# Rescue - VM boot từ Volume

**Lưu ý:** Không nên thực hiện cách này do phải thao tác với databases của Openstack.

### Cách thức thực hiện:
- Chỉnh sửa DB của OpenStack để gỡ được volume boot khỏi máy ảo.
- Gắn 1 volume khác có sẵn SystemRescueCD vào máy ảo .
- Thực hiện reset password.
- Gỡ volume SystemRescueCD khỏi máy ảo, chỉnh sửa lại DB OpenStack để đưa máy ảo về trạng thái ban đầu.
- Boot máy ảo và đăng nhập bằng password mới.

# 1. Gỡ volume boot khỏi máy ảo
List danh sách máy ảo:
```
openstack server list
+--------------------------------------+-------------+---------+----------------------+----------+----------+
| ID                                   | Name        | Status  | Networks             | Image    | Flavor   |
+--------------------------------------+-------------+---------+----------------------+----------+----------+
| ...                                                                                                       |
| 710c7d8e-ce9a-4dc3-a7fe-cb9e4ca44ee0 | vm03-u20    | ACTIVE  | public1=10.10.32.172 |          | flv_1c1r |
| ...                                                                                                       |
+--------------------------------------+-------------+---------+----------------------+----------+----------+
```

List danh sách volume:
```
openstack volume list
+--------------------------------------+-----------+-----------+------+--------------------------------------+
| ID                                   | Name      | Status    | Size | Attached to                          |
+--------------------------------------+-----------+-----------+------+--------------------------------------+
| ...                                                                                                        |
| 11551f7d-224c-428f-abc8-4da6222b152c | vlm03-u20 | in-use    |   10 | Attached to vm03-u20 on /dev/vda     |
| 9574f65e-4de1-4e60-bc68-d8423da0272b | RescueCD  | available |    1 |                                      |
| ...                                                                                                        |
+--------------------------------------+-----------+-----------+------+--------------------------------------+
```

Tắt máy ảo cần rescue. Ví dụ này, ta thực hiện trên `vm03-u20` chạy Ubuntu 20.04:
```
openstack server stop vm03-u20
```

Sau khi tắt, tiến hành detach volume boot của `vm03-u20`
```
openstack server remove volume vm03-u20 vlm03-u20
```

Truy cập DB, update bản ghi của volume boot của máy ảo
```
mysql -uroot -pWelcome123

use nova;

update block_device_mapping set device_name = '/dev/vdl', boot_index=1 where volume_id = '11551f7d-224c-428f-abc8-4da6222b152c' and deleted = 0;

use cinder;

update volume_attachment set mountpoint='/dev/vdl' where volume_id ='11551f7d-224c-428f-abc8-4da6222b152c' and deleted=0;

exit
```

`11551f7d-224c-428f-abc8-4da6222b152c` : ID của volume boot - `vlm03-u20`

Sau khi cập nhật, kiểm tra lại volume
```
openstack volume list
+--------------------------------------+-----------+-----------+------+--------------------------------------+
| ID                                   | Name      | Status    | Size | Attached to                          |
+--------------------------------------+-----------+-----------+------+--------------------------------------+
| ...                                                                                                        |
| 11551f7d-224c-428f-abc8-4da6222b152c | vlm03-u20 | in-use    |   10 | Attached to vm03-u20 on /dev/vdl     | 
| 9574f65e-4de1-4e60-bc68-d8423da0272b | RescueCD  | available |    1 |                                      |
| ...                                                                                                        |
+--------------------------------------+-----------+-----------+------+--------------------------------------+
```

# 2. Tạo volume SystemRescueCD
## 2.1. Tạo image SystemRescueCD
### 2.1.1. Tải SystemRescueCD
```
wget https://osdn.net/projects/systemrescuecd/storage/releases/6.1.7/systemrescuecd-amd64-6.1.7.iso
```

### 2.1.2. Trên máy chủ tạo image, tải gói `syslinux`. Đây là 1 gói các lightweight master boot record để khởi động OS.
```
yum install -y syslinux
```

### 2.1.3. Sử dụng công cụ `isohybrid`, cho phép ISO image có thể được boot qua BIOS từ disk storage device (USB)
```
isohybrid systemrescuecd-amd64-6.1.7.iso
```

### 2.1.4. Tạo image ISO SystemRescueCD lên Glance
```
openstack image create --file systemrescuecd-amd64-6.1.7.iso --public systemrescuecd-amd64
```

### 2.1.5. Đặt các metadata cho image
```
openstack image set --property hw_cdrom_bus=ide --property hw_disk_bus=ide systemrescuecd-amd64
```

## 2.2. Tạo volume với image SystemRescueCD vừa tạo
```
openstack volume create --size 1 --image systemrescuecd-amd64 RescueCD
```

## 2.3. Mount volume SystemRescueCD vào máy ảo
```
openstack server add volume vm03-u20 RescueCD
```

Kiểm tra lại ta đã thấy volume RescueCD được attached vào `vm03-u20`
```
openstack volume list
+--------------------------------------+-----------+-----------+------+--------------------------------------+
| ID                                   | Name      | Status    | Size | Attached to                          |
+--------------------------------------+-----------+-----------+------+--------------------------------------+
| ...                                                                                                        |
| 11551f7d-224c-428f-abc8-4da6222b152c | vlm03-u20 | in-use    |   10 | Attached to vm03-u20 on /dev/vdl     |
| 9574f65e-4de1-4e60-bc68-d8423da0272b | RescueCD  | in-use    |    1 | Attached to vm03-u20 on /dev/vdb     |
| ...                                                                                                        |
+--------------------------------------+-----------+-----------+------+--------------------------------------+
```

## 2.4. Truy cập Mysql và thực hiện update bản ghi của Volume Rescue
```
mysql -uroot -pWelcome123

use nova;

update block_device_mapping set device_name = '/dev/vda', boot_index=1 where volume_id = '9574f65e-4de1-4e60-bc68-d8423da0272b' and deleted = 0;

use cinder;

update volume_attachment set mountpoint='/dev/vda' where volume_id ='9574f65e-4de1-4e60-bc68-d8423da0272b' and deleted=0;

exit
```

`9574f65e-4de1-4e60-bc68-d8423da0272b` : ID volume SystemRescueCD

## 2.5. Khởi động máy ảo
```
openstack server start vm03-u20
```

# 3. Thực hiện reset password VM
## 3.1. Truy cập Horizon để vào console của VM
Truy cập console của `vm03-u20`, ta sẽ thấy giao diện của SystemRescueCD.

<img src="..\images\Screenshot_90.png">

Lựa chọn option đầu tiên `Boot SystemRescueCD using default options`. VM sẽ boot vào cmd của RescueCD:

<img src="..\images\Screenshot_91.png">

## 3.2. List các ổ đĩa của máy ảo:
```
fdisk -l
```

<img src="..\images\Screenshot_93.png">

Ta thấy ổ đĩa `/dev/vdb2` là ổ đĩa chứa OS máy ảo

## 3.3. Mount ổ đĩa chứa OS vào thư mục `/mnt/root`
```
mkdir /mnt/root
mount /dev/vdb2 /mnt/root
```

## 3.4. Thay đổi root directory từ SystemRescueCD sang thư mục `/mnt/root`
```
chroot /mnt/root /bin/bash
```

## 3.5. Tiến hành đổi password cho user `root`
```
passwd root
```

# 4. Umount volume SystemRescue và mount lại volume boot của máy ảo
Thực hiện trên node controller
## 4.1. Tắt VM
```
openstack server stop vm03-u20
```

## 4.2. Update lại bản ghi của volume SystemRescueCD
```
mysql -uroot -pWelcome123

use nova;

update block_device_mapping set device_name = '/dev/vdb', boot_index=1 where volume_id = '9574f65e-4de1-4e60-bc68-d8423da0272b' and deleted = 0;

use cinder;

update volume_attachment set mountpoint='/dev/vdb' where volume_id ='9574f65e-4de1-4e60-bc68-d8423da0272b' and deleted=0;

exit
```

`9574f65e-4de1-4e60-bc68-d8423da0272b` : ID của volume Rescue

## 4.3. Gỡ bỏ volume Rescue khỏi máy ảo
```
openstack server remove volume vm03-u20 RescueCD
```

## 4.4. Update bản ghi của volume boot
```
mysql -uroot -pWelcome123

use nova;

update block_device_mapping set device_name = '/dev/vda', boot_index=1 where volume_id = '11551f7d-224c-428f-abc8-4da6222b152c' and deleted = 0;

use cinder;

update volume_attachment set mountpoint='/dev/vda' where volume_id ='11551f7d-224c-428f-abc8-4da6222b152c' and deleted=0;

exit 
```

`11551f7d-224c-428f-abc8-4da6222b152c` : ID volume boot của máy ảo

Kiểm tra lại:
```
openstack volume list
+--------------------------------------+-----------+-----------+------+--------------------------------------+
| ID                                   | Name      | Status    | Size | Attached to                          |
+--------------------------------------+-----------+-----------+------+--------------------------------------+
| ...                                                                                                        |
| 11551f7d-224c-428f-abc8-4da6222b152c | vlm03-u20 | in-use    |   10 | Attached to vm03-u20 on /dev/vda     |
| 9574f65e-4de1-4e60-bc68-d8423da0272b | RescueCD  | available |    1 |                                      |
| ...                                                                                                        |
+--------------------------------------+-----------+-----------+------+--------------------------------------+
```

# 5. Khởi động máy ảo và đăng nhập bằng mật khẩu vừa đổi.
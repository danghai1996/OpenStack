# Nova rescue - VM boot từ Local disk

Openstack cung cấp chức năng để rescue các instnace trong các trường hợp lỗi hệ thống, mất mát filesystem, mất SSH key, cấu hình network bị sai hoặc dùng để khôi phục mật khẩu.

Lưu ý: Trong quá trình boot thì instance disk và recuse disk có thể trùng UUID , vì thế trong 1 một số trường hợp thì instance đã vào mode rescue nhưng vẫn boot từ local disk

Ví dụ một vài trường hợp bạn muốn sửa chữa một hệ thống bị bằng cách sử dụng rescue boot:
- Mất ssh key và muốn kích hoạt tàm thời chế độ đăng nhập bằng password
- Cấu hình mạng gặp lỗi
- Cấu hình boot lỗi

# Hướng dẫn Reset password cho các máy ảo trên hệ thống OPS sử dụng SystemRecuseCD
## 1. Tạo image SystemRescueCD
### 1.1. Tải SystemRescueCD
```
wget https://osdn.net/projects/systemrescuecd/storage/releases/6.1.7/systemrescuecd-amd64-6.1.7.iso
```

### 1.2. Trên máy chủ tạo image, tải gói `syslinux`. Đây là 1 gói các lightweight master boot record để khởi động OS.
```
yum install -y syslinux
```

### 1.3. Sử dụng công cụ `isohybrid`, cho phép ISO image có thể được boot qua BIOS từ disk storage device (USB)
```
isohybrid systemrescuecd-amd64-6.1.7.iso
```

### 1.4. Tạo image ISO SystemRescueCD lên Glance
```
openstack image create --file systemrescuecd-amd64-6.1.7.iso --public systemrescuecd-amd64
```

### 1.5. Đặt các metadata cho image
```
openstack image set --property hw_cdrom_bus=ide --property hw_disk_bus=ide systemrescuecd-amd64
```

## 2. Cấu hình trên OpenStack
Để thực hiện việc reset password máy ảo, ta sẽ sử dụng 1 tính năng của nova là Rescue VM. Tính năng này sẽ đặt máy ảo vào trạng thái Rescue, trong đó VM sẽ boot vào 1 image được chỉ định.

### 2.1. Cấu hình `shutdown_timeout` trên node OpenStack Controller
Khi thực hiện Rescue, VM sẽ được soft reboot trước khi boot vào SystemRescueCD, nếu thời gian soft_reboot lâu hơn thời gian mặc định của nova (60s), VM sẽ bị hard reboot. Để quá trình reset password thành công, phải đảm bảo quá trình soft reboot này của VM thành công tốt đẹp. Để thực hiện điều đó, cấu hình giá trị `shutdown_timeout` cao hơn mức mặc định của nova. 

Sửa file `/etc/nova/nova.conf`, đặt thời gian timeout là 5p (300s):
```
[DEFAULT]
shutdown_timeout = 300
```

Khởi động lại service của Nova
```
systemctl restart \
openstack-nova-api.service \
openstack-nova-scheduler.service \
openstack-nova-conductor.service \
openstack-nova-novncproxy.service
```

## 3. Thực hiện Reset password VM
### 3.1. Đưa máy ảo về trạng thái Rescue. 
**Lưu ý:** máy ảo phải đang ở trạng thái Active (Running) trước khi chuyển về Rescue.

List danh sách máy ảo:
```
openstack server list
+--------------------------------------+-------------+---------+----------------------+----------+----------+
| ID                                   | Name        | Status  | Networks             | Image    | Flavor   |
+--------------------------------------+-------------+---------+----------------------+----------+----------+
| 710c7d8e-ce9a-4dc3-a7fe-cb9e4ca44ee0 | vm03-u20    | ACTIVE  | public1=10.10.32.172 |          | flv_1c1r |
| 581a4ad0-81bc-4eea-bbbe-9ec94c160496 | vm04-u20    | ACTIVE  | public1=10.10.32.179 | ubuntu20 | flv_1c1r |
| 6a3678f3-6c05-4248-acf7-32ccea9a74ab | vm02-cirros | ACTIVE  | public1=10.10.32.178 |          | flv_1c1r |
| 687d0cd3-1ee9-4092-a7bb-fd29c597f5c2 | vm01-cirros | SHUTOFF | public1=10.10.32.177 |          | flv_1c1r |
+--------------------------------------+-------------+---------+----------------------+----------+----------+
```

Đưa máy ảo về trạng thái rescue
```
openstack server rescue --image systemrescuecd-amd64 vm04-u20
```

**Trong đó:**
- `systemrescuecd-amd64`: tên image SystemRescueCD đã tạo ở trên
- `vm04-u20`: tên máy ảo cần reset password

### 3.2. Truy cập Horizon, ta sẽ thấy trạng thái Rescue của VM `vm04-u20`

<img src="..\images\Screenshot_89.png">

Truy cập console của `vm04-u20`, ta sẽ thấy giao diện của SystemRescueCD.

<img src="..\images\Screenshot_90.png">

Lựa chọn option đầu tiên `Boot SystemRescueCD using default options`. VM sẽ boot vào cmd của RescueCD:

<img src="..\images\Screenshot_91.png">

### 3.3. List các ổ đĩa của máy ảo:
```
fdisk -l
```

<img src="..\images\Screenshot_92.png">

Ta thấy ổ đĩa `/dev/sdb2` là ổ đĩa chứa OS máy ảo

### 3.4. Mount ổ đĩa chứa OS vào thư mục `/mnt/root`
```
mkdir /mnt/root
mount /dev/sdb2 /mnt/root
```

### 3.5. Thay đổi root directory từ SystemRescueCD sang thư mục `/mnt/root`
```
chroot /mnt/root /bin/bash
```

### 3.6. Tiến hành đổi password cho user `root`
```
passwd root
```

### 3.7. Trên node Controller, gỡ bỏ trạng thái Rescue của máy ảo
```
openstack server unrescue vm04-u20
```

### 3.8. Login vào VM bằng mật khẩu vừa đổi.

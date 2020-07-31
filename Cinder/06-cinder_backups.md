# Cấu hình Cinder backups với backends là NFS

## Cấu hình NFS làm backends cho Cinder
> Trên host chạy Cinder volume service

Dùng trình soạn thảo vi mở file `/etc/cinder/nfs_shares` :
```
vi /etc/cinder/nfs_shares 
```
Thêm: `/var/lib/nfs-share` là thư mục mà chúng ta sẽ share khi cấu hình NFS (thư mục chứa các file backups).
```
10.10.31.166:/var/lib/nfs-share
```

Phân quyền và đổi chủ sở hữu cho file /etc/cinder/nfs_shares
```
chown root:cinder /etc/cinder/nfs_shares
chmod 0640 /etc/cinder/nfs_shares
```

Cấu hình cinder volume service sử dụng `/etc/cinder/nfs_shares` file đã được tạo phía trên.
```
vi /etc/cinder/cinder.conf
```
Thêm:
```
[backend_defaults]
nfs_shares_config = /etc/cinder/nfs_shares
volume_driver = cinder.volume.drivers.nfs.NfsDriver
```

Khởi động lại các dịch vụ để hoàn tất quá trình cài đặt.
```
systemctl enable openstack-cinder-volume.service target.service
systemctl start openstack-cinder-volume.service target.service
systemctl restart openstack-cinder-volume.service target.service rpcbind nfs-server
```

## Cấu hình Block Storage NFS
> Trên node NFS
Cài đặt và cấu hình NFS
```
yum install nfs-utils -y
mkdir /var/lib/nfs-share
echo "/var/lib/nfs-share 10.10.31.0/24(rw,no_root_squash)" > /etc/exports 
systemctl restart rpcbind nfs-server
systemctl enable rpcbind nfs-server
```

Nếu firewall đang được bật
```
firewall-cmd --add-service=nfs --permanent
firewall-cmd --reload
```

## Trên Storage node
Cài các gói:
```
yum install --enablerepo=centos-openstack-train,epel -y openstack-cinder targetcli python-keystone python-openstackclient
```

Thực hiện sửa cấu hình trong file `/etc/cinder/cinder.conf` trên Storage node
```
vi /etc/cinder/cinder.conf 
```
Sửa các phần sau:
```
[DEFAULT]
enabled_backends = lvm,nfs
...

[nfs]
volume_driver = cinder.volume.drivers.nfs.NfsDriver
nfs_shares_config = /etc/cinder/nfs_shares
volume_backend_name = nfsdriver-1
nfs_mount_point_base = $state_path/mnt_nfs
```

Cài đặt và cấu hình NFS client trên Storage node
```
yum -y install nfs-utils	
systemctl enable rpcbind
systemctl start rpcbind

cat <<EOF > /etc/cinder/nfs_shares  
10.10.31.166:/var/lib/nfs-share
EOF

chmod 640 /etc/cinder/nfs_shares
chgrp cinder /etc/cinder/nfs_shares

systemctl restart openstack-cinder-volume
chown  -R cinder. /var/lib/cinder/mnt_nfs/
```
Khởi động lại dịch vụ để hoàn tất cài đặt
```
systemctl enable openstack-cinder-volume.service target.service
systemctl start openstack-cinder-volume.service target.service
systemctl restart openstack-cinder-volume.service target.service rpcbind nfs-server
```

## Tạo type NFS trên Controller node
> Thực hiện trên Controller node

Tạo type volume:
```
openstack volume type create nfs
openstack volume type set nfs --property volume_backend_name=nfsdriver-1
```

Kiểm tra lại:
```
openstack volume type list --long
```

Khởi tạo NFS DISK
```
openstack volume create --type  nfs --size 5 disk_nfs
```

Restart service backup
```
systemctl start openstack-cinder-backup
systemctl enable openstack-cinder-backup
systemctl restart openstack-cinder-backup.service
```

Kiểm tra:
```
openstack volume service list
```

## Cấu hình Cinder backups
> Thực hiện trên node Storage

Chỉnh sửa file cấu hình của Cinder `/etc/cinder/cinder.conf`
```
vi /etc/cinder/cinder.conf
```
```
[DEFAULT]
backup_driver=cinder.backup.drivers.nfs.NFSBackupDriver
backup_share=10.10.31.166:/var/lib/nfs-share
```

Khởi động lại dịch vụ:
```
systemctl start openstack-cinder-backup
systemctl enable openstack-cinder-backup
systemctl restart openstack-cinder-backup.service
```

### Lưu ý: Có thể lab all in one trên 1 node Controller

------

# Backups và restore volume
Để có thể tạo backup từ một volume thì volume đó phải đang ở trạng thái `available`. Nếu volume đang được `attach` vào máy ảo thì cần phải gỡ nó ra hoặc chuyển đổi status của nó bằng tài khoản admin, sau khi backup xong thì `attach` lại.

## Backups volume
```
openstack volume backup create --name <tên_bản_backup> [--incremental] [--force] <VOLUME>
```
**Trong đó:**
- `<VOLUME>` : tên hoặc ID của volume

- `incremental` : để chỉ ra kiểu backup muốn thực hiện là incremental backup (không có cờ này thì mặc định sẽ thực hiện full backup)

- `force` : là cờ chỉ ra sự cho phép hoặc không cho phép thực hiện backup volume trong khi đang được `attack` vào máy ảo. Khi không có cờ `force`, volume sẽ chỉ được back up khi nó ở trạng thái `available` (nghĩa là đang không được gắn vào máy ảo). Khi trạng thái của volume là `in-use`, nó đang được gán vào một máy ảo nào đó, muốn backup nó cần bật cờ `force`, khi đó dữ liệu trong volume sẽ bị ngắt quãng. Mặc định cờ force sẽ được thiết lập bằng `FALSE`.

## Restore volume
```
openstack volume backup restore BACKUP_ID VOLUME_ID
```
Sau khi restore cần reboot lại VM.

- Khi restore từ full backup nó sẽ gọi là full restore.

- Khi restore từ incremental backup, danh sách backup được liệt kê dựa trên IDs của parent backups. Một full restore được thực hiện dựa trên full backup đầu tiên và các lần incremental backup sau đó được đặt theo đúng thứ tự.

## **Ví dụ:**
Ta tạo 1 volume từ image cirros và tạo 1 vm với volume đó:
```
openstack volume create --type nfs --size 5 --image cirros vlm_nfs_cirros

openstack server create --flavor flv_01 --volume vlm_nfs_cirros \
--nic net-id=4fce44b1-3f19-4ccb-8084-889d552f7dd9 \
--security-group 3fdbfd81-e5ee-4a40-9db8-566c77c828db \
vm01-test-nfs-backup
```

Truy cập VM và tạo ra một vài file:
```
echo "Test-Backups-Restore-NFS" > haidd.txt
```

Tắt VM và chuyển trạng thái volume về available:
```
openstack server stop vm01-test-nfs-backup

openstack volume set --state available vlm_nfs_cirros
```

Kiểm tra lại trạng thái của volume:
```
openstack volume list
+--------------------------------------+----------------+-----------+------+-----------------------------------------------+
| ID                                   | Name           | Status    | Size | Attached to                                   |
+--------------------------------------+----------------+-----------+------+-----------------------------------------------+
| 53939c51-e4a8-47d3-a6e2-9e156a1dc6f5 | vlm_nfs_cirros | available |    5 | Attached to vm01-test-nfs-backup on /dev/vda  |
| f9871b5a-4a73-4ed9-b934-b82b1659583a | vlm01-cirros   | in-use    |    6 | Attached to vm01-cirros on /dev/vda           |
+--------------------------------------+----------------+-----------+------+-----------------------------------------------+
```

Backups volume:
```
openstack volume backup create --name bak_vm01_cirros vlm_nfs_cirros
+-------+--------------------------------------+
| Field | Value                                |
+-------+--------------------------------------+
| id    | 55ae49fd-d7c0-4c66-8ef1-a98cf0db25bc |
| name  | bak_vm01_cirros                      |
+-------+--------------------------------------+
```

Khi backups xong sẽ thấy trạng thái là `available`:
```
openstack volume backup list
+--------------------------------------+------------------+-------------+-----------+------+
| ID                                   | Name             | Description | Status    | Size |
+--------------------------------------+------------------+-------------+-----------+------+
| 55ae49fd-d7c0-4c66-8ef1-a98cf0db25bc | bak_vm01_cirros  | None        | available |    5 |
| 5b2563d1-b043-4b1b-a111-ae432f9357c7 | backups_disk_nfs | None        | available |    5 |
+--------------------------------------+------------------+-------------+-----------+------+
```

Bật máy ảo và xóa file đã tạo ở trên đi:
```
openstack server start vm01-test-nfs-backup
```
Trên máy ảo, xóa file
```
rm -f haidd.txt
```

Thực hiện restore bản backup đã tạo:
```
openstack volume backup restore 55ae49fd-d7c0-4c66-8ef1-a98cf0db25bc 53939c51-e4a8-47d3-a6e2-9e156a1dc6f5
```

Kiểm tra trạng thái restore sẽ là `restoring-backup`. Sau khi hoàn thành sẽ có status là `available`
```
openstack volume list
+--------------------------------------+----------------+------------------+------+-----------------------------------------------+
| ID                                   | Name           | Status           | Size | Attached to                                   |
+--------------------------------------+----------------+------------------+------+-----------------------------------------------+
| 53939c51-e4a8-47d3-a6e2-9e156a1dc6f5 | vlm_nfs_cirros | restoring-backup |    5 | Attached to vm01-test-nfs-backup on /dev/vda  |
| f9871b5a-4a73-4ed9-b934-b82b1659583a | vlm01-cirros   | in-use           |    6 | Attached to vm01-cirros on /dev/vda           |
+--------------------------------------+----------------+------------------+------+-----------------------------------------------+
```

Sau khi restore xong, ta reboot máy ảo và kiểm tra xem file tạo lúc đầu đã được restore chưa
```
cat haidd.txt
Test-Backups-Restore-NFS
```

Vậy là ta đã restore thành công.

Tiến hành đổi status của volume từ `available` về `in-use`
```
openstack volume set --state in-use vlm_nfs_cirros
```

Việc backups sẽ khá ngốn tài nguyên:
```
htop
```
<img src="..\images\Screenshot_76.png">

# File cấu hình Cinder

**Vị trí lưu file cấu hình Cinder:** `/etc/cinder/cinder.conf`

## 1. Khai báo DB
```
[database]
connection = mysql+pymysql://cinder:Welcome123@10.10.31.166/cinder
```

- `connection = ...` : chỉ ra địa chỉ mà cần kết nối database. Khai báo password :`Welcome123` cho database `cinder` và địa chỉ : `10.10.31.166` (hostname hoặc IP) của node Controller

## 2. Khai báo Message Queue
```
[DEFAULT]
transport_url = rabbit://openstack:Welcome123@10.10.31.166
```

- `transport_url` : đường dẫn, user, mật khẩu, IP Controller

## 3. Khai báo xác thực Keystone
```
[DEFAULT]
auth_strategy = keystone

[keystone_authtoken]
www_authenticate_uri = http://10.10.31.166:5000
auth_url = http://10.10.31.166:5000
memcached_servers = 10.10.31.166:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = cinder
password = Welcome123
```

- `auth_strategy = keystone` : Cấu hình sử dụng Keystone để xác thực
- `www_authenticate_uri = http://10.10.31.166:5000` : Cấu hình enpoint Identity service
- `auth_url = http://10.10.31.166:5000` : URL để xác thực Identity service
- `memcached_servers = 10.10.31.166:11211` : Địa chỉ Memcache-server
- `auth_type = password` : Hình thức xác thực sử dụng password
- `project_domain_name = default` : Chỉ định project domain name openstack
- `user_domain_name = default` : Chỉ định user domain name openstack
- `project_name = service` : Chỉ định project name openstack
- `username = cinder` : Chỉ định username của cinder
- `password = Welcome123` : Chỉ định password của cinder

## 4. IP management
```
[DEFAULT]
my_ip = 10.10.31.166
```

- `my_ip = 10.10.31.166` : Địa chỉ IP management của Storage Node

## 5. Cấu hình Storage Backend
### 5.1. LVM Backend
```
[DEFAULT]
enabled_backends = lvm

[lvm]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_group = cinder-volumes
target_protocol = iscsi
target_helper = lioadm
```
- `enabled_backends = lvm` : Sử dụng backend là LVM. Đối với multiple backend chỉ cần dấu phẩy giữa các backend (ví dụ : enable_backends = lvm,nfs,glusterfs)
- `volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver`: Chỉ định driver mà LVM sử dụng
- `volume_group = cinder-volumes` :  Chỉ định vgroup mà tạo lúc cài đặt. Sử dụng câu lệnh `vgs` hoặc `vgdisplay` để xem thông tin về vgroup đã tạo.
- `target_protocol = iscsi` : Xác định giao thức iSCSI cho iSCSI volumes mới, được tạo ra với tgtadm hoặc lioadm. Để kích hoạt RDMA , tham số lên lên được đặt là "iser" . Hỗ trợ cho giao thức iSCSI giá trị là "iscsi" và "iser"
- `target_helper = lioadm` : Chỉ ra iSCSI sử dụng. Mặc định là `tgtadm`. Có các tùy chọn sau: 
    - `lioadm` hỗ trợ LIO iSCSI
    - `scstadmin` cho SCST
    - `iseradm` cho ISER
    - `ietadm` cho iSCSI

### 5.2. GlusterFS Backend
```
[DEFAULT]
enabled_backends = glusterfs
[glusterfs]
volume_driver = cinder.volume.drivers.glusterfs.GlusterfsDriver
glusterfs_shares_config = /etc/cinder/glusterfs_shares
glusterfs_mount_point_base = $state_path/mnt_gluster
```
- `enabled_backends = glusterfs` : Sử dụng backend là Glusterfs

- `volume_driver = cinder.volume.drivers.glusterfs.GlusterfsDriver` : Chỉ định driver mà Glusterfs sử dụng

- `glusterfs_shares_config = /etc/cinder/glusterfs_shares` : File cấu hình để kết nối tới GlusterFS.

- `glusterfs_mount_point_base = $state_path/mnt_gluster` : Mount point tới GlusterFS

    - Nội dung file glusterfs_shares
        ```
        <địa_chỉ_IP>:/cinder-vol
        ```
        - `<địa_chỉ_IP>` : IP của GlusterFS Pool.
        - `cinder-vol` : Tên volume đã tạo ở GlusterFS

## 6. Cấu hình khác
```
[DEFAULT]
my_ip = 10.10.31.166
glance_api_servers = http://10.10.31.166:9292

[oslo_concurrency]
lock_path = /var/lib/cinder/tmp
```
- `my_ip = 10.10.31.166`: IP Management của Storage node
- `glance_api_servers = http://10.10.31.166:9292`: URL kết nối tới Glance
- `lock_path = /var/lib/cinder/tmp`: Khai báo thư mục chứa lock_path
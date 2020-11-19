# Tích hợp Openstack HA với CEPH

# Phần 1: Chuẩn bị
## Cài đặt các gói bổ sung trên các CTL, COM
> ### Thực hiện trên các node CTL và COM
Update repo
```
cat <<EOF> /etc/yum.repos.d/ceph.repo
[ceph]
name=Ceph packages for $basearch
baseurl=https://download.ceph.com/rpm-luminous/el7/x86_64/
enabled=1
priority=2
gpgcheck=1
gpgkey=https://download.ceph.com/keys/release.asc

[ceph-noarch]
name=Ceph noarch packages
baseurl=https://download.ceph.com/rpm-luminous/el7/noarch
enabled=1
priority=2
gpgcheck=1
gpgkey=https://download.ceph.com/keys/release.asc

[ceph-source]
name=Ceph source packages
baseurl=https://download.ceph.com/rpm-luminous/el7/SRPMS
enabled=0
priority=2
gpgcheck=1
gpgkey=https://download.ceph.com/keys/release.asc
EOF

yum update -y
```

Cài đặt các gói
```
yum install epel-release -y 
yum install python-rbd -y
yum install ceph-common -y
```

## Tạo pool trên CEPH
> ## Thực hiện trên node cephaio
```
ceph osd pool create volumes 64 64
ceph osd pool create vms 16 16
ceph osd pool create images 8 8
ceph osd pool create backups 32 32

rbd pool init volumes
rbd pool init vms
rbd pool init images
rbd pool init backups
```

Copy cấu hình sang các node CTL và COM
```
ssh 10.10.34.161 sudo tee /etc/ceph/ceph.conf < /etc/ceph/ceph.conf
ssh 10.10.34.162 sudo tee /etc/ceph/ceph.conf < /etc/ceph/ceph.conf
ssh 10.10.34.163 sudo tee /etc/ceph/ceph.conf < /etc/ceph/ceph.conf

ssh 10.10.34.164 sudo tee /etc/ceph/ceph.conf < /etc/ceph/ceph.conf
ssh 10.10.34.165 sudo tee /etc/ceph/ceph.conf < /etc/ceph/ceph.conf
```

# Phần 2: Cấu hình CEPH làm backend cho Glance-Images
## Tạo key và chuyển key sang các node CTL
> ## Thực hiện trên node cephaio
Tạo key Glance
```
cd /ceph-deploy

ceph auth get-or-create client.glance mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=images' > ceph.client.glance.keyring
```

Chuyển key glance sang 3 node CTL
```
ceph auth get-or-create client.glance | ssh 10.10.34.161 sudo tee /etc/ceph/ceph.client.glance.keyring
ceph auth get-or-create client.glance | ssh 10.10.34.162 sudo tee /etc/ceph/ceph.client.glance.keyring
ceph auth get-or-create client.glance | ssh 10.10.34.163 sudo tee /etc/ceph/ceph.client.glance.keyring
```

## Cấu hình trên các node CTL
### Dừng service Glance
> ### Thực hiện trên cả 3 node CTL
```
systemctl stop openstack-glance-api.service \
  openstack-glance-registry.service
```

> ## Trên node CTL1 
Phân quyền
```
sudo chown glance:glance /etc/ceph/ceph.client.glance.keyring
sudo chmod 0640 /etc/ceph/ceph.client.glance.keyring
```

Thêm cấu hình `/etc/glance/glance-api.conf`
```
[DEFAULT]
show_image_direct_url = True
...

[glance_store]
#stores = file,http
#default_store = file
#filesystem_store_datadir = /var/lib/glance/images/
default_store = rbd
stores = file,http,rbd
rbd_store_pool = images
rbd_store_user = glance
rbd_store_ceph_conf = /etc/ceph/ceph.conf
rbd_store_chunk_size = 8
```

Restart lại dịch vụ glance
```
systemctl start openstack-glance-api.service \
  openstack-glance-registry.service
```

> ## Trên node CTL2
Phân quyền
```
sudo chown glance:glance /etc/ceph/ceph.client.glance.keyring
sudo chmod 0640 /etc/ceph/ceph.client.glance.keyring
```

Thêm cấu hình `/etc/glance/glance-api.conf`
```
[DEFAULT]
show_image_direct_url = True
...

[glance_store]
#stores = file,http
#default_store = file
#filesystem_store_datadir = /var/lib/glance/images/
default_store = rbd
stores = file,http,rbd
rbd_store_pool = images
rbd_store_user = glance
rbd_store_ceph_conf = /etc/ceph/ceph.conf
rbd_store_chunk_size = 8
```

Restart lại dịch vụ glance
```
systemctl start openstack-glance-api.service \
  openstack-glance-registry.service
```

> ## Trên node CTL3
Phân quyền
```
sudo chown glance:glance /etc/ceph/ceph.client.glance.keyring
sudo chmod 0640 /etc/ceph/ceph.client.glance.keyring
```

Thêm cấu hình `/etc/glance/glance-api.conf`
```
[DEFAULT]
show_image_direct_url = True
...

[glance_store]
#stores = file,http
#default_store = file
#filesystem_store_datadir = /var/lib/glance/images/
default_store = rbd
stores = file,http,rbd
rbd_store_pool = images
rbd_store_user = glance
rbd_store_ceph_conf = /etc/ceph/ceph.conf
rbd_store_chunk_size = 8
```

Restart lại dịch vụ glance
```
systemctl start openstack-glance-api.service \
  openstack-glance-registry.service
```

## Kiểm tra
Test theo test case sau:

- Bật Glance tại CTL 1, tắt Glance trên CTL 2 3, test up image
- Bật Glance tại CTL 2, tắt Glance trên CTL 1 3, test up image
- Bật Glance tại CTL 3, tắt Glance trên CTL 1 2, test up image

Tắt glance
```
systemctl stop openstack-glance-api.service \
  openstack-glance-registry.service
```

Up thử Image (thực hiện tại CTL1), Up Image với dịch vụ Glance CTL 1 up, CTL 2 3 down
```
source admin-openrc
wget http://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img

openstack image create "cirros-ceph-ctl1" \
  --file cirros-0.3.5-x86_64-disk.img \
  --disk-format qcow2 --container-format bare \
  --public
```

Quay lại kiểm tra trên node Ceph
```
rbd -p images ls
```

Up thử Image (thực hiện tại CTL2), Up Image với dịch vụ Glance CTL 2 up, CTL 1 3 down
```
source admin-openrc
wget http://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img

openstack image create "cirros-ceph-ctl2" \
  --file cirros-0.3.5-x86_64-disk.img \
  --disk-format qcow2 --container-format bare \
  --public
```

Quay lại kiểm tra trên node Ceph
```
rbd -p images ls
```

Up thử Image (thực hiện tại CTL3), Up Image với dịch vụ Glance CTL 3 up, CTL 1 2 down
```
source admin-openrc
wget http://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img

openstack image create "cirros-ceph-ctl3" \
  --file cirros-0.3.5-x86_64-disk.img \
  --disk-format qcow2 --container-format bare \
  --public
```

Quay lại kiểm tra trên node Ceph
```
rbd -p images ls
```

# Phần 3: Cấu hình CEPH làm backend cho Cinder-volume, Cinder-backup
> ## Thực hiện trên node cephaio
Tạo key cinder và cinder-backup
```
cd /ceph-deploy
ceph auth get-or-create client.cinder mon 'allow r, allow command "osd blacklist", allow command "blacklistop"' osd 'allow class-read object_prefix rbd_children, allow rwx pool=volumes, allow rwx pool=images' > ceph.client.cinder.keyring
ceph auth get-or-create client.cinder-backup mon 'profile rbd' osd 'profile rbd pool=backups' > ceph.client.cinder-backup.keyring
```

Chuyển key cinder và key cinder-backup sang các node CTL
```
ceph auth get-or-create client.cinder | ssh 10.10.34.161 sudo tee /etc/ceph/ceph.client.cinder.keyring
ceph auth get-or-create client.cinder-backup | ssh 10.10.34.161 sudo tee /etc/ceph/ceph.client.cinder-backup.keyring

ceph auth get-or-create client.cinder | ssh 10.10.34.162 sudo tee /etc/ceph/ceph.client.cinder.keyring
ceph auth get-or-create client.cinder-backup | ssh 10.10.34.162 sudo tee /etc/ceph/ceph.client.cinder-backup.keyring

ceph auth get-or-create client.cinder | ssh 10.10.34.163 sudo tee /etc/ceph/ceph.client.cinder.keyring
ceph auth get-or-create client.cinder-backup | ssh 10.10.34.163 sudo tee /etc/ceph/ceph.client.cinder-backup.keyring
```

Chuyển key cinder tới các node COM
```
ceph auth get-or-create client.cinder | ssh 10.10.34.164 sudo tee /etc/ceph/ceph.client.cinder.keyring
ceph auth get-key client.cinder | ssh 10.10.34.164 tee /root/client.cinder

ceph auth get-or-create client.cinder | ssh 10.10.34.165 sudo tee /etc/ceph/ceph.client.cinder.keyring
ceph auth get-key client.cinder | ssh 10.10.34.165 tee /root/client.cinder
```

> ## Trên 3 node CTL
### Dừng dịch vụ Cinder trên cả 3 node CTL
> ### Trên cả 3 node CTL
```
systemctl stop openstack-cinder-api.service openstack-cinder-volume.service openstack-cinder-scheduler.service
```

> ## Trên node CTL1
Phân quyền
```
sudo chown cinder:cinder /etc/ceph/ceph.client.cinder*
sudo chmod 0640 /etc/ceph/ceph.client.cinder*
```

Bổ sung thêm cấu hình `/etc/cinder/cinder.conf` trên node controller
```
[DEFAULT]
....
## Thêm các giá trị này
notification_driver = messagingv2
enabled_backends = ceph
glance_api_version = 2
backup_driver = cinder.backup.drivers.ceph
backup_ceph_conf = /etc/ceph/ceph.conf
backup_ceph_user = cinder-backup
backup_ceph_chunk_size = 134217728
backup_ceph_pool = backups
backup_ceph_stripe_unit = 0
backup_ceph_stripe_count = 0
restore_discard_excess_bytes = true
host=ceph

## Bổ sung section ceph
[ceph]
volume_driver = cinder.volume.drivers.rbd.RBDDriver
volume_backend_name = ceph
rbd_pool = volumes
rbd_ceph_conf = /etc/ceph/ceph.conf
rbd_flatten_volume_from_snapshot = false
rbd_max_clone_depth = 5
rbd_store_chunk_size = 4
rados_connect_timeout = -1
rbd_user = cinder
rbd_secret_uuid = 414ba151-4068-40c6-9d7b-84998ce6a5a6
report_discard_supported = true
```

Restart lại dịch vụ
```
systemctl restart openstack-cinder-api.service openstack-cinder-volume.service
systemctl restart openstack-cinder-scheduler.service
```

Kiểm tra:
```
[root@ctl01 ~]# openstack volume service list
+------------------+-----------+------+---------+-------+----------------------------+
| Binary           | Host      | Zone | Status  | State | Updated At                 |
+------------------+-----------+------+---------+-------+----------------------------+
| cinder-scheduler | ctl01     | nova | enabled | down  | 2020-11-19T07:44:06.000000 |
| cinder-scheduler | ctl02     | nova | enabled | down  | 2020-11-19T07:43:57.000000 |
| cinder-scheduler | ctl03     | nova | enabled | down  | 2020-11-19T07:44:07.000000 |
| cinder-scheduler | ceph      | nova | enabled | up    | 2020-11-19T07:46:20.000000 |
| cinder-volume    | ceph@ceph | nova | enabled | up    | 2020-11-19T07:46:31.000000 |
+------------------+-----------+------+---------+-------+----------------------------+
```

**Lưu ý:**
- Chỉ quan tâm `cinder-scheduler: ceph` và `cinder-volume: ceph@ceph` phải `up`

Tạo mới type cinder ceph:
```
cinder type-create ceph
cinder type-key ceph set volume_backend_name=ceph
```

> ## Trên node CTL2
Phân quyền
```
sudo chown cinder:cinder /etc/ceph/ceph.client.cinder*
sudo chmod 0640 /etc/ceph/ceph.client.cinder*
```

Bổ sung thêm cấu hình `/etc/cinder/cinder.conf` trên node controller
```
[DEFAULT]
....
## Thêm các giá trị này
notification_driver = messagingv2
enabled_backends = ceph
glance_api_version = 2
backup_driver = cinder.backup.drivers.ceph
backup_ceph_conf = /etc/ceph/ceph.conf
backup_ceph_user = cinder-backup
backup_ceph_chunk_size = 134217728
backup_ceph_pool = backups
backup_ceph_stripe_unit = 0
backup_ceph_stripe_count = 0
restore_discard_excess_bytes = true
host=ceph

## Bổ sung section ceph
[ceph]
volume_driver = cinder.volume.drivers.rbd.RBDDriver
volume_backend_name = ceph
rbd_pool = volumes
rbd_ceph_conf = /etc/ceph/ceph.conf
rbd_flatten_volume_from_snapshot = false
rbd_max_clone_depth = 5
rbd_store_chunk_size = 4
rados_connect_timeout = -1
rbd_user = cinder
rbd_secret_uuid = 414ba151-4068-40c6-9d7b-84998ce6a5a6
report_discard_supported = true
```

Restart lại dịch vụ
```
systemctl restart openstack-cinder-api.service openstack-cinder-volume.service
systemctl restart openstack-cinder-scheduler.service
```

Kiểm tra tương tự CTL1:
```
[root@ctl02 ~]# openstack volume service list
+------------------+-----------+------+---------+-------+----------------------------+
| Binary           | Host      | Zone | Status  | State | Updated At                 |
+------------------+-----------+------+---------+-------+----------------------------+
| cinder-scheduler | ctl01     | nova | enabled | down  | 2020-11-19T07:44:06.000000 |
| cinder-scheduler | ctl02     | nova | enabled | down  | 2020-11-19T07:43:57.000000 |
| cinder-scheduler | ctl03     | nova | enabled | down  | 2020-11-19T07:44:07.000000 |
| cinder-scheduler | ceph      | nova | enabled | up    | 2020-11-19T07:51:20.000000 |
| cinder-volume    | ceph@ceph | nova | enabled | up    | 2020-11-19T07:51:21.000000 |
+------------------+-----------+------+---------+-------+----------------------------+
```

> ## Trên node CTL3
Phân quyền
```
sudo chown cinder:cinder /etc/ceph/ceph.client.cinder*
sudo chmod 0640 /etc/ceph/ceph.client.cinder*
```

Bổ sung thêm cấu hình `/etc/cinder/cinder.conf` trên node controller
```
[DEFAULT]
....
## Thêm các giá trị này
notification_driver = messagingv2
enabled_backends = ceph
glance_api_version = 2
backup_driver = cinder.backup.drivers.ceph
backup_ceph_conf = /etc/ceph/ceph.conf
backup_ceph_user = cinder-backup
backup_ceph_chunk_size = 134217728
backup_ceph_pool = backups
backup_ceph_stripe_unit = 0
backup_ceph_stripe_count = 0
restore_discard_excess_bytes = true
host=ceph

## Bổ sung section ceph
[ceph]
volume_driver = cinder.volume.drivers.rbd.RBDDriver
volume_backend_name = ceph
rbd_pool = volumes
rbd_ceph_conf = /etc/ceph/ceph.conf
rbd_flatten_volume_from_snapshot = false
rbd_max_clone_depth = 5
rbd_store_chunk_size = 4
rados_connect_timeout = -1
rbd_user = cinder
rbd_secret_uuid = 414ba151-4068-40c6-9d7b-84998ce6a5a6
report_discard_supported = true
```

Restart lại dịch vụ
```
systemctl restart openstack-cinder-api.service openstack-cinder-volume.service
systemctl restart openstack-cinder-scheduler.service
```

Kiểm tra tương tự CTL1, CTL2
```
[root@ctl03 ~]# openstack volume service list
+------------------+-----------+------+---------+-------+----------------------------+
| Binary           | Host      | Zone | Status  | State | Updated At                 |
+------------------+-----------+------+---------+-------+----------------------------+
| cinder-scheduler | ctl01     | nova | enabled | down  | 2020-11-19T07:44:06.000000 |
| cinder-scheduler | ctl02     | nova | enabled | down  | 2020-11-19T07:43:57.000000 |
| cinder-scheduler | ctl03     | nova | enabled | down  | 2020-11-19T07:44:07.000000 |
| cinder-scheduler | ceph      | nova | enabled | up    | 2020-11-19T07:52:59.000000 |
| cinder-volume    | ceph@ceph | nova | enabled | up    | 2020-11-19T07:52:54.000000 |
+------------------+-----------+------+---------+-------+----------------------------+
```

> ## Thực hiện trên các node COM

> ### Trên node COM1
Khởi tạo 1 uuid mới cho Cinder
```
uuidgen

8bf533a8-80b0-42f3-9d6e-1ccd2dfda989
```

**Lưu ý:** UUID sẽ sử dụng cho tất cả các COM nên chỉ tạo 1 lần đầu tiên

Tạo file xml cho phép Ceph RBD (Rados Block Device) xác thực với libvirt thông qua uuid vừa tạo
```
cat > ceph-secret.xml <<EOF
<secret ephemeral='no' private='no'>
<uuid>8bf533a8-80b0-42f3-9d6e-1ccd2dfda989</uuid>
<usage type='ceph'>
	<name>client.cinder secret</name>
</usage>
</secret>
EOF

sudo virsh secret-define --file ceph-secret.xml
```
Output:
```
Secret 8bf533a8-80b0-42f3-9d6e-1ccd2dfda989 created
```

Gán giá trị của `client.cinder` cho `uuid`
```
virsh secret-set-value --secret 8bf533a8-80b0-42f3-9d6e-1ccd2dfda989 --base64 $(cat /root/client.cinder)
```
Output
```
Secret value set
```


Khởi động lại dịch vụ
```
systemctl restart openstack-nova-compute
```

> ### Trên node COM2
Tạo file xml cho phép Ceph RBD (Rados Block Device) xác thực với libvirt thông qua uuid vừa tạo
```
cat > ceph-secret.xml <<EOF
<secret ephemeral='no' private='no'>
<uuid>8bf533a8-80b0-42f3-9d6e-1ccd2dfda989</uuid>
<usage type='ceph'>
	<name>client.cinder secret</name>
</usage>
</secret>
EOF

sudo virsh secret-define --file ceph-secret.xml
```
Output:
```
Secret 8bf533a8-80b0-42f3-9d6e-1ccd2dfda989 created
```

Gán giá trị của `client.cinder` cho `uuid`
```
virsh secret-set-value --secret 8bf533a8-80b0-42f3-9d6e-1ccd2dfda989 --base64 $(cat /root/client.cinder)
```
Output
```
Secret value set
```


Khởi động lại dịch vụ
```
systemctl restart openstack-nova-compute
```

## Kiểm tra:
Test theo test case sau:
- Bật Cinder tại CTL 1, tắt Cinder trên CTL 2 3, test up tạo mới Volume, boot VM với Cinder Volume trên horizon
- Bật Cinder tại CTL 2, tắt Cinder trên CTL 1 3, test up tạo mới Volume, boot VM với Cinder Volume trên horizon
- Bật Cinder tại CTL 3, tắt Cinder trên CTL 1 2, test up tạo mới Volume, boot VM với Cinder Volume trên horizon

```
systemctl stop openstack-cinder-api.service openstack-cinder-volume.service openstack-cinder-scheduler.service

openstack volume service list
```

Kiểm tra tại Ceph
```
rbd -p volumes ls
```

# Phần 4: Cấu hình CEPH làm backend cho Nova-Compute
> ## Trên node cephaio
Tạo key cho nova:
```
cd /ceph-deploy/

ceph auth get-or-create client.nova mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=vms, allow rx pool=images' -o /etc/ceph/ceph.client.nova.keyring 
```

Copy sang các node COM
```
ceph auth get-or-create client.nova | ssh 10.10.34.164 sudo tee /etc/ceph/ceph.client.nova.keyring
ceph auth get-key client.nova | ssh 10.10.34.164 tee /root/client.nova

ceph auth get-or-create client.nova | ssh 10.10.34.165 sudo tee /etc/ceph/ceph.client.nova.keyring
ceph auth get-key client.nova | ssh 10.10.34.166 tee /root/client.nova
```

## Thực hiện trên các node COM
> ### Trên node COM1
Phân quyền
```
chgrp nova /etc/ceph/ceph.client.nova.keyring
chmod 0640 /etc/ceph/ceph.client.nova.keyring
```

Tạo file XML cho Ceph kết nối Compute
```
cat << EOF > nova-ceph.xml
<secret ephemeral="no" private="no">
<uuid>805b9716-7fe8-45dd-8e1e-5dfdeff8b9be</uuid>
<usage type="ceph">
<name>client.nova secret</name>
</usage>
</secret>
EOF

sudo virsh secret-define --file nova-ceph.xml

virsh secret-set-value --secret 805b9716-7fe8-45dd-8e1e-5dfdeff8b9be --base64 $(cat /root/client.nova)
```

Chỉnh sửa file `/etc/nova/nova.conf` tại COM
```
[libvirt]
...
images_rbd_pool=vms
images_type=rbd
rbd_secret_uuid=805b9716-7fe8-45dd-8e1e-5dfdeff8b9be
rbd_user=nova
images_rbd_ceph_conf = /etc/ceph/ceph.conf
```

Restart service
```
systemctl restart openstack-nova-compute 
```

> ### Trên node COM2
Phân quyền
```
chgrp nova /etc/ceph/ceph.client.nova.keyring
chmod 0640 /etc/ceph/ceph.client.nova.keyring
```

Tạo file XML cho Ceph kết nối Compute
```
cat << EOF > nova-ceph.xml
<secret ephemeral="no" private="no">
<uuid>805b9716-7fe8-45dd-8e1e-5dfdeff8b9be</uuid>
<usage type="ceph">
<name>client.nova secret</name>
</usage>
</secret>
EOF

sudo virsh secret-define --file nova-ceph.xml

virsh secret-set-value --secret 805b9716-7fe8-45dd-8e1e-5dfdeff8b9be --base64 $(cat /root/client.nova)
```

Chỉnh sửa file `/etc/nova/nova.conf` tại COM
```
[libvirt]
...
images_rbd_pool=vms
images_type=rbd
rbd_secret_uuid=805b9716-7fe8-45dd-8e1e-5dfdeff8b9be
rbd_user=nova
images_rbd_ceph_conf = /etc/ceph/ceph.conf
```

Restart service
```
systemctl restart openstack-nova-compute 
```

## Kiểm tra
Boot VM từ local disk nova

Kiểm tra tại Ceph khi VM boot thành công
```
rbd -p volumes ls
```
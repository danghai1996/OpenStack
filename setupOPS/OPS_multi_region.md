# Multi Region

# Khái niệm Multi Region trong OPS
## Tổng quan
Trong Openstack có rất nhiều khái niệm dễ hiểu lầm như Domain, Region, Multi Site. Sau đây mình sẽ làm rõ các khái niệm

## Domain
Một hệ thống Cloud Openstack có thể được sử dụng bởi nhiều TỔ CHỨC, DOANH NGHIỆP, CÁ NHÂN, v.v, đồng thời openstack cũng cho phép tạo project có tên giống nhau.

Ví dụ, Phòng Kỹ Thuật của Nhân Hòa sử dụng project có tên là phong_ky_thuat, Phòng Kỹ Thuật của công ty Cloud365 cũng đặt tên project là phong_ky_thuat, khi đó trên hệ thống openstack sẽ có 2 project "phong_ky_thuat" nhưng lại thuộc sử hữu của 2 công ty khác nhau, điều này sẽ gây lẫn cho người quản trị.

Để hệ thống Openstack trở nên rõ ràng hơn, Keystone bổ sung khái niệm "Domain" nhằm cô lập các project giữa các tổ chức. Tức mỗi doanh nghiệp, tổ chức, cá nhân sẽ có 1 domain riêng, tại domain mỗi tổ chức, họ chỉ có thể thấy các project và user mà họ sở hữu, độc lập với "Domain" khác.

Quay lại ví dụ trên, công ty Nhân hoà sẽ sở hữu domain 'nhanhoa', công ty Cloud365 sẽ sở hữu domain 'cloud365'. Khi đó người quản trị của Nhân Hòa chỉ thấy các project, user thuộc sử hữu domain 'nhanhoa', tương tự với công ty Cloud365

<img src="..\images\Screenshot_40.png">

## Region
Khi hệ thống Cloud Openstack phát triển tới mức độ nhất định, việc triển khai 1 cụm Openstack là không đủ, các doanh nghiệp sẽ tính đến việc triển khai nhiều hệ thống Openstack tại nhiều địa điểm khác nhau phục vụ cho bài toán tối ưu vị trí địa lý.

Ví dụ: Công ty Nhân Hòa đã có một cụm Openstack tại Hà Nội, Công ty quyết định triển khai một cụm Openstack khác tại Đà Nẵng, một cụm Openstack Hồ Chí Minh để cung cấp dịch vụ tốt nhất cho khách hàng tại Hà Nội, Đà Năng, Hồ Chí Minh, đồng thời có yêu cầu 3 cụm Openstack phải sử dụng chung một hệ thống xác thực để phục vụ cho việc quản trị người dùng.

Khái niệm Region hay Multi Region sẽ được áp dụng cho ví dụ trên, khi chúng ta muốn triển khai nhiều cụm openstack khác nhau nhưng lại muốn sử dụng chúng Keystone.

<img src="..\images\Screenshot_142.png">

## Multi Site
Khái niệm Multi Site cũng gần giống với khái niệm Region nhưng khác biệt điểm, chúng ta sẽ có nhiều cụm Openstack nhưng sẽ không chia sẻ dịch vụ Keystone (Mỗi cụm đều chạy độc lập).

# Các mô hình triển khai Multi Regions
Để thực hiện triển khải Multi Region, ta sẽ cần triển khai các cụm OPS kháu nhau và chung hệ thống xác thực là project Keystone.

**Ví dụ:** 3 cụm OPS được đặt tại 3 DC khác nhau: Hà Nội, Đà Nẵng, TP.Hồ Chí Minh

## Centralized Keystone DB
Tức ta sẽ có 1 DB trung tâm của Keystone, các Region sẽ kết nối với Database thông qua đường WAN. (Không sử dụng Keystone DB)

## Asyncronous Keystone DB
Mỗi Region sẽ có 1 DB tuy nhiên sẽ duy trì 1 Master (Cho phép đọc ghi) còn lại sẽ là Slave (Chỉ đọc). Khi đó dữ liệu sẽ đồng bộ giữa các DB tại các Region

## Syncronous (Clustered) Keystone DB: 
Sử dụng MySQL/MariaDB Galera Cluster, Keystone database tại 3 Region sẽ được đồng bộ.

### Ưu nhược điểm các loại
Các tùy chọn sẽ có ưu nhược điểm khác nhau.

||Centralized Keystone DB|Asyncronous Keystone DB|Syncronous (Clustered) Keystone DB|
|-|-|-|-|
|Ưu điểm||||
|Nhược điểm||||

# Triển khai Multi Region trong Openstack Queens
### Chuẩn bị:
- Triển khai 2 cụm OPS theo [docs cài đặt manual](./OPS-queen-centos7-install.md).

## Mô hình
### Mô hình tích hợp
### Mô hình triển khai
**Mô hình:**

<img src="..\images\multi-region-queens.png">

**IP Planning:**

<img src="..\images\multi-region-ip-planing-queen.png">

# Triển khai
**Lưu ý:**

- Theo docs manual, Region mặc định sẽ là RegionOne, vì vậy chúng ta sẽ không phải cấu hình 2 node Controller 10.10.34.162 và compute 10.10.34.163
- Trong bài, mình sẽ chỉ cấu hình 2 node controller 10.10.34.169 và compute 10.10.34.170 sang thành RegionTwo, chia sẻ Keystone, Horizon với RegionOne
- Keystone chia sẻ giữa 2 cụm Openstack sẽ nằm trên Controller (của RegionOne) 10.10.34.162

## Bước 1: Tạo RegionTwo trên Controller 34.162
> Thực hiện trên node Controller RegioneOne 34.162

Tạo mới Region
```
keystone-manage bootstrap --bootstrap-password Welcome123 \
  --bootstrap-admin-url http://10.10.34.162:5000/v3/ \
  --bootstrap-internal-url http://10.10.34.162:5000/v3/ \
  --bootstrap-public-url http://10.10.34.162:5000/v3/ \
  --bootstrap-region-id RegionTwo
```

Kết quả
```
[root@controller1 ~]# openstack region list
+-----------+---------------+-------------+
| Region    | Parent Region | Description |
+-----------+---------------+-------------+
| RegionOne | None          |             |
| RegionTwo | None          |             |
+-----------+---------------+-------------+
```

Sau khi sử dụng keystone khởi tạo RegionTwo, keystone sẽ tự động tạo thêm endpoint identity mới
```
openstack endpoint list --service identity
+----------------------------------+-----------+--------------+--------------+---------+-----------+------------------------------+
| ID                               | Region    | Service Name | Service Type | Enabled | Interface | URL                          |
+----------------------------------+-----------+--------------+--------------+---------+-----------+------------------------------+
| 22ab059420f04df8890c4f1d3f4669d1 | RegionTwo | keystone     | identity     | True    | admin     | http://10.10.34.162:5000/v3/ |
| 60ef2b41c0014abeb5572458b69e6342 | RegionTwo | keystone     | identity     | True    | public    | http://10.10.34.162:5000/v3/ |
| 7238f9e83c6b48aea1e17775ff4ec8f9 | RegionOne | keystone     | identity     | True    | admin     | http://10.10.34.162:5000/v3/ |
| 7d68628998ee4c10a4637e0e0bdd3892 | RegionTwo | keystone     | identity     | True    | internal  | http://10.10.34.162:5000/v3/ |
| 8ea1bf2ad3de4c39aa1f2e5be8485e11 | RegionOne | keystone     | identity     | True    | public    | http://10.10.34.162:5000/v3/ |
| 9125846d310e4484b6803a9ed4aeeb8c | RegionOne | keystone     | identity     | True    | internal  | http://10.10.34.162:5000/v3/ |
+----------------------------------+-----------+--------------+--------------+---------+-----------+------------------------------+
```

## Bước 2: Khởi tạo các endpoint RegionTwo cho các service nova, glance, neutron
> Thực hiện trên node Controller RegioneOne 34.162

Lưu ý các endpoint tạo cho RegionTwo sẽ sử dụng IP của node Contrller của RegionTwo 10.10.34.169

```
openstack endpoint create --region RegionTwo image public http://10.10.34.169:9292
openstack endpoint create --region RegionTwo image admin http://10.10.34.169:9292
openstack endpoint create --region RegionTwo image internal http://10.10.34.169:9292

openstack endpoint create --region RegionTwo network public http://10.10.34.169:9696
openstack endpoint create --region RegionTwo network internal http://10.10.34.169:9696
openstack endpoint create --region RegionTwo network admin http://10.10.34.169:9696

openstack endpoint create --region RegionTwo compute public http://10.10.34.169:8774/v2.1
openstack endpoint create --region RegionTwo compute admin http://10.10.34.169:8774/v2.1
openstack endpoint create --region RegionTwo compute internal http://10.10.34.169:8774/v2.1

openstack endpoint create --region RegionTwo placement public http://10.10.34.169:8778
openstack endpoint create --region RegionTwo placement admin http://10.10.34.169:8778
openstack endpoint create --region RegionTwo placement internal http://10.10.34.169:8778

openstack endpoint create --region RegionTwo volumev2 public http://10.10.34.169:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionTwo volumev2 internal http://10.10.34.169:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionTwo volumev2 admin http://10.10.34.169:8776/v2/%\(project_id\)s

openstack endpoint create --region RegionTwo volumev3 public http://10.10.34.169:8776/v3/%\(project_id\)s
openstack endpoint create --region RegionTwo volumev3 internal http://10.10.34.169:8776/v3/%\(project_id\)s
openstack endpoint create --region RegionTwo volumev3 admin http://10.10.34.169:8776/v3/%\(project_id\)s
```

Kiểm tra
```
[root@controller1 ~]# openstack endpoint list --region RegionTwo
+----------------------------------+-----------+--------------+--------------+---------+-----------+--------------------------------------------+
| ID                               | Region    | Service Name | Service Type | Enabled | Interface | URL                                        |
+----------------------------------+-----------+--------------+--------------+---------+-----------+--------------------------------------------+
| 22ab059420f04df8890c4f1d3f4669d1 | RegionTwo | keystone     | identity     | True    | admin     | http://10.10.34.162:5000/v3/               |
| 2eb4186411964b2eb31cfdf781f61743 | RegionTwo | cinderv3     | volumev3     | True    | admin     | http://10.10.34.169:8776/v3/%(project_id)s |
| 45fae5987fd54c4596d8b88387c16c40 | RegionTwo | cinderv3     | volumev3     | True    | internal  | http://10.10.34.169:8776/v3/%(project_id)s |
| 45ff222e65de4fef85fb190bf3121f05 | RegionTwo | nova         | compute      | True    | admin     | http://10.10.34.169:8774/v2.1              |
| 5cd9c790940144f581791c1f52fa9d58 | RegionTwo | glance       | image        | True    | admin     | http://10.10.34.169:9292                   |
| 60ef2b41c0014abeb5572458b69e6342 | RegionTwo | keystone     | identity     | True    | public    | http://10.10.34.162:5000/v3/               |
| 704cfb6d3e8e4ec997a044f89a875606 | RegionTwo | neutron      | network      | True    | public    | http://10.10.34.169:9696                   |
| 7d68628998ee4c10a4637e0e0bdd3892 | RegionTwo | keystone     | identity     | True    | internal  | http://10.10.34.162:5000/v3/               |
| 7dfeb858c93f4676ac42f4fe3964e449 | RegionTwo | placement    | placement    | True    | admin     | http://10.10.34.169:8778                   |
| 7e3e5351f544465b97f3821a038994e7 | RegionTwo | nova         | compute      | True    | public    | http://10.10.34.169:8774/v2.1              |
| a64dfb69b07e47a4b953f35224b9119c | RegionTwo | placement    | placement    | True    | public    | http://10.10.34.169:8778                   |
| a8318cc089eb4d2aacc7d0d447701328 | RegionTwo | cinderv2     | volumev2     | True    | internal  | http://10.10.34.169:8776/v2/%(project_id)s |
| aea1cc4b07854cb29f2d9efe3b564789 | RegionTwo | glance       | image        | True    | internal  | http://10.10.34.169:9292                   |
| aebf242f38f1464eb49aee4d94bda110 | RegionTwo | neutron      | network      | True    | admin     | http://10.10.34.169:9696                   |
| b2358c2ba77f4fb7af25ca279e935f4e | RegionTwo | placement    | placement    | True    | internal  | http://10.10.34.169:8778                   |
| bae2d04233104d6ea7f2bbfcfda08fa7 | RegionTwo | cinderv2     | volumev2     | True    | admin     | http://10.10.34.169:8776/v2/%(project_id)s |
| d2b2459ddcb0465886b534499c03675b | RegionTwo | cinderv2     | volumev2     | True    | public    | http://10.10.34.169:8776/v2/%(project_id)s |
| db671a87446f4dfeb28d01cc7422a634 | RegionTwo | cinderv3     | volumev3     | True    | public    | http://10.10.34.169:8776/v3/%(project_id)s |
| eb0f03e6472d4f9e915b7579a04c445f | RegionTwo | neutron      | network      | True    | internal  | http://10.10.34.169:9696                   |
| eeb6b188c6384aa9855ecf4bbf427d24 | RegionTwo | nova         | compute      | True    | internal  | http://10.10.34.169:8774/v2.1              |
| fa0e35e1d58f490e851f2621531dd5a0 | RegionTwo | glance       | image        | True    | public    | http://10.10.34.169:9292                   |
+----------------------------------+-----------+--------------+--------------+---------+-----------+--------------------------------------------+
```

## Bước 3: Tạo admin openstack resource của RegionTwo
> Thực hiện trên cả 2 node Controller RegioneOne 34.162 và RegionTwo 34.169

```
cat << EOF >> admin-openrc-r2
export OS_REGION_NAME=RegionTwo
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=Welcome123
export OS_AUTH_URL=http://10.10.34.162:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
EOF
```

## Bước 4: Chỉnh sửa service Glance của RegionTwo
> Thực hiện trên node Controller RegioneTwo 34.169

Với Glance, chúng ta sẽ chỉnh sửa mục xác thực keystone về CTL 10.10.34.162

Ta sẽ thực hiện chỉnh sửa 2 file : `/etc/glance/glance-api.conf` và `/etc/glance/glance-registry.conf`

Thực hiện sửa các mục là :

Module `[keystone_authtoken]`:
- Sửa `auth_uri = http://10.10.34.169:5000` thành -> `auth_url = http://10.10.34.162:5000`
- Sửa `auth_url = http://10.10.34.169:5000` thành -> `auth_url = http://10.10.34.162:5000`
- Sửa `region_name = RegionOne` -> `region_name = RegionTwo`
File `/etc/glance/glance-api.conf`
```conf
[DEFAULT]
bind_host = 10.10.34.169
registry_host = 10.10.34.169
[cors]
[database]
connection = mysql+pymysql://glance:Welcome123@10.10.34.169/glance
[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
[image_format]
[keystone_authtoken]
auth_uri = http://10.10.34.162:5000
auth_url = http://10.10.34.162:5000
memcached_servers = 10.10.34.169:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = Welcome123
region_name = RegionTwo
[matchmaker_redis]
[oslo_concurrency]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
driver = messagingv2

[oslo_messaging_rabbit]
rabbit_ha_queues = true
rabbit_retry_interval = 1
rabbit_retry_backoff = 2
amqp_durable_queues= true

[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[paste_deploy]
flavor = keystone
[profiler]
[store_type_location_strategy]
[task]
[taskflow_executor]
```

File `/etc/glance/glance-registry.conf`
```conf
[DEFAULT]
bind_host = 10.10.34.169
[database]
connection = mysql+pymysql://glance:Welcome123@10.10.34.169/glance
[keystone_authtoken]
auth_uri = http://10.10.34.162:5000
auth_url = http://10.10.34.162:5000
memcached_servers = 10.10.34.169
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = Welcome123
region_name = RegionTwo
[matchmaker_redis]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
driver = messagingv2

[oslo_messaging_rabbit]
rabbit_ha_queues = true
rabbit_retry_interval = 1
rabbit_retry_backoff = 2
amqp_durable_queues= true

[oslo_messaging_zmq]
[oslo_policy]
[paste_deploy]
flavor = keystone
[profiler]
```

Khởi động lại dịch vụ
```
systemctl restart openstack-glance-api.service openstack-glance-registry.service
```

Upload image 
```
openstack image create "cirrosr2" --file cirros-0.3.5-x86_64-disk.img --disk-format qcow2 --container-format bare --public
```

Kiểm tra: thực hiện trên cả 2 node Controller RegionOne và RegionTwo đều sẽ cho kết quả giống nhau:
```
source admin-openrc-r2
openstack image list
+--------------------------------------+----------+--------+
| ID                                   | Name     | Status |
+--------------------------------------+----------+--------+
| 7a3b05fa-cfd7-445c-abd3-52df0ce22442 | cirros   | active |
| 4261a249-ab9b-49e0-8205-8ff0ba70c183 | cirrosr2 | active |
+--------------------------------------+----------+--------+
```

## Bước 5: Chỉnh sửa service Nova
> Thực hiện trên node Controller (34.169) và Compute (34.170) của RegioneTwo 

### Thực hiện trên node Controller RegionTwo 34.169
Với Nova, ta sẽ chỉnh sửa file cấu hình `/etc/nova/nova.conf`

Module `[cinder]`:
- Sửa `os_region_name = RegionOne` thành -> `os_region_name = RegionTwo`

Module `[keystone_authtoken]`:
- Sửa `auth_url = http://10.10.34.169:5000/v3` thành -> `auth_url = http://10.10.34.162:5000/v3`
- Sửa `region_name = RegionOne` thành -> `region_name = RegionTwo`

Module `[neutron]`
- Sửa `auth_url = http://10.10.34.169:35357` thành -> `auth_url = http://10.10.34.162:35357`
- Sửa `region_name = RegionOne` thành -> `region_name = RegionTwo`

Module `[placement]`
- Sửa `os_region_name = RegionOne` thành -> `os_region_name = RegionTwo`
- Sửa `auth_url = http://10.10.34.169:5000/v3` thành -> `auth_url = http://10.10.34.162:5000/v3`

File `/etc/nova/nova.conf`
```conf
[DEFAULT]
my_ip = 10.10.34.169
enabled_apis = osapi_compute,metadata
use_neutron = True
osapi_compute_listen=10.10.34.169
metadata_host=10.10.34.169
metadata_listen=10.10.34.169
metadata_listen_port=8775
firewall_driver = nova.virt.firewall.NoopFirewallDriver
transport_url = rabbit://openstack:Welcome123@10.10.34.169:5672
[api]
auth_strategy = keystone
[api_database]
connection = mysql+pymysql://nova:Welcome123@10.10.34.169/nova_api
[barbican]
[cache]
backend = oslo_cache.memcache_pool
enabled = true
memcache_servers = 10.10.34.169:11211
[cells]
[cinder]
os_region_name = RegionTwo
[compute]
[conductor]
[console]
[consoleauth]
[cors]
[crypto]
[database]
connection = mysql+pymysql://nova:Welcome123@10.10.34.169/nova
[devices]
[ephemeral_storage_encryption]
[filter_scheduler]
[glance]
api_servers = http://10.10.34.169:9292
[guestfs]
[healthcheck]
[hyperv]
[ironic]
[key_manager]
[keystone]
[keystone_authtoken]
auth_url = http://10.10.34.162:5000/v3
memcached_servers = 10.10.34.169:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = Welcome123
region_name = RegionTwo
[libvirt]
[matchmaker_redis]
[metrics]
[mks]
[neutron]
url = http://10.10.34.169:9696
auth_url = http://10.10.34.162:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionTwo
project_name = service
username = neutron
password = Welcome123
service_metadata_proxy = true
metadata_proxy_shared_secret = Welcome123
[notifications]
[osapi_v21]
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
driver = messagingv2

[oslo_messaging_rabbit]
rabbit_ha_queues = true
rabbit_retry_interval = 1
rabbit_retry_backoff = 2
amqp_durable_queues= true

[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[pci]
[placement]
os_region_name = RegionTwo
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://10.10.34.162:5000/v3
username = placement
password = Welcome123
[quota]
[rdp]
[remote_debug]
[scheduler]
discover_hosts_in_cells_interval = 300
[serial_console]
[service_user]
[spice]
[upgrade_levels]
[vault]
[vendordata_dynamic_auth]
[vmware]
[vnc]
novncproxy_host=10.10.34.169
enabled = true
vncserver_listen = 10.10.34.169
vncserver_proxyclient_address = 10.10.34.169
novncproxy_base_url = http://10.10.34.169:6080/vnc_auto.html
[workarounds]
[wsgi]
[xenserver]
[xvp]
```

Khởi động lại dịch vụ:
```
systemctl restart openstack-nova-api.service openstack-nova-scheduler.service openstack-nova-consoleauth.service openstack-nova-conductor.service openstack-nova-novncproxy.service
```

### Thực hiện trên node Compute RegionTwo 34.170
Chỉnh sửa file cấu hình nova `/etc/nova/nova.conf`

Module `[cinder]`
- Sửa `os_region_name = RegionOne` thành -> `os_region_name = RegionTwo`

Module `[keystone_authtoken]`
- Sửa `auth_url = http://10.10.34.169:5000/v3` -> `auth_url = http://10.10.34.162:5000/v3`
- Sửa `region_name = RegionOne` thành -> `region_name = RegionTwo`

Module `[neutron]`
- Sửa `auth_url = http://10.10.34.169:35357` thành -> `auth_url = http://10.10.34.162:35357`
- Sửa `region_name = RegionOne` thành -> `region_name = RegionTwo`

Module `[placement]`
- Sửa `auth_url = http://10.10.34.169:5000/v3` thành -> `auth_url = http://10.10.34.162:5000/v3`
- Sửa `os_region_name = RegionOne` -> `os_region_name = RegionTwo`


File cấu hình `/etc/nova/nova.conf`:
```conf
[DEFAULT]
enabled_apis = osapi_compute,metadata
transport_url = rabbit://openstack:Welcome123@10.10.34.169:5672
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
[api]
auth_strategy = keystone
[api_database]
[barbican]
[cache]
[cells]
[cinder]
os_region_name = RegionTwo
[compute]
[conductor]
[console]
[consoleauth]
[cors]
[crypto]
[database]
[devices]
[ephemeral_storage_encryption]
[filter_scheduler]
[glance]
api_servers = http://10.10.34.169:9292
[guestfs]
[healthcheck]
[hyperv]
[ironic]
[key_manager]
[keystone]
[keystone_authtoken]
auth_url = http://10.10.34.162:5000/v3
memcached_servers = 10.10.34.169:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = Welcome123
region_name = RegionTwo
[libvirt]
virt_type = kvm
[matchmaker_redis]
[metrics]
[mks]
[neutron]
url = http://10.10.34.169:9696
auth_url = http://10.10.34.162:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionTwo
project_name = service
username = neutron
password = Welcome123
[notifications]
[osapi_v21]
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
driver = messagingv2

[oslo_messaging_rabbit]
rabbit_ha_queues = true
rabbit_retry_interval = 1
rabbit_retry_backoff = 2
amqp_durable_queues= true
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[pci]
[placement]
os_region_name = RegionTwo
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://10.10.34.162:5000/v3
username = placement
password = Welcome123
[quota]
[rdp]
[remote_debug]
[scheduler]
discover_hosts_in_cells_interval = 300
[serial_console]
[service_user]
[spice]
[upgrade_levels]
[vault]
[vendordata_dynamic_auth]
[vmware]
[vnc]
enabled = True
server_listen = 0.0.0.0
server_proxyclient_address = 10.10.34.170
novncproxy_base_url = http://10.10.34.169:6080/vnc_auto.html
[workarounds]
[wsgi]
[xenserver]
[xvp]
```

Khởi động lại dịch vụ tại node Compute
```
systemctl restart libvirtd.service openstack-nova-compute
```

**Lưu ý:** Kiểm tra log tại Nova Compute
```
cat /var/log/nova/nova-compute.log | grep 'placement'

2020-09-06 21:18:21.875 2190 ERROR nova.scheduler.client.report [req-40e95940-d60b-481e-b783-bb2182318b8a - - - - -] [req-6ee8511d-897b-4c60-a74d-14251dd3a960] Failed to retrieve resource provider tree from placement API for UUID cba64545-11bc-444f-9253-971f7da921d2. Got 401: {"error": {"message": "The request you have made requires authentication.", "code": 401, "title": "Unauthorized"}}.
```

Nếu xuất hiện, kiểm tra config tại nova controller và nova compute, sau đó khởi động os CTL 34.169 vầ 34.170 thuộc RegionTwo (lỗi có thể do db hoặc cache)

Kiểm tra service trên node Controller của RegionOne (34.162)
```
[root@controller1 ~]# source admin-openrc-r2
[root@controller1 ~]# openstack compute service list --os-region-name RegionTwo
+----+------------------+-------------+----------+---------+-------+----------------------------+
| ID | Binary           | Host        | Zone     | Status  | State | Updated At                 |
+----+------------------+-------------+----------+---------+-------+----------------------------+
|  1 | nova-conductor   | controller1 | internal | enabled | up    | 2020-09-06T14:35:08.000000 |
|  2 | nova-consoleauth | controller1 | internal | enabled | up    | 2020-09-06T14:35:11.000000 |
|  3 | nova-scheduler   | controller1 | internal | enabled | up    | 2020-09-06T14:35:08.000000 |
|  6 | nova-compute     | compute1    | nova     | enabled | up    | 2020-09-06T14:35:08.000000 |
+----+------------------+-------------+----------+---------+-------+----------------------------+
```

## Bước 6: Chỉnh sửa service Cinder
> Thực hiện trên node Controller (34.169) của RegioneTwo

Ta thực hiện chỉnh sửa file cấu hình `/etc/cinder/cinder.conf`

Module `[DEFAULT]`
- Sửa `glance_api_servers = http://10.10.34.162:9292` thành -> `glance_api_servers = http://10.10.34.169:9292`

Module `[keystone_authtoken]`
- Sửa `auth_uri = http://10.10.34.169:5000` thành -> `auth_uri = http://10.10.34.162:5000`
- Sửa `auth_url = http://10.10.34.169:35357` thành -> `auth_url = http://10.10.34.162:35357`
- Sửa `region_name = RegionOne` thành -> `region_name = RegionTwo`

File cấu hình `/etc/cinder/cinder.conf`
```conf
[DEFAULT]
my_ip = 10.10.34.169
transport_url = rabbit://openstack:Welcome123@10.10.34.169:5672
auth_strategy = keystone
osapi_volume_listen = 10.10.34.169
enabled_backends = lvm
glance_api_servers = http://10.10.34.169:9292
[backend]
[backend_defaults]
[barbican]
[brcd_fabric_example]
[cisco_fabric_example]
[coordination]
[cors]
[database]
connection = mysql+pymysql://cinder:Welcome123@10.10.34.169/cinder
[fc-zone-manager]
[healthcheck]
[key_manager]
[keystone_authtoken]
auth_uri = http://10.10.34.162:5000
auth_url = http://10.10.34.162:35357
memcached_servers = 10.10.34.169:11211
auth_type = password
project_domain_id = default
user_domain_id = default
project_name = service
username = cinder
password = Welcome123
region_name = RegionTwo
[matchmaker_redis]
[nova]
[oslo_concurrency]
lock_path = /var/lib/cinder/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
driver = messagingv2

[oslo_messaging_rabbit]
rabbit_retry_interval = 1
rabbit_retry_backoff = 2
amqp_durable_queues = true
rabbit_ha_queues = true

[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[oslo_reports]
[oslo_versionedobjects]
[profiler]
[service_user]
[ssl]
[vault]
[lvm]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_group = cinder-volumes
iscsi_protocol = iscsi
iscsi_helper = lioadm
volume_backend_name = lvm
```

Khởi động lại dịch vụ
```
systemctl restart openstack-cinder-api.service
systemctl restart openstack-cinder-volume.service
systemctl restart openstack-cinder-scheduler.service
```

Kiểm tra dịch vụ trên node Controller 34.162 thuộc RegionOne
```
openstack volume service list --os-region-name RegionTwo
+------------------+-----------------+------+---------+-------+----------------------------+
| Binary           | Host            | Zone | Status  | State | Updated At                 |
+------------------+-----------------+------+---------+-------+----------------------------+
| cinder-scheduler | controller1     | nova | enabled | up    | 2020-09-06T14:52:25.000000 |
| cinder-volume    | controller1@lvm | nova | enabled | up    | 2020-09-06T14:52:27.000000 |
+------------------+-----------------+------+---------+-------+----------------------------+
```

Tạo 1 volume:
```
openstack volume create --size 5 vlm_test
```

Kiểm tra trên node Controller 34.162 của RegionOne
```
source admin-openrc-r2

openstack volume list
+--------------------------------------+----------+-----------+------+-------------+
| ID                                   | Name     | Status    | Size | Attached to |
+--------------------------------------+----------+-----------+------+-------------+
| c0a14080-3f6f-4e1d-8093-0d04fc748abe | vlm_test | available |    5 |             |
+--------------------------------------+----------+-----------+------+-------------+
```

## Bước 7: Chỉnh sửa service Neutron
> Thực hiện trên node Controller (34.169) và Compute (34.170) của RegioneTwo 

### Thực hiện trên node Controller 34.169
Chỉnh sửa file `/etc/neutron/neutron.conf`

Module `[keystone_authtoken]`:
- Sửa `auth_uri = http://10.10.34.169:5000` thành -> `auth_uri = http://10.10.34.162:5000`
- Sửa `auth_url = http://10.10.34.169:35357` thành -> `auth_url = http://10.10.34.162:35357`
- Sửa `region_name = RegionOne` thành -> `region_name = RegionTwo`

Module `[nova]`
- Sửa `auth_url = http://10.10.34.169:35357` thành -> `auth_url = http://10.10.34.162:35357`
- Sửa `region_name = RegionOne` thành -> `region_name = RegionTwo`

File cấu hình `/etc/neutron/neutron.conf`
```conf
[DEFAULT]
bind_host = 10.10.34.169
core_plugin = ml2
service_plugins = router
transport_url = rabbit://openstack:Welcome123@10.10.34.169:5672
auth_strategy = keystone
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true
allow_overlapping_ips = True
dhcp_agents_per_network = 2
[agent]
[cors]
[database]
connection = mysql+pymysql://neutron:Welcome123@10.10.34.169/neutron
[keystone_authtoken]
auth_uri = http://10.10.34.162:5000
auth_url = http://10.10.34.162:35357
memcached_servers = 10.10.34.169:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = Welcome123
region_name = RegionTwo
[matchmaker_redis]
[nova]
auth_url = http://10.10.34.162:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionTwo
project_name = service
username = nova
password = Welcome123
[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
driver = messagingv2

[oslo_messaging_rabbit]
rabbit_retry_interval = 1
rabbit_retry_backoff = 2
amqp_durable_queues = true
rabbit_ha_queues = true
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[quotas]
[ssl]
```

Khởi động lại dịch vụ
```
systemctl restart neutron-server.service neutron-linuxbridge-agent.service neutron-l3-agent.service
```

### Thực hiện trên node Compute RegionTwo 34.170
Chỉnh sửa file `/etc/neutron/neutron.conf`

Module `[keystone_authtoken]`
- Sửa `auth_uri = http://10.10.34.169:5000` thành -> `auth_uri = http://10.10.34.162:5000`
- Sửa `auth_url = http://10.10.34.169:35357` thành -> `auth_url = http://10.10.34.162:35357`
- Sửa `region_name = RegionOne` thành -> `region_name = RegionTwo`

File cấu hình `/etc/neutron/neutron.conf`
```conf
[DEFAULT]
transport_url = rabbit://openstack:Welcome123@10.10.34.169:5672
auth_strategy = keystone
[agent]
[cors]
[database]
[keystone_authtoken]
auth_uri = http://10.10.34.162:5000
auth_url = http://10.10.34.162:35357
memcached_servers = 10.10.34.169:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = Welcome123
region_name = RegionTwo
[matchmaker_redis]
[nova]
[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
driver = messagingv2

[oslo_messaging_rabbit]
rabbit_ha_queues = true
rabbit_retry_interval = 1
rabbit_retry_backoff = 2
amqp_durable_queues= true
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[quotas]
[ssl]
```

Khởi động lại dịch vụ
```
systemctl restart neutron-linuxbridge-agent.service neutron-dhcp-agent.service neutron-metadata-agent.service
```

Kiểm tra trên node Controller của RegionOne 34.162
```
source admin-openrc-r2

openstack network agent list --os-region-name RegionTwo
+--------------------------------------+--------------------+-------------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host        | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+-------------+-------------------+-------+-------+---------------------------+
| 6df3c5f4-fb48-40c2-87a2-eafe4f31298a | Metadata agent     | compute1    | None              | :-)   | UP    | neutron-metadata-agent    |
| 9911404f-a47e-4e6f-aecf-8328ce84e850 | DHCP agent         | compute1    | nova              | :-)   | UP    | neutron-dhcp-agent        |
| ad1d7392-081f-44f8-a0a7-7ec156fe7cae | L3 agent           | controller1 | nova              | :-)   | UP    | neutron-l3-agent          |
| d93a8ed7-f689-46ac-b98a-a098f23dbe2b | Linux bridge agent | compute1    | None              | :-)   | UP    | neutron-linuxbridge-agent |
| f5de4934-e83a-444b-b1da-e0914f896d37 | Linux bridge agent | controller1 | None              | :-)   | UP    | neutron-linuxbridge-agent |
+--------------------------------------+--------------------+-------------+-------------------+-------+-------+---------------------------+

openstack network list
+--------------------------------------+------------+--------------------------------------+
| ID                                   | Name       | Subnets                              |
+--------------------------------------+------------+--------------------------------------+
| f52c0616-1c29-437d-a074-6b362570c7cb | r2public1  | 79c0e166-5b97-43f7-a9ff-f5356247acdc |
| fbe72094-70a3-4c3c-ad14-913a17d64ec3 | r2private1 | f14f194a-b44b-42ad-99d7-a85ec37dabc5 |
+--------------------------------------+------------+--------------------------------------+
```

## 8. Chỉnh sửa Horizon
> Thực hiện trên node Controller (34.169) của RegionTwo

Sửa file `/etc/openstack-dashboard/local_settings`
- Sửa `OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST` thành -> `OPENSTACK_KEYSTONE_URL = "http://10.10.34.162:5000/v3"`

Restart service:
```
systemctl restart httpd.service memcached.service
```
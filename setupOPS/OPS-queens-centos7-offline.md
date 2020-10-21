# Cài đặt Openstack Queens offline trên CentOS-7

Sau khi đã backups các gói cài đặt OPS Queens. Ta sẽ sử dụng các gói đã backups để cài đặt các node của OPS.

[Hướng dẫn backup các gói cài đặt OPS Queens trên CentOS-7](./backups-package-OPS-queen-centos7.md)

### Mô hình

<img src="..\images\OPS-2node-queens.png">

## IP planning

<img src="..\images\ip-planing-queen.png">

**Yêu cầu thêm:** OS của các node phải cùng phiên bản với OS đã backups các gói cài đặt. Trong bài viết này là **CentOS-7.6.1810**


# Upload các gói cài đặt phù hợp đã backups lên các node
Sau khi backups thì các gói cài đặt, ta có thể upload lên các FTP server, hoặc copy ra USB, ... để lưu trữ.

Tiến hành tải các gói cài đặt về các node.

## 1. Node Controller
Tải thư mục các gói backups Controller về.

Kiểm tra các gói:
```
ls -l backup-ctl/
```

<img src="..\images\Screenshot_144.png">

## 2. Node Compute
Tải thư mục các gói backups Compute về.

Kiểm tra các gói:
```
ls -l backup-com/
```

<img src="..\images\Screenshot_145.png">

# I. Cài đặt môi trường
## 1. Cài đặt cơ bản
### 1.1. Trên node Controller
> Thực hiện trên node Controller

Thiết lập Hostname
```
hostnamectl set-hostname controller1
bash
```

Thiết lập IP
```
nmcli con modify eth0 ipv4.addresses 10.10.34.162/24
nmcli con modify eth0 ipv4.gateway 10.10.34.1
nmcli con modify eth0 ipv4.dns 8.8.8.8
nmcli con modify eth0 ipv4.method manual
nmcli con modify eth0 connection.autoconnect yes

nmcli con modify eth1 ipv4.addresses 10.10.31.162/24
nmcli con modify eth1 ipv4.method manual
nmcli con modify eth1 connection.autoconnect yes


nmcli con modify eth2 ipv4.addresses 10.10.35.162/24
nmcli con modify eth2 ipv4.method manual
nmcli con modify eth2 connection.autoconnect yes
```

Disable tường lửa và Selinux
```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
systemctl disable firewalld
systemctl stop firewalld
systemctl stop NetworkManager
systemctl disable NetworkManager
systemctl enable network
systemctl start network
```

Cấu hình các mode sysctl
```
echo 'net.ipv4.conf.all.arp_ignore = 1'  >> /etc/sysctl.conf
echo 'net.ipv4.conf.all.arp_announce = 2'  >> /etc/sysctl.conf
echo 'net.ipv4.conf.all.rp_filter = 2'  >> /etc/sysctl.conf
echo 'net.netfilter.nf_conntrack_tcp_be_liberal = 1'  >> /etc/sysctl.conf

cat << EOF >> /etc/sysctl.conf
net.ipv4.ip_nonlocal_bind = 1
net.ipv4.tcp_keepalive_time = 6
net.ipv4.tcp_keepalive_intvl = 3
net.ipv4.tcp_keepalive_probes = 6
net.ipv4.ip_forward = 1
net.ipv4.conf.all.rp_filter = 0
net.ipv4.conf.default.rp_filter = 0
EOF
```
Kiểm tra mode sysctl
```
sysctl -p
```

Khai báo file hosts:
```
echo "10.10.34.162 controller1" >> /etc/hosts
echo "10.10.34.163 compute1" >> /etc/hosts
```

Cài đặt các gói cho OPS Queens:
```
yum localinstall -y /root/backup-ctl/01-ops-ctr/*.rpm
```

Cài đặt các gói phần mềm thêm:
```
yum localinstall -y /root/backup-ctl/02-extra/*.rpm
```

Cài đặt các gói Python
```
yum localinstall -y /root/backup-ctl/03-python-ops/*.rpm
```

Reboot server
```
reboot
```

### 1.2. Trên node Compute1
> Thực hiện trên node Compute

Thiết lập Hostname
```
hostnamectl set-hostname compute1
bash
```

Thiết lập IP
```
nmcli con modify eth0 ipv4.addresses 10.10.34.163/24
nmcli con modify eth0 ipv4.gateway 10.10.34.1
nmcli con modify eth0 ipv4.dns 8.8.8.8
nmcli con modify eth0 ipv4.method manual
nmcli con modify eth0 connection.autoconnect yes

nmcli con modify eth1 ipv4.addresses 10.10.31.163/24
nmcli con modify eth1 ipv4.method manual
nmcli con modify eth1 connection.autoconnect yes


nmcli con modify eth2 ipv4.addresses 10.10.35.163/24
nmcli con modify eth2 ipv4.method manual
nmcli con modify eth2 connection.autoconnect yes
```

Disable tường lửa và Selinux
```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
systemctl disable firewalld
systemctl stop firewalld
systemctl stop NetworkManager
systemctl disable NetworkManager
systemctl enable network
systemctl start network
```

Cấu hình các mode sysctl
```
echo 'net.ipv4.conf.all.arp_ignore = 1'  >> /etc/sysctl.conf
echo 'net.ipv4.conf.all.arp_announce = 2'  >> /etc/sysctl.conf
echo 'net.ipv4.conf.all.rp_filter = 2'  >> /etc/sysctl.conf
echo 'net.netfilter.nf_conntrack_tcp_be_liberal = 1'  >> /etc/sysctl.conf

cat << EOF >> /etc/sysctl.conf
net.ipv4.ip_nonlocal_bind = 1
net.ipv4.tcp_keepalive_time = 6
net.ipv4.tcp_keepalive_intvl = 3
net.ipv4.tcp_keepalive_probes = 6
net.ipv4.ip_forward = 1
net.ipv4.conf.all.rp_filter = 0
net.ipv4.conf.default.rp_filter = 0
EOF

sysctl -p
```

Khai báo file hosts:
```
echo "10.10.34.162 controller1" >> /etc/hosts
echo "10.10.34.163 compute1" >> /etc/hosts
```

Cài đặt các package cho OpenStack Queens
```
yum localinstall -y /root/backup-com/01-ops-compute/*.rpm
```

Cài đặt các gói phần mềm mở rộng
```
yum localinstall -y /root/backup-com/02-extra/*.rpm
```

Cài đặt các gói Python
```
yum localinstall -y /root/backup-com/03-python-ops/*.rpm
```

Reboot server
```
reboot
```

### Quay lại node Controller
> Thực hiện trên node Controller
Tạo ssh key
```
ssh-keygen -t rsa -f /root/.ssh/id_rsa -q -P ""

ssh-copy-id -o StrictHostKeyChecking=no -i /root/.ssh/id_rsa.pub root@controller1
ssh-copy-id -o StrictHostKeyChecking=no -i /root/.ssh/id_rsa.pub root@compute1

scp /root/.ssh/id_rsa root@compute1:/root/.ssh/
```

## 2. Cài đặt NTP
### 2.1. Cài đặt trên node Controller
> Thực hiện trên node Controller
Trong bài lab này sẽ sử dụng Node Controller làm NTPD Server

Cài đặt từ các gói backups
```
yum localinstall -y /root/backup-ctl/04-chrony/*.rpm
```

Sao lưu file cấu hình của NTP:
```
cp /etc/chrony.conf /etc/chrony.conf.bak
```

Máy `controller` sẽ cập nhật thời gian từ internet hoặc máy chủ NTP. Các máy compute còn lại sẽ đồng bộ thời gian từ `controller`. Trong bài lab này sẽ sử dụng địa chỉ NTP của nội bộ.

Set timezone:
```
timedatectl set-timezone Asia/Ho_Chi_Minh
```

Sửa file cấu hình:
```
sed -i s'/0.centos.pool.ntp.org/10.10.34.130/'g /etc/chrony.conf

sed -i s'/server 1.centos.pool.ntp.org iburst/#server 1.centos.pool.ntp.org iburst/'g /etc/chrony.conf
sed -i s'/server 2.centos.pool.ntp.org iburst/#server 2.centos.pool.ntp.org iburst/'g /etc/chrony.conf
sed -i s'/server 3.centos.pool.ntp.org iburst/#server 3.centos.pool.ntp.org iburst/'g /etc/chrony.conf
```

Để cho phép các node compute kết nối với chrony trên node controller, ta sẽ sửa file cấu hình cho subnet chung của các node:
```
sed -i s'|#allow 192.168.0.0/16|allow 10.10.34.0/24|'g /etc/chrony.conf
```

Khởi động lại chrony sau khi sửa file cấu hình
```
systemctl restart chronyd
systemctl enable chronyd
```

Kiểm tra lại trạng thái của chrony xem đã OK hay chưa.
```
systemctl status chronyd
```

Kiểm tra xem đã đồng bộ thời gian chưa:
```
chronyc sources
```

### 2.2. Cài đặt trên node Compute1
> Thực hiện trên node Compute1

```
yum localinstall -y /root/backup-com/04-chrony/*.rpm 
```

Sao lưu file cấu hình của NTP:
```
cp /etc/chrony.conf /etc/chrony.conf.bak
```

Set timezone:
```
timedatectl set-timezone Asia/Ho_Chi_Minh
```

Các node compute sẽ đồng bộ thời gian từ node `controller1`. 

Sửa file cấu hình như sau:
```
sed -i 's/server 0.centos.pool.ntp.org iburst/server 10.10.34.162 iburst/g' /etc/chrony.conf

sed -i s'/server 1.centos.pool.ntp.org iburst/#server 1.centos.pool.ntp.org iburst/'g /etc/chrony.conf
sed -i s'/server 2.centos.pool.ntp.org iburst/#server 2.centos.pool.ntp.org iburst/'g /etc/chrony.conf
sed -i s'/server 3.centos.pool.ntp.org iburst/#server 3.centos.pool.ntp.org iburst/'g /etc/chrony.conf
```

Khởi động lại chrony sau khi sửa file cấu hình
```
systemctl restart chronyd
systemctl enable chronyd
```

Kiểm tra lại trạng thái của chrony xem đã OK hay chưa.
```
systemctl status chronyd
```

Kiểm tra xem đã đồng bộ thời gian chưa:
```
chronyc sources
```

## 3. Cài đặt và cấu hình Memcached
> Thực hiện trên node Controller

Cài đặt và cấu hình memcache
```
yum localinstall -y /root/backup-ctl/05-memcached/*.rpm
```
```
sed -i "s/-l 127.0.0.1,::1/-l 10.10.34.162 /g" /etc/sysconfig/memcached
```

Restart service memcached
```
systemctl enable memcached.service
systemctl restart memcached.service
```

## 4. Cài đặt và cấu hình MySQL galera
> Thực hiện trên node Controller

```
yum localinstall -y /root/backup-ctl/06-mysql/*.rpm
```

Backup cấu hình mysql
```
cp /etc/my.cnf.d/server.cnf /etc/my.cnf.d/server.cnf.orig
rm -rf /etc/my.cnf.d/server.cnf
```

Cấu hình mysql cho OpenStack
```
cat << EOF > /etc/my.cnf.d/openstack.cnf
[mysqld]
bind-address = 10.10.34.162 
default-storage-engine = innodb
innodb_file_per_table
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
EOF
```

Restart service
```
systemctl enable mariadb.service
systemctl restart mariadb.service
```

Đặt lại password cho user mysql. Ta đặt mật khẩu là `Welcome123`
```
mysql_secure_installation
```

## 5. Cài đặt và cấu hình RabbitMQ
> Thực hiện trên node Controller

```
yum localinstall -y /root/backup-ctl/07-rabbitmq-server/*.rpm
```

Cấu hình rabbitmq
```
systemctl enable rabbitmq-server.service
systemctl start rabbitmq-server.service
rabbitmq-plugins enable rabbitmq_management
systemctl restart rabbitmq-server
curl -O http://localhost:15672/cli/rabbitmqadmin
chmod a+x rabbitmqadmin
mv rabbitmqadmin /usr/sbin/
rabbitmqadmin list users
```

Thêm user openstack
```
rabbitmqctl add_user openstack Welcome123
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
rabbitmqctl set_user_tags openstack administrator
```

# II. Cài đặt các thành phần OpenStack
## 1. Cài đặt Keystone
> Thực hiện trên node Controller

Tạo db
```
mysql -u root -pWelcome123
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'Welcome123';
exit
```

Cài packages
```
yum localinstall -y /root/backup-ctl/08-keystone/*.rpm
```

Cấu hình bind port
```
cp /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/

sed -i -e 's/VirtualHost \*/VirtualHost 10.10.34.162/g' /etc/httpd/conf.d/wsgi-keystone.conf
sed -i -e 's/Listen 5000/Listen 10.10.34.162:5000/g' /etc/httpd/conf.d/wsgi-keystone.conf
sed -i -e 's/Listen 35357/Listen 10.10.34.162:35357/g' /etc/httpd/conf.d/wsgi-keystone.conf
sed -i -e 's/^Listen.*/Listen 10.10.34.162:80/g' /etc/httpd/conf/httpd.conf
```

Cấu hình keystone
```
cp /etc/keystone/keystone.conf /etc/keystone/keystone.conf.org
rm -rf /etc/keystone/keystone.conf

cat << EOF >> /etc/keystone/keystone.conf
[DEFAULT]
[assignment]
[auth]
[cache]
[catalog]
[cors]
[credential]
[database]
connection = mysql+pymysql://keystone:Welcome123@10.10.34.162/keystone
[domain_config]
[endpoint_filter]
[endpoint_policy]
[eventlet_server]
[federation]
[fernet_tokens]
[healthcheck]
[identity]
[identity_mapping]
[ldap]
[matchmaker_redis]
[memcache]
[oauth1]
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
[policy]
[profiler]
[resource]
[revoke]
[role]
[saml]
[security_compliance]
[shadow_users]
[signing]
[token]
provider = fernet
[tokenless_auth]
[trust]
EOF
```

Phân quyền file cấu hình
```
chown root:keystone /etc/keystone/keystone.conf
```

Sync db
```
su -s /bin/sh -c "keystone-manage db_sync" keystone
```

Set up fernet key
```
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
```

Bootstrap keystone
```
keystone-manage bootstrap --bootstrap-password Welcome123 \
--bootstrap-admin-url http://10.10.34.162:5000/v3/ \
--bootstrap-internal-url http://10.10.34.162:5000/v3/ \
--bootstrap-public-url http://10.10.34.162:5000/v3/ \
--bootstrap-region-id RegionOne
```

Enable và start httpd
```
systemctl enable httpd.service
systemctl restart httpd.service
```

Export biến môi trường
```
export OS_USERNAME=admin
export OS_PASSWORD=Welcome123
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://10.10.34.162:35357/v3
export OS_IDENTITY_API_VERSION=3
```

Tạo domain
```
openstack domain create --description "An Example Domain" example
openstack project create --domain default --description "Service Project" service
```

Tạo project và user
```
openstack project create --domain default  --description "Demo Project" demo
openstack user create --domain default --password Welcome123 demo
```

Tạo role và gắn role
```
openstack role create user
openstack role add --project demo --user demo user
```

Unset 2 biến môi trường
```
unset OS_AUTH_URL OS_PASSWORD
```

Tạo token
```
openstack --os-auth-url http://10.10.34.162:35357/v3 \
--os-project-domain-name Default --os-user-domain-name Default \
--os-project-name admin --os-username admin token issue
```

Tạo file xác thực
```
cat << EOF >> admin-openrc
export OS_REGION_NAME=RegionOne
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=Welcome123
export OS_AUTH_URL=http://10.10.34.162:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
EOF


cat << EOF >> demo-openrc
export OS_REGION_NAME=RegionOne
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=Welcome123
export OS_AUTH_URL=http://10.10.34.162:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
EOF
```

Kiểm tra cấu hình keystone
```
. admin-openrc
openstack token issue
```
Sau khi thực hiện câu lệnh hiện ra bảng token là OK

## 2. Cài đặt Glance
> Thực hiện trên node Controller

Tạo db
```
mysql -u root -pWelcome123
CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'Welcome123';
exit
```

Sử dụng biến môi trường
```
source admin-openrc
```

Tạo user
```
openstack user create --domain default --password Welcome123 glance
openstack role add --project service --user glance admin
openstack service create --name glance --description "OpenStack Image" image
```

Tạo endpoint
```
openstack endpoint create --region RegionOne image public http://10.10.34.162:9292
openstack endpoint create --region RegionOne image admin http://10.10.34.162:9292
openstack endpoint create --region RegionOne image internal http://10.10.34.162:9292
```

Cài packages
```
yum localinstall -y /root/backup-ctl/09-glance/*.rpm
```

Cấu hình glance api
```conf
cp /etc/glance/glance-api.conf /etc/glance/glance-api.conf.org 
rm -rf /etc/glance/glance-api.conf

cat << EOF >> /etc/glance/glance-api.conf
[DEFAULT]
bind_host = 10.10.34.162 
registry_host = 10.10.34.162 
[cors]
[database]
connection = mysql+pymysql://glance:Welcome123@10.10.34.162/glance
[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
[image_format]
[keystone_authtoken]
auth_uri = http://10.10.34.162:5000
auth_url = http://10.10.34.162:5000
memcached_servers = 10.10.34.162:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = Welcome123
region_name = RegionOne
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
EOF
```

Cấu hình glance registry
```conf
cp /etc/glance/glance-registry.conf /etc/glance/glance-registry.conf.org
rm -rf /etc/glance/glance-registry.conf

cat << EOF >> /etc/glance/glance-registry.conf
[DEFAULT]
bind_host = 10.10.34.162 
[database]
connection = mysql+pymysql://glance:Welcome123@10.10.34.162/glance
[keystone_authtoken]
auth_uri = http://10.10.34.162:5000
auth_url = http://10.10.34.162:5000
memcached_servers = 10.10.34.162 
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = Welcome123
region_name = RegionOne
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
EOF
```

Phân quyền file cấu hình
```
chown root:glance /etc/glance/glance-api.conf
chown root:glance /etc/glance/glance-registry.conf
```

Sync db
```
su -s /bin/sh -c "glance-manage db_sync" glance
```

Enable và start dịch vụ
```
systemctl enable openstack-glance-api.service openstack-glance-registry.service
systemctl start openstack-glance-api.service openstack-glance-registry.service
```

Download và tạo image
```
wget http://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img

openstack image create "cirros" \
--file cirros-0.3.5-x86_64-disk.img \
--disk-format qcow2 --container-format bare \
--public
```

List image hiện có:
```
openstack image list
+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| 5f296720-d6b4-43e7-af9f-f58a059b8b92 | cirros | active |
+--------------------------------------+--------+--------+
```

**Lưu ý:** Sau khi tạo images, mặc định image sẽ được đưa vào thư mục `/var/lib/glance/image`

## 3. Cài đặt Nova
### 3.1. Cài đặt Nova trên node Controller
> Thực hiện trên node Controller

Tạo database nova
```
mysql -u root -pWelcome123
CREATE DATABASE nova_api;
CREATE DATABASE nova;
CREATE DATABASE nova_cell0;
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY 'Welcome123';
exit
```

Tạo user và endpoint
```
openstack user create --domain default --password Welcome123 nova
openstack role add --project service --user nova admin
openstack service create --name nova --description "OpenStack Compute" compute

openstack endpoint create --region RegionOne compute public http://10.10.34.162:8774/v2.1
openstack endpoint create --region RegionOne compute admin http://10.10.34.162:8774/v2.1
openstack endpoint create --region RegionOne compute internal http://10.10.34.162:8774/v2.1

openstack user create --domain default --password Welcome123 placement
openstack role add --project service --user placement admin
openstack service create --name placement --description "Placement API" placement

openstack endpoint create --region RegionOne placement public http://10.10.34.162:8778
openstack endpoint create --region RegionOne placement admin http://10.10.34.162:8778
openstack endpoint create --region RegionOne placement internal http://10.10.34.162:8778
```

Tải packages
```
yum localinstall -y /root/backup-ctl/10-nova/*.rpm
```

Cấu hình nova
```
cp /etc/nova/nova.conf /etc/nova/nova.conf.org 
rm -rf /etc/nova/nova.conf

cat << EOF >> /etc/nova/nova.conf
[DEFAULT]
my_ip = 10.10.34.162 
enabled_apis = osapi_compute,metadata
use_neutron = True
osapi_compute_listen=10.10.34.162 
metadata_host=10.10.34.162 
metadata_listen=10.10.34.162 
metadata_listen_port=8775
firewall_driver = nova.virt.firewall.NoopFirewallDriver
transport_url = rabbit://openstack:Welcome123@10.10.34.162:5672
[api]
auth_strategy = keystone
[api_database]
connection = mysql+pymysql://nova:Welcome123@10.10.34.162/nova_api
[barbican]
[cache]
backend = oslo_cache.memcache_pool
enabled = true
memcache_servers = 10.10.34.162:11211
[cells]
[cinder]
os_region_name = RegionOne
[compute]
[conductor]
[console]
[consoleauth]
[cors]
[crypto]
[database]
connection = mysql+pymysql://nova:Welcome123@10.10.34.162/nova
[devices]
[ephemeral_storage_encryption]
[filter_scheduler]
[glance]
api_servers = http://10.10.34.162:9292
[guestfs]
[healthcheck]
[hyperv]
[ironic]
[key_manager]
[keystone]
[keystone_authtoken]
auth_url = http://10.10.34.162:5000/v3
memcached_servers = 10.10.34.162:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = Welcome123
region_name = RegionOne
[libvirt]
[matchmaker_redis]
[metrics]
[mks]
[neutron]
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
os_region_name = RegionOne
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
novncproxy_host=10.10.34.162 
enabled = true
vncserver_listen = 10.10.34.162 
vncserver_proxyclient_address = 10.10.34.162 
novncproxy_base_url = http://10.10.34.162:6080/vnc_auto.html
[workarounds]
[wsgi]
[xenserver]
[xvp]
EOF
```

Thêm vào file `00-nova-placement-api.conf`
```html
cat << 'EOF' >> /etc/httpd/conf.d/00-nova-placement-api.conf

<Directory /usr/bin>
    <IfVersion >= 2.4>
        Require all granted 
    </IfVersion>
    <IfVersion < 2.4>
        Order allow,deny
        Allow from all
    </IfVersion>
</Directory>
EOF
```

Cấu hình bind port cho nova-placement
```
sed -i -e 's/VirtualHost \*/VirtualHost 10.10.34.162/g' /etc/httpd/conf.d/00-nova-placement-api.conf
sed -i -e 's/Listen 8778/Listen 10.10.34.162:8778/g' /etc/httpd/conf.d/00-nova-placement-api.conf
```

Restart httpd
```
systemctl restart httpd
```

Sync db
```
su -s /bin/sh -c "nova-manage api_db sync" nova
su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
su -s /bin/sh -c "nova-manage db sync" nova
```

Xác nhận lại xem CELL0 đã được đăng ký hay chưa
```
su -s /bin/sh -c "nova-manage cell_v2 list_cells" nova
```

Màn hình sẽ xuất hiện kết quả như dưới là OK
```
+-------+--------------------------------------+-------------------------------------------+---------------------------------------------------+
|  Name |                 UUID                 |               Transport URL               |                Database Connection                |
+-------+--------------------------------------+-------------------------------------------+---------------------------------------------------+
| cell0 | 00000000-0000-0000-0000-000000000000 |                   none:/                  | mysql+pymysql://nova:****@10.10.34.162/nova_cell0 |
| cell1 | b3514c50-d263-4266-bdcd-87207ecb543b | rabbit://openstack:****@10.10.34.162:5672 |    mysql+pymysql://nova:****@10.10.34.162/nova    |
+-------+--------------------------------------+-------------------------------------------+---------------------------------------------------+
```

Enable và start service
```
systemctl enable openstack-nova-api.service \
openstack-nova-scheduler.service \
openstack-nova-consoleauth.service \
openstack-nova-conductor.service \
openstack-nova-novncproxy.service

systemctl start openstack-nova-api.service \
openstack-nova-scheduler.service \
openstack-nova-consoleauth.service \
openstack-nova-conductor.service \
openstack-nova-novncproxy.service
```

Kiểm tra lại dịch vụ
```
openstack compute service list
```

### 3.2. Cài đặt Nova trên node Compute
> Thực hiện trên node Compute

Cài nova
```
yum localinstall -y /root/backup-com/05-nova/*.rpm
```

Cấu hình nova
```
cp /etc/nova/nova.conf  /etc/nova/nova.conf.org
rm -rf /etc/nova/nova.conf

cat << EOF >> /etc/nova/nova.conf 
[DEFAULT]
enabled_apis = osapi_compute,metadata
transport_url = rabbit://openstack:Welcome123@10.10.34.162:5672
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
[api]
auth_strategy = keystone
[api_database]
[barbican]
[cache]
[cells]
[cinder]
os_region_name = RegionOne
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
api_servers = http://10.10.34.162:9292
[guestfs]
[healthcheck]
[hyperv]
[ironic]
[key_manager]
[keystone]
[keystone_authtoken]
auth_url = http://10.10.34.162:5000/v3
memcached_servers = 10.10.34.162:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = Welcome123
region_name = RegionOne
[libvirt]
virt_type = kvm
[matchmaker_redis]
[metrics]
[mks]
[neutron]
url = http://10.10.34.162:9696
auth_url = http://10.10.34.162:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
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
os_region_name = RegionOne
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
server_proxyclient_address = 10.10.34.163
novncproxy_base_url = http://10.10.34.162:6080/vnc_auto.html
[workarounds]
[wsgi]
[xenserver]
[xvp]
EOF
```

Phân quyền
```
chown root:nova /etc/nova/nova.conf
```

Enable và start nova
```
systemctl enable libvirtd.service openstack-nova-compute.service
systemctl start libvirtd.service openstack-nova-compute.service
```

Kiểm tra lại trên node Controller
```
openstack compute service list --service nova-compute
+----+--------------+----------+------+---------+-------+------------+
| ID | Binary       | Host     | Zone | Status  | State | Updated At |
+----+--------------+----------+------+---------+-------+------------+
|  6 | nova-compute | compute1 | nova | enabled | up    | None       |
+----+--------------+----------+------+---------+-------+------------+
```

## 4. Cài đặt Neutron
### 4.1. Cài đặt Neutron trên node Controller
> Thực hiện trên node Controller

Tạo database neutron
```
mysql -u root -pWelcome123
CREATE DATABASE neutron;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'Welcome123';
exit
```

Tạo user, endpoint trên 1 node
```
openstack user create --domain default --password Welcome123 neutron
openstack role add --project service --user neutron admin
openstack service create --name neutron --description "OpenStack Networking" network

openstack endpoint create --region RegionOne network public http://10.10.34.162:9696
openstack endpoint create --region RegionOne network internal http://10.10.34.162:9696
openstack endpoint create --region RegionOne network admin http://10.10.34.162:9696
```

Cài packages
```
yum localinstall -y /root/backup-ctl/11-neutron/*.rpm
```

Cấu hình neutron

**Lưu ý:** Mô hình này sử dụng mô hình mạng provider (flat) sử dụng linuxbridge
DHCP agent và metadata agent được chạy trên node compute
```
cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.org
rm -rf /etc/neutron/neutron.conf

cat << EOF >> /etc/neutron/neutron.conf
[DEFAULT]
bind_host = 10.10.34.162 
core_plugin = ml2
service_plugins = router
transport_url = rabbit://openstack:Welcome123@10.10.34.162:5672
auth_strategy = keystone
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true
allow_overlapping_ips = True
dhcp_agents_per_network = 2
[agent]
[cors]
[database]
connection = mysql+pymysql://neutron:Welcome123@10.10.34.162/neutron
[keystone_authtoken]
auth_uri = http://10.10.34.162:5000
auth_url = http://10.10.34.162:35357
memcached_servers = 10.10.34.162:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = Welcome123
region_name = RegionOne
[matchmaker_redis]
[nova]
auth_url = http://10.10.34.162:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = Welcome123
region_name = RegionOne
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
EOF
```

Cấu hình file `ml2`
```
cp /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugins/ml2/ml2_conf.ini.org
rm -rf /etc/neutron/plugins/ml2/ml2_conf.ini

cat << EOF >> /etc/neutron/plugins/ml2/ml2_conf.ini
[DEFAULT]
[l2pop]
[ml2]
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
mechanism_drivers = linuxbridge,l2population
extension_drivers = port_security
[ml2_type_flat]
[ml2_type_geneve]
[ml2_type_gre]
[ml2_type_vlan]
network_vlan_ranges = provider
[ml2_type_vxlan]
vni_ranges = 1:1000
[securitygroup]
enable_ipset = True
EOF
```

Cấu hình file LB agent
```
cp /etc/neutron/plugins/ml2/linuxbridge_agent.ini /etc/neutron/plugins/ml2/linuxbridge_agent.ini.org 
rm -rf /etc/neutron/plugins/ml2/linuxbridge_agent.ini

cat << EOF >> /etc/neutron/plugins/ml2/linuxbridge_agent.ini
[DEFAULT]
[agent]
[linux_bridge]
physical_interface_mappings = provider:eth1 
[network_log]
[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
[vxlan]
enable_vxlan = true
local_ip = 10.10.35.162
l2_population = true
EOF
```

Cấu hình trên file l3 agent
```
cp /etc/neutron/l3_agent.ini /etc/neutron/l3_agent.ini.org
rm -rf /etc/neutron/l3_agent.ini

cat << EOF >> /etc/neutron/l3_agent.ini
[DEFAULT]
interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
[agent]
[ovs]
EOF
```

Chỉnh sửa file `/etc/nova/nova.conf`
```
[neutron]
url = http://10.10.34.162:9696
auth_url = http://10.10.34.162:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = Welcome123
service_metadata_proxy = true
metadata_proxy_shared_secret = Welcome123
```

Restart lại dv nova-api
```
systemctl restart openstack-nova-api.service
```

Phân quyền file cấu hình
```
chown -R root:neutron /etc/neutron/
```

Tạo liên kết
```
ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
```

Sync db (bỏ qua các cảnh báo Warning)
```
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
```

Enable và start dịch vụ
```
systemctl restart openstack-nova-api.service \
openstack-nova-scheduler.service \
openstack-nova-consoleauth.service \
openstack-nova-conductor.service \
openstack-nova-novncproxy.service

systemctl enable neutron-server.service \
neutron-linuxbridge-agent.service \
neutron-l3-agent.service

systemctl restart neutron-server.service \
neutron-linuxbridge-agent.service \
neutron-l3-agent.service
```

### 4.2. Cài đặt Neutron trên node Compute
> Thực hiện trên node Compute

Cài neutron
```
yum localinstall -y /root/backup-com/06-neutron/*.rpm
```

Cấu hình neutron
```
cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.org 
rm -rf /etc/neutron/neutron.conf

cat << EOF >> /etc/neutron/neutron.conf
[DEFAULT]
transport_url = rabbit://openstack:Welcome123@10.10.34.162:5672
auth_strategy = keystone
[agent]
[cors]
[database]
[keystone_authtoken]
auth_uri = http://10.10.34.162:5000
auth_url = http://10.10.34.162:35357
memcached_servers = 10.10.34.162:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = Welcome123
region_name = RegionOne
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
EOF
```

Cấu hình file LB agent3
```
cp /etc/neutron/plugins/ml2/linuxbridge_agent.ini /etc/neutron/plugins/ml2/linuxbridge_agent.ini.org 
rm -rf /etc/neutron/plugins/ml2/linuxbridge_agent.ini

cat << EOF >> /etc/neutron/plugins/ml2/linuxbridge_agent.ini
[DEFAULT]
[agent]
[linux_bridge]
physical_interface_mappings = provider:eth1 
[network_log]
[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
[vxlan]
enable_vxlan = true
local_ip = 10.10.35.163
l2_population = true
EOF
```

Cấu hình dhcp agent
```
cp /etc/neutron/dhcp_agent.ini /etc/neutron/dhcp_agent.ini.org
rm -rf /etc/neutron/dhcp_agent.ini

cat << EOF >> /etc/neutron/dhcp_agent.ini
[DEFAULT]
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = true
force_metadata = True
[agent]
[ovs]
EOF
```

Cấu hình metadata agent
```
cp /etc/neutron/metadata_agent.ini /etc/neutron/metadata_agent.ini.org 
rm -rf /etc/neutron/metadata_agent.ini

cat << EOF >> /etc/neutron/metadata_agent.ini
[DEFAULT]
nova_metadata_host = 10.10.34.162
metadata_proxy_shared_secret = Welcome123
[agent]
[cache]
EOF
```

Phân quyền
```
chown root:neutron /etc/neutron/metadata_agent.ini /etc/neutron/neutron.conf /etc/neutron/dhcp_agent.ini /etc/neutron/plugins/ml2/linuxbridge_agent.ini
```

Restart dịch vụ nova
```
systemctl restart libvirtd.service openstack-nova-compute
```

Enable và start neutron
```
systemctl enable neutron-linuxbridge-agent.service \
neutron-dhcp-agent.service \
neutron-metadata-agent.service

systemctl restart neutron-linuxbridge-agent.service \
neutron-dhcp-agent.service \
neutron-metadata-agent.service
```

## 5. Cài đặt Horizon
> Thực hiện trên node Controller

Cài đặt packages
```
yum localinstall -y /root/backup-ctl/12-horizon/*.rpm
```

Tạo file direct
```
filehtml=/var/www/html/index.html
touch $filehtml
cat << EOF >> $filehtml
<html>
<head>
<META HTTP-EQUIV="Refresh" Content="0.5; URL=http://10.10.34.162/dashboard">
</head>
<body>
<center> <h1>Redirecting to OpenStack Dashboard</h1> </center>
</body>
</html>
EOF
```

Backup cấu hình
```
cp /etc/openstack-dashboard/local_settings /etc/openstack-dashboard/local_settings.org
```

Thay đổi cấu hình trong file `/etc/openstack-dashboard/local_settings`
```
ALLOWED_HOSTS = ['*',]
OPENSTACK_API_VERSIONS = {
"identity": 3,
"image": 2,
"volume": 2,
}
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = 'Default'
```

Lưu ý thêm `SESSION_ENGINE` vào trên dòng CACHE như bên dưới
```
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': ['10.10.34.162:11211',]
    }
}
OPENSTACK_HOST = "10.10.34.162"
OPENSTACK_KEYSTONE_URL = "http://10.10.34.162:5000/v3"
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
```

Lưu ý: Nếu chỉ sử dụng provider, chỉnh sửa các thông số sau
```
OPENSTACK_NEUTRON_NETWORK = {
    'enable_router': False,
    'enable_quotas': False,
    'enable_ipv6': False,
    'enable_distributed_router': False,
    'enable_ha_router': False,
    'enable_fip_topology_check': False,
}

TIME_ZONE = "Asia/Ho_Chi_Minh"
```

Thêm vào file `/etc/httpd/conf.d/openstack-dashboard.conf`
```
echo "WSGIApplicationGroup %{GLOBAL}" >> /etc/httpd/conf.d/openstack-dashboard.conf
```

Restart lại httpd
```
systemctl restart httpd.service memcached.service
```

Tiến hành truy cập địa chỉ `10.10.34.162` và đăng nhập bằng tài khoản `admin`/`Welcome123`

<img src="..\images\Screenshot_143.png">

## 6. Cài đặt Cinder
> Cài đặt trên node Controller

Tạo database cinder
```
mysql -u root -pWelcome123
CREATE DATABASE cinder;
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' IDENTIFIED BY 'Welcome123';  
exit
```

Tạo service, user và endpoint
```
openstack user create --domain default --password Welcome123 cinder
openstack role add --project service --user cinder admin
openstack service create --name cinderv2 --description "OpenStack Block Storage" volumev2
openstack service create --name cinderv3 --description "OpenStack Block Storage" volumev3


openstack endpoint create --region RegionOne volumev2 public http://10.10.34.162:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionOne volumev2 internal http://10.10.34.162:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionOne volumev2 admin http://10.10.34.162:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionOne volumev3 public http://10.10.34.162:8776/v3/%\(project_id\)s
openstack endpoint create --region RegionOne volumev3 internal http://10.10.34.162:8776/v3/%\(project_id\)s
openstack endpoint create --region RegionOne volumev3 admin http://10.10.34.162:8776/v3/%\(project_id\)s
```

Cài package Cinder với LVM
```
yum localinstall -y /root/backup-ctl/13-cinder/*.rpm
```

Khởi động dịch vụ LVM và cho phép khởi động cùng hệ thống.
```
systemctl enable lvm2-lvmetad.service
systemctl start lvm2-lvmetad.service
```

Kiểm tra dung lượng disk:
```
lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0              11:0    1 1024M  0 rom
vda             252:0    0   60G  0 disk
├─vda1          252:1    0    1G  0 part /boot
└─vda2          252:2    0   59G  0 part
  ├─centos-root 253:0    0 35.6G  0 lvm  /
  ├─centos-swap 253:1    0    6G  0 lvm  [SWAP]
  └─centos-home 253:2    0 17.4G  0 lvm  /home
vdb             252:16   0   60G  0 disk
```

Tạo LVM physical volume `/dev/vdb`
```
pvcreate /dev/vdb
```

Tạo LVM volume group `cinder-volumes`
```
vgcreate cinder-volumes /dev/vdb
```

Sửa file `/etc/lvm/lvm.conf` dòng 141, để LVM chỉ scan ổ `vdb` cho block storage
```
devices {
    ...
    filter = [ "a/vdb/", "r/.*/"]
```

Sửa cấu hình cinder
```
cp /etc/cinder/cinder.conf /etc/cinder/cinder.conf.bak 
rm -rf /etc/cinder/cinder.conf

cat << EOF >> /etc/cinder/cinder.conf
[DEFAULT]
my_ip = 10.10.34.162 
transport_url = rabbit://openstack:Welcome123@10.10.34.162:5672
auth_strategy = keystone
osapi_volume_listen = 10.10.34.162 
enabled_backends = lvm
glance_api_servers = http://10.10.34.162:9292
[backend]
[backend_defaults]
[barbican]
[brcd_fabric_example]
[cisco_fabric_example]
[coordination]
[cors]
[database]
connection = mysql+pymysql://cinder:Welcome123@10.10.34.162/cinder
[fc-zone-manager]
[healthcheck]
[key_manager]
[keystone_authtoken]
auth_uri = http://10.10.34.162:5000
auth_url = http://10.10.34.162:35357
memcached_servers = 10.10.34.162:11211
auth_type = password
project_domain_id = default
user_domain_id = default
project_name = service
username = cinder
password = Welcome123
region_name = RegionOne
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
EOF
```

Phân quyền file cấu hình
```
chown root:cinder /etc/cinder/cinder.conf
```

Sync db
```
su -s /bin/sh -c "cinder-manage db sync" cinder
```

Chỉnh sửa file `/etc/nova/nova.conf`
```
[cinder]
os_region_name = RegionOne
```

Restart dịch vụ nova api
```
systemctl restart openstack-nova-api.service
```

Enable và start dịch vụ
```
systemctl enable openstack-cinder-api.service \
openstack-cinder-volume.service \
openstack-cinder-scheduler.service

systemctl restart openstack-cinder-api.service \
openstack-cinder-volume.service \
openstack-cinder-scheduler.service
```
# Các bước cài đặt Openstack Queen trên CentOS 7

## IP planning

<img src="..\images\ip-planing-queen.png">

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
nmcli con modify eth0 ipv4.addresses 10.10.31.166/24
nmcli con modify eth0 ipv4.gateway 10.10.31.1
nmcli con modify eth0 ipv4.dns 8.8.8.8
nmcli con modify eth0 ipv4.method manual
nmcli con modify eth0 connection.autoconnect yes

nmcli con modify eth1 ipv4.addresses 10.10.32.166/24
nmcli con modify eth1 ipv4.method manual
nmcli con modify eth1 connection.autoconnect yes


nmcli con modify eth2 ipv4.addresses 10.10.35.166/24
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

Khai báo repo và cài đặt các gói cho OPS-Queens:
```
echo '[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.2/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1' >> /etc/yum.repos.d/MariaDB.repo

yum install -y epel-release
yum update -y
yum install -y centos-release-openstack-queens \
open-vm-tools python2-PyMySQL vim telnet wget curl 
yum install -y python-openstackclient openstack-selinux 
yum upgrade -y
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
nmcli con modify eth0 ipv4.addresses 10.10.31.167/24
nmcli con modify eth0 ipv4.gateway 10.10.31.1
nmcli con modify eth0 ipv4.dns 8.8.8.8
nmcli con modify eth0 ipv4.method manual
nmcli con modify eth0 connection.autoconnect yes

nmcli con modify eth1 ipv4.addresses 10.10.32.167/24
nmcli con modify eth1 ipv4.method manual
nmcli con modify eth1 connection.autoconnect yes


nmcli con modify eth2 ipv4.addresses 10.10.35.167/24
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

Khai báo repo và cài đặt các package cho OpenStack Queens
```
echo '[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.2/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1' >> /etc/yum.repos.d/MariaDB.repo

yum install -y epel-release
yum update -y
yum install -y centos-release-openstack-queens \
open-vm-tools python2-PyMySQL vim telnet wget curl 
yum install -y python-openstackclient openstack-selinux 
yum upgrade -y
```

Reboot server
```
reboot
```

## 2. Cài đặt NTP
### 2.1. Cài đặt trên node Controller
> Thực hiện trên node Controller
Trong bài lab này sẽ sử dụng Node Controller làm NTPD Server

Cài đặt package
```
yum -y install chrony
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
sed -i s'/0.centos.pool.ntp.org/10.10.35.150/'g /etc/chrony.conf

sed -i s'/server 1.centos.pool.ntp.org iburst/#server 1.centos.pool.ntp.org iburst/'g /etc/chrony.conf
sed -i s'/server 2.centos.pool.ntp.org iburst/#server 2.centos.pool.ntp.org iburst/'g /etc/chrony.conf
sed -i s'/server 3.centos.pool.ntp.org iburst/#server 3.centos.pool.ntp.org iburst/'g /etc/chrony.conf
```

Để cho phép các node compute kết nối với chrony trên node controller, ta sẽ sửa file cấu hình cho subnet chung của các node:
```
sed -i s'|#allow 192.168.0.0/16|allow 10.10.31.0/24|'g /etc/chrony.conf
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
yum install -y chrony 
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
sed -i 's/server 0.centos.pool.ntp.org iburst/server controller1 iburst/g' /etc/chrony.conf

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

## 3. Cài đặt MariaDB trên node Controller
> Chỉ thực hiện trên node Controller

Cài đặt package
```
yum install mariadb mariadb-server -y
```

Thêm config của mysql cho openstack
```
cat << EOF >> /etc/my.cnf.d/openstack.cnf 
[mysqld]
bind-address = 10.10.31.166
        
default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
EOF
```

Enable và start MariaDB
```
systemctl enable mariadb.service
systemctl start mariadb.service
```

Cài đặt passwd cho MySQL. Ta đặt pass là `Welcome123`
```
mysql_secure_installation <<EOF

y
Welcome123
Welcome123
y
y
y
y
EOF
```

## 4. Cài đặt RabbitMQ (Chỉ cài đặt trên node Controller)
> Thực hiện trên node Controller

Cài đặt package
```
yum install rabbitmq-server -y
```

Enable và start rabbitmq-server
```
systemctl start rabbitmq-server
systemctl enable rabbitmq-server
systemctl status rabbitmq-server
```

Cấu hình cho rabbitmq-server
```
rabbitmq-plugins enable rabbitmq_management
systemctl restart rabbitmq-server
curl -O http://localhost:15672/cli/rabbitmqadmin
chmod a+x rabbitmqadmin
mv rabbitmqadmin /usr/sbin/
rabbitmqadmin list users
```

Tạo user và gán quyền
```
rabbitmqctl add_user openstack Welcome123
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
systemctl enable rabbitmq-server.service
rabbitmqctl set_user_tags openstack administrator
```
Kiểm tra user vừa tạo
```
rabbitmqadmin list users
```
Đăng nhập vào Dashboard quản trị của Rabbit-mq
```
http://10.10.31.166:15672
user: openstack
password: Welcome123
```

## 5. Cài đặt Memcached (Chỉ cài đặt trên node Controller)
> Cài đặt trên node Controller

Cài đặt package
```
yum install memcached python-memcached -y 
```

Cấu hình cho memcached
```
sed -i "s/-l 127.0.0.1,::1/-l 10.10.31.166/g" /etc/sysconfig/memcached
```

Enable và start memcached
```
systemctl enable memcached.service
systemctl start memcached.service
```

# II. Cài đặt các project của OPS
## 1. Cài đặt Keystone
> Cấu hình trên node Controller

Tạo database, user và phân quyền cho keystone
- Tên database: `keystone`
- Tên user của database: `keystone`
- Mật khẩu: `Welcome123`

```
mysql -uroot -pWelcome123

CREATE DATABASE keystone;

GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'Welcome123';

GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'Welcome123';

GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'10.10.31.166' IDENTIFIED BY 'Welcome123';

FLUSH PRIVILEGES;

exit
```

Cài đặt package
```
yum install -y openstack-keystone httpd mod_wsgi
```

Backup cấu hình
```
mv /etc/keystone/keystone.{conf,conf.bk}
```

Cấu hình cho Keystone
```yaml
cat << EOF >> /etc/keystone/keystone.conf
[DEFAULT]
[assignment]
[auth]
[cache]
[catalog]
[cors]
[credential]
[database]
connection = mysql+pymysql://keystone:Welcome123@10.10.31.166/keystone
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
#driver = messagingv2
[oslo_messaging_rabbit]
#rabbit_retry_interval = 1
#rabbit_retry_backoff = 2
#amqp_durable_queues = true
#rabbit_ha_queues = true
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

Phân quyền lại config file
```
chown root:keystone /etc/keystone/keystone.conf
```

Đồng bộ database cho keystone
```
su -s /bin/sh -c "keystone-manage db_sync" keystone
```

Thiết lập Fernet key
```
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
```

Thiết lập boostrap cho Keystone
```
keystone-manage bootstrap --bootstrap-password Welcome123 \
--bootstrap-admin-url http://10.10.31.166:5000/v3/ \
--bootstrap-internal-url http://10.10.31.166:5000/v3/ \
--bootstrap-public-url http://10.10.31.166:5000/v3/ \
--bootstrap-region-id RegionOne
```

Cấu hình apache cho keystone
```
sed -i 's|#ServerName www.example.com:80|ServerName 10.10.31.166|g' /etc/httpd/conf/httpd.conf 
```

Create symlink cho keystone api
```
ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
ls /etc/httpd/conf.d/
```

Start & Enable apache
```
systemctl enable httpd.service
systemctl restart httpd.service
systemctl status httpd.service
```

Tạo file biến môi trường `openrc-admin` cho tài khoản quản trị
```
cat << EOF >> admin-openrc
export export OS_REGION_NAME=RegionOne
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=Welcome123
export OS_AUTH_URL=http://10.10.31.166:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
export PS1='[\u@\h \W(admin-openrc)]\$ '
EOF
```

Tạo file biến môi trường openrc-demo cho tài khoản demo
```
cat << EOF >> demo-openrc
export export OS_REGION_NAME=RegionOne
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=Welcome123
export OS_AUTH_URL=http://10.10.31.166:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
export PS1='[\u@\h \W(demo-openrc)]\$ '
EOF
```

Thực thi biến môi trường
```
source /root/admin-openrc
```

Kiểm tra lại hoạt động của keystone
```
openstack token issue
```

Màn hình xuất hiện như bên dưới là OK.
```
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                                                                                   |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2020-09-01T10:22:54+0000                                                                                                                                                                |
| id         | gAAAAABfThLuz786MQF5Z_nGQixRPlm0u0me0gjGov18eGTYs_aFX0axiSLTCc7TFbNanAa1JZmqPKR51Z7LKQOXgbKZXF7hrkh9c9h-CaA8SFr-Swuv7RX_xXlGG7F616-86dvtPN3gpEmX5GCMl92-CAc0tBgzGaSzGAB0WB4FCooRoxt0-44 |
| project_id | f9b90a09e7ad4cd791de7534fb34f39a                                                                                                                                                        |
| user_id    | 1950975c353942759b66acabb6dca5a4                                                                                                                                                        |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

### Tạo domain, projects, users, và roles
Sử dụng biến môi trường
```
source admin-openrc 
```

Tạo PJ Service
```
openstack project create --domain default --description "Service Project" service
```

Tạo PJ demo
```
openstack project create --domain default --description "Demo Project" demo
```

Tạo User demo và password
```
openstack user create --domain default --password Welcome123 demo
```

Tạo roles user
```
openstack role create user
```

Thêm roles user trên PJ demo
```
openstack role add --project demo --user demo user
```

## 2. Cài đặt Glance (Images Service)
> Chỉ cấu hình trên node Controller

Tạo database cho glance
```
mysql -uroot -pWelcome123

CREATE DATABASE glance;

GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'Welcome123';

GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'Welcome123';

GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'10.10.31.166' IDENTIFIED BY 'Welcome123';

FLUSH PRIVILEGES;

exit
```

Sử dụng biến môi trường
```
source admin-openrc
```

Tạo user glance
```
openstack user create --domain default --password Welcome123 glance
```

Thêm roles admin cho user glance trên project service
```
openstack role add --project service --user glance admin
```

Kiểm tra lại user glance
```
openstack role list --user glance --project service
```

Khởi tạo dịch vụ glance
```
openstack service create --name glance --description "OpenStack Image" image
```

Tạo các enpoint cho glane
```
openstack endpoint create --region RegionOne image public http://10.10.31.166:9292
openstack endpoint create --region RegionOne image internal http://10.10.31.166:9292
openstack endpoint create --region RegionOne image admin http://10.10.31.166:9292
```

Cài đặt package
```
yum install -y openstack-glance
```

Backup cấu hình glance-api
```
mv /etc/glance/glance-api.{conf,conf.bk}
```

Cấu hình glance-api
```
cat << EOF >> /etc/glance/glance-api.conf
[DEFAULT]
bind_host = 10.10.31.166
registry_host = 10.10.31.166
[cors]
[database]
connection = mysql+pymysql://glance:Welcome123@10.10.31.166/glance
[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
[image_format]
[keystone_authtoken]
auth_uri = http://10.10.31.166:5000
auth_url = http://10.10.31.166:5000
memcached_servers = 10.10.31.166:11211
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
#driver = messagingv2
[oslo_messaging_rabbit]
#rabbit_ha_queues = true
#rabbit_retry_interval = 1
#rabbit_retry_backoff = 2
#amqp_durable_queues= true
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

Phân quyền lại file cấu hình
```
chown root:glance /etc/glance/glance-api.conf
```

Backup cấu hình glance-registry
```
mv /etc/glance/glance-registry.{conf,conf.bk}
```

Cấu hình glance-registry
```
cat << EOF >> /etc/glance/glance-registry.conf
[DEFAULT]
bind_host = 10.10.31.166
[database]
connection = mysql+pymysql://glance:Welcome123@10.10.31.166/glance
[keystone_authtoken]
auth_uri = http://10.10.31.166:5000
auth_url = http://10.10.31.166:5000
memcached_servers = 10.10.31.166
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
#driver = messagingv2
[oslo_messaging_rabbit]
#rabbit_ha_queues = true
#rabbit_retry_interval = 1
#rabbit_retry_backoff = 2
#amqp_durable_queues= true
[oslo_messaging_zmq]
[oslo_policy]
[paste_deploy]
flavor = keystone
[profiler]
EOF
```

Phân quyền lại file cấu hình
```
chown root:glance /etc/glance/glance-registry.conf
```

Đồng bộ database cho glance
```
su -s /bin/sh -c "glance-manage db_sync" glance
```

Enable và restart Glance
```
systemctl enable openstack-glance-api.service openstack-glance-registry.service
systemctl start openstack-glance-api.service openstack-glance-registry.service
```

Download image cirros:
```
wget http://download.cirros-cloud.net/0.5.1/cirros-0.5.1-x86_64-disk.img
```

Upload image lên Glance
```
openstack image create "cirros" --file cirros-0.5.1-x86_64-disk.img \
--disk-format qcow2 --container-format bare --public
```


# Các bước cài đặt Openstack Train trên CentOS 7

# Mô hình và IP planning
### Mô hình

<img src="..\images\OPS-3node-train.png">

### IP planning

<img src="..\images\ip_planning.png">

# Các bước cài đặt
## 1. Cài đặt cơ bản
### 1.1. Trên Controller
> Thực hiện trên node Controller

Update các gói phần mềm và cài đặt các gói cơ bản:
```
yum update -y

yum install epel-release -y

yum update -y 

yum install -y wget byobu git vim
```

Thiết lập hostname
```
hostnamectl set-hostname controller

bash
```

Khai báo file `/etc/hosts`
```
echo "127.0.0.1 localhost" > /etc/hosts
echo "10.10.31.166 controller" >> /etc/hosts
echo "10.10.31.167 compute1" >> /etc/hosts
echo "10.10.31.168 compute2" >> /etc/hosts
```

Thiết lập IP theo phân hoạch cho `controller`
```
nmcli con modify eth0 ipv4.addresses 10.10.31.166/24
nmcli con modify eth0 ipv4.gateway 10.10.31.1
nmcli con modify eth0 ipv4.dns 8.8.8.8
nmcli con modify eth0 ipv4.method manual
nmcli con modify eth0 connection.autoconnect yes

nmcli con modify eth1 ipv4.addresses 10.10.34.166/24
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
reboot
```

### 1.2. Trên node compute1
> Thực hiện trên node compute1

Update và cài các gói cơ bản:
```
yum update -y

yum install epel-release -y

yum update -y 

yum install -y wget byobu git vim 
```

Thiết lập hostname:
```
hostnamectl set-hostname compute1

bash
```

Khai báo file `/etc/hosts`
```
echo "127.0.0.1 localhost" > /etc/hosts
echo "10.10.31.166 controller" >> /etc/hosts
echo "10.10.31.167 compute1" >> /etc/hosts
echo "10.10.31.168 compute2" >> /etc/hosts
```

Thiết lập IP theo phân hoạch cho `compute1`
```
nmcli con modify eth0 ipv4.addresses 10.10.31.167/24
nmcli con modify eth0 ipv4.gateway 10.10.31.1
nmcli con modify eth0 ipv4.dns 8.8.8.8
nmcli con modify eth0 ipv4.method manual
nmcli con modify eth0 connection.autoconnect yes

nmcli con modify eth1 ipv4.addresses 10.10.34.167/24
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
reboot
```

### 1.3. Trên node compute2
> Thực hiện trên node compute2 

Thực hiện tương tự trên node compute1

```
yum update -y

yum install epel-release -y

yum update -y 

yum install -y wget byobu git vim 

hostnamectl set-hostname compute2
bash

echo "127.0.0.1 localhost" > /etc/hosts
echo "10.10.31.166 controller" >> /etc/hosts
echo "10.10.31.167 compute1" >> /etc/hosts
echo "10.10.31.168 compute2" >> /etc/hosts

nmcli con modify eth0 ipv4.addresses 10.10.31.168/24
nmcli con modify eth0 ipv4.gateway 10.10.31.1
nmcli con modify eth0 ipv4.dns 8.8.8.8
nmcli con modify eth0 ipv4.method manual
nmcli con modify eth0 connection.autoconnect yes

nmcli con modify eth1 ipv4.addresses 10.10.34.168/24
nmcli con modify eth1 ipv4.method manual
nmcli con modify eth1 connection.autoconnect yes


nmcli con modify eth2 ipv4.addresses 10.10.35.168/24
nmcli con modify eth2 ipv4.method manual
nmcli con modify eth2 connection.autoconnect yes

sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
sudo systemctl disable firewalld
sudo systemctl stop firewalld
sudo systemctl stop NetworkManager
sudo systemctl disable NetworkManager
sudo systemctl enable network
sudo systemctl start network
reboot
```

## 2. Cài đặt OpenStack
Thực hiện cài đặt các gói trên OpenStack:

### 2.1. Cài đặt package cho OPS trên cả 3 node
> Khai báo repo cho OpenStack Train trên cả tất cả các node.

```
yum -y install centos-release-openstack-train

yum -y upgrade

yum -y install crudini wget vim

yum -y install python-openstackclient openstack-selinux python2-PyMySQL

yum -y update
```

### 2.2. Cài đặt NTP
#### 2.2.1. Cài đặt NTP trên node Controller
> Cài đặt đồng bộ thời gian cho `controller`. Trong hướng dẫn này sử dụng chrony để làm NTP.
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
sed -i s'/0.centos.pool.ntp.org/10.10.34.130/'g /etc/chrony.conf

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

Kết quả như dưới đây là thành công (thể hiện ở dấu *)
```
210 Number of sources = 1
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* 10.10.34.130                  5   6    17     3  -4123ns[ -147us] +/-   44ms
```

#### 2.2.1. Cài đặt NTP trên 2 node compute
> Cài đặt NTP trên 2 node compute

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

2 node compute sẽ đồng bộ thời gian từ node `controller`. 

Sửa file cấu hình như sau:
```
sed -i 's/server 0.centos.pool.ntp.org iburst/server controller iburst/g' /etc/chrony.conf

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

Kết quả như dưới đây là thành công (thể hiện ở dấu * controller)
```
210 Number of sources = 1
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* controller                    6   6    17     8  +3952ns[  -47us] +/-   45ms
```

Kiểm tra lại thời gian sau khi đồng bộ:
```
timedatectl
```
Kết quả:
```
      Local time: Wed 2020-06-24 09:46:09 +07
  Universal time: Wed 2020-06-24 02:46:09 UTC
        RTC time: Wed 2020-06-24 02:46:08
       Time zone: Asia/Ho_Chi_Minh (+07, +0700)
     NTP enabled: yes
NTP synchronized: yes
 RTC in local TZ: no
      DST active: n/a
```

### 2.3. Cài đặt và cấu hình memcached
Cơ chế xác thực dịch vụ cho các dịch vụ sử dụng `memcached` để cache các token.

> `Memcached` thường chạy trên node `controller` -> Cài đặt `memchached` trên node `controller`

Cài đặt memcached
```
yum -y install memcached python-memcached
```

Backups file cấu hình `memcached`:
```
cp /etc/sysconfig/memcached /etc/sysconfig/memcached.bak
```

Chỉnh sửa file cấu hình `memcached`:

Cấu hình dịch vụ để sử dụng địa chỉ IP quản lý của node `controller`. Điều này là để cho phép truy cập bởi các node khác thông qua dải mạng VLAN MGNT (dải mạng quản lý)
```
sed -i "s/-l 127.0.0.1,::1/-l 127.0.0.1,::1,10.10.31.166/g" /etc/sysconfig/memcached
```

Khởi động lại memcached
```
systemctl enable memcached.service

systemctl restart memcached.service
```

### 2.4. Cài đặt và cấu hình MariaDB trên node Controller
> Cài đặt và câu hình MariaDB trên node Controller

Hầu hết các dịch vụ của OPS sử dụng cơ sở dữ liệu SQL để lưu thông tin. DB thường sẽ chạy trên node `controller`. Các dịch vụ OpenStack cũng hỗ trợ các cơ sở dữ liệu SQL khác bao gồm PostgreSQL.


Cài đặt MariaDB
```
yum -y install mariadb mariadb-server python2-PyMySQL
```

Tạo và chỉnh sửa file cấu hình của Openstack: `/etc/my.cnf.d/openstack.cnf`

Ta sẽ tạo phần `[mysqld]` và set `bind-address` thành IP của node `controller` để các node khác truy cập quan mạng quản lý (VLAN MGNT)
```
cat <<EOF> /etc/my.cnf.d/openstack.cnf
[mysqld]
bind-address = 10.10.31.166

default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
EOF
```

Nội dung file `/etc/my.cnf.d/openstack.cnf`
```conf
[mysqld]
bind-address = 10.10.31.166

default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
```

Khởi động lại MariaDB
```
systemctl enable mariadb.service

systemctl start mariadb.service
```

Bảo mật dịch vụ SQL bằng cách chạy lệnh:
```
mysql_secure_installation
```
Trong bài này, ta đặt password là `Welcome123`

Sau khi xác thực bảo mật xong, ta truy cập db với user `root` với pass `Welcome123` thực hiện các lệnh sau:
```
mysql -u root -p

GRANT ALL PRIVILEGES ON *.* TO 'root'@'10.10.31.166' IDENTIFIED BY 'Welcome123' WITH GRANT OPTION;

FLUSH PRIVILEGES;

DROP USER 'root'@'::1';

exit
```

### 2.5.  Cài đặt & cấu hình rabbitmq trên máy `controller`
> Cài đặt và cấu hình rabbitmq trên node `controller`

OpenStack sử dụng hàng đợi tin nhắn (**Message queue**) để phối hợp các hoạt động và thông tin trạng thái giữa các dịch vụ.

Message queue thường chạy trên node `controller`. OpenStack hỗ trợ một số dịch vụ Message queue như là: RabbitMQ, Qpid, và ZeroMQ.

Tuy nhiên, hầu hết các bản phân phối gói OpenStack đều hỗ trợ dịch vụ hàng đợi tin nhắn cụ thể. Ta sẽ sử dụng Rabiitmq bởi vì hầu hết các phiên bản đều hỗ trợ nó.

Cài đặt `rabbitmq`
```
yum -y install rabbitmq-server
```

Khởi động `rabbitmq`
```
systemctl enable rabbitmq-server.service

systemctl start rabbitmq-server.service
```

Khai báo plugin cho `rabbitmq`
```
rabbitmq-plugins enable rabbitmq_management
```

Cấu hình trang quản lý rabbitmq trên UI
```
curl -O http://localhost:15672/cli/rabbitmqadmin

chmod a+x rabbitmqadmin

mv rabbitmqadmin /usr/sbin/
```

Tạo user `openstack` với mật khẩu `Welcome123` (có thể thay đổi pass tùy ý)
```
rabbitmqctl add_user openstack Welcome123
```

Cho phép user `openstack` vừa tạo có thể cấu hình, viết, đọc và truy cập:
```
# Cấp quyền
rabbitmqctl set_permissions openstack ".*" ".*" ".*"

# Set tag adminnistrator
rabbitmqctl set_user_tags openstack administrator

# Show danh sách user
rabbitmqadmin list users
```

Danh sách các user rabbitmq
```
+-----------+--------------------------------+--------------------------------------------------+---------------+
|   name    |       hashing_algorithm        |                  password_hash                   |     tags      |
+-----------+--------------------------------+--------------------------------------------------+---------------+
| guest     | rabbit_password_hashing_sha256 | QrQUtUlqQ6CbfMIj5SpE5405FUOeRYs+JAEJ6o6x6yC1yA1R | administrator |
| openstack | rabbit_password_hashing_sha256 | w6SBXYJ08Bxw+yK80mHNu/UJDanfxhOPAsereCmTJfPLfv9E | administrator |
+-----------+--------------------------------+--------------------------------------------------+---------------+
```

Sau đó có thể đăng nhập vào UI của rabbitmq bằng URL `http://IP_MANAGER_CONTROLLER:15672` với user và mật khẩu ở trên để kiểm tra. 

Trong bài này là `http://10.10.31.166:15672/` với user/pass: `openstack`/`Welcome123`

<img src="..\images\Screenshot_7.png">

Giao diện của RabiitMQ sau khi đăng nhập:

<img src="..\images\Screenshot_8.png">

### 2.6. Cài đặt và cấu hình Etcd trên node Controller
> Etcd được cài đặt trên node `controller`

ETCD là một ứng dụng lưu trữ dữ liệu phân tán theo theo kiểu key-value, nó được các services trong OpenStack sử dụng lưu trữ cấu hình, theo dõi các trạng thái dịch vụ và các tình huống khác.

Cài đặt etcd:
```
yum -y install etcd
```

Sao lưu file cấu hình của etcd
```
cp /etc/etcd/etcd.conf /etc/etcd/etcd.conf.bak
```

Chỉnh sửa file cấu hình của etcd. Lưu ý thay đúng IP (10.10.31.166) và hostname của `controller` đã được thiết lập ở trước đó.
```
sed -i '/ETCD_DATA_DIR=/cETCD_DATA_DIR="/var/lib/etcd/default.etcd"' /etc/etcd/etcd.conf

sed -i '/ETCD_LISTEN_PEER_URLS=/cETCD_LISTEN_PEER_URLS="http://10.10.31.166:2380"' /etc/etcd/etcd.conf

sed -i '/ETCD_LISTEN_CLIENT_URLS=/cETCD_LISTEN_CLIENT_URLS="http://10.10.31.166:2379"' /etc/etcd/etcd.conf

sed -i '/ETCD_NAME=/cETCD_NAME="controller"' /etc/etcd/etcd.conf

sed -i '/ETCD_INITIAL_ADVERTISE_PEER_URLS=/cETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.10.31.166:2380"' /etc/etcd/etcd.conf

sed -i '/ETCD_ADVERTISE_CLIENT_URLS=/cETCD_ADVERTISE_CLIENT_URLS="http://10.10.31.166:2379"' /etc/etcd/etcd.conf

sed -i '/ETCD_INITIAL_CLUSTER=/cETCD_INITIAL_CLUSTER="controller=http://10.10.31.166:2380"' /etc/etcd/etcd.conf

sed -i '/ETCD_INITIAL_CLUSTER_TOKEN=/cETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-01"' /etc/etcd/etcd.conf

sed -i '/ETCD_INITIAL_CLUSTER_STATE=/cETCD_INITIAL_CLUSTER_STATE="new"' /etc/etcd/etcd.conf
```

Hoặc vào sửa file bằng vi, vim, ...Chỉnh sửa các dòng trong file cấu hình 2 mục `#[Member]` và `#[Clustering]`
```conf
#[Member]
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="http://10.10.31.166:2380"
ETCD_LISTEN_CLIENT_URLS="http://10.10.31.166:2379"
ETCD_NAME="controller"
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.10.31.166:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://10.10.31.166:2379"
ETCD_INITIAL_CLUSTER="controller=http://10.10.31.166:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-01"
ETCD_INITIAL_CLUSTER_STATE="new"
```

Kích hoạt và khởi động `etcd`
```
systemctl enable etcd

systemctl restart etcd
```

Kiểm tra trạng thái dịch vụ
```
systemctl status etcd
```

### 2.7. Cài đặt và cấu hình Keystone
> Keystone được cài đặt trên node controller

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

Cài đặt keystone
```
yum -y install openstack-keystone httpd mod_wsgi
```

Sao lưu file cấu hình của keystone
```
cp /etc/keystone/keystone.conf /etc/keystone/keystone.conf.bak
```

Dùng lệnh crudini để sửa các dòng cần thiết file keystone
```
crudini --set /etc/keystone/keystone.conf database connection mysql+pymysql://keystone:Welcome123@10.10.31.166/keystone

crudini --set /etc/keystone/keystone.conf token provider fernet
```

Đảm bảo phân đúng quyền cho file cấu hình của keystone
```
chown root:keystone /etc/keystone/keystone.conf
```

Đồng bộ để sinh database cho keystone
```
su -s /bin/sh -c "keystone-manage db_sync" keystone
```

Sinh các file cho fernet
```
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone

keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
```

Sau khi chạy 02 lệnh ở trên, ta sẽ thấy thư mục `/etc/keystone/fernet-keys` được sinh ra và chứa các file key của fernet

Thiết lập boottrap cho keystone
```
keystone-manage bootstrap --bootstrap-password Welcome123 \
--bootstrap-admin-url http://10.10.31.166:5000/v3/ \
--bootstrap-internal-url http://10.10.31.166:5000/v3/ \
--bootstrap-public-url http://10.10.31.166:5000/v3/ \
--bootstrap-region-id RegionOne
```

Keystone sẽ sử dụng httpd để chạy service, các request vào keystone sẽ thông qua httpd. Do vậy cần cấu hình httpd để keystone sử dụng.

Sửa cấu hình httpd, mở file `/etc/httpd/conf/httpd.conf` để thêm sau dòng 95 cấu hình bên dưới (hoắc sửa dòng 95 cũng được)
```
ServerName controller
```

Tạo liên kết cho file /usr/share/keystone/wsgi-keystone.conf
```
ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
```

Khởi động và kích hoạt httpd
```
systemctl enable httpd.service

systemctl start httpd.service
```

Kiểm tra lại service của httpd
```
systemctl status httpd.service
```

Tạo file biến môi trường cho keystone
```
cat << EOF > /root/admin-openrc
export OS_USERNAME=admin
export OS_PASSWORD=Welcome123
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://10.10.31.166:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
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
| expires    | 2020-06-24T05:54:20+0000                                                                                                                                                                |
| id         | gAAAAABe8tx8KcGgaSULfkvS5w_8r2coWZ5s6zyK8PZ6cebj7mgT9aktjd-uw-XvcEdZmV2gW4eDfDJpVy4PmtWXEQTzsIPrQtng_yTOtN9vs9VzLAP0GfXSsSGSEubwihd3qXmLiLwuuFul9gemyVGcJ20UwE46Z4qCtLrdqeqbACpcjgRGFpE |
| project_id | 46161aba23f04e8d86f0ae79910a5c1d                                                                                                                                                        |
| user_id    | e4dbcd43e62a42798852f1f3637930bd                                                                                                                                                        |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

Khai báo user `demo`, project `demo`
```
openstack project create service --domain default --description "Service Project" 
openstack project create demo --domain default --description "Demo Project" 
openstack user create demo --domain default --password Welcome123
openstack role create user
openstack role add --project demo --user demo user
```

Kết thúc bước cài đặt keystone. Chuyển sang bước cài đặt tiếp theo.

### 2.8. Cài đặt và cấu hình Glance
> Cài đặt Glance trên node `controller`

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

Khai báo user cho service glance

Thực thi biến môi trường để sử dụng được CLI của OpenStack
```
source /root/admin-openrc
```

Tạo user, project cho glance
```
openstack user create  glance --domain default --password Welcome123

openstack role add --project service --user glance admin

openstack service create --name glance --description "OpenStack Image" image

openstack endpoint create --region RegionOne image public http://10.10.31.166:9292

openstack endpoint create --region RegionOne image internal http://10.10.31.166:9292

openstack endpoint create --region RegionOne image admin http://10.10.31.166:9292
```

Cài đặt glance

Cài đặt glance và các gói cần thiết.
```
yum install -y openstack-glance

yum install -y MySQL-python

yum install -y python-devel
```

Sao lưu file cấu hình glance
```
cp /etc/glance/glance-api.conf /etc/glance/glance-api.conf.bak
```

Cấu hình glance
```
crudini --set /etc/glance/glance-api.conf database connection  mysql+pymysql://glance:Welcome123@10.10.31.166/glance

crudini --set /etc/glance/glance-api.conf keystone_authtoken www_authenticate_uri http://10.10.31.166:5000
crudini --set /etc/glance/glance-api.conf keystone_authtoken auth_url  http://10.10.31.166:5000
crudini --set /etc/glance/glance-api.conf keystone_authtoken memcached_servers 10.10.31.166:11211
crudini --set /etc/glance/glance-api.conf keystone_authtoken auth_type password 
crudini --set /etc/glance/glance-api.conf keystone_authtoken project_domain_name Default
crudini --set /etc/glance/glance-api.conf keystone_authtoken user_domain_name Default
crudini --set /etc/glance/glance-api.conf keystone_authtoken project_name service
crudini --set /etc/glance/glance-api.conf keystone_authtoken username glance
crudini --set /etc/glance/glance-api.conf keystone_authtoken password Welcome123

crudini --set /etc/glance/glance-api.conf paste_deploy flavor keystone

crudini --set /etc/glance/glance-api.conf glance_store stores file,http
crudini --set /etc/glance/glance-api.conf glance_store default_store file
crudini --set /etc/glance/glance-api.conf glance_store filesystem_store_datadir /var/lib/glance/images/
```

Đồng bộ database cho glance
```
su -s /bin/sh -c "glance-manage db_sync" glance
```

Khởi động và kích hoạt glance
```
systemctl enable openstack-glance-api.service

systemctl start openstack-glance-api.service
```

Tải image và import vào glance
```
wget http://download.cirros-cloud.net/0.4.0/cirros-0.4.0-x86_64-disk.img

openstack image create "cirros" --file cirros-0.4.0-x86_64-disk.img --disk-format qcow2 --container-format bare --public
```

Kiểm tra lại xem image đã được up hay chưa bằng cách
```
openstack image list
```

Danh sách các image đã được up hiện ra tương tự sau:
```
+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| f6c2a88b-c725-4662-a3ab-30197af7d14b | cirros | active |
+--------------------------------------+--------+--------+
```

### 2.9. Cài đặt và cấu hình Placement
> Thực hiện cài đặt và cấu hình Placement trên node Controller

Thực hiện tạo database, user, mật khẩu cho placement.
```
mysql -uroot -pWelcome123 

CREATE DATABASE placement;

GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'localhost' IDENTIFIED BY 'Welcome123';

GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'%' IDENTIFIED BY 'Welcome123';

GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'10.10.31.166' IDENTIFIED BY 'Welcome123';

FLUSH PRIVILEGES;

exit
```

Thực thi biến môi trường để sử dụng được CLI của OpenStack
```
source /root/admin-openrc
```

Tạo service, gán quyền, enpoint cho placement.
```
openstack user create  placement --domain default --password Welcome123

openstack role add --project service --user placement admin

openstack service create --name placement --description "Placement API" placement

openstack endpoint create --region RegionOne placement public http://10.10.31.166:8778

openstack endpoint create --region RegionOne placement internal http://10.10.31.166:8778

openstack endpoint create --region RegionOne placement admin http://10.10.31.166:8778
```

Cài đặt placement
```
yum install -y openstack-placement-api
```

Sao lưu file cấu hình của placement
```
cp /etc/placement/placement.conf /etc/placement/placement.conf.bak
```

Cấu hình placement
```
crudini --set  /etc/placement/placement.conf placement_database connection mysql+pymysql://placement:Welcome123@10.10.31.166/placement

crudini --set  /etc/placement/placement.conf api auth_strategy keystone

crudini --set  /etc/placement/placement.conf keystone_authtoken auth_url  http://10.10.31.166:5000/v3

crudini --set  /etc/placement/placement.conf keystone_authtoken memcached_servers 10.10.31.166:11211

crudini --set  /etc/placement/placement.conf keystone_authtoken auth_type password

crudini --set  /etc/placement/placement.conf keystone_authtoken project_domain_name Default

crudini --set  /etc/placement/placement.conf keystone_authtoken user_domain_name Default

crudini --set  /etc/placement/placement.conf keystone_authtoken project_name service

crudini --set  /etc/placement/placement.conf keystone_authtoken username placement

crudini --set  /etc/placement/placement.conf keystone_authtoken password Welcome123
```

Khai báo phân quyền cho placement
```
cat <<EOF>> /etc/httpd/conf.d/00-nova-placement-api.conf
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

Tạo các bảng, đồng bộ dữ liệu cho placement
```
su -s /bin/sh -c "placement-manage db sync" placement
```

Khởi động lại httpd
```
systemctl restart httpd
```

### 2.10. Cài đặt Nova
#### 2.10.1. Cài đặt nova trên node controller
Tạo các database, user, mật khẩu cho services nova
```
mysql -uroot -pWelcome123

CREATE DATABASE nova_api;
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'10.10.31.166' IDENTIFIED BY 'Welcome123';

CREATE DATABASE nova;
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'10.10.31.166' IDENTIFIED BY 'Welcome123';

CREATE DATABASE nova_cell0;
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'10.10.31.166' IDENTIFIED BY 'Welcome123';
FLUSH PRIVILEGES;

exit
```

Thực thi biến môi trường để sử dụng được CLI của OpenStack
```
source /root/admin-openrc
```

Tạo endpoint cho nova
```
openstack user create nova --domain default --password Welcome123

openstack role add --project service --user nova admin

openstack service create --name nova --description "OpenStack Compute" compute

openstack endpoint create --region RegionOne compute public http://10.10.31.166:8774/v2.1

openstack endpoint create --region RegionOne compute internal http://10.10.31.166:8774/v2.1

openstack endpoint create --region RegionOne compute admin http://10.10.31.166:8774/v2.1
```

Cài đặt các gói cho nova
```
yum install -y openstack-nova-api openstack-nova-conductor openstack-nova-novncproxy openstack-nova-scheduler
```
Sao lưu file cấu hình của nova
```
cp /etc/nova/nova.conf /etc/nova/nova.conf.bak
```

Cấu hình cho nova
```
crudini --set /etc/nova/nova.conf DEFAULT my_ip 10.10.31.166
crudini --set /etc/nova/nova.conf DEFAULT use_neutron true
crudini --set /etc/nova/nova.conf DEFAULT firewall_driver nova.virt.firewall.NoopFirewallDriver
crudini --set /etc/nova/nova.conf DEFAULT enabled_apis osapi_compute,metadata
crudini --set /etc/nova/nova.conf DEFAULT transport_url rabbit://openstack:Welcome123@10.10.31.166:5672/

crudini --set /etc/nova/nova.conf api_database connection mysql+pymysql://nova:Welcome123@10.10.31.166/nova_api
crudini --set /etc/nova/nova.conf database connection mysql+pymysql://nova:Welcome123@10.10.31.166/nova
crudini --set /etc/nova/nova.conf api connection  mysql+pymysql://nova:Welcome123@10.10.31.166/nova

crudini --set /etc/nova/nova.conf keystone_authtoken www_authenticate_uri http://10.10.31.166:5000/
crudini --set /etc/nova/nova.conf keystone_authtoken auth_url http://10.10.31.166:5000/
crudini --set /etc/nova/nova.conf keystone_authtoken memcached_servers 10.10.31.166:11211
crudini --set /etc/nova/nova.conf keystone_authtoken auth_type password
crudini --set /etc/nova/nova.conf keystone_authtoken project_domain_name Default
crudini --set /etc/nova/nova.conf keystone_authtoken user_domain_name Default
crudini --set /etc/nova/nova.conf keystone_authtoken project_name service
crudini --set /etc/nova/nova.conf keystone_authtoken username nova
crudini --set /etc/nova/nova.conf keystone_authtoken password Welcome123

crudini --set /etc/nova/nova.conf vnc enabled true 
crudini --set /etc/nova/nova.conf vnc server_listen \$my_ip
crudini --set /etc/nova/nova.conf vnc server_proxyclient_address \$my_ip

crudini --set /etc/nova/nova.conf glance api_servers http://10.10.31.166:9292

crudini --set /etc/nova/nova.conf oslo_concurrency lock_path /var/lib/nova/tmp

crudini --set /etc/nova/nova.conf placement region_name RegionOne
crudini --set /etc/nova/nova.conf placement project_domain_name Default
crudini --set /etc/nova/nova.conf placement project_name service
crudini --set /etc/nova/nova.conf placement auth_type password
crudini --set /etc/nova/nova.conf placement user_domain_name Default
crudini --set /etc/nova/nova.conf placement auth_url http://10.10.31.166:5000/v3
crudini --set /etc/nova/nova.conf placement username placement
crudini --set /etc/nova/nova.conf placement password Welcome123

crudini --set /etc/nova/nova.conf scheduler discover_hosts_in_cells_interval 300

crudini --set /etc/nova/nova.conf neutron url http://10.10.31.166:9696
crudini --set /etc/nova/nova.conf neutron auth_url http://10.10.31.166:5000
crudini --set /etc/nova/nova.conf neutron auth_type password
crudini --set /etc/nova/nova.conf neutron project_domain_name Default
crudini --set /etc/nova/nova.conf neutron user_domain_name Default
crudini --set /etc/nova/nova.conf neutron project_name service
crudini --set /etc/nova/nova.conf neutron username neutron
crudini --set /etc/nova/nova.conf neutron password Welcome123
crudini --set /etc/nova/nova.conf neutron service_metadata_proxy True
crudini --set /etc/nova/nova.conf neutron metadata_proxy_shared_secret Welcome123
```

Thực hiện các lệnh để sinh các bảng cho nova
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

Màn hình sẽ xuất hiện kết quả
```
+-------+--------------------------------------+--------------------------------------------+---------------------------------------------------+----------+
|  Name |                 UUID                 |               Transport URL                |                Database Connection                | Disabled |
+-------+--------------------------------------+--------------------------------------------+---------------------------------------------------+----------+
| cell0 | 00000000-0000-0000-0000-000000000000 |                   none:/                   | mysql+pymysql://nova:****@10.10.31.166/nova_cell0 |  False   |
| cell1 | 425797ea-22c7-4cfb-8400-e1e56e273882 | rabbit://openstack:****@10.10.31.166:5672/ |    mysql+pymysql://nova:****@10.10.31.166/nova    |  False   |
+-------+--------------------------------------+--------------------------------------------+---------------------------------------------------+----------+
```

Kích hoạt các dịch vụ của nova
```
systemctl enable \
openstack-nova-api.service \
openstack-nova-scheduler.service \
openstack-nova-conductor.service \
openstack-nova-novncproxy.service
```

Khởi động các dịch vụ của nova
```
systemctl start \
openstack-nova-api.service \
openstack-nova-scheduler.service \
openstack-nova-conductor.service \
openstack-nova-novncproxy.service
```

Kiểm tra lại xem dịch vụ của nova đã hoạt động hay chưa.
```
openstack compute service list
```

Kết quả như sau là OK
```
+----+----------------+------------+----------+---------+-------+----------------------------+
| ID | Binary         | Host       | Zone     | Status  | State | Updated At                 |
+----+----------------+------------+----------+---------+-------+----------------------------+
|  4 | nova-conductor | controller | internal | enabled | up    | 2020-06-24T07:37:52.000000 |
|  5 | nova-scheduler | controller | internal | enabled | up    | 2020-06-24T07:37:55.000000 |
+----+----------------+------------+----------+---------+-------+----------------------------+
```

#### 2.10.2. Cài đặt Nova trên node compute
> Cài đặt Nova trên node compute1

Cài đặt các gói của nova
```
yum install -y python-openstackclient openstack-selinux openstack-utils

yum install -y openstack-nova-compute
```

Sao lưu file cấu hình của nova
```
cp  /etc/nova/nova.conf  /etc/nova/nova.conf.bak
```

Cấu hình nova
```
crudini --set /etc/nova/nova.conf DEFAULT enabled_apis osapi_compute,metadata
crudini --set /etc/nova/nova.conf DEFAULT transport_url rabbit://openstack:Welcome123@10.10.31.166
crudini --set /etc/nova/nova.conf DEFAULT my_ip 10.10.31.167
crudini --set /etc/nova/nova.conf DEFAULT use_neutron true
crudini --set /etc/nova/nova.conf DEFAULT firewall_driver nova.virt.firewall.NoopFirewallDriver

crudini --set /etc/nova/nova.conf api_database connection mysql+pymysql://nova:Welcome123@10.10.31.166/nova_api

crudini --set /etc/nova/nova.conf database connection = mysql+pymysql://nova:Welcome123@10.10.31.166/nova

crudini --set /etc/nova/nova.conf api auth_strategy keystone

crudini --set /etc/nova/nova.conf keystone_authtoken www_authenticate_uri http://10.10.31.166:5000/
crudini --set /etc/nova/nova.conf keystone_authtoken auth_url http://10.10.31.166:5000/
crudini --set /etc/nova/nova.conf keystone_authtoken memcached_servers 10.10.31.166:11211
crudini --set /etc/nova/nova.conf keystone_authtoken auth_type password
crudini --set /etc/nova/nova.conf keystone_authtoken project_domain_name Default
crudini --set /etc/nova/nova.conf keystone_authtoken user_domain_name Default
crudini --set /etc/nova/nova.conf keystone_authtoken project_name service
crudini --set /etc/nova/nova.conf keystone_authtoken username nova
crudini --set /etc/nova/nova.conf keystone_authtoken password Welcome123

crudini --set /etc/nova/nova.conf vnc enabled true
crudini --set /etc/nova/nova.conf vnc server_listen 0.0.0.0
crudini --set /etc/nova/nova.conf vnc server_proxyclient_address \$my_ip
crudini --set /etc/nova/nova.conf vnc novncproxy_base_url http://10.10.31.166:6080/vnc_auto.html

crudini --set /etc/nova/nova.conf glance api_servers http://10.10.31.166:9292

crudini --set /etc/nova/nova.conf oslo_concurrency lock_path /var/lib/nova/tmp

crudini --set /etc/nova/nova.conf placement region_name RegionOne
crudini --set /etc/nova/nova.conf placement project_domain_name Default
crudini --set /etc/nova/nova.conf placement project_name service
crudini --set /etc/nova/nova.conf placement auth_type password
crudini --set /etc/nova/nova.conf placement user_domain_name Default
crudini --set /etc/nova/nova.conf placement auth_url http://10.10.31.166:5000/v3
crudini --set /etc/nova/nova.conf placement username placement
crudini --set /etc/nova/nova.conf placement password Welcome123

crudini --set /etc/nova/nova.conf libvirt virt_type  $(count=$(egrep -c '(vmx|svm)' /proc/cpuinfo); if [ $count -eq 0 ];then   echo "qemu"; else   echo "kvm"; fi)
```

Khởi động lại nova
```
systemctl enable libvirtd.service openstack-nova-compute.service

systemctl start libvirtd.service openstack-nova-compute.service
```

> Cài đặt trên node compute2 tương tự. (Chú ý thay đổi IP)

#### 2.10.3. Thêm node compute vào hệ thống
> Thực hiện trên node `controller`

Login vào máy chủ `controller` và thực hiện lệnh dưới để kiểm tra xem `compute1` và `compute2` đã up hay chưa:
```
source /root/admin-openrc

openstack compute service list --service nova-compute
```

Kết quả ta sẽ thấy như bên dưới là ok.
```
+----+--------------+----------+------+---------+-------+----------------------------+
| ID | Binary       | Host     | Zone | Status  | State | Updated At                 |
+----+--------------+----------+------+---------+-------+----------------------------+
|  6 | nova-compute | compute1 | nova | enabled | up    | 2020-06-24T08:01:25.000000 |
|  7 | nova-compute | compute2 | nova | enabled | up    | 2020-06-24T08:08:11.000000 |
+----+--------------+----------+------+---------+-------+----------------------------+
```

Thực hiện add node `compute` vào CELL
```
su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova
```

Kết quả màn hình sẽ hiển thị tương tự như bên dưới.
```
Found 2 cell mappings.
Skipping cell0 since it does not contain hosts.
Getting computes from cell 'cell1': 425797ea-22c7-4cfb-8400-e1e56e273882
Found 0 unmapped computes in cell: 425797ea-22c7-4cfb-8400-e1e56e273882
```

### 2.11. Cài đặt Neutron
#### 2.11.1. Cài đặt Neutron trên node Controller
> Cài đặt Neutron trên node Controller

Tạo database cho neutron
```
mysql -uroot -pWelcome123

CREATE DATABASE neutron;

GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'Welcome123';

GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'Welcome123';

GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'10.10.31.166' IDENTIFIED BY 'Welcome123';

FLUSH PRIVILEGES;

exit
```

Tạo project, user, endpoint cho neutron
```
source /root/admin-openrc

openstack user create neutron --domain default --password Welcome123

openstack role add --project service --user neutron admin

openstack service create --name neutron --description "OpenStack Compute" network

openstack endpoint create --region RegionOne network public http://10.10.31.166:9696

openstack endpoint create --region RegionOne network internal http://10.10.31.166:9696

openstack endpoint create --region RegionOne network admin http://10.10.31.166:9696
```

Cài đặt `neutron` cho `controller`
```
yum install -y openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables
```

Sao lưu các file cấu hình của neutron
```
cp  /etc/neutron/neutron.conf  /etc/neutron/neutron.conf.bak

cp /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugins/ml2/ml2_conf.ini.bak

cp  /etc/neutron/plugins/ml2/linuxbridge_agent.ini  /etc/neutron/plugins/ml2/linuxbridge_agent.ini.bak 

cp /etc/neutron/dhcp_agent.ini /etc/neutron/dhcp_agent.ini.bak

cp /etc/neutron/metadata_agent.ini /etc/neutron/metadata_agent.ini.bak
```

Cấu hình file `/etc/neutron/neutron.conf`
```
crudini --set  /etc/neutron/neutron.conf DEFAULT core_plugin ml2
crudini --set  /etc/neutron/neutron.conf DEFAULT service_plugins
crudini --set  /etc/neutron/neutron.conf DEFAULT transport_url rabbit://openstack:Welcome123@10.10.31.166
crudini --set  /etc/neutron/neutron.conf DEFAULT auth_strategy keystone
crudini --set  /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_status_changes True
crudini --set  /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_data_changes True 

crudini --set  /etc/neutron/neutron.conf database connection  mysql+pymysql://neutron:Welcome123@10.10.31.166/neutron

crudini --set  /etc/neutron/neutron.conf keystone_authtoken www_authenticate_uri http://10.10.31.166:5000
crudini --set  /etc/neutron/neutron.conf keystone_authtoken auth_url http://10.10.31.166:5000
crudini --set  /etc/neutron/neutron.conf keystone_authtoken memcached_servers 10.10.31.166:11211
crudini --set  /etc/neutron/neutron.conf keystone_authtoken auth_type password
crudini --set  /etc/neutron/neutron.conf keystone_authtoken project_domain_name default
crudini --set  /etc/neutron/neutron.conf keystone_authtoken user_domain_name default
crudini --set  /etc/neutron/neutron.conf keystone_authtoken project_name service
crudini --set  /etc/neutron/neutron.conf keystone_authtoken username neutron
crudini --set  /etc/neutron/neutron.conf keystone_authtoken password Welcome123

crudini --set /etc/neutron/neutron.conf nova auth_url http://10.10.31.166:5000
crudini --set /etc/neutron/neutron.conf nova auth_type password
crudini --set /etc/neutron/neutron.conf nova project_domain_name Default
crudini --set /etc/neutron/neutron.conf nova user_domain_name Default
crudini --set /etc/neutron/neutron.conf nova region_name RegionOne
crudini --set /etc/neutron/neutron.conf nova project_name service
crudini --set /etc/neutron/neutron.conf nova username nova
crudini --set /etc/neutron/neutron.conf nova password Welcome123

crudini --set /etc/neutron/neutron.conf oslo_concurrency lock_path /var/lib/neutron/tmp
```

Sửa file cấu hình của `/etc/neutron/plugins/ml2/ml2_conf.ini`
```
crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 type_drivers flat,vlan,vxlan
crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 tenant_network_types vxlan
crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 mechanism_drivers linuxbridge
crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 extension_drivers port_security          
crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2_type_flat flat_networks provider
crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2_type_vxlan vni_ranges 1:1000        
crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini securitygroup enable_ipset True
```

Sửa file `/etc/neutron/plugins/ml2/linuxbridge_agent.ini`
```
crudini --set  /etc/neutron/plugins/ml2/linuxbridge_agent.ini linux_bridge physical_interface_mappings provider:eth1

crudini --set  /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan enable_vxlan True

crudini --set  /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan local_ip $(ip addr show dev eth2 scope global | grep "inet " | sed -e 's#.*inet ##g' -e 's#/.*##g')

crudini --set  /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup enable_security_group True

crudini --set  /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup firewall_driver neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```

Khai báo `sysctl`
```
echo 'net.bridge.bridge-nf-call-iptables = 1' >> /etc/sysctl.conf
echo 'net.bridge.bridge-nf-call-ip6tables = 1' >> /etc/sysctl.conf
modprobe br_netfilter
/sbin/sysctl -p
```

Tạo liên kết
```
ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
```

Thiết lập database
```
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
--config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
```

Khởi động và kích hoạt dịch vụ neutron
```
systemctl enable neutron-server.service \
neutron-linuxbridge-agent.service neutron-dhcp-agent.service \
neutron-metadata-agent.service
  
systemctl start neutron-server.service \
neutron-linuxbridge-agent.service neutron-dhcp-agent.service \
neutron-metadata-agent.service
```

Kiểm tra
```
openstack network agent list
```

#### 2.11.2. Cài đặt Neutron trên node compute
> Thực hiện cài đặt trên node `compute1`

Khai báo bổ sung cho nova
```
crudini --set /etc/nova/nova.conf neutron url http://10.10.31.166:9696
crudini --set /etc/nova/nova.conf neutron auth_url http://10.10.31.166:5000
crudini --set /etc/nova/nova.conf neutron auth_type password
crudini --set /etc/nova/nova.conf neutron project_domain_name Default
crudini --set /etc/nova/nova.conf neutron user_domain_name Default
crudini --set /etc/nova/nova.conf neutron project_name service
crudini --set /etc/nova/nova.conf neutron username neutron
crudini --set /etc/nova/nova.conf neutron password Welcome123
```

Cài đặt neutron
```
yum install -y openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables ipset
```

Sao lưu file cấu hình của neutron
```
cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.bak
cp /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugins/ml2/ml2_conf.ini.bak
cp /etc/neutron/plugins/ml2/linuxbridge_agent.ini /etc/neutron/plugins/ml2/linuxbridge_agent.ini.bak
cp /etc/neutron/dhcp_agent.ini /etc/neutron/dhcp_agent.ini.bak
cp /etc/neutron/metadata_agent.ini /etc/neutron/metadata_agent.ini.bak
```

Sửa file cấu hình của neutron `/etc/neutron/neutron.conf`
```
crudini --set /etc/neutron/neutron.conf DEFAULT auth_strategy keystone
crudini --set /etc/neutron/neutron.conf DEFAULT core_plugin ml2
crudini --set /etc/neutron/neutron.conf DEFAULT transport_url rabbit://openstack:Welcome123@10.10.31.166
crudini --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_status_changes true
crudini --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_data_changes true

crudini --set /etc/neutron/neutron.conf keystone_authtoken www_authenticate_uri http://10.10.31.166:5000
crudini --set /etc/neutron/neutron.conf keystone_authtoken auth_url http://10.10.31.166:5000
crudini --set /etc/neutron/neutron.conf keystone_authtoken memcached_servers 10.10.31.166:11211
crudini --set /etc/neutron/neutron.conf keystone_authtoken auth_type password
crudini --set /etc/neutron/neutron.conf keystone_authtoken project_domain_name Default
crudini --set /etc/neutron/neutron.conf keystone_authtoken user_domain_name Default
crudini --set /etc/neutron/neutron.conf keystone_authtoken project_name service
crudini --set /etc/neutron/neutron.conf keystone_authtoken username neutron
crudini --set /etc/neutron/neutron.conf keystone_authtoken password Welcome123

crudini --set /etc/neutron/neutron.conf oslo_concurrency lock_path /var/lib/neutron/tmp
```

Khai báo `sysctl`
```
echo 'net.bridge.bridge-nf-call-iptables = 1' >> /etc/sysctl.conf
echo 'net.bridge.bridge-nf-call-ip6tables = 1' >> /etc/sysctl.conf
modprobe br_netfilter
/sbin/sysctl -p
```

Sửa file `/etc/neutron/plugins/ml2/linuxbridge_agent.ini`
```
crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini linux_bridge physical_interface_mappings provider:eth1
crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan enable_vxlan True
crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan local_ip $(ip addr show dev eth2 scope global | grep "inet " | sed -e 's#.*inet ##g' -e 's#/.*##g')
crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup enable_security_group True
crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup firewall_driver neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```

Khai báo cho file /etc/neutron/metadata_agent.ini
```
crudini --set /etc/neutron/metadata_agent.ini DEFAULT nova_metadata_host 10.10.31.166
crudini --set /etc/neutron/metadata_agent.ini DEFAULT metadata_proxy_shared_secret Welcome123
```

Khai báo cho file /etc/neutron/dhcp_agent.ini
```
crudini --set /etc/neutron/dhcp_agent.ini DEFAULT interface_driver neutron.agent.linux.interface.BridgeInterfaceDriver
crudini --set /etc/neutron/dhcp_agent.ini DEFAULT enable_isolated_metadata True
crudini --set /etc/neutron/dhcp_agent.ini DEFAULT dhcp_driver neutron.agent.linux.dhcp.Dnsmasq
crudini --set /etc/neutron/dhcp_agent.ini DEFAULT force_metadata True
```

Kích hoạt neutron
```
systemctl enable neutron-linuxbridge-agent.service
systemctl enable neutron-metadata-agent.service
systemctl enable neutron-dhcp-agent.service
```

Khởi động neutron
```
systemctl start neutron-linuxbridge-agent.service
systemctl start neutron-metadata-agent.service
systemctl start neutron-dhcp-agent.service
systemctl restart openstack-nova-compute.service
```
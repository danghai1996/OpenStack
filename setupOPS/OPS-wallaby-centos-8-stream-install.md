# Triển khai OpenStack bản Wallaby trên CentOS Stream 8

# Các bước cài đặt:
Cấu hình enviroment trên cả 2 node
- IP, hostname, Disable firewall, SELinux
- Cài đặt NTP

Cài đặt MariaDB trên node Controller

Cài đặt RabbitMQ lên Controller

Cài đặt Memcache -> CTL

Cài đặt ETCD -> CTL

Cài đặt các project


# Mô hình và IPPlanning
### Mô hình:

<img src="..\images\ops-wallaby\Screenshot_1.png">

### IP Planning

<img src="..\images\ops-wallaby\Screenshot_2.png">

# I. Cài đặt môi trường
## 1. Cài đặt cơ bản
### 1.1. Trên node Controller
> Thực hiện trên node Contrller
Thiết lập Hostname
```
hostnamectl set-hostname controller1
bash
```

Thiết lập IP:
```
nmcli con modify ens3 ipv4.addresses 10.10.30.51/24
nmcli con modify ens3 ipv4.gateway 10.10.30.1
nmcli con modify ens3 ipv4.dns 8.8.8.8
nmcli con modify ens3 ipv4.method manual
nmcli con modify ens3 connection.autoconnect yes

nmcli con modify ens4 ipv4.addresses 10.10.31.51/24
nmcli con modify ens4 ipv4.method manual
nmcli con modify ens4 connection.autoconnect yes


nmcli con modify ens5 ipv4.addresses 10.10.32.51/24
nmcli con modify ens5 ipv4.method manual
nmcli con modify ens5 connection.autoconnect yes
```

Khai báo file `/etc/hosts`
```
echo "10.10.30.51 controller1" >> /etc/hosts
echo "10.10.30.52 compute01" >> /etc/hosts
```

Disable Firewall and SELinux:
```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
systemctl disable firewalld
systemctl stop firewalld
```

Khởi động lại:
```
init 6
```

Update và cài đặt 1 số gói:
```
dnf update -y

dnf install wget vim -y
```

> ## Snapshot node CTL `Set-IP`

### 1.2. Trên node Compute01
> Thực hiện trên node compute01

Thiết lập Hostname
```
hostnamectl set-hostname compute01
bash
```

Thiết lập IP:
```
nmcli con modify ens3 ipv4.addresses 10.10.30.52/24
nmcli con modify ens3 ipv4.gateway 10.10.30.1
nmcli con modify ens3 ipv4.dns 8.8.8.8
nmcli con modify ens3 ipv4.method manual
nmcli con modify ens3 connection.autoconnect yes

nmcli con modify ens4 ipv4.addresses 10.10.31.52/24
nmcli con modify ens4 ipv4.method manual
nmcli con modify ens4 connection.autoconnect yes


nmcli con modify ens5 ipv4.addresses 10.10.32.52/24
nmcli con modify ens5 ipv4.method manual
nmcli con modify ens5 connection.autoconnect yes
```

Khai báo file `/etc/hosts`
```
echo "10.10.30.51 controller1" >> /etc/hosts
echo "10.10.30.52 compute01" >> /etc/hosts
```

Disable Firewall and SELinux:
```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
systemctl disable firewalld
systemctl stop firewalld
```

Update và cài đặt 1 số gói:
```
dnf update -y

dnf install wget vim -y
```

### Thực hiện kiểm tra trên các node:
- Ping ra internet bằng IP và domain name.
- Ping ra gateway của các interface
- Ping tới các IP của các node trong topo.

> ## Snapshot node COM01 `Set-IP`

## 2. Thêm repo OpenStack Wallaby và cài đặt một số gói
> Thực hiện trên tất cả các node

Thêm repo OpenStack Wallaby và upgrade OS để cập nhật 1 số gói Python3 từ Openstack Wallaby repository
```
dnf -y install centos-release-openstack-wallaby

sed -i -e "s/enabled=1/enabled=0/g" /etc/yum.repos.d/CentOS-OpenStack-wallaby.repo

dnf --enablerepo=centos-openstack-wallaby -y upgrade
```



## 3. Cài đặt NTP
### 3.1. Cài đặt NTP server trên node Controller
> Thực hiện trên node Controller1

Cài đặt Chrony:
```
dnf -y install chrony
```

Sao lưu file cấu hình của NTP:
```
cp /etc/chrony.conf /etc/chrony.conf.bak
```

Node `controller` sẽ cập nhật thời gian từ internet hoặc node chủ NTP. Các node compute còn lại sẽ đồng bộ thời gian từ `controller`. Trong bài lab này sẽ sử dụng địa chỉ NTP của nội bộ.

Set timezone:
```
timedatectl set-timezone Asia/Ho_Chi_Minh
```

Sửa file cấu hình:
```
sed -i s'/pool 2.centos.pool.ntp.org iburst/server 10.10.30.100 iburst/'g /etc/chrony.conf
```

Để cho phép các node compute kết nối với chrony trên node controller, ta sẽ sửa file cấu hình cho subnet chung của các node:
```
sed -i s'|#allow 192.168.0.0/16|allow 10.10.30.0/24|'g /etc/chrony.conf
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
[root@controller1 ~]# chronyc sources
210 Number of sources = 1
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* 10.10.30.100                  4   6    17    21  -2493ns[ -154us] +/-   14ms
```

### 3.2. Cài đặt NTP trên node Compute
> Thực hiện trên node Compute

Cài đặt Chrony:
```
dnf -y install chrony
```

Sao lưu file cấu hình của NTP:
```
cp /etc/chrony.conf /etc/chrony.conf.bak
```

Set timezone:
```
timedatectl set-timezone Asia/Ho_Chi_Minh
```

Sửa file cấu hình như sau:
```
sed -i s'/pool 2.centos.pool.ntp.org iburst/server 10.10.30.51 iburst/'g /etc/chrony.conf
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
[root@compute01 ~]# chronyc sources
210 Number of sources = 1
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* controller1                   5   6    17     0   +264ns[  -26us] +/-   14ms
```

Kiểm tra lại thời gian sau khi đồng bộ:
```
timedatectl
```
Kết quả:
```
               Local time: Wed 2021-04-28 22:33:48 +07
           Universal time: Wed 2021-04-28 15:33:48 UTC
                 RTC time: Wed 2021-04-28 15:33:47
                Time zone: Asia/Ho_Chi_Minh (+07, +0700)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

## 4. Cài đặt và cấu hình MariaDB trên node Controller
> Thực hiện trên node Controller

Cài đặt MariaDB 10.3
```
dnf module -y install mariadb:10.3
```

Tạo và chỉnh sửa file cấu hình của Openstack: /etc/my.cnf.d/openstack.cnf

Ta sẽ tạo module `[mysqld]` và set `bind-address` thành IP của node `controller` để các node khác truy cập qua mạng quản lý (VLAN MGNT)
```
cat <<EOF> /etc/my.cnf.d/openstack.cnf
[mysqld]
bind-address = 10.10.30.51

default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
EOF
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

Sau khi xác thực bảo mật xong, ta truy cập db với user root với pass `Welcome123` thực hiện các lệnh sau:
```
mysql -uroot -pWelcome123

GRANT ALL PRIVILEGES ON *.* TO 'root'@'10.10.30.51' IDENTIFIED BY 'Welcome123' WITH GRANT OPTION;

FLUSH PRIVILEGES;

DROP USER 'root'@'::1';

exit
```
# Hướng dẫn cách backups các gói cài đặt OPS Queens trên CentOS-7

## Chuẩn bị:
2 máy cài đặt CentOS-7 giống nhau: Trong bài viết này là bản: **7.6.1810**
- Có kết nối Internet

# I. Trên node cài đặt Controller
## 1. Tạo thư mục lưu các gói cài đặt:
```
mkdir /root/backup-ctl
cd /root/backup-ctl
mkdir 01-ops-ctr  02-extra  03-python-ops  04-chrony  05-memcached  06-mysql  07-rabbitmq-server  08-keystone  09-glance  10-nova  11-neutron  12-horizon  13-cinder
cd
```

## 2. Backup các gói
### 2.1. Các gói cài đặt OPS Queens:
Backups các gói
```
yum install --downloadonly --downloaddir=/root/backup-ctl/01-ops-ctr/ centos-release-openstack-queens
```

Thực hiện cài đặt các gói vừa tải:
```
yum localinstall -y /root/backup-ctl/01-ops-ctr/*.rpm
```

### 2.2. Các gói phần mềm hỗ trợ
Backups:
```
yum install --downloadonly --downloaddir=/root/backup-ctl/02-extra/ crudini wget vim
```

Cài đặt:
```
yum localinstall -y /root/backup-ctl/02-extra/*.rpm
```

### 2.3. Các gói Python
Backups:
```
yum install --downloadonly --downloaddir=/root/backup-ctl/03-python-ops/ python-openstackclient openstack-selinux python2-PyMySQL
```

Cài đặt
```
yum localinstall -y /root/backup-ctl/03-python-ops/*.rpm
```

### 2.4. Chrony
Backups
```
yum install --downloadonly --downloaddir=/root/backup-ctl/04-chrony/ chrony
```

Cài đặt
```
yum localinstall -y /root/backup-ctl/04-chrony/*.rpm
```

### 2.5. Memcached
Backup
```
yum install --downloadonly --downloaddir=/root/backup-ctl/05-memcached/ memcached
```

Cài đặt:
```
yum localinstall -y /root/backup-ctl/05-memcached/*.rpm
```

### 2.6. Mysql
Cài đặt repo:
```
echo '[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.2/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1' >> /etc/yum.repos.d/MariaDB.repo
```

Backup
```
yum install --downloadonly --downloaddir=/root/backup-ctl/06-mysql/ mariadb mariadb-server python2-PyMySQL
```

Cài đặt:
```
yum localinstall -y /root/backup-ctl/06-mysql/*.rpm
```

### 2.7. RabbirMQ
Backup
```
yum install --downloadonly --downloaddir=/root/backup-ctl/07-rabbitmq-server/ rabbitmq-server
```

Cài đặt:
```
yum localinstall -y /root/backup-ctl/07-rabbitmq-server/*.rpm
```

### 2.8. Keystone
Backup
```
yum install --downloadonly --downloaddir=/root/backup-ctl/08-keystone/ openstack-keystone httpd mod_wsgi
```

Cài đặt:
```
yum localinstall -y /root/backup-ctl/08-keystone/*.rpm
```

### 2.9. Glance
Backup
```
yum install --downloadonly --downloaddir=/root/backup-ctl/09-glance/ openstack-glance
```

Cài đặt:
```
yum localinstall -y /root/backup-ctl/09-glance/*.rpm
```

### 2.10. Nova
Backup
```
yum install --downloadonly --downloaddir=/root/backup-ctl/10-nova/ openstack-nova-api openstack-nova-conductor openstack-nova-console openstack-nova-novncproxy openstack-nova-scheduler openstack-nova-placement-api
```

Cài đặt:
```
yum localinstall -y /root/backup-ctl/10-nova/*.rpm
```

### 2.10. Neutron
Backup
```
yum install --downloadonly --downloaddir=/root/backup-ctl/11-neutron/ openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables
```

Cài đặt:
```
yum localinstall -y /root/backup-ctl/11-neutron/*.rpm
```

### 2.11. Horizon
Backup
```
yum install --downloadonly --downloaddir=/root/backup-ctl/12-horizon/ openstack-dashboard
```

Cài đặt:
```
yum localinstall -y /root/backup-ctl/12-horizon/*.rpm
```

### 2.12. Cinder
Backup
```
yum install --downloadonly --downloaddir=/root/backup-ctl/13-cinder/ openstack-cinder targetcli python-keystone lvm2
```

Cài đặt:
```
yum localinstall -y /root/backup-ctl/13-cinder/*.rpm
```

> ### Vậy là ta đã backups xong các gói cài đặt trên node Controller.

# II. Trên node cài đặt Compute
Chỉnh sửa `/etc/yum.repos.d/CentOS-QEMU-EV.repo`
```
sed -i 's|baseurl=http:\/\/mirror.centos.org\/$contentdir\/$releasever\/virt\/$basearch\/kvm-common\/|baseurl=http:\/\/mirror.centos.org\/centos\/7\/virt\/x86_64\/kvm-common\/|g' /etc/yum.repos.d/CentOS-QEMU-EV.repo
```

## 1. Tạo thư mục lưu các gói cài đặt:
```
mkdir /root/backup-com
cd /root/backup-com
mkdir 01-ops-compute  02-extra  03-python-ops  04-chrony  05-nova  06-neutron
cd
```

## 2. Backup các gói
### 2.1. Các gói cài đặt OPS Queens:
Backup
```
yum install --downloadonly --downloaddir=/root/backup-com/01-ops-compute/ centos-release-openstack-queens
```

Cài đặt
```
yum localinstall -y /root/backup-com/01-ops-compute/*.rpm
```

### 2.2. Gói phần mềm mở rộng
```
yum install --downloadonly --downloaddir=/root/backup-com/02-extra/ crudini wget vim
```

Cài đặt
```
yum localinstall -y /root/backup-com/02-extra/*.rpm
```

### 2.3. Các gói Python
```
yum install --downloadonly --downloaddir=/root/backup-com/03-python-ops/ python-openstackclient openstack-selinux python2-PyMySQL
```

Cài đặt
```
yum localinstall -y /root/backup-com/03-python-ops/*.rpm
```

### 2.4. Chronyd
```
yum install --downloadonly --downloaddir=/root/backup-com/04-chrony/ chrony
``` 

Cài đặt
```
yum localinstall -y /root/backup-com/04-chrony/*.rpm
```

### 2.5. Nova
```
yum install --downloadonly --downloaddir=/root/backup-com/05-nova/ openstack-nova-compute libvirt-client
```

Cài đặt
```
yum localinstall -y /root/backup-com/05-nova/*.rpm
```

### 2.6. Neutron
```
yum install --downloadonly --downloaddir=/root/backup-com/06-neutron/ openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables
```

Cài đặt
```
yum localinstall -y /root/backup-com/06-neutron/*.rpm
```

> ### Vậy là ta đã backups xong các gói cài đặt trên node Compute.

# Lưu trữ các gói cài đặt
Trên 2 node, ta đã có 2 thư mục chứa các gói cài đặt cần thiết để cài đặt OPS:
- **Controller**: `/root/backup-ctl/`
- **Compute**: `/root/backup-com/`

Upload 2 thư mục lên 1 FTP server hoặc một nơi lưu trữ để có thể sử dụng khi cài đặt các node của OPS-Queens.
# Các bước cài đặt Openstack Queens với Local Repo

## Mục tiêu:
Cài đặt được OpenStack mà không cần sử dụng đến Internet


# Cài đặt
## 1. Backups gói cài đặt
Cài đặt theo docs: [Cài đặt OPS Queens trên CentOS-7](./OPS-queen-centos7-install.md)

Đến các bước cài đặt gói, ta tiến hành tạo thư mục để backups các gói và tải các gói vào thư mục đã tạo.

### Trên node Controller
Tạo các thư mục theo từng phần cài đặt:
```
mkdir /root/backup-ctl
cd /root/backup-ctl
mkdir 01-ops-ctr  02-extra  03-python-ops  04-chrony  05-memcached  06-mysql  07-rabbitmq-server  08-keystone  09-glance  10-nova  11-neutron  12-horizon  13-cinder
```

Khi đến các bước cài đặt, ta tiến hành sử dụng các lệnh tương ứng dưới đây để lưu gói cài:

1. Cài đặt OPS Queens
```
yum install --downloadonly --downloaddir=/root/backup-ctl/01-ops-ctr/ centos-release-openstack-queens
```

2. Các gói phần mềm hỗ trợ
```
yum install --downloadonly --downloaddir=/root/backup-ctl/02-extra/ crudini wget vim
```

3. Các gói Python
```
yum install --downloadonly --downloaddir=/root/backup-ctl/03-python-ops/ python-openstackclient openstack-selinux python2-PyMySQL
```

4. Gói cài đặt chronyd
```
yum install --downloadonly --downloaddir=/root/backup-ctl/04-chrony/ chrony
```

5. Memcached
```
yum install --downloadonly --downloaddir=/root/backup-ctl/05-memcached/ memcached
```

6. Mysql
```
yum install --downloadonly --downloaddir=/root/backup-ctl/06-mysql/ mariadb mariadb-server python2-PyMySQL
```

7. RabbirMQ
```
yum install --downloadonly --downloaddir=/root/backup-ctl/07-rabbitmq-server/ rabbitmq-server
```

8. Keystone
```
yum install --downloadonly --downloaddir=/root/backup-ctl/08-keystone/ openstack-keystone httpd mod_wsgi
```

9. Glance
```
yum install --downloadonly --downloaddir=/root/backup-ctl/09-glance/ openstack-glance
```

10. Nova
```
yum install --downloadonly --downloaddir=/root/backup-ctl/10-nova/ openstack-nova-api openstack-nova-conductor openstack-nova-console openstack-nova-novncproxy openstack-nova-scheduler openstack-nova-placement-api
```

11. Neutron
```
yum install --downloadonly --downloaddir=/root/backup-ctl/11-neutron/ openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables
```

12. Horizon
```
yum install --downloadonly --downloaddir=/root/backup-ctl/12-horizon/ openstack-dashboard
```

13. Cinder
```
yum install --downloadonly --downloaddir=/root/backup-ctl/13-cinder/ openstack-cinder targetcli python-keystone lvm2
```

### Trên node Compute:
Tạo các thư mục theo từng phần cài đặt:
```
mkdir /root/backup-com
cd /root/backup-com
mkdir 01-ops-compute  02-extra  03-python-ops  04-chrony  05-nova  06-neutron
```

Khi đến các bước cài đặt, ta tiến hành sử dụng các lệnh tương ứng dưới đây để lưu gói cài:
1. OPS Queens
```
yum install --downloadonly --downloaddir=/root/backup-com/01-ops-compute/ centos-release-openstack-queens
```

2. Gói phần mềm mở rộng
```
yum install --downloadonly --downloaddir=/root/backup-com/02-extra/ crudini wget vim
```

3. Các gói Python
```
yum install --downloadonly --downloaddir=/root/backup-com/03-python-ops/ python-openstackclient openstack-selinux python2-PyMySQL
```

4. Chronyd
```
yum install --downloadonly --downloaddir=/root/backup-com/04-chrony/ chrony 
```

5. Nova
```
yum install --downloadonly --downloaddir=/root/backup-com/05-nova/ openstack-nova-compute libvirt-client
```

6. Neutron
```
yum install --downloadonly --downloaddir=/root/backup-com/06-neutron/ openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables
```

## 2. Sử dụng các gói đã backups để cài đặt offline
Copy thư mục backup sang các node cần cài đặt

Cài đặt theo docs. [Cài đặt OPS Queens trên CentOS-7](./OPS-queen-centos7-install.md)

Đến các bước cài đặt gói, ta sử dụng lệnh lệnh tương ứng với các thư mục backups:
```
yum localinstall -y /root/backup/<thư_mục_backups>/* 
```

Ví dụ: Cài đặt gói `centos-release-openstack-queens` trên CTL
```
yum localinstall -y /root/backup-ctl/01-ops-ctr/*
```
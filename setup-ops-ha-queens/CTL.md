# Cấu hình các node Controller

# Phần 1: Thiết lập ban đầu
> ## Trên node CTL1
Set hostname :
```
hostnamectl set-hostname ctl01
```

Cấu hình network:
```
nmcli con mod eth0 ipv4.address 10.10.34.161/24
nmcli con mod eth0 ipv4.gateway 10.10.34.1
nmcli con mod eth0 ipv4.dns 8.8.8.8
nmcli con mod eth0 ipv4.method manual
nmcli con mod eth0 connection.autoconnect yes

nmcli con mod eth1 ipv4.address 10.10.31.161/24
nmcli con mod eth1 ipv4.method manual
nmcli con mod eth1 connection.autoconnect yes

nmcli con mod eth2 ipv4.address 10.10.32.161/24
nmcli con mod eth2 ipv4.method manual
nmcli con mod eth2 connection.autoconnect yes

nmcli con mod eth3 ipv4.address 10.10.33.161/24
nmcli con mod eth3 ipv4.method manual
nmcli con mod eth3 connection.autoconnect yes
```

Selinux, Firewalld
```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
systemctl stop firewalld
systemctl disable firewalld
```

CMD log
```
curl -Lso- https://raw.githubusercontent.com/nhanhoadocs/scripts/master/Utilities/cmdlog.sh | bash
```


Cấu hình chronyd
```
Lấy time từ IP: 10.10.34.130
```

Reboot 
```
init 6
```

Chuẩn bị sysctl
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

Cấu hình hostname
```
echo "10.10.34.161 ctl01" >> /etc/hosts
echo "10.10.34.162 ctl02" >> /etc/hosts
echo "10.10.34.163 ctl03" >> /etc/hosts
echo "10.10.34.164 com01" >> /etc/hosts
echo "10.10.34.165 com02" >> /etc/hosts
```

Setup keypair
```
ssh-keygen -t rsa -f /root/.ssh/id_rsa -q -P ""
ssh-copy-id -o StrictHostKeyChecking=no -i /root/.ssh/id_rsa.pub root@ctl02
ssh-copy-id -o StrictHostKeyChecking=no -i /root/.ssh/id_rsa.pub root@ctl03
```

> ## Trên node CTL2
Set hostname :
```
hostnamectl set-hostname ctl02
bash
```

Cấu hình network:
```
nmcli con mod eth0 ipv4.address 10.10.34.162/24
nmcli con mod eth0 ipv4.gateway 10.10.34.1
nmcli con mod eth0 ipv4.dns 8.8.8.8
nmcli con mod eth0 ipv4.method manual
nmcli con mod eth0 connection.autoconnect yes

nmcli con mod eth1 ipv4.address 10.10.31.162/24
nmcli con mod eth1 ipv4.method manual
nmcli con mod eth1 connection.autoconnect yes

nmcli con mod eth2 ipv4.address 10.10.32.162/24
nmcli con mod eth2 ipv4.method manual
nmcli con mod eth2 connection.autoconnect yes

nmcli con mod eth3 ipv4.address 10.10.33.162/24
nmcli con mod eth3 ipv4.method manual
nmcli con mod eth3 connection.autoconnect yes

systemctl restart NetworkManager
```

Selinux, Firewalld
```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
systemctl stop firewalld
systemctl disable firewalld
```

CMD log
```
curl -Lso- https://raw.githubusercontent.com/nhanhoadocs/scripts/master/Utilities/cmdlog.sh | bash
```


Cấu hình chronyd
```
Lấy time từ IP: 10.10.34.130
```

Reboot 
```
init 6
```

Chuẩn bị sysctl
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

Cấu hình hostname
```
echo "10.10.34.161 ctl01" >> /etc/hosts
echo "10.10.34.162 ctl02" >> /etc/hosts
echo "10.10.34.163 ctl03" >> /etc/hosts
echo "10.10.34.164 com01" >> /etc/hosts
echo "10.10.34.165 com02" >> /etc/hosts
```

Setup keypair
```
ssh-keygen -t rsa -f /root/.ssh/id_rsa -q -P ""
ssh-copy-id -o StrictHostKeyChecking=no -i /root/.ssh/id_rsa.pub root@ctl01
ssh-copy-id -o StrictHostKeyChecking=no -i /root/.ssh/id_rsa.pub root@ctl03
```

> ## Trên node CTL3
Set hostname :
```
hostnamectl set-hostname ctl03
bash
```

Cấu hình network:
```
nmcli con mod eth0 ipv4.address 10.10.34.163/24
nmcli con mod eth0 ipv4.gateway 10.10.34.1
nmcli con mod eth0 ipv4.dns 8.8.8.8
nmcli con mod eth0 ipv4.method manual
nmcli con mod eth0 connection.autoconnect yes

nmcli con mod eth1 ipv4.address 10.10.31.163/24
nmcli con mod eth1 ipv4.method manual
nmcli con mod eth1 connection.autoconnect yes

nmcli con mod eth2 ipv4.address 10.10.32.163/24
nmcli con mod eth2 ipv4.method manual
nmcli con mod eth2 connection.autoconnect yes

nmcli con mod eth3 ipv4.address 10.10.33.163/24
nmcli con mod eth3 ipv4.method manual
nmcli con mod eth3 connection.autoconnect yes

systemctl restart NetworkManager
```

Selinux, Firewalld
```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
systemctl stop firewalld
systemctl disable firewalld
```

CMD log
```
curl -Lso- https://raw.githubusercontent.com/nhanhoadocs/scripts/master/Utilities/cmdlog.sh | bash
```


Cấu hình chronyd
```
Lấy time từ IP: 10.10.34.130
```

Reboot 
```
init 6
```

Chuẩn bị sysctl
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

Cấu hình hostname
```
echo "10.10.34.161 ctl01" >> /etc/hosts
echo "10.10.34.162 ctl02" >> /etc/hosts
echo "10.10.34.163 ctl03" >> /etc/hosts
echo "10.10.34.164 com01" >> /etc/hosts
echo "10.10.34.165 com02" >> /etc/hosts
```

Setup keypair
```
ssh-keygen -t rsa -f /root/.ssh/id_rsa -q -P ""
ssh-copy-id -o StrictHostKeyChecking=no -i /root/.ssh/id_rsa.pub root@ctl01
ssh-copy-id -o StrictHostKeyChecking=no -i /root/.ssh/id_rsa.pub root@ctl02
```

> ## Tắt, snapshot `env` trên cả 3 node

# Phần 2: Setup Galera Cluster
## Setup repo và cài đặt MariaDB
> ## Trên tất cả các node CTL
```
echo '[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.2/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1' >> /etc/yum.repos.d/MariaDB.repo

# Update khi cài 1 cụm mới với OS mới. Còn không thì nên dùng OS cùng phiên bản với các node đã triển khai
yum -y update

yum install -y mariadb mariadb-server
systemctl stop mariadb
```

## Cấu hình
> ## Node CTL1
Cấu hình:
```
cp /etc/my.cnf.d/server.cnf /etc/my.cnf.d/server.cnf.bak

echo '[server]
[mysqld]
bind-address=10.10.34.161

[galera]
wsrep_on=ON
wsrep_provider=/usr/lib64/galera/libgalera_smm.so
wsrep_cluster_address="gcomm://10.10.34.161,10.10.34.162,10.10.34.163"
binlog_format=row
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
wsrep_cluster_name="galera_cluster"
bind-address=10.10.34.161
wsrep_node_address="10.10.34.161"
wsrep_node_name="ctl01"
wsrep_sst_method=rsync

skip-name-resolve
max_connections = 10240
innodb_locks_unsafe_for_binlog=1
query_cache_size=0
query_cache_type=0
innodb_log_file_size=100M
innodb_file_per_table
innodb_flush_log_at_trx_commit=2
[embedded]
[mariadb]
[mariadb-10.2]
' > /etc/my.cnf.d/server.cnf
```

> ## Node CTL2
Cấu hình
```
cp /etc/my.cnf.d/server.cnf /etc/my.cnf.d/server.cnf.bak

echo '[server]
[mysqld]
bind-address=10.10.34.162

[galera]
wsrep_on=ON
wsrep_provider=/usr/lib64/galera/libgalera_smm.so
wsrep_cluster_address="gcomm://10.10.34.161,10.10.34.162,10.10.34.163"
binlog_format=row
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
wsrep_cluster_name="galera_cluster"
bind-address=10.10.34.162
wsrep_node_address="10.10.34.162"
wsrep_node_name="ctl02"
wsrep_sst_method=rsync

skip-name-resolve
max_connections = 10240
innodb_locks_unsafe_for_binlog=1
query_cache_size=0
query_cache_type=0
innodb_log_file_size=100M
innodb_file_per_table
innodb_flush_log_at_trx_commit=2
[embedded]
[mariadb]
[mariadb-10.2]
' > /etc/my.cnf.d/server.cnf
```

> ## Node CTL3
Cấu hình
```
cp /etc/my.cnf.d/server.cnf /etc/my.cnf.d/server.cnf.bak

echo '[server]
[mysqld]
bind-address=10.10.34.163

[galera]
wsrep_on=ON
wsrep_provider=/usr/lib64/galera/libgalera_smm.so
wsrep_cluster_address="gcomm://10.10.34.161,10.10.34.162,10.10.34.163"
binlog_format=row
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
wsrep_cluster_name="galera_cluster"
bind-address=10.10.34.163
wsrep_node_address="10.10.34.163"
wsrep_node_name="ctl03"
wsrep_sst_method=rsync

skip-name-resolve
max_connections = 10240
innodb_locks_unsafe_for_binlog=1
query_cache_size=0
query_cache_type=0
innodb_log_file_size=100M
innodb_file_per_table
innodb_flush_log_at_trx_commit=2
[embedded]
[mariadb]
[mariadb-10.2]
' > /etc/my.cnf.d/server.cnf
```

## Khởi tạo Cluster
> ## Trên node CTL1
Khởi tạo Cluster
```
galera_new_cluster
systemctl start mariadb
systemctl enable mariadb
```

> ## Trên node CTL2 và CTL3
Start service Mariadb
```
systemctl start mariadb
systemctl enable mariadb
```

## Kiểm tra:
> ## Trên các node
```
mysql -u root -e "SHOW STATUS LIKE 'wsrep_cluster_size'"
```

Kết quả
```
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 3     |
+--------------------+-------+
```

## Đặt mật khẩu và phân quyền cho MariaDB
> ## Trên node CTL1
```
password_galera_root=Welcome123
cat << EOF | mysql -uroot
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '$password_galera_root';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' IDENTIFIED BY '$password_galera_root';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON *.* TO 'root'@'10.10.34.161' IDENTIFIED BY '$password_galera_root';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON *.* TO 'root'@'10.10.34.162' IDENTIFIED BY '$password_galera_root';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON *.* TO 'root'@'10.10.34.163' IDENTIFIED BY '$password_galera_root';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON *.* TO 'root'@'127.0.0.1' IDENTIFIED BY '$password_galera_root';FLUSH PRIVILEGES;

GRANT ALL PRIVILEGES ON *.* TO 'root'@'ctl01' IDENTIFIED BY '$password_galera_root';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON *.* TO 'root'@'ctl02' IDENTIFIED BY '$password_galera_root';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON *.* TO 'root'@'ctl03' IDENTIFIED BY '$password_galera_root';FLUSH PRIVILEGES;
EOF
```

## Cấu hình HAProxy check Mysql
> ## Trên node CTL1
Cài đặt và cấu hình plugin check Mysql
```
yum install rsync xinetd crudini git -y
git clone https://github.com/thaonguyenvan/percona-clustercheck
cp percona-clustercheck/clustercheck /usr/local/bin

cat << EOF >> /etc/xinetd.d/mysqlchk
service mysqlchk
{
      disable = no
      flags = REUSE
      socket_type = stream
      port = 9200
      wait = no
      user = nobody
      server = /usr/local/bin/clustercheck
      log_on_failure += USERID
      only_from = 0.0.0.0/0
      per_source = UNLIMITED
}
EOF
```

Tạo service
```
echo 'mysqlchk 9200/tcp # MySQL check' >> /etc/services
```

Tạo tk check mysql
```
mysql -uroot -pWelcome123
GRANT PROCESS ON *.* TO 'clustercheckuser'@'localhost' IDENTIFIED BY 'clustercheckpassword!';
FLUSH PRIVILEGES;
EXIT
```

Bật xinetd
```
systemctl start xinetd
systemctl enable xinetd
```

Kiểm tra:
```
[root@ctl01 ~]# clustercheck
HTTP/1.1 200 OK
Content-Type: text/plain
Connection: close
Content-Length: 40

Percona XtraDB Cluster Node is synced.
```

> ## Trên node CTL2
Cài đặt và cấu hình plugin check Mysql
```
yum install rsync xinetd crudini git -y
git clone https://github.com/thaonguyenvan/percona-clustercheck
cp percona-clustercheck/clustercheck /usr/local/bin


cat << EOF >> /etc/xinetd.d/mysqlchk
service mysqlchk
{
      disable = no
      flags = REUSE
      socket_type = stream
      port = 9200
      wait = no
      user = nobody
      server = /usr/local/bin/clustercheck
      log_on_failure += USERID
      only_from = 0.0.0.0/0
      per_source = UNLIMITED
}
EOF
```

Tạo service
```
echo 'mysqlchk 9200/tcp # MySQL check' >> /etc/services
```

Bật xinetd
```
systemctl start xinetd
systemctl enable xinetd
```

Kiểm tra:
```
[root@ctl02 ~]# clustercheck
HTTP/1.1 200 OK
Content-Type: text/plain
Connection: close
Content-Length: 40

Percona XtraDB Cluster Node is synced.
```

> ## Trên node CTL3
Cài đặt và cấu hình plugin check Mysql
```
yum install rsync xinetd crudini git -y
git clone https://github.com/thaonguyenvan/percona-clustercheck
cp percona-clustercheck/clustercheck /usr/local/bin


cat << EOF >> /etc/xinetd.d/mysqlchk
service mysqlchk
{
      disable = no
      flags = REUSE
      socket_type = stream
      port = 9200
      wait = no
      user = nobody
      server = /usr/local/bin/clustercheck
      log_on_failure += USERID
      only_from = 0.0.0.0/0
      per_source = UNLIMITED
}
EOF
```

Tạo service
```
echo 'mysqlchk 9200/tcp # MySQL check' >> /etc/services
```

Bật xinetd
```
systemctl start xinetd
systemctl enable xinetd
```

Kiểm tra:
```
[root@ctl03 ~]# clustercheck
HTTP/1.1 200 OK
Content-Type: text/plain
Connection: close
Content-Length: 40

Percona XtraDB Cluster Node is synced.
```

> ## Snapshot `galera-cluster` trên cả 3 node CTL

# Phần 3: Cài đặt RabbitMQ Cluster
## Cài đặt môi trường và RabbitMQ Cluster
> ## Thực hiện trên cả 3 node CTL1, CTL2, CTL3
```
yum -y install epel-release

yum -y install erlang socat wget

wget https://www.rabbitmq.com/releases/rabbitmq-server/v3.6.10/rabbitmq-server-3.6.10-1.el7.noarch.rpm
rpm --import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
rpm -Uvh rabbitmq-server-3.6.10-1.el7.noarch.rpm

systemctl start rabbitmq-server
systemctl enable rabbitmq-server
systemctl status rabbitmq-server

rabbitmq-plugins enable rabbitmq_management
chown -R rabbitmq:rabbitmq /var/lib/rabbitmq/
```

## Cấu hình Cluster RabbitMQ
> ## Node CTL1
Khởi tạo RabbitMQ Cluster:
```
rabbitmqctl add_user openstack Welcome123
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
rabbitmqctl set_user_tags openstack administrator
rabbitmqctl set_policy ha-all '^(?!amq\.).*' '{"ha-mode": "all"}'


scp -p /var/lib/rabbitmq/.erlang.cookie ctl02:/var/lib/rabbitmq/.erlang.cookie
scp -p /var/lib/rabbitmq/.erlang.cookie ctl03:/var/lib/rabbitmq/.erlang.cookie

rabbitmqctl start_app
rabbitmqctl cluster_status
```

Kiểm tra:
```
[root@ctl01 ~]# rabbitmqctl cluster_status
Cluster status of node rabbit@ctl01
[{nodes,[{disc,[rabbit@ctl01]}]},
 {running_nodes,[rabbit@ctl01]},
 {cluster_name,<<"rabbit@ctl01">>},
 {partitions,[]},
 {alarms,[{rabbit@ctl01,[]}]}]
```

> ## Node CTL2
Thực hiện join cluster
```
chown rabbitmq:rabbitmq /var/lib/rabbitmq/.erlang.cookie
chmod 400 /var/lib/rabbitmq/.erlang.cookie

systemctl restart rabbitmq-server.service

rabbitmqctl stop_app
rabbitmqctl join_cluster rabbit@ctl01
rabbitmqctl start_app
```

> ## Node CTL3
Thực hiện join cluster
```
chown rabbitmq:rabbitmq /var/lib/rabbitmq/.erlang.cookie
chmod 400 /var/lib/rabbitmq/.erlang.cookie

systemctl restart rabbitmq-server.service

rabbitmqctl stop_app
rabbitmqctl join_cluster rabbit@ctl01
rabbitmqctl start_app
```

> ## Snapshot 3 node `rabbitMQ`

# Phần 4: Triển khai PCS
## Chuẩn bị môi trường 
> ## Trên tất cả các node CTL1, CTL2, CTL3
```
yum install pacemaker corosync haproxy pcs fence-agents-all resource-agents psmisc policycoreutils-python -y

echo Welcome123 | passwd --stdin hacluster

systemctl enable pcsd.service pacemaker.service corosync.service haproxy.service

systemctl start pcsd.service
```

## Cấu hình Cluster
> ## Trên node CTL1
```
pcs cluster auth ctl01 ctl02 ctl03 -u hacluster -p Welcome123
pcs cluster setup --name ha_cluster ctl01 ctl02 ctl03

pcs cluster enable --all
pcs cluster start --all

pcs property set stonith-enabled=false
pcs property set no-quorum-policy=ignore
pcs property set default-resource-stickiness="INFINITY"

pcs resource create vip_public ocf:heartbeat:IPaddr2 ip=10.10.34.170 cidr_netmask=24 \
meta migration-threshold=3 failure-timeout=60 resource-stickiness=1 \
op monitor interval=5 timeout=20 \
op start interval=0 timeout=30 \
op stop interval=0 timeout=30

pcs resource create p_haproxy systemd:haproxy \
	meta migration-threshold=3 failure-timeout=120 target-role=Started \
	op monitor interval=30 timeout=60 \
	op start interval=0 timeout=60 \
	op stop interval=0 timeout=60   

pcs constraint colocation add vip_public with p_haproxy score=INFINITY
pcs constraint order start vip_public then start p_haproxy
```

## Cấu hình HAProxy
> ## Thực hiện trên 3 node CTL1, CTL2, CTL3
```
cp /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.org 
rm -rf /etc/haproxy/haproxy.cfg

cat >> /etc/haproxy/haproxy.cfg << EOF
global
    daemon
    group  haproxy
    log  /dev/log local0
    log /dev/log    local1 notice
    maxconn  16000
    pidfile  /var/run/haproxy.pid
    stats  socket /var/lib/haproxy/stats
    tune.bufsize  32768
    tune.maxrewrite  1024
    user  haproxy

  
defaults
    log  global
    maxconn  8000
    mode  http
    option  redispatch
    option  http-server-close
    option  splice-auto
    retries  3
    timeout  http-request 20s
    timeout  queue 1m
    timeout  connect 10s
    timeout  client 1m
    timeout  server 1m
    timeout  check 10s

listen stats
    bind 10.10.34.170:8080
    mode http
    stats enable
    stats uri /stats
    stats realm HAProxy\ Statistics

listen mysqld 
    bind 10.10.34.170:3306
    balance  leastconn
    mode  tcp
    option  httpchk
    option  tcplog
    option  clitcpka
    option  srvtcpka
    timeout client  28801s
    timeout server  28801s    
    server controller1 10.10.34.161:3306 check port 9200 inter 5s fastinter 2s rise 3 fall 3 
    server controller2 10.10.34.162:3306 check port 9200 inter 5s fastinter 2s rise 3 fall 3 backup
    server controller3 10.10.34.163:3306 check port 9200 inter 5s fastinter 2s rise 3 fall 3 backup


listen keystone-5000
    bind 10.10.34.170:5000 
    option  httpchk
    option  httplog
    option  httpclose
    balance source
    server controller1 10.10.34.161:5000  check inter 5s fastinter 2s downinter 2s rise 3 fall 3
    server controller2 10.10.34.162:5000  check inter 5s fastinter 2s downinter 2s rise 3 fall 3
    server controller3 10.10.34.163:5000  check inter 5s fastinter 2s downinter 2s rise 3 fall 3

listen keystone-35357
    bind 10.10.34.170:35357
    option  httpchk
    option  httplog
    option  httpclose
    balance source
    server controller1 10.10.34.161:35357  check inter 5s fastinter 2s downinter 2s rise 3 fall 3
    server controller2 10.10.34.162:35357  check inter 5s fastinter 2s downinter 2s rise 3 fall 3
    server controller3 10.10.34.163:35357  check inter 5s fastinter 2s downinter 2s rise 3 fall 3


listen nova-api-8774
    bind 10.10.34.170:8774 
    option  httpchk
    option  httplog
    option  httpclose
    timeout server  600s
    server controller1 10.10.34.161:8774  check inter 5s fastinter 2s downinter 3s rise 3 fall 3
    server controller2 10.10.34.162:8774  check inter 5s fastinter 2s downinter 3s rise 3 fall 3
    server controller3 10.10.34.163:8774  check inter 5s fastinter 2s downinter 3s rise 3 fall 3

listen nova-metadata-api
    bind 10.10.34.170:8775 
    option  httpchk
    option  httplog
    option  httpclose
    server controller1 10.10.34.161:8775  check inter 5s fastinter 2s downinter 3s rise 3 fall 3
    server controller2 10.10.34.162:8775  check inter 5s fastinter 2s downinter 3s rise 3 fall 3
    server controller3 10.10.34.163:8775  check inter 5s fastinter 2s downinter 3s rise 3 fall 3

listen nova-novncproxy
	bind 10.10.34.170:6080
    balance  source
    option  httplog
    server controller1 10.10.34.161:6080  check
    server controller2 10.10.34.162:6080  check
    server controller3 10.10.34.163:6080  check
    
listen nova_placement_api
    bind 10.10.34.170:8778
    balance source
    option tcpka
    option tcplog
    http-request del-header X-Forwarded-Proto
    server controller1 10.10.34.161:8778 check inter 2000 rise 2 fall 5
    server controller2 10.10.34.162:8778 check inter 2000 rise 2 fall 5
    server controller3 10.10.34.163:8778 check inter 2000 rise 2 fall 5    
    
listen glance-api
    bind 10.10.34.170:9292
    option  httpchk /versions
    option  httplog
    option  httpclose
    timeout server  11m
    server controller1 10.10.34.161:9292  check inter 5s fastinter 2s downinter 3s rise 3 fall 3
    server controller2 10.10.34.162:9292  check inter 5s fastinter 2s downinter 3s rise 3 fall 3
    server controller3 10.10.34.163:9292  check inter 5s fastinter 2s downinter 3s rise 3 fall 3

listen glance-registry
    bind 10.10.34.170:9191 
    timeout server  11m
    server controller1 10.10.34.161:9191  check
    server controller2 10.10.34.162:9191  check
    server controller3 10.10.34.163:9191  check

listen neutron
    bind 10.10.34.170:9696 
    option  httpchk
    option  httplog
    option  httpclose
    balance source
    server controller1 10.10.34.161:9696  check inter 5s fastinter 2s downinter 3s rise 3 fall 3
    server controller2 10.10.34.162:9696  check inter 5s fastinter 2s downinter 3s rise 3 fall 3
    server controller3 10.10.34.163:9696  check inter 5s fastinter 2s downinter 3s rise 3 fall 3

listen cinder-api
    bind 10.10.34.170:8776 
    option  httpchk
    option  httplog
    option  httpclose
    server controller1 10.10.34.161:8776  check inter 5s fastinter 2s downinter 3s rise 3 fall 3
    server controller2 10.10.34.162:8776 backup check inter 5s fastinter 2s downinter 3s rise 3 fall 3
    server controller3 10.10.34.163:8776 backup check inter 5s fastinter 2s downinter 3s rise 3 fall 3

  
listen horizon
    bind 10.10.34.170:80
    balance  source
    mode  http
    option  forwardfor
    option  httpchk
    option  httpclose
    option  httplog
    stick  on src
    stick-table  type ip size 200k expire 30m
    timeout  client 3h
    timeout  server 3h
    server controller1 10.10.34.161:80  weight 1 check
    server controller2 10.10.34.162:80  weight 1 check
    server controller3 10.10.34.163:80  weight 1 check
EOF
```

## Restart service
> ## Thực hiện trên node CTL1
```
pcs resource restart p_haproxy
pcs resource cleanup
```

Kiểm tra:
```
[root@ctl01 ~]# pcs status
Cluster name: ha_cluster
Stack: corosync
Current DC: ctl02 (version 1.1.23-1.el7-9acf116022) - partition with quorum
Last updated: Tue Nov 17 17:08:23 2020
Last change: Tue Nov 17 17:07:36 2020 by root via crm_resource on ctl01

3 nodes configured
2 resource instances configured

Online: [ ctl01 ctl02 ctl03 ]

Full list of resources:

 vip_public     (ocf::heartbeat:IPaddr2):       Started ctl01
 p_haproxy      (systemd:haproxy):      Started ctl01

Daemon Status:
  corosync: active/enabled
  pacemaker: active/enabled
  pcsd: active/enabled
```

Hoặc truy cập: 
```
http://10.10.34.170:8080/stats
```

# Phần 5: Cài đặt các gói cần thiết cho OPS
> ## Thực hiện trên tất cả các node CTL
```
yum -y install centos-release-openstack-queens
yum -y install crudini wget vim
yum -y install python-openstackclient openstack-selinux python2-PyMySQL
```

# Phần 6: Cấu hình memcache
**Lưu ý:** Thực hiện xong từng node mới chuyển node thực hiện
> ## Trên node CTL1
```
yum install -y memcached

sed -i "s/-l 127.0.0.1,::1/-l 10.10.34.161/g" /etc/sysconfig/memcached

systemctl enable memcached.service
systemctl restart memcached.service
```

> ## Trên node CTL2
```
yum install -y memcached

sed -i "s/-l 127.0.0.1,::1/-l 10.10.34.162/g" /etc/sysconfig/memcached

systemctl enable memcached.service
systemctl restart memcached.service
```

> ## Trên node CTL3
```
yum install -y memcached

sed -i "s/-l 127.0.0.1,::1/-l 10.10.34.163/g" /etc/sysconfig/memcached

systemctl enable memcached.service
systemctl restart memcached.service
```

> ## Snapshot `memcached`

# Phần 7: Cài đặt Keystone
## Tạo database
> ## node CTL1
```
mysql -u root -pWelcome123
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
IDENTIFIED BY 'Welcome123';

GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'ctl01' IDENTIFIED BY 'Welcome123';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'ctl02' IDENTIFIED BY 'Welcome123';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'ctl03' IDENTIFIED BY 'Welcome123';FLUSH PRIVILEGES;
exit
```

## Cài đặt các gói
> ## Thực hiện trên tất cả các node CTL1, CTL2, CTL3
```
yum install openstack-keystone httpd mod_wsgi -y
```

## Cấu hình bind port
> ## Trên node CTL1
Cấu hình bind port keystone
```
cp /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
sed -i -e 's/VirtualHost \*/VirtualHost 10.10.34.161/g' /etc/httpd/conf.d/wsgi-keystone.conf
sed -i -e 's/Listen 5000/Listen 10.10.34.161:5000/g' /etc/httpd/conf.d/wsgi-keystone.conf
sed -i -e 's/Listen 35357/Listen 10.10.34.161:35357/g' /etc/httpd/conf.d/wsgi-keystone.conf
sed -i -e 's/^Listen.*/Listen 10.10.34.161:80/g' /etc/httpd/conf/httpd.conf
```

> ## Trên node CTL2
Cấu hình bind port keystone
```
cp /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
sed -i -e 's/VirtualHost \*/VirtualHost 10.10.34.162/g' /etc/httpd/conf.d/wsgi-keystone.conf
sed -i -e 's/Listen 5000/Listen 10.10.34.162:5000/g' /etc/httpd/conf.d/wsgi-keystone.conf
sed -i -e 's/Listen 35357/Listen 10.10.34.162:35357/g' /etc/httpd/conf.d/wsgi-keystone.conf
sed -i -e 's/^Listen.*/Listen 10.10.34.162:80/g' /etc/httpd/conf/httpd.conf
```

> ## Trên node CTL3
Cấu hình bind port keystone
```
cp /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
sed -i -e 's/VirtualHost \*/VirtualHost 10.10.34.163/g' /etc/httpd/conf.d/wsgi-keystone.conf
sed -i -e 's/Listen 5000/Listen 10.10.34.163:5000/g' /etc/httpd/conf.d/wsgi-keystone.conf
sed -i -e 's/Listen 35357/Listen 10.10.34.163:35357/g' /etc/httpd/conf.d/wsgi-keystone.conf
sed -i -e 's/^Listen.*/Listen 10.10.34.163:80/g' /etc/httpd/conf/httpd.conf
```

## Cấu hình Keystone
> ## Trên node CTL1
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
connection = mysql+pymysql://keystone:Welcome123@10.10.34.170/keystone
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
[oslo_messaging_rabbit]
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

Phân quyền
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

Chuyển 2 thư mục này sang 2 CTL còn lại
```
scp -r /etc/keystone/credential-keys /etc/keystone/fernet-keys root@ctl02:/etc/keystone/

scp -r /etc/keystone/credential-keys /etc/keystone/fernet-keys root@ctl03:/etc/keystone/
```

Bootstrap
```
keystone-manage bootstrap --bootstrap-password Welcome123 \
  --bootstrap-admin-url http://10.10.34.170:5000/v3/ \
  --bootstrap-internal-url http://10.10.34.170:5000/v3/ \
  --bootstrap-public-url http://10.10.34.170:5000/v3/ \
  --bootstrap-region-id RegionOne
```

Enable và start httpd
```
systemctl enable httpd.service
systemctl start httpd.service
```

Export biến môi trường
```
export OS_USERNAME=admin
export OS_PASSWORD=Welcome123
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://10.10.34.170:35357/v3
export OS_IDENTITY_API_VERSION=3
```

Tạo domain, user
```
openstack domain create --description "An Example Domain" example
openstack project create --domain default \
  --description "Service Project" service

openstack project create --domain default \
  --description "Demo Project" demo

openstack user create --domain default \
  --password Welcome123 demo

openstack role create user
openstack role add --project demo --user demo user

unset OS_AUTH_URL OS_PASSWORD

openstack --os-auth-url http://10.10.34.170:35357/v3 \
  --os-project-domain-name Default --os-user-domain-name Default \
  --os-project-name admin --os-username admin token issue
```

Tạo file xác thực
```
cat << EOF >> admin-openrc
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=Welcome123
export OS_AUTH_URL=http://10.10.34.170:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
EOF

cat << EOF >> demo-openrc
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=Welcome123
export OS_AUTH_URL=http://10.10.34.170:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
EOF
```

> ## Trên node CTL2, CTL3
Thực hiện lần lượt trên từng node

Tạo file xác thực:
```
cat << EOF >> admin-openrc
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=Welcome123
export OS_AUTH_URL=http://10.10.34.170:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
EOF

cat << EOF >> demo-openrc
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=Welcome123
export OS_AUTH_URL=http://10.10.34.170:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
EOF
```

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
connection = mysql+pymysql://keystone:Welcome123@10.10.34.170/keystone
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
[oslo_messaging_rabbit]
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

chown root:keystone /etc/keystone/keystone.conf
chown -R keystone:keystone /etc/keystone/credential-keys /etc/keystone/fernet-keys

systemctl enable httpd.service
systemctl start httpd.service
```

## Kiểm tra
Test theo test case sau:

- Bật HTTPD tại CTL 1, tắt HTTP trên CTL 2 3, lấy token
- Bật HTTPD tại CTL 2, tắt HTTP trên CTL 1 3, lấy token
- Bật HTTPD tại CTL 3, tắt HTTP trên CTL 1 2, lấy token

Bảo đảm các test case đều pass
```
systemctl stop httpd
source admin-openrc
openstack token issue
```

Kiểm tra đồng thời HAProxy Stats: http://10.10.34.170:8080/stats

> ## Snapshot `keystone`

# Phần 8: Cài đặt Glance
## Tạo database, user, endpoint
> ## Trên node CTL1
Tạo Database
```sql
mysql -u root -pWelcome123

CREATE DATABASE glance;

GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' \
  IDENTIFIED BY 'Welcome123';

GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \
  IDENTIFIED BY 'Welcome123';

GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'ctl01' IDENTIFIED BY 'Welcome123';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'ctl02' IDENTIFIED BY 'Welcome123';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'ctl03' IDENTIFIED BY 'Welcome123';FLUSH PRIVILEGES;
exit
```

Tạo user
```
openstack user create --domain default --password Welcome123 glance

openstack role add --project service --user glance admin

openstack service create --name glance \
  --description "OpenStack Image" image
```

Tạo endpoint
```
openstack endpoint create --region RegionOne \
  image public http://10.10.34.170:9292

openstack endpoint create --region RegionOne \
  image admin http://10.10.34.170:9292

openstack endpoint create --region RegionOne \
  image internal http://10.10.34.170:9292
```

## Cài đặt package Glance
> ## Thực hiện trên tất cả các node CTL1, CTL2, CTL3
```
yum install -y openstack-glance
```

## Cấu hình glance-api và glance-registry
> ## Trên node CTL1
Cấu hình glance-api
```
cp /etc/glance/glance-api.conf /etc/glance/glance-api.conf.org 
rm -rf /etc/glance/glance-api.conf

cat << EOF >> /etc/glance/glance-api.conf
[DEFAULT]
bind_host = 10.10.34.161
registry_host = 10.10.34.170
[cors]
[database]
connection = mysql+pymysql://glance:Welcome123@10.10.34.170/glance
[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
[image_format]
[keystone_authtoken]
auth_uri = http://10.10.34.170:5000
auth_url = http://10.10.34.170:5000
memcached_servers = 10.10.34.161:11211,10.10.34.162:11211,10.10.34.163:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = Welcome123
[matchmaker_redis]
[oslo_concurrency]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
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

Cấu hình glance-registry
```
cp /etc/glance/glance-registry.conf /etc/glance/glance-registry.conf.org
rm -rf /etc/glance/glance-registry.conf

cat << EOF >> /etc/glance/glance-registry.conf
[DEFAULT]
bind_host = 10.10.34.161
[database]
connection = mysql+pymysql://glance:Welcome123@10.10.34.170/glance
[keystone_authtoken]
auth_uri = http://10.10.34.170:5000
auth_url = http://10.10.34.170:5000
memcached_servers = 10.10.34.161:11211,10.10.34.162:11211,10.10.34.163:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = Welcome123
[matchmaker_redis]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_policy]
[paste_deploy]
flavor = keystone
[profiler]
EOF
```

Phân quyền
```
chown root:glance /etc/glance/glance-api.conf
chown root:glance /etc/glance/glance-registry.conf
```

Sync DB
```
su -s /bin/sh -c "glance-manage db_sync" glance
```

Enable và start dịch vụ
```
systemctl enable openstack-glance-api.service \
  openstack-glance-registry.service

systemctl start openstack-glance-api.service \
  openstack-glance-registry.service
```

Download và tạo image
```
wget http://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img

openstack image create "cirros" \
  --file cirros-0.3.5-x86_64-disk.img \
  --disk-format qcow2 --container-format bare \
  --public
```

> ## node CTL2
Cấu hình glance-api
```
cp /etc/glance/glance-api.conf /etc/glance/glance-api.conf.org 
rm -rf /etc/glance/glance-api.conf

cat << EOF >> /etc/glance/glance-api.conf
[DEFAULT]
bind_host = 10.10.34.162
registry_host = 10.10.34.170
[cors]
[database]
connection = mysql+pymysql://glance:Welcome123@10.10.34.170/glance
[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
[image_format]
[keystone_authtoken]
auth_uri = http://10.10.34.170:5000
auth_url = http://10.10.34.170:5000
memcached_servers = 10.10.34.161:11211,10.10.34.162:11211,10.10.34.163:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = Welcome123
[matchmaker_redis]
[oslo_concurrency]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
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

Cấu hình glance-registry
```
cp /etc/glance/glance-registry.conf /etc/glance/glance-registry.conf.org
rm -rf /etc/glance/glance-registry.conf
cat << EOF >> /etc/glance/glance-registry.conf
[DEFAULT]
bind_host = 10.10.34.162
[database]
connection = mysql+pymysql://glance:Welcome123@10.10.34.170/glance
[keystone_authtoken]
auth_uri = http://10.10.34.170:5000
auth_url = http://10.10.34.170:5000
memcached_servers = 10.10.34.161:11211,10.10.34.162:11211,10.10.34.163:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = Welcome123
[matchmaker_redis]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_policy]
[paste_deploy]
flavor = keystone
[profiler]
EOF
```

Phân quyền
```
chown root:glance /etc/glance/glance-api.conf
chown root:glance /etc/glance/glance-registry.conf
```

Enable và start dịch vụ
```
systemctl enable openstack-glance-api.service \
  openstack-glance-registry.service

systemctl start openstack-glance-api.service \
  openstack-glance-registry.service
```

> ## Node CTL3
Cấu hình glance-api
```
cp /etc/glance/glance-api.conf /etc/glance/glance-api.conf.org 
rm -rf /etc/glance/glance-api.conf

cat << EOF >> /etc/glance/glance-api.conf
[DEFAULT]
bind_host = 10.10.34.163
registry_host = 10.10.34.170
[cors]
[database]
connection = mysql+pymysql://glance:Welcome123@10.10.34.170/glance
[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
[image_format]
[keystone_authtoken]
auth_uri = http://10.10.34.170:5000
auth_url = http://10.10.34.170:5000
memcached_servers = 10.10.34.161:11211,10.10.34.162:11211,10.10.34.163:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = Welcome123
[matchmaker_redis]
[oslo_concurrency]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
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

Cấu hình glance-registry
```
cp /etc/glance/glance-registry.conf /etc/glance/glance-registry.conf.org
rm -rf /etc/glance/glance-registry.conf
cat << EOF >> /etc/glance/glance-registry.conf
[DEFAULT]
bind_host = 10.10.34.163
[database]
connection = mysql+pymysql://glance:Welcome123@10.10.34.170/glance
[keystone_authtoken]
auth_uri = http://10.10.34.170:5000
auth_url = http://10.10.34.170:5000
memcached_servers = 10.10.34.161:11211,10.10.34.162:11211,10.10.34.163:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = Welcome123
[matchmaker_redis]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_policy]
[paste_deploy]
flavor = keystone
[profiler]
EOF
```

Phân quyền
```
chown root:glance /etc/glance/glance-api.conf
chown root:glance /etc/glance/glance-registry.conf
```

Enable và start dịch vụ
```
systemctl enable openstack-glance-api.service \
  openstack-glance-registry.service

systemctl start openstack-glance-api.service \
  openstack-glance-registry.service
```

## Copy image từ CTL1 sang CTL2, CTL3
> ## Trên CTL1
```
[root@ctl01 ~]# cd /var/lib/glance/images/
[root@ctl01 images]# ls
36a40c72-7b6f-4fc3-98f3-dd7a3da7e89e

scp 36a40c72-7b6f-4fc3-98f3-dd7a3da7e89e root@ctl02:/var/lib/glance/images/

scp 36a40c72-7b6f-4fc3-98f3-dd7a3da7e89e root@ctl03:/var/lib/glance/images/
```

> ## Trên CTL2, CTL3
```
chown -R glance:glance /var/lib/glance/images
```

## Kiểm tra
Test theo test case sau:
- Bật Glance services tại CTL 1, tắt Glance services trên CTL 2 3, get list image
- Bật Glance services tại CTL 2, tắt Glance services trên CTL 1 3, get list image
- Bật Glance services tại CTL 3, tắt Glance services trên CTL 1 2, get list image

Bảo đảm các test case đều pass
```
systemctl stop openstack-glance-api.service \
  openstack-glance-registry.service

openstack image list
```
Kiểm tra đồng thời HAProxy Stats: http://10.10.34.170:8080/stats

# Phần 9: Cài đặt Nova
## Cài đặt các gói
> ## Thực hiện trên tất cả các node
```
yum install -y openstack-nova-api openstack-nova-conductor \
  openstack-nova-console openstack-nova-novncproxy \
  openstack-nova-scheduler openstack-nova-placement-api
```

## Tạo database, user, endpoint
> ## Thực hiện trên node CTL1
Tạo database
```
mysql -u root -pWelcome123

CREATE DATABASE nova_api;
CREATE DATABASE nova;
CREATE DATABASE nova_cell0;

GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' \
  IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'ctl01' IDENTIFIED BY 'Welcome123';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'ctl02' IDENTIFIED BY 'Welcome123';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'ctl03' IDENTIFIED BY 'Welcome123';FLUSH PRIVILEGES;


GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \
  IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'ctl01' IDENTIFIED BY 'Welcome123';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'ctl02' IDENTIFIED BY 'Welcome123';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'ctl03' IDENTIFIED BY 'Welcome123';FLUSH PRIVILEGES;


GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' \
  IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'ctl01' IDENTIFIED BY 'Welcome123';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'ctl02' IDENTIFIED BY 'Welcome123';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'ctl03' IDENTIFIED BY 'Welcome123';FLUSH PRIVILEGES;
exit
```

Tạo user và endpoint
```
openstack user create --domain default --password Welcome123 nova
openstack role add --project service --user nova admin
openstack service create --name nova \
  --description "OpenStack Compute" compute

openstack endpoint create --region RegionOne \
  compute public http://10.10.34.170:8774/v2.1
openstack endpoint create --region RegionOne \
  compute admin http://10.10.34.170:8774/v2.1
openstack endpoint create --region RegionOne \
  compute internal http://10.10.34.170:8774/v2.1

openstack user create --domain default --password Welcome123 placement
openstack role add --project service --user placement admin
openstack service create --name placement --description "Placement API" placement
  
openstack endpoint create --region RegionOne placement public http://10.10.34.170:8778
openstack endpoint create --region RegionOne placement admin http://10.10.34.170:8778
openstack endpoint create --region RegionOne placement internal http://10.10.34.170:8778
```

## Cấu hình Nova
> ## Trên node CTL1
Cấu hình file nova.conf
```
cp /etc/nova/nova.conf /etc/nova/nova.conf.org 
rm -rf /etc/nova/nova.conf

cat << EOF >> /etc/nova/nova.conf
[DEFAULT]
my_ip = 10.10.34.161
enabled_apis = osapi_compute,metadata
use_neutron = True
osapi_compute_listen=10.10.34.161
metadata_host=10.10.34.161
metadata_listen=10.10.34.161
metadata_listen_port=8775
firewall_driver = nova.virt.firewall.NoopFirewallDriver
transport_url = rabbit://openstack:Welcome123@10.10.34.161:5672,openstack:Welcome123@10.10.34.162:5672,openstack:Welcome123@10.10.34.163:5672
[api]
auth_strategy = keystone
[api_database]
connection = mysql+pymysql://nova:Welcome123@10.10.34.170/nova_api
[barbican]
[cache]
backend = oslo_cache.memcache_pool
enabled = true
memcache_servers = 10.10.34.161:11211,10.10.34.162:11211,10.10.34.163:11211
[cells]
[cinder]
[compute]
[conductor]
[console]
[consoleauth]
[cors]
[crypto]
[database]
connection = mysql+pymysql://nova:Welcome123@10.10.34.170/nova
[devices]
[ephemeral_storage_encryption]
[filter_scheduler]
[glance]
api_servers = http://10.10.34.170:9292
[guestfs]
[healthcheck]
[hyperv]
[ironic]
[key_manager]
[keystone]
[keystone_authtoken]
auth_url = http://10.10.34.170:5000/v3
memcached_servers = 10.10.34.161:11211,10.10.34.162:11211,10.10.34.163:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = Welcome123
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
auth_url = http://10.10.34.170:5000/v3
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
novncproxy_host=10.10.34.161
enabled = true
#vncserver_listen = 10.10.34.161
#vncserver_proxyclient_address = 10.10.34.161
novncproxy_base_url = http://10.10.34.170:6080/vnc_auto.html
[workarounds]
[wsgi]
[xenserver]
[xvp]
EOF
```

Thêm vào file `00-nova-placement-api.conf`
```xml
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
sed -i -e 's/VirtualHost \*/VirtualHost 10.10.34.161/g' /etc/httpd/conf.d/00-nova-placement-api.conf
sed -i -e 's/Listen 8778/Listen 10.10.34.161:8778/g' /etc/httpd/conf.d/00-nova-placement-api.conf

systemctl restart httpd
```

Sync db
```
su -s /bin/sh -c "nova-manage api_db sync" nova
su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
su -s /bin/sh -c "nova-manage db sync" nova
```

Enable và start service
```
systemctl enable openstack-nova-api.service \
  openstack-nova-scheduler.service openstack-nova-consoleauth.service \
  openstack-nova-conductor.service openstack-nova-novncproxy.service

systemctl restart openstack-nova-api.service \
  openstack-nova-scheduler.service openstack-nova-consoleauth.service \
  openstack-nova-conductor.service openstack-nova-novncproxy.service

openstack compute service list
```

> ## Trên node CTL2
Cấu hình nova.conf
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
transport_url = rabbit://openstack:Welcome123@10.10.34.161:5672,openstack:Welcome123@10.10.34.162:5672,openstack:Welcome123@10.10.34.163:5672
[api]
auth_strategy = keystone
[api_database]
connection = mysql+pymysql://nova:Welcome123@10.10.34.170/nova_api
[barbican]
[cache]
backend = oslo_cache.memcache_pool
enabled = true
memcache_servers = 10.10.34.161:11211,10.10.34.162:11211,10.10.34.163:11211
[cells]
[cinder]
[compute]
[conductor]
[console]
[consoleauth]
[cors]
[crypto]
[database]
connection = mysql+pymysql://nova:Welcome123@10.10.34.170/nova
[devices]
[ephemeral_storage_encryption]
[filter_scheduler]
[glance]
api_servers = http://10.10.34.170:9292
[guestfs]
[healthcheck]
[hyperv]
[ironic]
[key_manager]
[keystone]
[keystone_authtoken]
auth_url = http://10.10.34.170:5000/v3
memcached_servers = 10.10.34.161:11211,10.10.34.162:11211,10.10.34.163:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = Welcome123
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
auth_url = http://10.10.34.170:5000/v3
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
#vncserver_listen = 10.10.34.162
#vncserver_proxyclient_address = 10.10.34.162
novncproxy_base_url = http://10.10.34.170:6080/vnc_auto.html
[workarounds]
[wsgi]
[xenserver]
[xvp]
EOF
```

Thêm vào file `00-nova-placement-api.conf`
```
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

systemctl restart httpd
```

Enable và start service 
```
systemctl enable openstack-nova-api.service \
  openstack-nova-scheduler.service openstack-nova-consoleauth.service \
  openstack-nova-conductor.service openstack-nova-novncproxy.service

systemctl start openstack-nova-api.service \
  openstack-nova-scheduler.service openstack-nova-consoleauth.service \
  openstack-nova-conductor.service openstack-nova-novncproxy.service
```

> ## Trên node CTL3
Cấu hình nova
```
cp /etc/nova/nova.conf /etc/nova/nova.conf.org 
rm -rf /etc/nova/nova.conf

cat << EOF >> /etc/nova/nova.conf
[DEFAULT]
my_ip = 10.10.34.163
enabled_apis = osapi_compute,metadata
use_neutron = True
osapi_compute_listen=10.10.34.163
metadata_host=10.10.34.163
metadata_listen=10.10.34.163
metadata_listen_port=8775
firewall_driver = nova.virt.firewall.NoopFirewallDriver
transport_url = rabbit://openstack:Welcome123@10.10.34.161:5672,openstack:Welcome123@10.10.34.162:5672,openstack:Welcome123@10.10.34.163:5672
[api]
auth_strategy = keystone
[api_database]
connection = mysql+pymysql://nova:Welcome123@10.10.34.170/nova_api
[barbican]
[cache]
backend = oslo_cache.memcache_pool
enabled = true
memcache_servers = 10.10.34.161:11211,10.10.34.162:11211,10.10.34.163:11211
[cells]
[cinder]
[compute]
[conductor]
[console]
[consoleauth]
[cors]
[crypto]
[database]
connection = mysql+pymysql://nova:Welcome123@10.10.34.170/nova
[devices]
[ephemeral_storage_encryption]
[filter_scheduler]
[glance]
api_servers = http://10.10.34.170:9292
[guestfs]
[healthcheck]
[hyperv]
[ironic]
[key_manager]
[keystone]
[keystone_authtoken]
auth_url = http://10.10.34.170:5000/v3
memcached_servers = 10.10.34.161:11211,10.10.34.162:11211,10.10.34.163:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = Welcome123
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
auth_url = http://10.10.34.170:5000/v3
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
novncproxy_host=10.10.34.163
enabled = true
#vncserver_listen = 10.10.34.163
#vncserver_proxyclient_address = 10.10.34.163
novncproxy_base_url = http://10.10.34.170:6080/vnc_auto.html
[workarounds]
[wsgi]
[xenserver]
[xvp]
EOF
```

Thêm vào file 00-nova-placement-api.conf
```
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
sed -i -e 's/VirtualHost \*/VirtualHost 10.10.34.163/g' /etc/httpd/conf.d/00-nova-placement-api.conf
sed -i -e 's/Listen 8778/Listen 10.10.34.163:8778/g' /etc/httpd/conf.d/00-nova-placement-api.conf
systemctl restart httpd
```

Enable và start service
```
systemctl enable openstack-nova-api.service \
  openstack-nova-scheduler.service openstack-nova-consoleauth.service \
  openstack-nova-conductor.service openstack-nova-novncproxy.service

systemctl start openstack-nova-api.service \
  openstack-nova-scheduler.service openstack-nova-consoleauth.service \
  openstack-nova-conductor.service openstack-nova-novncproxy.service
```

## Kiểm tra
Test theo các use case:

- Bật Nova services tại CTL 1, tắt Nova services trên CTL 2 3, list nova services
- Bật Nova services tại CTL 2, tắt Nova services trên CTL 1 3, list nova services
- Bật Nova services tại CTL 3, tắt Nova services trên CTL 1 2, list nova services

Bảo đảm các test case đều pass
```
systemctl stop openstack-nova-api.service \
  openstack-nova-scheduler.service openstack-nova-consoleauth.service \
  openstack-nova-conductor.service openstack-nova-novncproxy.service

openstack compute service list
```

Kiểm tra đồng thời HAProxy Stats: http://10.10.34.170:8080/stats

> ## Snapshot `nova`

# Phần 10: Cài đặt Neutron
## Cài đặt các package 
> ## Thực hiện trên tất cả các node CTL
```
yum install openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables -y
```

## Tạo database, user, endpoint
Tạo DB:
```
mysql -u root -pWelcome123
CREATE DATABASE neutron;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' \
  IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' \
  IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'ctl01' IDENTIFIED BY 'Welcome123';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'ctl02' IDENTIFIED BY 'Welcome123';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'ctl03' IDENTIFIED BY 'Welcome123';FLUSH PRIVILEGES;
exit
```

Tạo user, endpoint trên 1 node
```
openstack user create --domain default --password Welcome123 neutron
openstack role add --project service --user neutron admin
openstack service create --name neutron \
  --description "OpenStack Networking" network
  
openstack endpoint create --region RegionOne \
    network public http://10.10.34.170:9696
openstack endpoint create --region RegionOne \
  network internal http://10.10.34.170:9696
openstack endpoint create --region RegionOne \
  network admin http://10.10.34.170:9696
```

## Cấu hình Neutron
> ## Trên node CTL1
Cấu hình neutron.conf
```
cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.org
rm -rf /etc/neutron/neutron.conf

cat << EOF >> /etc/neutron/neutron.conf
[DEFAULT]
bind_host = 10.10.34.161
core_plugin = ml2
transport_url = rabbit://openstack:Welcome123@10.10.34.161:5672,openstack:Welcome123@10.10.34.162:5672,openstack:Welcome123@10.10.34.163:5672
auth_strategy = keystone
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true
allow_overlapping_ips = True
dhcp_agents_per_network = 2
service_plugins = neutron.services.qos.qos_plugin.QoSPlugin

[agent]
[cors]
[database]
connection = mysql+pymysql://neutron:Welcome123@10.10.34.170/neutron
[keystone_authtoken]
auth_uri = http://10.10.34.170:5000
auth_url = http://10.10.34.170:35357
memcached_servers = 10.10.34.161:11211,10.10.34.162:11211,10.10.34.163:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = Welcome123
[matchmaker_redis]
[nova]
auth_url = http://10.10.34.170:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = Welcome123
[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
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

Cấu hình file ml2
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
extension_drivers = port_security,qos
[ml2_type_flat]
flat_networks = provider
[ml2_type_geneve]
[ml2_type_gre]
[ml2_type_vlan]
#network_vlan_ranges = provider
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
extensions = qos
[linux_bridge]
physical_interface_mappings = provider:eth1 
[network_log]
[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
[vxlan]
enable_vxlan = true
local_ip = 10.10.32.161
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
url = http://10.10.34.170:9696
auth_url = http://10.10.34.170:35357
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

Restart lại service nova
```
systemctl restart openstack-nova-api.service \
    openstack-nova-scheduler.service \
    openstack-nova-consoleauth.service \
    openstack-nova-conductor.service \
    openstack-nova-novncproxy.service
```

Phân quyền
```
chown -R root:neutron /etc/neutron/
```

Tạo liên kết
```
ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
```

Sync database:
```
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
```

Enable và start dịch vụ
```
systemctl enable neutron-server.service neutron-linuxbridge-agent.service neutron-l3-agent.service

systemctl restart neutron-server.service neutron-linuxbridge-agent.service neutron-l3-agent.service

openstack network agent list
```

> ## Trên node CTL2
Cấu hình `neutron.conf`
```
cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.org
rm -rf /etc/neutron/neutron.conf

cat << EOF >> /etc/neutron/neutron.conf
[DEFAULT]
bind_host = 10.10.34.162
core_plugin = ml2
transport_url = rabbit://openstack:Welcome123@10.10.34.161:5672,openstack:Welcome123@10.10.34.162:5672,openstack:Welcome123@10.10.34.163:5672
auth_strategy = keystone
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true
allow_overlapping_ips = True
dhcp_agents_per_network = 2
service_plugins = neutron.services.qos.qos_plugin.QoSPlugin

[agent]
[cors]
[database]
connection = mysql+pymysql://neutron:Welcome123@10.10.34.170/neutron
[keystone_authtoken]
auth_uri = http://10.10.34.170:5000
auth_url = http://10.10.34.170:35357
memcached_servers = 10.10.34.161:11211,10.10.34.162:11211,10.10.34.163:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = Welcome123
[matchmaker_redis]
[nova]
auth_url = http://10.10.34.170:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = Welcome123
[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
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

Cấu hình file ml2
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
extension_drivers = port_security,qos
[ml2_type_flat]
flat_networks = provider
[ml2_type_geneve]
[ml2_type_gre]
[ml2_type_vlan]
#network_vlan_ranges = provider
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
extensions = qos
[linux_bridge]
physical_interface_mappings = provider:eth1 
[network_log]
[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
[vxlan]
enable_vxlan = true
local_ip = 10.10.32.162
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

Chỉnh sửa file /etc/nova/nova.conf
```
[neutron]
url = http://10.10.34.170:9696
auth_url = http://10.10.34.170:35357
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

Restart lại service nova
```
systemctl restart openstack-nova-api.service \
openstack-nova-scheduler.service \
openstack-nova-consoleauth.service \
openstack-nova-conductor.service \
openstack-nova-novncproxy.service
```

Phân quyền
```
chown -R root:neutron /etc/neutron/
```

Tạo liên kết
```
ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
```

Enable và start dịch vụ
```
systemctl enable neutron-server.service neutron-linuxbridge-agent.service neutron-l3-agent.service

systemctl restart neutron-server.service neutron-linuxbridge-agent.service neutron-l3-agent.service
```

> ## Trên node CTL3
Cấu hình neutron.conf
```
cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.org
rm -rf /etc/neutron/neutron.conf

cat << EOF >> /etc/neutron/neutron.conf
[DEFAULT]
bind_host = 10.10.34.163
core_plugin = ml2
transport_url = rabbit://openstack:Welcome123@10.10.34.161:5672,openstack:Welcome123@10.10.34.162:5672,openstack:Welcome123@10.10.34.163:5672
auth_strategy = keystone
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true
allow_overlapping_ips = True
dhcp_agents_per_network = 2
service_plugins = neutron.services.qos.qos_plugin.QoSPlugin

[agent]
[cors]
[database]
connection = mysql+pymysql://neutron:Welcome123@10.10.34.170/neutron
[keystone_authtoken]
auth_uri = http://10.10.34.170:5000
auth_url = http://10.10.34.170:35357
memcached_servers = 10.10.34.161:11211,10.10.34.162:11211,10.10.34.163:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = Welcome123
[matchmaker_redis]
[nova]
auth_url = http://10.10.34.170:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = Welcome123
[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
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

Cấu hình file ml2
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
extension_drivers = port_security,qos
[ml2_type_flat]
flat_networks = provider
[ml2_type_geneve]
[ml2_type_gre]
[ml2_type_vlan]
#network_vlan_ranges = provider
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
extensions = qos
[linux_bridge]
physical_interface_mappings = provider:eth1 
[network_log]
[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
[vxlan]
enable_vxlan = true
local_ip = 10.10.32.163
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
url = http://10.10.34.170:9696
auth_url = http://10.10.34.170:35357
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

Restart lại service nova
```
systemctl restart openstack-nova-api.service \
openstack-nova-scheduler.service \
openstack-nova-consoleauth.service \
openstack-nova-conductor.service \
openstack-nova-novncproxy.service
```

Phân quyền
```
chown -R root:neutron /etc/neutron/
```

Tạo liên kết
```
ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
```

Enable và start dịch vụ
```
systemctl enable neutron-server.service neutron-linuxbridge-agent.service neutron-l3-agent.service

systemctl restart neutron-server.service neutron-linuxbridge-agent.service neutron-l3-agent.service
```

## Kiểm tra
Test lại dịch vụ:

- Bật Neutron service tại CTL 1, tắt Neutron service trên CTL 2 3, list Neutron service
- Bật Neutron service tại CTL 2, tắt Neutron service trên CTL 1 3, list Neutron service
- Bật Neutron service tại CTL 3, tắt Neutron service trên CTL 1 2, list Neutron service

```
systemctl stop neutron-server.service neutron-linuxbridge-agent.service neutron-l3-agent.service

openstack network agent list
```

Kiểm tra đồng thời HAProxy Stats: http://10.10.34.170:8080/stats

> ## Snapshot `Neutron`

# Phần 11: Cấu hình Horizon
## Tải package và tạo file direct
> ## Thực hiện trên tất cả các node
Tải các package
```
yum install openstack-dashboard -y
```

Tạo file direct
```
filehtml=/var/www/html/index.html
touch $filehtml
cat << EOF >> $filehtml
<html>
<head>
<META HTTP-EQUIV="Refresh" Content="0.5; URL=http://10.10.34.170/dashboard">
</head>
<body>
<center> <h1>Redirecting to OpenStack Dashboard</h1> </center>
</body>
</html>
EOF
```

## Cấu hình Horizon
> ## Trên node CTL1
Backup file cấu hình
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

### Lưu ý thêm SESSION_ENGINE vào trên dòng CACHE như bên dưới
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
CACHES = {
    'default': {
         'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
         'LOCATION': ['10.10.34.161:11211','10.10.34.162:11211','10.10.34.163:11211',]
    }
}
OPENSTACK_HOST = "10.10.34.170"
OPENSTACK_KEYSTONE_URL = "http://10.10.34.170:5000/v3"
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"

### Lưu ý: Nếu chỉ sử dụng provider, chỉnh sửa các thông số sau
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

Scp sang 2 node còn lại. Nên xóa 2 file cấu hình trên 2 node còn lại trước
```
scp /etc/openstack-dashboard/local_settings root@ctl02:/etc/openstack-dashboard/

scp /etc/openstack-dashboard/local_settings root@ctl03:/etc/openstack-dashboard/
```

Thêm vào file `/etc/httpd/conf.d/openstack-dashboard.conf`
```
echo "WSGIApplicationGroup %{GLOBAL}" >> /etc/httpd/conf.d/openstack-dashboard.conf
```

Restart lại httpd
```
systemctl restart httpd.service memcached.service
```

> ## Trên node CTL2 và CTL3
Kiểm tra file `/etc/openstack-dashboard/local_settings` -> Bảo đảm nội dùng giống cấu hình trên CTL 1

Phân quyền cho cấu hình mới

**Lưu ý:** Yêu cầu thực hiện xong bước chuyển cấu hình qua 2 node còn lại trên node ctl01
```
chown root:apache /etc/openstack-dashboard/local_settings
```

Thêm vào file `/etc/httpd/conf.d/openstack-dashboard.conf`
```
echo "WSGIApplicationGroup %{GLOBAL}" >> /etc/httpd/conf.d/openstack-dashboard.conf
```

Restart lại httpd
```
systemctl restart httpd.service memcached.service
```

## Kiểm tra
Truy cập địa chỉ VIP:
```
http://10.10.34.170/
```

Đồng thời kiểm tra HAProxy Stats: `http://10.10.34.170:8080/stats`

> ## Snapshot `horizon`

# Phần 12: Cấu hình Cinder
**Yêu cầu:** [Cài đặt node CEPH-AIO trước](./ceph-aio.md)

## Cài đặt Cinder
> ## Thực hiện trên tất cả các node CTL
```
yum install openstack-cinder -y
```

## Tạo Database, endpoint
> ## Thực hiện trên node CTL1
Tạo databases
```sql
mysql -u root -pWelcome123
CREATE DATABASE cinder;
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' \
  IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' \
  IDENTIFIED BY 'Welcome123';  
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'ctl01' IDENTIFIED BY 'Welcome123';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'ctl02' IDENTIFIED BY 'Welcome123';FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'ctl03' IDENTIFIED BY 'Welcome123';FLUSH PRIVILEGES;
exit
```

Tạo endpoint
```
openstack user create --domain default --password Welcome123 cinder
openstack role add --project service --user cinder admin
openstack service create --name cinderv2 \
  --description "OpenStack Block Storage" volumev2
  
openstack service create --name cinderv3 \
  --description "OpenStack Block Storage" volumev3
  
openstack endpoint create --region RegionOne \
  volumev2 public http://10.10.34.170:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionOne \
  volumev2 internal http://10.10.34.170:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionOne \
  volumev2 admin http://10.10.34.170:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionOne \
  volumev3 public http://10.10.34.170:8776/v3/%\(project_id\)s
openstack endpoint create --region RegionOne \
  volumev3 internal http://10.10.34.170:8776/v3/%\(project_id\)s
openstack endpoint create --region RegionOne \
  volumev3 admin http://10.10.34.170:8776/v3/%\(project_id\)s
```

## Cấu hình Cinder
> ## Trên node CTL1
```
cp /etc/cinder/cinder.conf /etc/cinder/cinder.conf.bak 
rm -rf /etc/cinder/cinder.conf

cat << EOF >> /etc/cinder/cinder.conf
[DEFAULT]
my_ip = 10.10.34.161
transport_url = rabbit://openstack:Welcome123@10.10.34.161:5672,openstack:Welcome123@10.10.34.162:5672,openstack:Welcome123@10.10.34.163:5672
auth_strategy = keystone
osapi_volume_listen = 10.10.34.161
[backend]
[backend_defaults]
[barbican]
[brcd_fabric_example]
[cisco_fabric_example]
[coordination]
[cors]
[database]
connection = mysql+pymysql://cinder:Welcome123@10.10.34.170/cinder
[fc-zone-manager]
[healthcheck]
[key_manager]
[keystone_authtoken]
auth_uri = http://10.10.34.170:5000
auth_url = http://10.10.34.170:35357
memcached_servers = 10.10.34.161:11211,10.10.34.162:11211,10.10.34.163:11211
auth_type = password
project_domain_id = default
user_domain_id = default
project_name = service
username = cinder
password = Welcome123
[matchmaker_redis]
[nova]
[oslo_concurrency]
lock_path = /var/lib/cinder/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
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
EOF
```

Phân quyền
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

Restart lại dịch vụ nova api
```
systemctl restart openstack-nova-api.service
```

Enable và start dịch vụ
```
systemctl enable openstack-cinder-api.service openstack-cinder-volume.service openstack-cinder-scheduler.service
systemctl start openstack-cinder-api.service openstack-cinder-volume.service openstack-cinder-scheduler.service
```

Kiểm tra:
```
openstack volume service list

+------------------+-------+------+---------+-------+----------------------------+
| Binary           | Host  | Zone | Status  | State | Updated At                 |
+------------------+-------+------+---------+-------+----------------------------+
| cinder-scheduler | ctl01 | nova | enabled | up    | 2020-11-19T04:07:21.000000 |
+------------------+-------+------+---------+-------+----------------------------+
```

> ## Trên node CTL2
```
cp /etc/cinder/cinder.conf /etc/cinder/cinder.conf.bak 
rm -rf /etc/cinder/cinder.conf

cat << EOF >> /etc/cinder/cinder.conf
[DEFAULT]
my_ip = 10.10.34.162
transport_url = rabbit://openstack:Welcome123@10.10.34.161:5672,openstack:Welcome123@10.10.34.162:5672,openstack:Welcome123@10.10.34.163:5672
auth_strategy = keystone
osapi_volume_listen = 10.10.34.162
[backend]
[backend_defaults]
[barbican]
[brcd_fabric_example]
[cisco_fabric_example]
[coordination]
[cors]
[database]
connection = mysql+pymysql://cinder:Welcome123@10.10.34.170/cinder
[fc-zone-manager]
[healthcheck]
[key_manager]
[keystone_authtoken]
auth_uri = http://10.10.34.170:5000
auth_url = http://10.10.34.170:35357
memcached_servers = 10.10.34.161:11211,10.10.34.162:11211,10.10.34.163:11211
auth_type = password
project_domain_id = default
user_domain_id = default
project_name = service
username = cinder
password = Welcome123
[matchmaker_redis]
[nova]
[oslo_concurrency]
lock_path = /var/lib/cinder/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
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
EOF
```

Phân quyền
```
chown root:cinder /etc/cinder/cinder.conf
```

Chỉnh sửa file `/etc/nova/nova.conf`
```
[cinder]
os_region_name = RegionOne
```

Restart lại dịch vụ nova api
```
systemctl restart openstack-nova-api.service
```

Enable và start dịch vụ
```
systemctl enable openstack-cinder-api.service openstack-cinder-volume.service openstack-cinder-scheduler.service
systemctl start openstack-cinder-api.service openstack-cinder-volume.service openstack-cinder-scheduler.service
```

Kiểm tra:
```
openstack volume service list

+------------------+-------+------+---------+-------+----------------------------+
| Binary           | Host  | Zone | Status  | State | Updated At                 |
+------------------+-------+------+---------+-------+----------------------------+
| cinder-scheduler | ctl01 | nova | enabled | up    | 2020-11-19T04:12:01.000000 |
| cinder-scheduler | ctl02 | nova | enabled | up    | 2020-11-19T04:11:38.000000 |
+------------------+-------+------+---------+-------+----------------------------+
```

> ## Trên node CTL3
```
cp /etc/cinder/cinder.conf /etc/cinder/cinder.conf.bak 
rm -rf /etc/cinder/cinder.conf

cat << EOF >> /etc/cinder/cinder.conf
[DEFAULT]
my_ip = 10.10.34.163
transport_url = rabbit://openstack:Welcome123@10.10.34.161:5672,openstack:Welcome123@10.10.34.162:5672,openstack:Welcome123@10.10.34.163:5672
auth_strategy = keystone
osapi_volume_listen = 10.10.34.163
[backend]
[backend_defaults]
[barbican]
[brcd_fabric_example]
[cisco_fabric_example]
[coordination]
[cors]
[database]
connection = mysql+pymysql://cinder:Welcome123@10.10.34.170/cinder
[fc-zone-manager]
[healthcheck]
[key_manager]
[keystone_authtoken]
auth_uri = http://10.10.34.170:5000
auth_url = http://10.10.34.170:35357
memcached_servers = 10.10.34.161:11211,10.10.34.162:11211,10.10.34.163:11211
auth_type = password
project_domain_id = default
user_domain_id = default
project_name = service
username = cinder
password = Welcome123
[matchmaker_redis]
[nova]
[oslo_concurrency]
lock_path = /var/lib/cinder/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
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
EOF
```

Phân quyền
```
chown root:cinder /etc/cinder/cinder.conf
```

Chỉnh sửa file `/etc/nova/nova.conf`
```
[cinder]
os_region_name = RegionOne
```

Restart lại dịch vụ nova api
```
systemctl restart openstack-nova-api.service
```

Enable và start dịch vụ
```
systemctl enable openstack-cinder-api.service openstack-cinder-volume.service openstack-cinder-scheduler.service
systemctl start openstack-cinder-api.service openstack-cinder-volume.service openstack-cinder-scheduler.service
```

Kiểm tra:
```
openstack volume service list
+------------------+-------+------+---------+-------+----------------------------+
| Binary           | Host  | Zone | Status  | State | Updated At                 |
+------------------+-------+------+---------+-------+----------------------------+
| cinder-scheduler | ctl01 | nova | enabled | up    | 2020-11-19T04:15:02.000000 |
| cinder-scheduler | ctl02 | nova | enabled | up    | 2020-11-19T04:14:59.000000 |
| cinder-scheduler | ctl03 | nova | enabled | up    | 2020-11-19T04:14:50.000000 |
+------------------+-------+------+---------+-------+----------------------------+
```

## Kiểm tra
Test theo test case sau:

- Bật cinder tại CTL 1, tắt cinder trên CTL 2 3, lấy list service
- Bật cinder tại CTL 2, tắt cinder trên CTL 1 3, lấy list service
- Bật cinder tại CTL 3, tắt cinder trên CTL 1 2, lấy list service

Sau khi cấu hình Cinder tại CTL 1 2 3
```
systemctl stop openstack-cinder-api.service openstack-cinder-volume.service openstack-cinder-scheduler.service
openstack volume service list
```

Kiểm tra đồng thời HAProxy Stats: http://10.10.34.170:8080/stats

# Phần 13: Tích hợp OPS HA-CEPH
[Tích hợp OPS HA với CEPH](./ceph-ops-ha.md)
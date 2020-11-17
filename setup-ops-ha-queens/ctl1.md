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


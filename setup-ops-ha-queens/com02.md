# Cấu hình trên Compute02

# Phần 1: Chuẩn bị
Set hostname :
```
hostnamectl set-hostname com02
bash
```

Cấu hình network:
```
nmcli con mod eth0 ipv4.address 10.10.34.165/24
nmcli con mod eth0 ipv4.gateway 10.10.34.1
nmcli con mod eth0 ipv4.dns 8.8.8.8
nmcli con mod eth0 ipv4.method manual
nmcli con mod eth0 connection.autoconnect yes

nmcli con mod eth1 ipv4.address 10.10.31.165/24
nmcli con mod eth1 ipv4.method manual
nmcli con mod eth1 connection.autoconnect yes

nmcli con mod eth2 ipv4.address 10.10.32.165/24
nmcli con mod eth2 ipv4.method manual
nmcli con mod eth2 connection.autoconnect yes

nmcli con mod eth3 ipv4.address 10.10.33.165/24
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

Cài đặt CMD-log
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

> ## Snapshot `env`

# Phần 2: Cài đặt các gói cần thiết
```
yum -y install centos-release-openstack-queens
yum -y install crudini wget vim
yum -y install python-openstackclient openstack-selinux python2-PyMySQL
```

# Phần 3: Cài đặt Nova
## Cài đặt gói
```
yum install openstack-nova-compute libvirt-client -y
```

## Cấu hình nova
```
cp /etc/nova/nova.conf  /etc/nova/nova.conf.org
rm -rf /etc/nova/nova.conf

cat << EOF >> /etc/nova/nova.conf 
[DEFAULT]
enabled_apis = osapi_compute,metadata
transport_url = rabbit://openstack:Welcome123@10.10.34.161:5672,openstack:Welcome123@10.10.34.162:5672,openstack:Welcome123@10.10.34.163:5672
my_ip = 10.10.34.165
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
[api]
auth_strategy = keystone
[api_database]
[barbican]
[cache]
[cells]
[cinder]
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
virt_type = qemu
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
server_proxyclient_address = 10.10.34.165
novncproxy_base_url = http://10.10.34.170:6080/vnc_auto.html
[workarounds]
[wsgi]
[xenserver]
[xvp]
EOF
```

Khởi động dịch vụ
```
chown root:nova /etc/nova/nova.conf
systemctl enable libvirtd.service openstack-nova-compute.service
systemctl restart libvirtd.service openstack-nova-compute.service
```


Về CTL 1 thực hiện `openstack compute service list`

Kết quả
```
[root@ctl01 ~]# openstack compute service list
+----+------------------+-------+----------+---------+-------+----------------------------+
| ID | Binary           | Host  | Zone     | Status  | State | Updated At                 |
+----+------------------+-------+----------+---------+-------+----------------------------+
|  1 | nova-consoleauth | ctl01 | internal | enabled | up    | 2020-11-18T10:03:07.000000 |
|  4 | nova-conductor   | ctl01 | internal | enabled | up    | 2020-11-18T10:03:06.000000 |
|  7 | nova-scheduler   | ctl01 | internal | enabled | up    | 2020-11-18T10:03:07.000000 |
| 25 | nova-consoleauth | ctl02 | internal | enabled | up    | 2020-11-18T10:03:01.000000 |
| 28 | nova-scheduler   | ctl02 | internal | enabled | up    | 2020-11-18T10:03:09.000000 |
| 31 | nova-conductor   | ctl02 | internal | enabled | up    | 2020-11-18T10:03:02.000000 |
| 46 | nova-consoleauth | ctl03 | internal | enabled | up    | 2020-11-18T10:03:09.000000 |
| 49 | nova-scheduler   | ctl03 | internal | enabled | up    | 2020-11-18T10:03:01.000000 |
| 52 | nova-conductor   | ctl03 | internal | enabled | up    | 2020-11-18T10:03:10.000000 |
| 68 | nova-compute     | com01 | nova     | enabled | up    | 2020-11-18T10:03:05.000000 |
| 71 | nova-compute     | com02 | nova     | enabled | up    | 2020-11-18T10:03:03.000000 |
+----+------------------+-------+----------+---------+-------+----------------------------+
```

# Phần 4: Cài đặt Neutron
## Cài đặt gói
```
yum install openstack-neutron openstack-neutron-ml2 \
  openstack-neutron-linuxbridge ebtables -y
```

## Cấu hình
```
cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.org 
rm -rf /etc/neutron/neutron.conf

cat << EOF >> /etc/neutron/neutron.conf
[DEFAULT]
transport_url = rabbit://openstack:Welcome123@10.10.34.161:5672,openstack:Welcome123@10.10.34.162:5672,openstack:Welcome123@10.10.34.163:5672
auth_strategy = keystone
[agent]
[cors]
[database]
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
[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
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
[quotas]
[ssl]
EOF
```

## Cấu hình LB Agent
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
local_ip = 10.10.32.165
l2_population = true
EOF
```

## Cấu hình DHCP Agent
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

## Cấu hình metadata agent
```
cp /etc/neutron/metadata_agent.ini /etc/neutron/metadata_agent.ini.org 
rm -rf /etc/neutron/metadata_agent.ini

cat << EOF >> /etc/neutron/metadata_agent.ini
[DEFAULT]
nova_metadata_host = 10.10.34.170
metadata_proxy_shared_secret = Welcome123
[agent]
[cache]
EOF
```

## Thêm vào file `/etc/nova/nova.conf`
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
```

## Phân quyền
```
chown root:neutron /etc/neutron/metadata_agent.ini /etc/neutron/neutron.conf /etc/neutron/dhcp_agent.ini /etc/neutron/plugins/ml2/linuxbridge_agent.ini
```

## Khởi động lại dịch vụ nova-compute
```
systemctl restart libvirtd.service openstack-nova-compute
```

## Khởi động dịch vụ
```
systemctl enable neutron-linuxbridge-agent.service \
neutron-dhcp-agent.service \
neutron-metadata-agent.service

systemctl restart neutron-linuxbridge-agent.service \
neutron-dhcp-agent.service \
neutron-metadata-agent.service
```

## Kiểm tra trên CTL
Kiểm tra các network agent
```
[root@ctl01 ~]# openstack network agent list
+--------------------------------------+--------------------+-------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host  | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+-------+-------------------+-------+-------+---------------------------+
| 0e44056c-f066-4658-81b4-4a81f5ce34b6 | DHCP agent         | com01 | nova              | :-)   | UP    | neutron-dhcp-agent        |
| 14140874-4070-4d94-b67a-f75825dc1b95 | Linux bridge agent | ctl02 | None              | :-)   | UP    | neutron-linuxbridge-agent |
| 39c83d89-070a-4f5c-aeab-cd3bece89427 | Metadata agent     | com02 | None              | :-)   | UP    | neutron-metadata-agent    |
| 46a0c055-aa2f-4cad-b331-1b3b6baf9eab | Linux bridge agent | com01 | None              | :-)   | UP    | neutron-linuxbridge-agent |
| 5ea59807-c0dd-4121-9d9a-de89ae9e7323 | Linux bridge agent | com02 | None              | :-)   | UP    | neutron-linuxbridge-agent |
| 65bb1e14-48ae-4d99-96bb-abafdb660b84 | DHCP agent         | com02 | nova              | :-)   | UP    | neutron-dhcp-agent        |
| 91caaf8c-0bea-473a-9060-101558d98324 | Metadata agent     | com01 | None              | :-)   | UP    | neutron-metadata-agent    |
| 9b1319e5-c9ea-42b6-a83c-c27bab0dc350 | Linux bridge agent | ctl01 | None              | :-)   | UP    | neutron-linuxbridge-agent |
| fb7c381e-9a2b-4ee0-9b50-111bb404fa6c | Linux bridge agent | ctl03 | None              | :-)   | UP    | neutron-linuxbridge-agent |
+--------------------------------------+--------------------+-------+-------------------+-------+-------+---------------------------+
```


# File cấu hình Neutron

## File `/etc/neutron/neutron.conf`
### Section `[DEFAULT]`
- `auth_strategy = keystone` : Loại hình xác thực

- `core_plugin = ml2` : Pluginn cốt lõi mà Neutron sử dụng

- `transport_url = rabbit://openstack:Welcome123@10.10.31.166` : Đường dẫn của Rabbit qeue


### Section `[database]`
- `connection = mysql+pymysql://neutron:Welcome123@10.10.31.166/neutron` : thông tin DB của Neutron

### Section `[keystone_authtoken]`
Thông tin xác thực với Keystone

```conf
[keystone_authtoken]
www_authenticate_uri = http://10.10.31.166:5000
auth_url = http://10.10.31.166:5000
memcached_servers = 10.10.31.166:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = Welcome123
```

### Section `[nova]`
Thông tin Nova

```conf
[nova]
auth_url = http://10.10.31.166:5000
auth_type = password
project_domain_name = Default
user_domain_name = Default
region_name = RegionOne
project_name = service
username = nova
password = Welcome123
```

## File `/etc/neutron/plugins/ml2/ml2_conf.ini`
### Section `[ml2]`
```conf
[ml2]
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
mechanism_drivers = linuxbridge
extension_drivers = port_security
```

- `type_drivers : local,flat,vlan,gre,vxlan,geneve` : các loại driver mạng được sử dụng 
- `tenant_network_types` : Danh sách theo thứ tự các kiểu mạng cho tenant network. 
- `mechanism_drivers` : cơ chế network sử dụng. Có thể là OpenvSwicth hoặc LinuxBridge
- `extension_drivers` : các driver mở rộng thêm

### Section `[ml2_type_flat]`
```conf
[ml2_type_flat]
flat_networks = provider
```

- `flat_networks` : Tên mạng vật lý dùng làm flat network

### Section `[ml2_type_vxlan]`
```conf
[ml2_type_vxlan]
vni_ranges = 1:1000
```

### Section `[securitygroup]`
- `enable_security_group = True` : kích hoạt security group API
- `firewall_driver = None` : driver cho security groups firewall trong l2 agent
- `enable_ipset = True` : Tăng tốc các security groups.

## File `/etc/neutron/plugins/ml2/linuxbridge_agent.ini`
### Section `[linux_bridge]`
- `physical_interface_mappings = <physical_network>:<physical_interface>` : các mạng vật lý có ánh xạ với các interface

### Section `[vxlan]`
- `enable_vxlan = True` : Enable VXLAN

- `local_ip = 10.10.35.166` : Địa chỉ ip của overlay (tunnel) network endpoint.

### Section `[securitygroup]`
- `enable_security_group = True` : kích hoạt security group API

- `firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver` : driver cho security groups firewall trong l2 agent

## File `/etc/neutron/metadata_agent.ini`
Cấu hình trên node Compute:
```conf
[DEFAULT]
nova_metadata_host = 10.10.31.166
metadata_proxy_shared_secret = Welcome123
[cache]
```
- `nova_metadata_host` : Địa chỉ IP hoặc DNS name của Nova metadata
- `metadata_proxy_shared_secret` : Secret key để xác minh. Nó cần khớp với config key `password` của Nova trong section `[neutron]` của Nova

## File `/etc/neutron/dhcp_agent.ini`
Cấu hình trên node Compute
```conf
[DEFAULT]
interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
enable_isolated_metadata = True
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
force_metadata = True
```
- `interface_driver` : Driver được sử dụng để quản lý virtual interface
- `enable_isolated_metadata` : Máy chủ DHCP có thể hỗ trợ cung cấp metadata cho các mạng isolated. Tùy chọn này không có tác dụng nào khi `force_metadata` được đặt giá trị là True
- `dhcp_driver` : Driver sử dụng để quản lý DHCP server
- `force_metadata` : Đặt giá trị này sẽ buộc DHCP server phải kết nối tới các host routes cụ thể vào DHCP request. Nếu tùy chọn này được đặt, thì metadata service sẽ được kích hoạt cho tất cả các mạng.


# Xem thêm
- https://docs.openstack.org/neutron/train/configuration/config.html

- https://docs.openstack.org/neutron/train/configuration/config.html

- https://docs.openstack.org/neutron/train/configuration/ml2-conf.html

- https://docs.openstack.org/neutron/train/configuration/linuxbridge-agent.html
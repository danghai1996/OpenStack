# Các lệnh thường dùng trong Neutron

## Network
### 1. List các network
List network
```
openstack network list
```

List subnet
```
openstack subnet list
```

### 2. Tạo network
```
openstack network create \
--share --provider-physical-network <tên provider nw> \
--provider-network-type <kiểu network> <tên-nw>
```

- `--provider-physical-network` : Tên mạng vật lý mà mạng ảo được triển khai
- `--provider-network-type` : Cơ chế vật lý mà mạng ảo được thực hiện: flat, geneve, gre, local, vlan, vxlan.


Ví dụ: 
```
openstack network create \
> --share --provider-physical-network provider \
> --provider-network-type flat public1
```

### 3. Tạo subnet
Tạo subnet cho network vừa tạo, tại đây ta khai báo dải mạng cấp dhcp và dns, gateway cho network.

**Lưu ý:** đối với **external network** thì dải mạng khai báo phải trùng với dải provider để máy ảo ra ngoài.
```
openstack subnet create --network <tên_mạng_chứa_subnet> \
--allocation-pool start=<IP_bắt đầu>,end=<IP_kết thúc> \
--dns-nameserver <DNS_server> --gateway <IP gateway> \
--subnet-range <địa chỉ mạng> \
<tên_subnet>
```

Ví dụ:
```
openstack subnet create --network public1 \
--allocation-pool start=10.10.32.170,end=10.10.32.180 \
--dns-nameserver 8.8.8.8 --gateway 10.10.32.1 \
--subnet-range 10.10.32.0/24 \
sub1pub1
```


### 4. Show thông tin một mạng
Show thông tin 1 mạng
```
openstack network show <tên_network>
```

Show thông tin 1 subnet
```
openstack subnet show <tên_subnet>
```

## Security group
### 1. List security group
```
openstack security group list
```

### 2. Tạo security group
```
openstack security group create
    [--description <description>]
    [--project <project>]
    [--stateful | --stateless]
    [--project-domain <project-domain>]
    [--tag <tag> | --no-tag]
    <name>
```

Ví dụ:
```
openstack security group create sg-test
```

### 3. Thêm rule vào 1 security group
```
openstack security group rule create
    [--remote-ip <ip-address> | --remote-group <group>]
    [--dst-port <port-range>]
    [--protocol <protocol>]
    [--description <description>]
    [--icmp-type <icmp-type>]
    [--icmp-code <icmp-code>]
    [--ingress | --egress]
    [--ethertype <ethertype>]
    [--project <project>]
    [--project-domain <project-domain>]
    <group>
```

Ví dụ: Cho phép ping tới VM
```
openstack security group rule create \
> --protocol icmp \
> sg-test
```

### 4. Hiển thị rule đang có của 1 security group
```
openstack security group rule list <tên_security_group>
```

Ví dụ:
```
openstack security group rule list sg-test
+--------------------------------------+-------------+-----------+-----------+------------+-----------------------+
| ID                                   | IP Protocol | Ethertype | IP Range  | Port Range | Remote Security Group |
+--------------------------------------+-------------+-----------+-----------+------------+-----------------------+
| 216f052e-c81a-4314-9437-6698daacdbb7 | icmp        | IPv4      | 0.0.0.0/0 |            | None                  |
| 45c5c2f8-ea5f-41a4-8828-3ddf04612861 | None        | IPv6      | ::/0      |            | None                  |
| d099f4fc-8035-43d5-9e59-e24e24f39272 | None        | IPv4      | 0.0.0.0/0 |            | None                  |
+--------------------------------------+-------------+-----------+-----------+------------+-----------------------+
```

### 5. Xóa 1 security group
```
openstack security group delete <group>
```

### 6. Gán security group vào máy ảo
```
nova add-secgroup <tên_máy> <tên_security_group>
```

Ví dụ:
```
nova add-secgroup vm01 sg-test
```

### 7. Xóa secutity group khỏi máy ảo
```
nova remove-secgroup <tên_máy> <tên_security_group>
```

Ví dụ:
```
nova remove-secgroup vm01 default
```

## Cấu hình IP cho VM
List port của 1 VM
```
openstack port list --server <tên_VM>
```
Ví dụ:
```
openstack port list --server vm01
+--------------------------------------+------+-------------------+-----------------------------------------------------------------------------+--------+
| ID                                   | Name | MAC Address       | Fixed IP Addresses                                                          | Status |
+--------------------------------------+------+-------------------+-----------------------------------------------------------------------------+--------+
| ae7c0409-a4b1-4379-be19-3c2c9aeea571 |      | fa:16:3e:0d:a6:aa | ip_address='10.10.32.171', subnet_id='0d1158d3-38e2-4d91-ba1e-38897fb5f238' | ACTIVE |
+--------------------------------------+------+-------------------+-----------------------------------------------------------------------------+--------+
```

### 1. Yêu cầu 1 IP mới cho VM
Ngắt kết nối và xóa port với VM
```
nova interface-detach <tên_instance> <ID_port>
openstack port delete <ID_port>
```

- Ví dụ:
    ```
    nova interface-detach vm01 ae7c0409-a4b1-4379-be19-3c2c9aeea571
    openstack port delete ae7c0409-a4b1-4379-be19-3c2c9aeea571
    ```

Tạo 1 port mới:
```
openstack port create --fixed-ip subnet=<subnet> --network <network> <name_port>
```

- Ví dụ:
    ```
    openstack port create --fixed-ip subnet=sub1pub1 --network public1 port01
    ```

Gán port mới tạo vào VM
```
nova interface-attach --port-id <port_ID> <tên_instance>
```

- Ví dụ:
    ```
    nova interface-attach --port-id 3089b3a0-442f-48d6-bbf9-cfec14885069 vm01
    +------------+--------------------------------------+
    | Property   | Value                                |
    +------------+--------------------------------------+
    | ip_address | 10.10.32.171                         |
    | mac_addr   | fa:16:3e:99:1a:a9                    |
    | net_id     | fa7e2242-d7a9-4619-875b-2f05991a4223 |
    | port_id    | 3089b3a0-442f-48d6-bbf9-cfec14885069 |
    | port_state | DOWN                                 |
    | tag        | -                                    |
    +------------+--------------------------------------+
    ```

Khởi động lại máy ảo
```
openstack server reboot <tên_instance>
```


## Xem thêm:
- https://docs.openstack.org/python-openstackclient/latest/cli/command-objects/security-group.html
- https://docs.openstack.org/python-openstackclient/latest/cli/command-objects/security-group-rule.html
- https://docs.openstack.org/python-openstackclient/latest/cli/command-objects/network.html
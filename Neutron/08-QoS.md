# QoS (Quality of Service) - Neutron

# 1. Giới thiệu
QoS được định nghĩa là khả năng đảm bảo các yêu cấu mạng nhất định như bandwidth, latency (độ trễ), jitter (độ giật) và reliability (độ tin cậy) để đáp ứng thỏa thuận về Service Level Agreement (SLA) giữa các nhà cũng cấp ứng dụng và end users.

Ví dụ đơn giản đó là chúng ta có thể set bandwidth cho từng loại traffic (những traffic như Voice over IP, streaming,... cần được ưu tiên hơn chẳng hạn)

QoS policies có thể được áp dụng:

- Theo từng network: Tất cả các ports được gán vào network nơi có QoS policies sẽ được áp dụng.
- Theo từng port: Các port cụ thể sẽ được áp dụng các policies, khi port đã có policies rồi thì nó sẽ bị ghi đè.

Trong OPS, QoS là một advanced service plug-in, nó được khai báo trong code của neutron và cung cấp thông qua ml2 extension driver.

# 2. QoS trong Neutron
Trong Neutron hiện đang hỗ trợ các rule QoS sau:

- `banwitth_limit`: hỗ trợ giới hạn băng thông tối đa trên từng network, port và IP floating
- `dhcp_marking`: hỗ trợ giới hạn băng thông dựa trên DSCP value. - Với QoS. Marking là 1 task nhỏ trong Classtifycation, (và tất nhiên marking lúc này là DSCP cho Difserv). Classtifycation có 2 task là identify gói tin và marking gói tin, sau đó đẩy vào các queuing, dùng scheduling để quyết định gói nào ra trước, gói nào phải chờ.
- `minimum_bandwidth`: giới hạn băng thông tối đa dựa lên kiểu kết nối.

Bảng dưới đây là một số các backends, QoS rules được hỗ trợ và các hướng đi của trafic (nhìn từ VM)

<img src="..\images\Screenshot_119.png">

# 3. Configure
Để enable service, thực hiện các bước sau:

## 3.1. Trên Network nodes (ở đây là node Controller)
Thêm QoS service vào `service_plugins` trong section `[DEFAULT]` trong file `/etc/neutron/neutron.conf`:
```
[DEFAULT]
service_plugins = router,qos
```

Khai báo driver `qos` trong file `/etc/neutron/plugins/ml2/ml2_conf.ini`:
```
[ml2]
extension_drivers = port_security, qos
```

Sửa file `/etc/neutron/plugins/ml2/<agent_name>_agent.ini`, tại đây là LinuxBridge
`/etc/neutron/plugins/ml2/linuxbridge_agent.ini`
```
[agent]
extensions = qos
```

Nếu Open vSwitch agent được sử dụng, sửa file `/etc/neutron/plugins/ml2/openvswitch_agent.ini`
```
[agent]
extensions = qos
```

Restart service
```
systemctl restart neutron-server.service neutron-linuxbridge-agent.service neutron-dhcp-agent.service neutron-metadata-agent.service
```

## 3.2. Trên Compute node
Sửa file `/etc/neutron/plugins/ml2/<agent_name>_agent.ini`, tại đây là LinuxBridge
`/etc/neutron/plugins/ml2/linuxbridge_agent.ini`
```
[agent]
extensions = qos
```

Restart service
```
systemctl restart neutron-linuxbridge-agent.service neutron-metadata-agent.service neutron-dhcp-agent.service openstack-nova-compute.service
```

## 3.3. Cấu hình `policy.json`


# 4. User workflow
Tạo 1 QoS policy và các rule giới hạn băng thông:
```
openstack network qos policy create bw-limiter
```

Tạo rule limit băng thông. Ta sẽ tạo rule cho cả chiều ra và vào đối với VM:
```
openstack network qos rule create --type bandwidth-limit --max-kbps 3000 \
--max-burst-kbits 300 --egress bw-limiter

openstack network qos rule create --type bandwidth-limit --max-kbps 3000 \
--max-burst-kbits 300 --ingress bw-limiter
```

**Note:** QoS yêu cầu chỉ số burst để chắc chắn sự đúng đắn các các rule set bandwith trên các OpenvSwitch và Linux Bridge. Nếu không set trong quá trình đặt rule thì mặc định chỉ số này sẽ về 80% bandwidth của các gói TCP thông thường. Nếu giá trị burst quá thấp sẽ gây ra việc giảm băng thông so với thông số cấu hình

Kiểm tra lại rule hiện có trong QoS policy vừa tạo:
```
 openstack network qos rule list bw-limiter
+--------------------------------------+--------------------------------------+-----------------+----------+-----------------+----------+-----------+-----------+
| ID                                   | QoS Policy ID                        | Type            | Max Kbps | Max Burst Kbits | Min Kbps | DSCP mark | Direction |
+--------------------------------------+--------------------------------------+-----------------+----------+-----------------+----------+-----------+-----------+
| 23b71377-6b4e-491a-be89-d3014ca85c93 | 6f55c133-a342-4dc2-b752-2e28e75d0df6 | bandwidth_limit |     3000 |             300 |          |           | egress    |
| c2beae05-003b-4646-9864-7aefd542d9b4 | 6f55c133-a342-4dc2-b752-2e28e75d0df6 | bandwidth_limit |     3000 |             300 |          |           | ingress   |
+--------------------------------------+--------------------------------------+-----------------+----------+-----------------+----------+-----------+-----------+
```


List các port hiện có:
```
openstack port list
+--------------------------------------+------+-------------------+------------------------------------------------------------------------------+--------+
| ID                                   | Name | MAC Address       | Fixed IP Addresses                                                           | Status |
+--------------------------------------+------+-------------------+------------------------------------------------------------------------------+--------+
| 1df9b075-c8f4-4892-8061-eb11c0e5f52d |      | fa:16:3e:46:ca:71 | ip_address='192.168.1.100', subnet_id='bff09b33-a595-4897-bc60-b8208d5d2a18' | ACTIVE |
| b26f70d8-58ed-4129-9a33-ec834fdbd256 |      | fa:16:3e:13:c5:26 | ip_address='10.10.32.177', subnet_id='02890e65-8a2c-48f6-bdbb-495ebdb7bb9b'  | ACTIVE |
| d3345c7e-e042-46f6-be31-a1164d8ad215 |      | fa:16:3e:53:b3:fd | ip_address='10.10.32.170', subnet_id='02890e65-8a2c-48f6-bdbb-495ebdb7bb9b'  | ACTIVE |
| ffdf139d-8c25-4985-87ac-163be95cce8a |      | fa:16:3e:b8:8b:0f | ip_address='10.10.32.178', subnet_id='02890e65-8a2c-48f6-bdbb-495ebdb7bb9b'  | ACTIVE |
+--------------------------------------+------+-------------------+------------------------------------------------------------------------------+--------+
```

Gán QoS policy vào port hoặc network cụ thể:
```
openstack port set --qos-policy bw-limiter b26f70d8-58ed-4129-9a33-ec834fdbd256
```

Kiểm tra lại port lại xem đã được gán QoS rule chưa:
```
openstack port show b26f70d8-58ed-4129-9a33-ec834fdbd256
+-------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field                   | Value                                                                                                                                                   |
+-------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+
| ...                                                                                                                                                   |
| qos_policy_id           | 6f55c133-a342-4dc2-b752-2e28e75d0df6                                                                                                                    |
| ...                                                                                                                                                   |
+-------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+
```

Tuy nhiên, việc sử dụng lệnh show port trên chưa chắc chắn là rule còn được áp dụng trên port đó không. Vì khi gán rule mà VM bị reboot sẽ bị mất. Ta cần kiểm tra trên node Compute mà VM chạy:

- Trên node Controller, show port để lấy MAC address của port:
    ```
    openstack port show b26f70d8-58ed-4129-9a33-ec834fdbd256
    ++--------------------------------------------------------------------------------------------------------------------------------------+
    | Field                   | Value                                                                                                       |
    ++--------------------------------------------------------------------------------------------------------------------------------------+
    | mac_address             | fa:16:3e:13:c5:26                                                                                           |
    | ...                                                                                                                                   |
    ++--------------------------------------------------------------------------------------------------------------------------------------+
    ```

- Trên node Compute chạy VM, tìm tap-port của VM với MAC address. Tuy nhiên, cần đổi `fa` -> `fe`. Như ví dụ đây, sẽ là `fe:16:3e:13:c5:26 `
    ```
    ip a | grep -C 2 "fe:16:3e:13:c5:26"
        valid_lft forever preferred_lft forever
    7: tapb26f70d8-58: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc tbf master brqa64987e0-19 state UNKNOWN group default qlen 1000
        link/ether fe:16:3e:13:c5:26 brd ff:ff:ff:ff:ff:ff
        inet6 fe80::fc16:3eff:fe13:c526/64 scope link
        valid_lft forever preferred_lft forever
    ```

- Kiểm tra trên node Compute bằng lệnh:
    ```
    tc qdisc show dev tapb26f70d8-58

    qdisc tbf 8001: root refcnt 2 rate 3072Kbit burst 38400b lat 50.0ms
    qdisc ingress ffff: parent ffff:fff1 ----------------
    ```
    Như vậy là port của VM đã được gán QoS rule thành công.

Để detach port khỏi QoS rule:
```
openstack port unset --no-qos-policy <id port>
```

### Sửa rule
List các rule trong một QoS
```
openstack network qos rule list bw-limiter
+--------------------------------------+--------------------------------------+-----------------+----------+-----------------+----------+-----------+-----------+
| ID                                   | QoS Policy ID                        | Type            | Max Kbps | Max Burst Kbits | Min Kbps | DSCP mark | Direction |
+--------------------------------------+--------------------------------------+-----------------+----------+-----------------+----------+-----------+-----------+
| 23b71377-6b4e-491a-be89-d3014ca85c93 | 6f55c133-a342-4dc2-b752-2e28e75d0df6 | bandwidth_limit |     3000 |             300 |          |           | egress    |
+--------------------------------------+--------------------------------------+-----------------+----------+-----------------+----------+-----------+-----------+
```

Show chi tiết 1 rule:
```
openstack network qos rule show bw-limiter 23b71377-6b4e-491a-be89-d3014ca85c93
+----------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field          | Value                                                                                                                                                   |
+----------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+
| direction      | egress                                                                                                                                                  |
| id             | 23b71377-6b4e-491a-be89-d3014ca85c93                                                                                                                    |
| location       | cloud='', project.domain_id=, project.domain_name='Default', project.id='5b4c1d2155004acf849cd3aac03b8f36', project.name='admin', region_name='', zone= |
| max_burst_kbps | 300                                                                                                                                                     |
| max_kbps       | 3000                                                                                                                                                    |
| name           | None                                                                                                                                                    |
| project_id     |                                                                                                                                                         |
+----------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+
``` 

Sửa rule:
```
openstack network qos rule set --max-kbps 1000 --ingress bw-limiter 23b71377-6b4e-491a-be89-d3014ca85c93

openstack network qos rule set --max-burst-kbit 100 --ingress bw-limiter 23b71377-6b4e-491a-be89-d3014ca85c93
```

Xem lại rule
```
openstack network qos rule show bw-limiter 23b71377-6b4e-491a-be89-d3014ca85c93
+----------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field          | Value                                                                                                                                                   |
+----------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+
| direction      | ingress                                                                                                                                                 |
| id             | 23b71377-6b4e-491a-be89-d3014ca85c93                                                                                                                    |
| location       | cloud='', project.domain_id=, project.domain_name='Default', project.id='5b4c1d2155004acf849cd3aac03b8f36', project.name='admin', region_name='', zone= |
| max_burst_kbps | 100                                                                                                                                                     |
| max_kbps       | 1000                                                                                                                                                    |
| name           | None                                                                                                                                                    |
| project_id     |                                                                                                                                                         |
+----------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+
```

Sau khi sửa, rule sẽ tự động cập nhật trên port đã được gán. Không cần gán lại

### Xóa 1 rule
```
openstack network qos rule delete <QoS-policy> <ID-rule>
```

# 5. Ví dụ
Ta tạo 2 VM như sau:
|VM|IP|OS|
|-|-|-|
|VM1|10.10.32.173|Ubuntu 20.04|
|VM2|10.10.32.176|Ubuntu 20.04|

Tiến hành cài đặt `iperf` trên cả 2 VM:
```
apt-get install -y iperf3
```

Trước khi gán rule QoS:
### **Trên VM1**
```
iperf3 -s
-----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------
Accepted connection from 10.10.32.176, port 49092
[  5] local 10.10.32.173 port 5201 connected to 10.10.32.176 port 49094
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-1.00   sec   162 MBytes  1.36 Gbits/sec
[  5]   1.00-2.00   sec   322 MBytes  2.70 Gbits/sec
[  5]   2.00-3.00   sec   406 MBytes  3.40 Gbits/sec
[  5]   3.00-4.00   sec   432 MBytes  3.62 Gbits/sec
[  5]   4.00-5.00   sec   551 MBytes  4.62 Gbits/sec
[  5]   5.00-6.00   sec   521 MBytes  4.37 Gbits/sec
[  5]   6.00-7.00   sec   549 MBytes  4.60 Gbits/sec
[  5]   7.00-8.00   sec   532 MBytes  4.46 Gbits/sec
[  5]   8.00-9.00   sec   555 MBytes  4.66 Gbits/sec
[  5]   9.00-10.00  sec   547 MBytes  4.59 Gbits/sec
[  5]  10.00-10.00  sec  1.58 MBytes  5.01 Gbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-10.00  sec  4.47 GBytes  3.84 Gbits/sec                  receiver
```

### **Trên VM2**
```
iperf3 -c 10.10.32.173
Connecting to host 10.10.32.173, port 5201
[  5] local 10.10.32.176 port 49094 connected to 10.10.32.173 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec   164 MBytes  1.38 Gbits/sec    3    506 KBytes
[  5]   1.00-2.00   sec   323 MBytes  2.71 Gbits/sec    2    853 KBytes
[  5]   2.00-3.00   sec   405 MBytes  3.40 Gbits/sec    0   1.13 MBytes
[  5]   3.00-4.00   sec   432 MBytes  3.62 Gbits/sec    2   1.37 MBytes
[  5]   4.00-5.00   sec   552 MBytes  4.64 Gbits/sec    0   1.60 MBytes
[  5]   5.00-6.00   sec   520 MBytes  4.36 Gbits/sec    1   1.78 MBytes
[  5]   6.00-7.00   sec   549 MBytes  4.60 Gbits/sec    4   1.95 MBytes
[  5]   7.00-8.00   sec   532 MBytes  4.46 Gbits/sec    3   2.08 MBytes
[  5]   8.00-9.00   sec   555 MBytes  4.66 Gbits/sec    3   2.21 MBytes
[  5]   9.00-10.00  sec   546 MBytes  4.59 Gbits/sec    2   2.32 MBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec  4.47 GBytes  3.84 Gbits/sec   20             sender
[  5]   0.00-10.00  sec  4.47 GBytes  3.84 Gbits/sec                  receiver

iperf Done.
```

Thực hiện gán rule QoS giới hạn băng thông VM1:
```
openstack port set --qos-policy bw-limiter bb3e0086-7064-4745-b773-a583a7b68096
```

Thực hiện lại `iperf`:

### VM1
```
iperf3 -s
-----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------
Accepted connection from 10.10.32.176, port 49096
[  5] local 10.10.32.173 port 5201 connected to 10.10.32.176 port 49098
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-1.00   sec   392 KBytes  3.21 Mbits/sec
[  5]   1.00-2.00   sec   358 KBytes  2.93 Mbits/sec
[  5]   2.00-3.00   sec   359 KBytes  2.94 Mbits/sec
[  5]   3.00-4.00   sec   358 KBytes  2.93 Mbits/sec
[  5]   4.00-5.00   sec   359 KBytes  2.94 Mbits/sec
[  5]   5.00-6.00   sec   359 KBytes  2.94 Mbits/sec
[  5]   6.00-7.00   sec   358 KBytes  2.93 Mbits/sec
[  5]   7.00-8.00   sec   359 KBytes  2.94 Mbits/sec
[  5]   8.00-9.00   sec   359 KBytes  2.94 Mbits/sec
[  5]   9.00-10.00  sec   358 KBytes  2.93 Mbits/sec
[  5]  10.00-10.14  sec  49.5 KBytes  2.87 Mbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-10.14  sec  3.58 MBytes  2.96 Mbits/sec                  receiver
```

### VM2
```
iperf3 -c 10.10.32.173
Connecting to host 10.10.32.173, port 5201
[  5] local 10.10.32.176 port 49098 connected to 10.10.32.173 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec   643 KBytes  5.27 Mbits/sec    0   46.7 KBytes
[  5]   1.00-2.00   sec   382 KBytes  3.13 Mbits/sec    4   41.0 KBytes
[  5]   2.00-3.00   sec   382 KBytes  3.13 Mbits/sec    0   52.3 KBytes
[  5]   3.00-4.00   sec   382 KBytes  3.13 Mbits/sec    2   42.4 KBytes
[  5]   4.00-5.00   sec   255 KBytes  2.08 Mbits/sec    0   46.7 KBytes
[  5]   5.00-6.00   sec   382 KBytes  3.13 Mbits/sec    0   52.3 KBytes
[  5]   6.00-7.00   sec   382 KBytes  3.13 Mbits/sec    2   41.0 KBytes
[  5]   7.00-8.00   sec   382 KBytes  3.13 Mbits/sec    0   52.3 KBytes
[  5]   8.00-9.00   sec   382 KBytes  3.13 Mbits/sec    2   41.0 KBytes
[  5]   9.00-10.00  sec   255 KBytes  2.08 Mbits/sec    0   52.3 KBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec  3.74 MBytes  3.13 Mbits/sec   10             sender
[  5]   0.00-10.14  sec  3.58 MBytes  2.96 Mbits/sec                  receiver

iperf Done.
```

# Tham khảo
- https://docs.openstack.org/neutron/latest/admin/config-qos.html
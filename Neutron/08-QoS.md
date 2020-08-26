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








# Tham khảo
- https://docs.openstack.org/neutron/latest/admin/config-qos.html
- 
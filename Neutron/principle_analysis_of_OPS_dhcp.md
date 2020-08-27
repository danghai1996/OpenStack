# Bài dịch - PRINCIPLE ANALYSIS OF OPENSTACK DHCP

Link bài gốc: http://www.99cloud.net/10262.html%EF%BC%8F

# 1. Mô tả ngắn gọn về nguyên tắc DHCP
**DHCP** : Dynamic Host Configuration Protocol. DHCP server sẽ điều khiển các range IP, và client có thể tự động xin cấp IP từ nó.

## Workflow 
1. Client tạo ra bản tin DISCOVER để yêu cầu cấp phát địa chỉ IP và gửi đi tới các Server. (Do chưa biết chính xác địa chỉ Server cấp IP cho mình nên gói tin sẽ gửi ở dạng Broadcast)

2. Các Server nhận bản tin DISCOVER của Client gửi:

    2.1. Nó sẽ kiểm tra xem địa chỉ IP nào phù hợp để cấp cho Client.

    2.2. Server tạo bản tin OFFER (chứa thông tin về IP và các thông số cấu hình khác mà Client yêu cầu để có thể sử dụng để truy cập Internet)

    2.3. Các Server sẽ gửi bản tin OFFER dưới dạng Broadcast.

3. Client nhận các gói OFFER

    3.1. Client chọn OFFER (có thể là gói tin đầu tiên nhận được , hoặc là gói có chứa IP mà nó đã từng dùng trước đó ). Còn nếu không nhận được gói OFFER nào thì nó sẽ gửi lại gói DISCOVER 1 lần nữa:

    3.2. Tạo gói REQUEST và gửi dưới dạng Broadcast tới tất cả các Server. Nếu nó nhận OFFER từ Server nào thì gói REQUEST gửi về Server đó sẽ mang ý nghĩa đồng ý nhận IP, còn các Server khác thì thông báo là không nhận OFFER đó.

4. Server nhận bản tin REQUEST (Đối với các Server không được nhận OFFER thì sẽ bỏ qua gói tin này)

    4.1. Server xử lí gói tin REQUEST: Kiểm tra xem IP này còn sử dụng được không.

    4.2. Nếu còn sử dụng được thì nó ghi lại thông tin và gửi lại gói tin ACK cho Client. Nếu không thì sẽ gửi lại PNAK để quay lại bước 1.

# 2. Tổng quan về neutron-dhcp-agent
Khi OPS tạo 1 máy ảo, nó sẽ tự động gán địa chỉ IP cho máy ảo thông qua service Neutron DHCP. Thành phần đó là neutron-dhcp-agent service chạy trên các node network.

# 3. File cấu hình
File cấu hình của DHCP service trên node network
```
/etc/neutron/dhcp_agent.ini
```

<img src="..\images\Screenshot_120.png">

- `dhcp_driver` : Mặc định, sử dụng `neutron.agent.linux.dhcp.Dnsmasq`
- `interface_driver` : sử dụng LinuxBridge để quản lỹ các interface của máy ảo

Khi mạng được tạo và dhcp function của subnet tương ứng được bật, Neutron sẽ bắt đầu tiến trình dnsmasq trên node network để cung cấp dhcp service cho mạng.
```
ps aux | grep dnsmasq
```

<img src="..\images\Screenshot_121.png">

Giải thích tham số:
- `--interface` : port được dnsmasq sử dụng để theo dõi các request/response để cung cấp DHCP service
- `--dhcp-hostsfile` : File lưu thông tin DHCP server. DNSmasq sẽ nhận được sự tương ứng giữa port và MAC-address từ file này

# 4. Network namespace
Neutron cung cấp DHCP độc lập cho mỗi mạng thông qua namespace. Cho phép tenant tạo ra các mạng overlapping network

Namespace tương ứng với DHCP được đặt tên kiểu : `qdhcp-network_id`

Có thể xem qua lệnh `ip netns` (trên node network). 

<img src="..\images\Screenshot_122.png">

# 5. OpenStack DHCP obtain ip process analysis
Khi khởi tạo VM1, Neutron sẽ cấp phát 1 port cho nó và đồng bộ hóa thông tin địa chỉ MAC và thông tin IP với file host của dnsmasq, như trong file dưới đây:
```
cat /var/lib/neutron/dhcp/a64987e0-1992-4a56-bf2c-b68d7755f9a3/host

fa:16:3e:53:b3:fd,host-10-10-32-170.openstacklocal,10.10.32.170
fa:16:3e:b8:8b:0f,host-10-10-32-178.openstacklocal,10.10.32.178
fa:16:3e:35:1d:59,host-10-10-32-176.openstacklocal,10.10.32.176
fa:16:3e:dc:34:65,host-10-10-32-173.openstacklocal,10.10.32.173
```

Đồng thời, nova-compute cũng tạo 1 file XML của instance có phần network như sau:
```
virsh edit instance-00000002

<interface type='bridge'>
    <mac address='fa:16:3e:dc:34:65'/>
    <source bridge='brqa64987e0-19'/>
    <target dev='tapbb3e0086-70'/>
    <model type='virtio'/>
    <mtu size='1500'/>
    <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
</interface>
```

## Khởi động máy ảo lần đầu tiên
1. Khi máy ảo vm1 khởi động lần đầu tiên, nó sẽ gửi 1 message dhcpdiscover broadcast, message sẽ được nhận trong toàn bộ mạng (vlan)

2. dhcpdiscover broadcast message sẽ được gửi tới tại tap2c6747c1-2a và dnsmasq sẽ listen nó. dnsmasq sẽ kiểm tra file host của mạng tương ứng và tìm các tùy chọn tương ứng, từ đó, dnsmasq sẽ dùng dhcpoffer message để set IP, subnetmask, ... Thông in sẽ được gửi tới vm1

3. vm1 sẽ gửi 1 dhcprequest broadcast message để xác nhận việc chấp nhận IP từ dhcpoffer message

4. dnsmasq xác nhận dhcpack message được gửi bởi dhcprequest message của vm1. SAu khi nhận được dhcpack message, vm1 bắt đầu sử dụng IP được nhận và toàn bộ quá trình kết thúc.

5. vm1 khởi tạo arp broadcast để lấy địa chỉ gateway MAC

6. Tương tác với địa chỉ nội bộ 169.254.169.254

7. vm1 sẽ gửi icmp đến gateway. Sau khi xác nhận, toàn bộ quá trình kết thúc


## Quá trình khởi động lại máy ảo
1. Khi khởi động lại, vm1 sẽ sử dụng địa chỉ cuối cùng để gửi direct broadcast dhcqrequest message yêu cầu tiếp tục sử dụng địa IP. Sau khi dnsmasq xác nhận, nó sẽ gửi dhcpack message để xác nhận, và vm1 sẽ nhận dhcqack message để tiếp tục sử dụng IP

2. vm1 khởi tạo 1 arp broadcast để lấy gateway MAC

3. tương tác với địa chỉ nội bộ 169.254.169.254

4. vm1 sẽ gửi icmp đến gateway. Sau khi xác nhận, toàn bộ quá trình kết thúc

# 6. Các tình huống nhiều subnet trong mạng
1. Khi có nhiều subnet trong mạng và dhcp được bật, Neutron sẽ cấu hình nhiều IP tương ứng trên tap device (network: dhcp port) trong namespace của mạng.

    <img src="..\images\Screenshot_125.png">

2. Đồng thời, cập nhật file host dhcp thông qua neutron database (2 IP được cấu hình cùng địa chỉ MAC)

    <img src="..\images\Screenshot_124.png">

3. Cuối cùng, khởi động lại dnsmasq process trên node network, và update `–dhcp-range` và `–dhcp-hostsfile` để làm cho quá trình hoạt động trên nhiều dài dhcp để đạt được 1 network và nhiều subnet dhcp service.

# 7. VM được truy cập thông qua tên miền
dnsmasq là 1 phần mềm nguồn mở hỗ trợ chức năng DHCP và DNS. Neutron hỗ trợ chức năng phan giải tên miền của các máy ảo trong cùng mạng bằng cách gọi dnsmasq

1. Khi tạo vm1, Neutron sẽ cấp phát 1 port cho nó và đồng bộ domain name và thông tin IP đến file host của dnsmasq
    ```
    cat /var/lib/neutron/dhcp/a64987e0-1992-4a56-bf2c-b68d7755f9a3/host
    ```

    <img src="..\images\Screenshot_123.png">

2. Tên miền tương ứng của `10.10.32.176` là `host-10-10-32-176.openstacklocal`.

3. Sau khi VM được bật, Neutron sẽ cập nhật `/etc/resolv.conf` của VM và sử dụng IP của dhcp port nhưu là DNS address (dnsmasq sẽ lắng nghe từ địa chỉ này) để thực hiện chức năng dns. 

# 8. Phân tích triển khai DHCP HA
**Tiền phân tích:**

Sau khi triển khai dhcp agent hoàn tất, dhcp đã chạy trên node network và state đã ok

1. Tạo 1 network. Neutron sẽ tạo 1 namespace và tap device tương ứng trên các node mạng

2. Trên Horizon

    <img src="..\images\Screenshot_126.png">

3. Khi tạo máy ảo, Neutron sẽ assign 1 port cho nó. Trong kịch bản HA, Thông tinn địa chỉ MAC và IP sẽ được cập nhật và file host của các node network. Nói cách khác, các file host sử dụng các node network như nhau.

4. Khi máy ảo khởi động, nó sẽ gửi message broadcast dhcpdiscover và tất cả các tab device trên node network sẽ nhận được message, vì vậy tất cả dnsmasq sẽ gửi lại response dhcpoffer

5. Máy ảo phản hổi message dhcpoffer đầu tiên nhận được bằng broadcast. dhcprequest message sẽ chứa IP của dnsmasq, vì vậy chỉ có dnsmasq tương ứng sẽ gửi message dhcpack để xác nhận, và các dnsmasq khác sẽ khoog xử lý message sau khi nhận được.

6. Sau khi nhận được message dhcpack xác nhận, máy ảo sẽ kiểm tra IP có thể sử dụng hay không. Nếu có, nó sẽ cập nhật thông tin và bắt đầu sử dụng IP
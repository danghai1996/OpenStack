# Bài dịch - LOW-LEVEL NETWORK IMPLEMENTATION MODEL

Link bài gốc: http://www.99cloud.net/10741.html%EF%BC%8F

# 1. Lời nói đầu
Một số người nói rằng Neutron khó học, và nếu tôi không tin vào ma quỷ, tôi phải xuyên thủng phần Neutron này.

Loạt bài này sẽ giới thiệu kiến trúc triển khai của Neutron, mô hình triền khai mạng, mô hình tài nguyên cấp trên, hỗ trợ kỹ thuật cơ bản, mục đích thiết kế và các trường hợp thực tế nói chung. Mục đích là để nắm bắt tình hình tổng thể của Neutron. Bài viết có tham khả và trích dẫn số lượng lớn tài liệu, sách báo, có link ở cuối bài.

# 2. Sự phát triển từ mạng truyền thống (TRADITIONAL NETWORK) lên mạng ảo (VIRTUALIZED NETWORK)
**Mạng truyền thống**

<img src="..\images\Screenshot_127.png">

**Virtual network**

<img src="..\images\Screenshot_128.png">

**Distributed virtual network** (Mạng ảo phân tán)

<img src="..\images\Screenshot_129.png">

# 3. Sự phát triển từ Single plane network tới Hybrid plane network
### Single-plane tenant sharing network
Tất cả tenants chia sẻ 1 mạng (IP address pool) và chỉ có thể có 1 loại mạng duy nhất (VLAN hoặc Flat)

- Không có private network (No private network)
- Không cô lập tenant (No tenant isolation)
- Không có định tuyến Layer 3 (No Layer 3 routing)

<img src="..\images\Screenshot_130.png">

### Multi-plane tenant shared network
Có nhiểu mạng chia sẻ (Shared networks) để tenants lựa chọn

- Không có private network (No private network)
- Không cô lập tenant (No tenant isolation)
- Không có định tuyến Layer 3 (No Layer 3 routing)

<img src="..\images\Screenshot_131.png">

### Mixed plane (shared/private) network
Sự kết hợp của share network và tenant private network.

- Có private network
- Không cô lập tenant (No tenant isolation)
- Không có định tuyến Layer 3 (No Layer 3 routing)

**Chú ý:** Vì nhiều tenant vẫn dựa vào share network (ví dụ: cần truy cập external network) nên tenant không đạt được sự cô lập hoàn toàn

<img src="..\images\Screenshot_132.png">

### Multi-plane tenant private network based on the operator's routing function (Các private network của tenant dựa trên chức năng định tuyến)
Mỗi tenant đều có private network của riêng mình, và thông tin liên lạc layer 3 giữa các network thông qua bộ router(public)

- Có private network
- Cô lập tenant(Tenant isolation)
- Chia sẻ định tuyến Layer3 (Shared Layer 3 routing)

<img src="..\images\Screenshot_133.png">

### Multi-plane tenant private multi-network based on private routers (Các tenant có nhiều private network dựa trên bộ định tuyến riêng)
Mỗi tenant có thể có vô số private network và router. Tenant có thể sử dụng các router private để có được thông tin liên lạc Layer 3 giữa các private network

- Có private network
- Cô lập tenant (Tenant isolation)
- Có định tuyến Layer 3 (There are three layers of routing)

<img src="..\images\Screenshot_134.png">

# 4. Sơ lược về Neutron
Neutron trong OPS là 1 project cung cấp "Network connectivity as a service" giữa các interface device (ví dụ: vNICs) được quản lý bởi các service trong OPS (ví dụ: Nova). OPS Networking service (Neutron) cung cấp 1 API cho phép người dùng thiết lập và xác định kết nối mạng và địa chỉ trong cloud.

Project code-name cho dịch vụ Networking là **Neutron**.

OPS networking xử lý việc tạo và quản lý cơ sở hạ tầng virtual network, bao gồm network, switches, subnet, router cho thiết bị quản lý bởi Nova. Các dịch vụ nâng cao như firewall hay VPN cũng có thể được sử dụng.

Neutron là 1 project cung cấp Network Connectivity as a Service cho OPS. Tất nhiên, Neutron có thể được tách ra khỏi hệ thống Keystone như 1 phần mềm trung gian SDN độc lập.

- **As a OPS component:** Cung cấp network service cho máy ảo OPS, bare metak và container

- **As SDN middleware:** Northbound (Hướng bắc) cung cấp tài nguyên interface trừu tượng, thống nhất (unified abstract resource interface). Và Southbound (hướng nam) với bộ điều khiển SDN không đồng nhất.

**Mục tiêu Neutron theo đuổi:** the distributed virtualized network in the cloud computing era and the multi-tenant isolated private multi-plane network.

Từ góc độ triển khai phần mềm, không khó để các nhà phát triển quen thuộc với lưu ý của Neutron: Trước tiên, Neutron team phải thiết kế mô hình Core API Model (Mô hình tài nguyên cốt lõi) và sau đó triển khai chúng. Đây cũng là phong cách triển khai của nhiều dự án OPS và rất đáng học hỏi. Khi thiết kế kiến trúc của phần mềm, đừng vọi code, mà trước tiên hay suy nghĩ thật rõ ràng về ý nghĩa của Service đó, **Service đó là gì, cho đối tượng nào, cung cấp dịch vụ gì và cung cấp nó như thế nào?** Bản chất của Network as a service cung cấp bởi Neutron là các tenant chỉ cần quan tâm đến service, không cần phải chi tiết triển khai và nội dung của service là định nghĩa của loại tài nguyên (API). Chỉ ở bản ổn định, tương thích và chuyển đổi ổn định của các API có thể kích hoạt APIs economy (1 hệ sinh thái xung quanh các open APIs), cho phép các phần mềm tồn tại trong thị trường open source khốc liệt.

Hơn nữa, ý định ban đầu của người thiết kế Neutron là định nghĩa phần mềm thuần túy, không cần thiết kế phần cứng mới, nhưng có thể phù hợp với các phần cứng hiện có. Đây là 1 ý tưởng thiết kế thực dụng. Các phần tử mạng physical/virtual network bên dưới được gọi thông qua Plugin-driver. Chức năng cung cấp sự hỗ trợ cho các service cấp trên. Đặc điểm này giúp Neutron nhận được sự quan tâm rộng rãi trong ngành.

# 5. Mô hình triển khai mạng Neutron
Trong quá trình tìm hiểu mô hình triển khai mạng Neutron, chúng tôi chủ yếu tìm ra ba câu hỏi:

- Làm thế nào để Neutron hỗ trợ nhiều loại network (VLAN, VXLAN) ?
- Làm cách nào để các máy ảo giữa các node compute giao tiếp trong cùng 1 tenant network?
- Cách máy ảo truy cập vào mạng external ?

### Tổng quan về mô hình triển khai mạng của Neutron
<img src="..\images\Screenshot_135.png">

### Mô hình triển khai Computing node Network
<img src="..\images\Screenshot_136.png">

Dưới đây, chúng tôi trực tiếp giới thiệu nếu VM traffic trên node compute được gửi ra khỏi node compute, và chén thiết bị mạng ảo liên quan trong quá trình này để giới thiệu cơ chế làm việc.

**Bước 1:** Lưu lượng được chuyến đến card mạng ảo vNIC thông qua TCP/IP stack kernel của VM để xử lý. vNIC là tap device cho phép các user mode program (GuestOS, qemu process) đưa dữ liệu vào kernel TCP/IP protocol stack. Tap device có thể chạy trên GuestOS và cung cấp các chức năng chính xác nhu physical NIC.

**Bước 2:** vNIC (tap device) của máy ảo không được kết nối trực tiếp với OvS Bridge, nhưng được chuyển tiếp đến OvS br-int (bridge tích hợp) thông qua Linux Bridge. Tại sao vNIC không kết nối trực tiếp tới br-int ? Điều này là do OvS chỉ có các quy tắc tường lửa tĩnh trước v2.5 và không hỗ trợ stateful firewal rules. Các rule này là sự hỗ trợ cơ bản của Neutron Security Group, cung cấp port (vNIC)-level security cho các máy ảo. Do đó, Linux Bridge được giới thiệu nhu 1 lớp bảo mật và iptables được sử dụng để lọc các stateful packet. Trong số đó, Linux Bridge `qbr` là tên viết tắt của quantum bridge

**Bước 3:** Linux Bridge và OvS được kết nối thông qua veth pair (cáp mạng ảo). 1 đầu của cáp được đặt tên là `qvb` (quantum veth bridge) và đầu còn lại là `qvo` (quantum veth ovs). Veth pair device luôn tồn tại thành từng cặp và được sử dụng để kết nối 2 thiết bị mạng ảo (virtual network device) nhằm mô phỏng quá trình truyền và nhận dữ liệu giữa các thiết bị ảo (virtual devices). Nguyên lý hoạt động của veth pair là đảo ngược hướng của dữ liệu liên lạc. Dữ liệu cần gửi sẽ được chuyển đổi thành dữ liệu cần nhận và sau đó được gửi tới kernel TCP/IP protocol stack để xử lý, và cuối cùng được chuyển hướng đến target device (network cable ở 1 phía khác)

**Bước 4:** OvS br-int là 1 network bridge tích hợp, như thiết bị local virtual switch của node compute, để hoàn thành việc xử lý lưu lượng máy ảo cục bộ (virtual machine traffic locally). Teanant flow được phân tách bằng VLAN IP local. (local VLAN của IP tenant network theo phân phối nội bộ (internal distribution))

- Lưu lượng do máy ảo gửi đi được gắn tag Local VLAN trong br-int, và lưu lượng truyền đến máy ảo sẽ bị xóa với Local VLAN tag trong br-int
- Layer 2 traffic giữa các máy ảo cục bộ được chuyển tiếp trực tiếp  trên br-int và nhận bởi port với cùng VLAN tag

**Bước 5:** Lưu lượng được truyền từ local VM đến remote VM (hoặc gateway) được gửi tới OvS `br-ethX`thông qua port `int-br-eth1` (thiết bị ngang hàng với OvS) trên OvS `br-int`. Cần lưu ý rằng br-ethX không phải static: khi network type là Flat, VLAN hãy sử dụng `br-ethX`; khi network type là Tunnel (VxLAN, GRE), `br-tun` sẽ thay thế `br-EthX`. Ví dụ trên là sử dụng VLAN network. Chúng tôi gọi chung `br-ethX` và `br-tun` như 1 tenant network layer bridge (TNB) để dễ mô tả

**Chú ý:** 
- qbr-xxx và OvS br-int được kết nối bằng veth pair device, `br-int` và `br-ethX/br-tun` được kết nối bởi 1 thiết bị vá lỗi ngang hàng (patch peer device). Cả 2 đều là virtual network cables, sự khác biệt là trước đây là 1 Linux virtual network device, còn sau này là 1 virtual network device được thực hiện bởi OvS
- Nếu tenant bridge là `br-tun` thay vì `br-ethX`, thì sẽ có port: `patch-int` trên `br-tun`, port:`patch-tun` trên `br-int`, thông qua `patch-int` và `patch-tun` nhận ra giao tiếp giữa tenant bridge `br-tun` và local bridge `br-int`

**Bước 6:** Khi lưu lượng máy ảo máy ảo chuyển giữa OvS `br-int` và OvS `br-ethX`, 1 hành động quan trọng và rất phức tạp là required-internal (yêu cầu nội bộ) và chuyển đổi VID bên ngoài. Đây là design concept tương thích nhiều lớp (layered compatibility), nhắc nhở tôi: tất cả các vấn đề máy tính có thể được giải quyết bằng cách đưa vào 1 lớp trung gian. Vấn đề của Neutron là nó hỗ trợ nhiểu network type (Flat, VLAN, VxLAN, GRE)

**Bước 7:** Cuối cùng, lưu lượng máy ảo dược gửi đến physical network thông qua physical card `ethX` được gắn trên TNB(br-ethX / br-tun). Mối liên hệ giữa OvS `br-int` và OvS `br-ethX` cũng được thực hiện qua veth pair.

**Bước 8:** Sau khi lưu lượng của máy ảo vào mạng vật lý, những gì còn lại là traditional network. Các network packet được chuyển tiếp dới các node network và compute khác bằng switch hoặc các thiết bị bridge khác (Điều này phụ thuộc vào cấu trúc liên kết mạng triển khai cụ thể, nói chung lưu lượng máy ảo sẽ chỉ được nhận bởi các node compute trong cùng 1 mạng)

# 6. VID CONVERSION INSIDE AND OUTSIDE
Từ quan điểm của lớp network, mô hình network của node compute có thể được chia thành: tenant network layer và local network layer.

Local network layer phân chia các máy ảo (port) trong các mạng khác nhau trong `br-int` theo VLAN ID. Local network chỉ hỗ trợ kiểu VLAN; trong khi tenant network layer cần hỗ trợ loại mạng non-tunnel kiểu network là Flat để thực hiện các yêu cầu của multi-type hybrid plane, VLAN (`br-ethX`) và tunnel network VxLAN, GRE (`br-tun`). Rõ ràng là phải có 1 lớp chuyển đổi giữa local network layer và tenant network layer. Đây là cái gọi là chuyển đổi VID.

VID là 1 khái niệm logic và có nhiểu loại VID khác nhau cho các tenant network type khác nhau.

<img src="..\images\Screenshot_137.png">

Cần lưu ý rằng range của VID được xác định bởi tenant - Tenant flows được phân tách bởi người dùng VLAN ID. (VID của tenant network được xác định bởi **User defined**, tại ví dụ này là VLAN). Cái gọi là **User defined** là range được cấu hình bởi người dùng cho các type network thông qua file cấu hình Neutron `/etc/neutron/plugins/ml2/ml2_conf.ini`

```
[ml2_type_vlan]
network_vlan_ranges = public:3001:4000

[ml2_type_vxlan]
vni_ranges = 1:1000

[ml2_type_gre]
tunnel_id_ranges = 1:1000
```

**Một lần nữa, VLAN ID của local network được chỉ định bởi thuật toán logic , trong khi VID của tenant network được tùy chỉnh bởi người dùng.**

Bạn có thể tự hỏi tại sao cần thực hiện chuyển đổi VID nội bộ và VID external khi tenant network và local network có cùng loại VLAN?

Để trả lời câu hỏi này, bạn chỉ cần suy nghĩ về nó 1 cách biện chứng: điều gì sẽ xảy ra nếu bạn tham gia mà không có sự chuyển đổi của VID nội bộ và external VID? Câu trả lời là sẽ có conflict giữa VID nội bộ và external trong kịch bản của tenant network với sự kết hợp của VLAN và VxLAN


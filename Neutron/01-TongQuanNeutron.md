# Tổng quan về Neutron - Openstack Networking

## 1. Giới thiệu chung về Neutron
OpenStack Networking cho phép bạn tạo và quản lí các network objects ví dụ như networks, subnets, và ports cho các services khác của OpenStack sử dụng. Với kiến trúc plugable, các plug-in có thể được sử dụng để triển khai các thiết bị và phần mềm khác nhau, nó khiến OpenStack có tính linh hoạt trong kiến trúc và triển khai.

Dịch vụ Networking trong OpenStack (neutron) cũng cấp API cho phép bạn định nghĩa các kết nối mạng và gán địa chỉ ở trong môi trường cloud. Nó cũng cho phép các nhà khai thác vận hành các công nghệ networking khác nhau cho phù hợp với mô hình điện toán đám mây của riêng họ. Neutron cũng cung cấp một API cho việc cấu hình cũng như quản lí các dịch vụ networking khác nhau từ L3 forwarding, NAT cho tới load balancing, perimeter firewalls, và virtual private networks.

Openstack là mô hình multitenancy. Tức mỗi tenant có thể tạo riêng nhiều private network, router, firewall, loadbalancer… Neutron có khả năng tách biệt các tài nguyên mạng giữa các tenant bằng giải pháp linux namespace. Mỗi network namespace riêng cho phép tạo các route, firewall rule, interface device riêng. Mỗi network hay router do tenant tạo ra đều hiện hữu dưới dạng 1 network namespace, từ đó các tenant có thể tạo các network trùng nhau (overlapping) nhưng vẫn độc lập mà không bị xung đột (isolated)

### Các thành phần của Neutron:
#### **API server:**

# Namespace trong Neutron

## 1. Overlapping network trong OPS
- Overlapping được hiểu là một mạng máy tính được xây dựng trên một nền tảng network mạng sẵn
- Openstack cung cấp môi trường Multi Tenant . Mỗi tenant cung cấp một mạng prviate , router, firewall , loadblancer riêng . - Nhờ namepsace cung cấp khả năng tách biết các tài nguyên mạng giữa các tenant - network namespace
Để xem được namepsace có thể sử dụng

    ```
    ip netns list
    ```
    Nếu bạn chưa thêm bất kỳ namespace nào, output sẽ trống. Namespace mặc định thì không được tính vào output của lệnh `ip netns list`

- Các namespace hiển thị dưới dạng
    - `qdhcp-*`
    - `qrouter-*`
    - `qlbaas-*`

## 2. Linux network namespaces
Trong network namespace, scoped `identifiers` là các thiết bị mạng, ví dụ như : `eth0`, tồn tại trong một namespace cụ thể

Linux khởi động với namespace mặc định, vì vậy nếu hệ điều hành của bạn không hoạt động gì đặc biệt, thì đó là vị trí của các thiết bị mạng (network devices). Nhưng cũng có thể tạo thêm các namespace khác với namespace mặc định và tạo các thiết bị mạng mới trong namespace đó, hoặc move một thiết bị hiện có từ name space này sang namespace khác.

Mỗi namespace đều có routing table (bảng định tuyến) và thực tế đây là lý do chính để namespace tồn tại. Bảng định tuyến được khóa bới IP đích. Vì vậy, network namespace là những gì bạn cần nếu bạn muốn cùng 1 IP đích có ý nghĩa khác nhau vào những thời điểm khác nhau

Mỗi namespace đều có bộ iptables riêng (cho cả IPv4 vag IPv6)
. Vì vậy, bạn có thể áp dụng các security khác nhau cho các luồng có cùng IP trong các namespace khác nhau, cũng như định tuyến khác nhau

Bất kỳ process của Linux nào đều chạy trong một network namespace cụ thể. Theo mặc định, process này được kế thừa từ parent process nhưng một process với các khả năng phù hợp có thể tự chuyển đổi sang một namespace khác. Trong thực tế, điều này chủ yếu được thực hiện bằng cách sử dụng lệnh `ip netns exec NETNS COMMAND...`, bắt đầu chạy lệnh trong namespace `NETNS`. Giả sử một process như vậy gửi một message tới địa chỉ IP `A.B.C.D`, ảnh hưởng của namespace A.B.C.D sẽ được tra cứu trong bảng định tuyến của namespace đó và điều đó sẽ xác định thiết bị mạng mà message được truyền qua.


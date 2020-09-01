# Multi Region

# Khái niệm Multi Region trong OPS
## Tổng quan
Trong Openstack có rất nhiều khái niệm dễ hiểu lầm như Domain, Region, Multi Site. Sau đây mình sẽ làm rõ các khái niệm

## Domain
Một hệ thống Cloud Openstack có thể được sử dụng bởi nhiều TỔ CHỨC, DOANH NGHIỆP, CÁ NHÂN, v.v, đồng thời openstack cũng cho phép tạo project có tên giống nhau.

Ví dụ, Phòng Kỹ Thuật của Nhân Hòa sử dụng project có tên là phong_ky_thuat, Phòng Kỹ Thuật của công ty Cloud365 cũng đặt tên project là phong_ky_thuat, khi đó trên hệ thống openstack sẽ có 2 project "phong_ky_thuat" nhưng lại thuộc sử hữu của 2 công ty khác nhau, điều này sẽ gây lẫn cho người quản trị.

Để hệ thống Openstack trở nên rõ ràng hơn, Keystone bổ sung khái niệm "Domain" nhằm cô lập các project giữa các tổ chức. Tức mỗi doanh nghiệp, tổ chức, cá nhân sẽ có 1 domain riêng, tại domain mỗi tổ chức, họ chỉ có thể thấy các project và user mà họ sở hữu, độc lập với "Domain" khác.

Quay lại ví dụ trên, công ty Nhân hoà sẽ sở hữu domain 'nhanhoa', công ty Cloud365 sẽ sở hữu domain 'cloud365'. Khi đó người quản trị của Nhân Hòa chỉ thấy các project, user thuộc sử hữu domain 'nhanhoa', tương tự với công ty Cloud365

<img src="..\images\Screenshot_40.png">

## Region
Khi hệ thống Cloud Openstack phát triển tới mức độ nhất định, việc triển khai 1 cụm Openstack là không đủ, các doanh nghiệp sẽ tính đến việc triển khai nhiều hệ thống Openstack tại nhiều địa điểm khác nhau phục vụ cho bài toán tối ưu vị trí địa lý.

Ví dụ: Công ty Nhân Hòa đã có một cụm Openstack tại Hà Nội, Công ty quyết định triển khai một cụm Openstack khác tại Đà Nẵng, một cụm Openstack Hồ Chí Minh để cung cấp dịch vụ tốt nhất cho khách hàng tại Hà Nội, Đà Năng, Hồ Chí Minh, đồng thời có yêu cầu 3 cụm Openstack phải sử dụng chung một hệ thống xác thực để phục vụ cho việc quản trị người dùng.

Khái niệm Region hay Multi Region sẽ được áp dụng cho ví dụ trên, khi chúng ta muốn triển khai nhiều cụm openstack khác nhau nhưng lại muốn sử dụng chúng Keystone.

<img src="..\images\Screenshot_142.png">

## Multi Site
Khái niệm Multi Site cũng gần giống với khái niệm Region nhưng khác biệt điểm, chúng ta sẽ có nhiều cụm Openstack nhưng sẽ không chia sẻ dịch vụ Keystone (Mỗi cụm đều chạy độc lập).

# Các mô hình triển khai Multi Regions
Để thực hiện triển khải Multi Region, ta sẽ cần triển khai các cụm OPS kháu nhau và chung hệ thống xác thực là project Keystone.

**Ví dụ:** 3 cụm OPS được đặt tại 3 DC khác nhau: Hà Nội, Đà Nẵng, TP.Hồ Chí Minh

## Centralized Keystone DB
Tức ta sẽ có 1 DB trung tâm của Keystone, các Region sẽ kết nối với Database thông qua đường WAN. (Không sử dụng Keystone DB)

## Asyncronous Keystone DB
Mỗi Region sẽ có 1 DB tuy nhiên sẽ duy trì 1 Master (Cho phép đọc ghi) còn lại sẽ là Slave (Chỉ đọc). Khi đó dữ liệu sẽ đồng bộ giữa các DB tại các Region

## Syncronous (Clustered) Keystone DB: 
Sử dụng MySQL/MariaDB Galera Cluster, Keystone database tại 3 Region sẽ được đồng bộ.

### Ưu nhược điểm các loại
Các tùy chọn sẽ có ưu nhược điểm khác nhau.

||Centralized Keystone DB|Asyncronous Keystone DB|Syncronous (Clustered) Keystone DB|
|-|-|-|-|
|Ưu điểm||||
|Nhược điểm||||
# Cloud Computing

## I. Cloud Computing
Cloud computing là mô hình cung cấp tài nguyên máy tính (mạng, máy chủ, storage, ứng dụng,...) trên internet theo nhu cầu của người dùng.

## II. Đặc điểm của cloud computing (5)
### 1. On-demand self-service
Khách hàng tự phục vụ, chủ động trong việc quản lý dịch vụ: khởi tạo, tạm dừng, gia hạn, ... dịch vụ không cần thông qua nhà cung cấp.

### 2. Broad network access
Khả năng truy cập trên mọi nền tảng thiết bị, mọi nơi qua internet

### 3. Resource pooling
Khách hàng được cung cấp tài nguyên (pool) có thể sử dụng, phân chia tùy vào nhu cầu.

Khả năng gộp-gom tài nguyên vật lý sau đó phân bổ 1 cách tự động cho người dùng theo nhu cầu

Ví dụ: Khách hàng được cấp tài nguyên là 5 Core, 10G Ram, 100G Disk. Khách hàng có thể chia nhỏ sử dụng tùy theo nhu cầu.
    
- Chia đều thành 5 máy : mỗi máy 1 Core, 2G Ram, 20G Disk.
- Sử dụng một phần: Một máy: 2 Core, 5G Ram, 80G Disk. Phần còn lại : 3 Core, 5G Ram, 20G Disk để dự trữ không sử dụng
- ...

### 4. Rapid elasticity
Khả năng co giãn, đàn hồi tài nguyên nhanh chóng và thuận tiện. Có thể cấp phát và thu hồi nhanh chóng theo nhu cầu khách hàng

### 5. Measured service
Khả năng đo lường tài nguyên để kiểm soát dịch vụ.

## III. Mô hình triển khai(4)
### 1. Private Cloud
<img src="https://i.imgur.com/qzyO7PV.png">

Private cloud được cung cấp để sử dụng trong nội bộ.

### 2. Public Cloud
<img src="https://i.imgur.com/KfgfCqt.png">

Public cloud là dịch vụ cloud cung cấp cho khách hàng thông qua internet. Thường là cho khách hàng thuê.

### 3. Hybrid Cloud
<img src="https://i.imgur.com/RDXj5yZ.png">

Hybrid cloud là sự kết hợp của Public cloud và Private Cloud. Cho phép ta khai thác điểm mạnh của từng mô hình cũng như đưa ra phương thức sử dụng tối ưu cho người dùng.

Mục đích của Hybrid cloud: tạo ra một môi trường thống nhất, tự động, có thể mở rộng, tận dụng tất cả những gì cơ sở hạ tầng đám mây công cộng có thể cung cấp, trong khi vẫn duy trì quyền kiểm soát dữ liệu quan trọng.

### 4. Community Cloud
<img src="https://i.imgur.com/Nplal4E.png">

Community Cloud được xây dựng để phục vụ cho 1 cộng đồng chung mối quan tâm nào đó sử dụng (ví dụ: chung ngành nghề, ...)

## IV. Mô hình dịch vụ(3)
<img src="https://i.imgur.com/NrDikKr.png">

### 1. Infrastructure as a Service (IaaS)
Cung cấp cho khách hàng tài nguyên tính toán, lưu trữ, ... (RAM, Disk, CPU, ...).

Người dùng có thể tùy ý cài đặt OS, ứng dụng

Ví dụ: Các dịch vụ cho thuê VM, cloud server, ....

### 2. Platform as a Service (PaaS)
Cung cấp cho người dùng nền tảng để khách hàng truy cập và phát triển ứng dụng của mình

Khách hàng có thể truy cập thông qua API, web interface, ...

Ví dụ: Tiki, Lazada, ...

### 3. Software as a Service (SaaS)
Khả năng cung cấp cho người dùng ứng dụng.

Người dùng không quan tâm ứng dụng chạy trên hạ tầng hay nền tảng nào.

Ví dụ: mail, office, ...
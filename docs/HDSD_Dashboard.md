# Hướng dẫn sử dụng Dashboard

Login vào địa chỉ: `http://10.10.31.166`, sau đó nhập các thông tin.

<img src="..\images\Screenshot_9.png">

Trong đó: 
- 1: Nhập domain là `default`
- 2: Nhập username là `admin`
- 3: Nhập password: Đã khai báo trước đó. Tại đây là `Welcome123`
- 4: Chọn `Sign In`

Sau khi đăng nhập xong chuyển sang bước khai báo các thành phần cần thiết để có đủ điều kiện tạo máy ảo.

## 1. Khai báo Network
### 1.1. Tạo provider network
Chọn Tab Admin -> Network -> Networks -> Create Network

<img src="..\images\Screenshot_10.png">

Tạo dải mạng public: chính là dải provider:

<img src="..\images\Screenshot_11.png">

- 1: Nhập tên của public network, tên này có thể là bất kỳ.
- 2: Chon project để tạo network, trong hướng dẫn sẽ chọn project `admin`
- 3: Cho chế độ mạng là `Flat`, một số hướng dẫn khác sẽ chọn là `Vlan`.
- 4: Nhập tên của Physical Network, tên này phải giống với tên trong bước khai báo của neutron ở cả controller và compute. Trong hướng dẫn này là `provider`.
- 5, 6: Tích chọn các mục này để đảm bảo đây là provider network và được dùng chung cho các người dùng khác nhau trên hệ thống OpenStack.
- 7: Chọn mục 7 để chuyển sang màn tiếp theo.


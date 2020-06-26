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

Tạo range cho dải provider trong phần **Subnet**

<img src="..\images\Screenshot_12.png">

Trong đó:
- 1: Nhập tên cho subnet provider network.
- 2: Nhập range sẽ làm provider network. Trong mô hình mạng này ta sẽ sử dụng provider network là `10.10.32.0/24`. Lưu ý range này cần lựa chọn đúng với mô hình mạng mà bạn sử dụng.
- 3: Mục nay để trống thì hệ thống sẽ tự động lấy IP đầu tiên của range. Trong trường hợp này openstack sẽ tự động lấy `10.10.32.1/24`.
- 4: Sau khi chọn xong các bước 1, 2, 3 thì chọn tiếp mục **Next** để chuyển sang màn tiếp theo.

Tại mục **Subnet Details**: khai báo range IP cấp cho các VM và DNS

<img src="..\images\Screenshot_13.png">

Trong đó:
- 1: Ta sẽ khai báo IP bắt đầu và IP kết thúc trong dải network provider theo cú pháp: `IP đầu tiên, IP kết thúc`
- 2: Ta nhập DNS Server 
- 3: Chọn create để kết thúc việc khai báo provider network, sau đó chuyển sang khai báo private network

Sau khi khai báo xong, ta sẽ thấy dải mạng vừa tạo như sau:

<img src="..\images\Screenshot_14.png">

Chọn chữ `public01` để quan sát thêm thông tin về network public vừa được tạo. Sau đó chọn tab port:

<img src="..\images\Screenshot_15.png">

Như hình trên ta thấy có IP `10.10.32.170`, đây là địa chỉ IP mà OpenStack lựa chọn để làm địa chỉ của DHCP server, địa chỉ IP này sẽ cấp DHCP cho các VM mỗi khi chúng được tạo ra.

Ta có thể đứng từ `controller` ping thử tới IP này, nếu có kết quả trả về thì phần khai báo network provider này đã thực hiện đúng.

Tiếp theo ta sẽ chuyển sang bước tạo private network.

### 1.2. Tạo private network
Chọn **Project** -> **Networks** -> **Network** -> **Create network**

<img src="..\images\Screenshot_16.png">

Trong tab Network, ta khai báo tên của private network

<img src="..\images\Screenshot_17.png">

Tab Subnet, khai báo tên subnet, dải mạng private, 

<img src="..\images\Screenshot_18.png">

Tab Subnet Details, ta sẽ chọn dải mạng cấp DHCP của dải private, DNS:

<img src="..\images\Screenshot_19.png">

Sau khi tạo xong ta sẽ thấy dải mạng private vừa tạo, có thể xem thông tin chi tiết bằng cách click chuột vào tên dải mạng đó.

<img src="..\images\Screenshot_20.png">

### 1.3. Cấu hình Security group
Mặc định thì OpenStack chặn toàn bộ các kết nối từ ngoài vào trong các VM, do vậy ta cần thực hiện mở các kết nối này để tiện thao tác.

Truy cập vào tab **Project** -> **Networks** -> **Security Group** -> **Manager Rules**

<img src="..\images\Screenshot_21.png">

Chọn **Add Rule**

<img src="..\images\Screenshot_22.png">

Chọn các rule cần thiết theo danh sách dưới, lưu ý chỉ chọn các rule cần thiết, trong hướng dẫn này sẽ mở các rule sau: `ALL ICMP`, `ALL TCP`, `ALL UDP`, `SSH`

<img src="..\images\Screenshot_23.png">

Thực hiện tương tự với các rule khác. sau khi add rule xong ta sẽ thấy các rule vừa thêm như sau:

<img src="..\images\Screenshot_24.png">

## 2. Tạo Flavor (gói cấu hình)
Ta sẽ tạo gói cấu hình cần để tạo VM

Chọn **Admin** -> **Compute** -> **Flavors** -> **Create Flavor**

<img src="..\images\Screenshot_25.png">

Nhập thông tin gói cấu hình

<img src="..\images\Screenshot_26.png">

Chọn sang tab Flavor Access, ta sẽ chọn project sẽ được sử dụng gói này. Trong bài lab này, ta chọn Admin

<img src="..\images\Screenshot_27.png">

Sau khi tạo xong, ta sẽ thấy Gói A được tạo:

<img src="..\images\Screenshot_28.png">


## 3. Tạo VM
Tạo VM:

<img src="..\images\Screenshot_29.png">

Đặt tên và mô tả VM:

<img src="..\images\Screenshot_30.png">

Chọn image:

<img src="..\images\Screenshot_31.png">

Chọn gói cấu hình:

<img src="..\images\Screenshot_32.png">

Chọn dải mạng

<img src="..\images\Screenshot_33.png">

Chọn **Launch Instance**

Chờ một lát để hệ thống tạo máy.

<img src="..\images\Screenshot_34.png">

Truy cập máy ảo qua console:

<img src="..\images\Screenshot_35.png">

Truy cập bằng thông tin: `cirros`/`gocubsgo`

<img src="..\images\Screenshot_36.png">

Kiểm tra IP và ping internet:

<img src="..\images\Screenshot_37.png">

Đứng từ máy controller ping tới VM:

<img src="..\images\Screenshot_38.png">
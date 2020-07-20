# Các file cấu hình của Glance

## Cấu hình cơ bản
Glance có một số tùy chọn mà bạn có thể sử dụng để cấu hình Glance API server, Glance Registry server và nhiều storage backens lưu trữ image cho Glance.

Chủ yếu các cấu hình đều được để trong file, Glance API server và Glance Registry server sử dụng hai file cấu hình khác nhau.

Khi Glance server khởi động, bạn có thể chỉ định file cấu hình được sử dụng. Nếu bạn không chỉ định cụ thể, Glance sẽ xem xét các thư mục sau để lấy file cấu hình:
```
~/.glance
~/
/etc/glance
/etc
```

Thư mục chứa các file cấu hình của Glance: `/etc/glance`

File cấu hình được tổ chức theo dạng Section, các tùy chọn theo dạng `key = value`

Glance gồm các file cấu hình cơ bản sau đây:
- **glance-api.conf** : File cấu hình cho API của image service.
- **glance-registry.conf** : File cấu hình cho glance image registry - nơi lưu trữ metadata về các images.
- **glance-scrubber.conf** : Được dùng để dọn dẹp các image đã được xóa
- **policy.json** : Bổ sung truy cập kiểm soát áp dụng cho các image service. Trong này, chúng tra có thể xác định vai trò, chính sách, làm tăng tính bảo mật trong Glane OpenStack.

Để có thể đổi path chứa các file cấu hình:
```
glance-api --config-dir=/etc/glance/glance-api.d
```

## File `glance-api.conf`
Đường dẫn: `/etc/glance/glance-api.conf`

- `[database]` : Cấu hình kết nối tới database trong section 
    ```
    [database]
    connection = mysql+pymysql://glance:Welcome123@10.10.31.166/glance
    ```
    **Trong đó:** 
    - `glance` : user truy cập database của glance
    - `Welcome123` : mật khẩu của user glance
    - `10.10.31.166` : IP node controller

- `[keystone_authtoken]` và `[paste_deploy]` : Cấu hình kết nối tới identity service tại section  với các thông tin cần thiết
    ```
    [keystone_authtoken]
    www_authenticate_uri = http://10.10.31.166:5000
    auth_url = http://10.10.31.166:5000
    memcached_servers = 10.10.31.166:11211
    auth_type = password
    project_domain_name = Default
    user_domain_name = Default
    project_name = service
    username = glance
    password = Welcome123

    [paste_deploy]
    flavor = keystone
    ```

- `[glance_store]` : Cấu hình kiểu lưu trữ image tại section
    ```
    [glance_store]
    stores = file,http
    default_store = file
    filesystem_store_datadir = /var/lib/glance/images/
    ```

    Ở đây có hai kiểu lưu trữ được sử dụng và mặc định sẽ sử dụng file system. Bạn có thể cấu hình để lưu trữ trên nhiều nơi khác nhau như sau:
    ```
    filesystem_store_datadirs=PATH:PRIORITY
    ```
    Trong đó: `* PATH` là đường dẫn tới thư mục chứa image `* PRIORITY` là mức độ ưu tiên.

    **Ví dụ:**
    ```
    filesystem_store_datadirs = /var/glance/store
    filesystem_store_datadirs = /var/glance/store1:100
    filesystem_store_datadirs = /var/glance/store2:200
    ```

## Nơi lưu image mặc định
Để kiểm tra thư mục lưu các image:
```
cat /etc/glance/glance-api.conf | grep "filesystem_store_datadir" | egrep -v "^#|^$"
```
OP:
```
filesystem_store_datadir = /var/lib/glance/images/
```

Kiểm tra các image hiện có:
```
ls -lh /var/lib/glance/images/
total 13M
-rw-r----- 1 glance glance 13M Jun 25 23:07 010ffd94-4d26-4dc6-be5b-1a7a31a3686a
```

Kiểm tra bằng command:
```
openstack image list
+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| 010ffd94-4d26-4dc6-be5b-1a7a31a3686a | cirros | active |
+--------------------------------------+--------+--------+
```

Ta thấy, image được lưu dưới dạng tên là ID của image đó.
# Tổng quan về project Nova trong OpenStack

# 1. Giới thiệu về Compute Service - Nova
- Nova là service chịu trách nhiệm chứa và quản lí các hệ thống cloud computing. OpenStack Nova là một project core trong Openstack, nhằm mục đích cấp phép các tài nguyên và quản lý số lượng lớn máy ảo. 

- OpenStack Compute chính là phần chính quan trọng nhất trong kiến trúc hệ thống Infrastructure-as-a-Service (IaaS). Phần lớn các modules của Nova được viết bằng Python.

- OpenStack Compute giao tiếp với các service khách của OPS:
    - OpenStack Identity (Keystone) để xác thực 
    - OpenStack Image (Glance) để lấy images
    - OpenStack Dashboard (Horizon) để lấy giao diện cho người dùng và người quản trị.
    - Ngoài ra còn có thể tương tác với các service khác : block storage, disk, baremetal compute instance

- Nova cho phép bạn điều khiển các máy ảo và networks, bạn cũng có thể quản lí các truy cập tới cloud từ users và projects. OpenStack Compute không chứa các phần mềm ảo hóa. Thay vào đó, nó sẽ định nghĩa các drivers để tương tác với các kĩ thuật ảo hóa khác chạy trên hệ điều hành của bạn và cung cấp các chức năn thông qua một web-based API.

# 2. Các thành phần của Nova

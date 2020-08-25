# Security Group

# Tổng quan
Security group là bộ các quy tắc để filter các IP, nó được áp dụng cho tất cả các instance để định nghĩa mạng truy cập và các máy ảo. Group rules được xác định cho các projects cụ thể, các user thuộc vào project nào thì có thể chỉnh sửa, thêm, xóa các rule của group tương ứng.

Tất cả các project đều có một security-groups mặc định là default được áp dụng cho bất kỳ một instance nào không được định nghĩa một security group nào khác. Nếu không thay đổi gì thì mặc định security group sẽ chặn tát cả các incoming traffic với instance của bạn.

Bạn có thể sử dụng option `allow_same_net_traffic` trong file `/etc/nova/nova.conf` để kiểm soát toàn bộ nếu các rules áp dụng cho host được chia sẻ mạng. Có hai giá trị có thể:

- **True (default)**

    - Hosts cũng nằm trên một subnet không được lọc và được cho phép đi qua đối với tất cả các loại traffic giữa chúng. 
    - Trên Flat network, tất cả các instances của tất cả các project đều không được lọc khi giao tiếp với nhau. 
    - Với VLAN, cho phép truy cập giữa các instance cùng project. 
    - Bạn cũng có thể mô phỏng option này bằng cách cấu hình default security group cho phép tất cả các traffic từ subnet.

- **False**
    - Security groups sẽ bắt buộc được áp dụng cho tất cả các kết nối, kể cả các kết nối cùng mạng.

Ngoài ra, số lượng tối đa các rules trong một security group được điểu khiển bởi `security_group_rules` và số lượng các security groups cho một project được điểu khiển bởi `security_groups` (xem [Manage quotas](https://docs.openstack.org/nova/rocky/admin/quotas2.html#manage-quotas))

# List và xem các security group hiện tại
List các security group hiện có trong project:
```
openstack security group list
openstack security group list
+--------------------------------------+---------+------------------------+----------------------------------+------+
| ID                                   | Name    | Description            | Project                          | Tags |
+--------------------------------------+---------+------------------------+----------------------------------+------+
| 37a041f3-fecf-4871-977e-4e1d493890ef | default | Default security group | dd170f6234c14cf9992060c0f67a17cb | []   |
| 5c1580e3-653f-4fc1-87cb-b05c324c50cf | default | Default security group |                                  | []   |
| c221dad8-8d00-4afa-b993-d654ab701275 | default | Default security group | 5b4c1d2155004acf849cd3aac03b8f36 | []   |
| c4954dfa-6a2b-4fb0-9a32-8f9d7d82f928 | default | Default security group | af57453a686740f18e48a8c4cf4ac994 | []   |
+--------------------------------------+---------+------------------------+----------------------------------+------+
```

Các security group có thể trùng tên nhau nên ta có thể sử dụng thêm option `--project` để xác định chính xác security group của project cần xem:
```
openstack security group list --project admin
+--------------------------------------+---------+------------------------+----------------------------------+------+
| ID                                   | Name    | Description            | Project                          | Tags |
+--------------------------------------+---------+------------------------+----------------------------------+------+
| c221dad8-8d00-4afa-b993-d654ab701275 | default | Default security group | 5b4c1d2155004acf849cd3aac03b8f36 | []   |
+--------------------------------------+---------+------------------------+----------------------------------+------+
```

Xem chi tiết security group:
```
openstack security group rule list <Security_group>
```
Ví dụ:
```
openstack security group rule list c221dad8-8d00-4afa-b993-d654ab701275
+--------------------------------------+-------------+-----------+-----------+------------+--------------------------------------+
| ID                                   | IP Protocol | Ethertype | IP Range  | Port Range | Remote Security Group                |
+--------------------------------------+-------------+-----------+-----------+------------+--------------------------------------+
| 0a6a6397-e832-4f60-8650-bc82412f7cc4 | tcp         | IPv4      | 0.0.0.0/0 | 1:65535    | None                                 |
| 0f6bb641-304f-4651-9509-8808cd46488e | tcp         | IPv4      | 0.0.0.0/0 | 22:22      | None                                 |
| 2cc9c11f-5c40-4d36-a6d8-8cf410acf245 | None        | IPv6      | ::/0      |            | c221dad8-8d00-4afa-b993-d654ab701275 |
| 3f8a3bd7-6f3e-4b16-b061-3102da920e85 | None        | IPv6      | ::/0      |            | None                                 |
| 7439ff0d-7761-4e0d-902f-fea9dc058420 | None        | IPv4      | 0.0.0.0/0 |            | c221dad8-8d00-4afa-b993-d654ab701275 |
| 7aa306b9-a090-4fde-9c05-00556cbd4ea9 | icmp        | IPv4      | 0.0.0.0/0 |            | None                                 |
| 86ea6c49-bf96-4a21-9077-54d701a043e2 | None        | IPv4      | 0.0.0.0/0 |            | None                                 |
| db2f4792-3e7e-486d-9dfd-24ece8c26247 | udp         | IPv4      | 0.0.0.0/0 | 1:65535    | None                                 |
+--------------------------------------+-------------+-----------+-----------+------------+--------------------------------------+
```

# Lab một số rule security group
Ban đầu, khi tạo mới 1 seciurity group sẽ chỉ có 2 rule mặc định:
```
openstack security group rule list sg-test
+--------------------------------------+-------------+-----------+-----------+------------+-----------------------+
| ID                                   | IP Protocol | Ethertype | IP Range  | Port Range | Remote Security Group |
+--------------------------------------+-------------+-----------+-----------+------------+-----------------------+
| a301d6e9-0ed9-42ad-bba3-9beb066856c7 | None        | IPv4      | 0.0.0.0/0 |            | None                  |
| ceacdd3e-78f6-46dd-8e97-8f577e160216 | None        | IPv6      | ::/0      |            | None                  |
+--------------------------------------+-------------+-----------+-----------+------------+-----------------------+
```



## Cho phép ping tới VM:
```
openstack security group rule create \
--protocol icmp \
sg-test
```

## Cho phép ssh tới VM
```
openstack security group rule create \
--protocol tcp \
--ingress \
--dst-port 22 \
sg-test
```

## Cho phép truy cập qua port 80 dịch vụ web
- Cài đặt apache lên VM (đối với Linux)
- Truy cập IP của VM sẽ không kết nối được

- Thực hiện add rule vào security group mà VM đang sử dụng:
    ```
    openstack security group rule create \
    --protocol tcp \
    --ingress \
    --dst-port 80 \
    sg-test
    ```

- Truy cập IP của VM bằng trình duyệt sẽ thấy trang Default Page Apache

# Tham khảo:
- https://docs.openstack.org/python-openstackclient/latest/cli/command-objects/security-group-rule.html
# Migrate VM trong Openstack

## Tổng quan
Migration là quá trình di chuyển máy ảo từ host vật lí này sang một host vật lí khác. Migration được sinh ra để làm nhiệm vụ bảo trì nâng cấp hệ thống. Ngày nay tính năng này đã được phát triển để thực hiện nhiều tác vụ hơn:
- **Cân bằng tải**: Di chuyển VMs tới các host khác khi phát hiện host đang chạy có dấu hiệu quá tải.
- **Bảo trì, nâng cấp hệ thống**: Di chuyển các VMs ra khỏi host trước khi tắt nó đi.
- **Khôi phục lại máy ảo khi host gặp lỗi**: Restart máy ảo trên một host khác.

Trong OpenStack, việc migrate được thực hiện giữa các node compute với nhau hoặc giữa các project trên cùng 1 node compute.

Openstack cung cấp 1 số phương pháp khác nhau đề Migrate VM. Các phương pháp này có các hạn chế khác nhau:
- Không thể live-migrate với shared storage.
- Không thể live-migrate nếu đã bật enabled.
- Không thể select target host nếu sử dụng nova migrate.

**Opensatck hỗ trợ hai kiểu migration đó là:**
- Cold migration: Non-live migration
- Live migration:
    - True live migration (shared storage or volume-based)
    - Block live migration

## So sánh 2 kiểu migrate
### Cold migrate
**Ưu điểm:**
- Đơn giản, dễ thực hiện
- Thực hiện được với mọi loại máy ảo

**Hạn chế:**
- Thời gian downtime lớn
- Không thể chọn host muốn migrate đến
- Quá trình migrate có thể mất một khoảng thời gian dài.

### Live migrate
**Ưu điểm:**
- Có thể chọn host muốn migrate đến
- Tiết kiệm chi phí, tăng sự linh hoạt trong khâu quản lý và vận hành
- Giảm thời gian downtime và gia tăng khả năng "cứu hộ" khi gặp sự cố.

**Nhược điểm:**
- Dù chọn được host nhưng vẫn có những giới hạn nhất định
- Quá trình migrate có thể fails nếu host bạn chọn không có đủ tài nguyên.
- Bạn không được can thiệp vào bất cứ tiến trình nào trong quá trình live migrate.
- Khó migrate với những máy ảo có dung lượng bộ nhớ lớn và trường hợp hai host khác CPU.

## Workflow 
### Cold migrate 
1. Tắt máy ảo (giống với virsh destroy) và ngắt kết nối với volume
2. Di chuyển thư mục hiện tại của máy ảo (instance_dir -> instance_dir_resize)
3. Nếu sử dụng QCOW2 với backing files (chế độ mặc định) thì image sẽ được convert thành dạng flat
4. Với shared storage, di chuyển thư mục chứa máy ảo. Nếu không, copy toàn bộ thông qua SCP.

### Live migrate
1. Kiểm tra lại xem storage backend có phù hợp với loại migrate sử dụng không
    - Thực hiện check shared storage với chế độ migrate thông thường
    - Không check khi sử dụng block migrations
    - Việc kiểm tra thực hiện trên cả 2 node gửi và nhận, chúng được điều phối bởi RPC call từ scheduler.
2. Đối với nơi nhận
    - Tạo các kết nối càn thiết với volume.
    - Nếu dùng block migration, tạo thêm thư mục chứa máy ảo, truyền vào đó những backing files còn thiếu từ Glance và tạo disk trống.
3. Tại nơi gửi, bắt đầu quá trình migration (qua url)
4. Khi hoàn thành, generate lại file XML và define lại nó ở nơi chứa máy ảo mới.

Trong live-migrate có hai loại là True và Block live migration. Dưới đây là mô tả một số các loại storage hỗ trợ hai loại migration này:

<img src="..\images\Screenshot_85.png">

# Hướng dẫn cấu hình 2 loại migrate
## Hướng dẫn cấu hình Cold migrate
Thực hiện migrate 1 VM từ `compute1`(10.10.31.167) sang `compute2`(10.10.31.168)
> ### Sử dụng SSH Tunneling để migrate hoặc resize VM
### Tại node Compute1
Sử dụng khóa gốc có trong thư mục `/root/.ssh/id_rsa` và `/root/.ssh/id_rsa.pub` hoặc tạo cặp khóa mới. Ở đây, ta sẽ tạo khóa mới

Tạo usermode
```
usermod -s /bin/bash nova
```

Tạo key-pair cho user nova
```
su nova
ssh-keygen
echo 'StrictHostKeyChecking no' >> /var/lib/nova/.ssh/config
cat /var/lib/nova/.ssh/id_rsa.pub > /var/lib/nova/.ssh/authorized_keys
chmod 600 /var/lib/nova/.ssh/id_rsa /var/lib/nova/.ssh/authorized_keys
exit
```

Thực hiện với quyền root, scp key pair tới node `compute2`. Nhập mật khẩu khi được yêu cầu.
```
scp -r /var/lib/nova/.ssh root@compute2:/var/lib/nova/
```

### Tại node Compute2
Thay đổi quyền của key pair cho user nova và add key pair đó vào SSH:
```
chown -R nova:nova /var/lib/nova/.ssh
```

Kiểm tra lại từ node `compute1` để chắc chắn user `nova` login được vào node `compute2` mà không cần sử dụng password 
```
su nova
ssh compute2
```

Kiểm tra tương tự trên node `compute2`
```
su nova
ssh compute1
```

#### Lưu ý
Nếu gặp phải lỗi:
```
This account is currently not available.
Connection to compute2 closed.
```

Kiểm tra shell của user `nova` trên node `compute2`, ta sẽ thấy `nologin`
```
cat /etc/passwd | grep "nova"
nova:x:162:162:OpenStack Nova Daemons:/var/lib/nova:/sbin/nologin
```

Bạn cần set usermod trên node `Compute2`
```
usermod -s /bin/bash nova
```

**Lưu ý:** Đối với trường hợp nhiều node compute, có thể sử dụng chung 1 cặp key, copy cặp key đó tới tất cả các node, để tất cả các node có thể xác thực lẫn nhau.

### Thực hiện trên cả 2 node
Restart service
```
systemctl restart libvirtd.service
systemctl restart openstack-nova-compute.service
```

### Thực hiện cold-migrate - Trên node Controller
Tắt máy ảo nếu nó đang chạy: (Nên tắt máy)
```
openstack server stop vm02-ovc
```

Migrate máy ảo
```
openstack server migrate vm02-ovc
```

Kiểm tra trạng thái
```
openstack server list
+--------------------------------------+----------+---------------+-----------------------+-------+---------+
| ID                                   | Name     | Status        | Networks              | Image | Flavor  |
+--------------------------------------+----------+---------------+-----------------------+-------+---------+
| 58607cad-aa47-4546-8531-4de7a1bdbb28 | vm02-ovc | VERIFY_RESIZE | public01=10.10.32.173 |       | flv_ovc |
+--------------------------------------+----------+---------------+-----------------------+-------+---------+
```

Xác thực resize:
```
openstack server resize confirm vm02-ovc
```


Kiểm tra lại kết quả:
```
openstack server list --long
```

## Hướng dẫn cấu hình Live-Migrate
**OpenStack hỗ trợ 2 loại live migrate:**
- True live migration (shared storage or volume-based) : Trong trường hợp này, máy ảo sẽ được di chuyển sửa dụng storage mà cả hai máy computes đều có thể truy cập tới. Nó yêu cầu máy ảo sử dụng block storage hoặc shared storage.
- Block live migration : Mất một khoảng thời gian lâu hơn để hoàn tất quá trình migrate bởi máy ảo được chuyển từ host này sang host khác. Tuy nhiên nó lại không yêu cầu máy ảo sử dụng hệ thống lưu trữ tập trung.

Các yêu cầu chung:
- Cả hai node nguồn và đích đều phải được đặt trên cùng subnet và có cùng loại CPU.
- Cả controller và compute đều phải phân giải được tên miền của nhau.
- Compute node buộc phải sử dụng KVM với libvirt.

**Lưu ý:** live-migration làm việc với 1 số loại VM và storage
- Shared storage: Cả 2 hypervisor có quyền truy cập shared storage chung.
- Block storage: VM sử dụng root disk, không tương thích với các loại read-only devices.
- Volume storage: VM sử dụng iSCSI volumes.

### Cấu hình live migrate
#### Tại các node compute
Chỉnh sửa cấu hình
```
sed -i 's/#listen_tls = 0/listen_tls = 0/g' /etc/libvirt/libvirtd.conf
sed -i 's/#listen_tcp = 1/listen_tcp = 1/g' /etc/libvirt/libvirtd.conf
sed -i 's/#auth_tcp = "sasl"/auth_tcp = "none"/g' /etc/libvirt/libvirtd.conf
sed -i 's/#LIBVIRTD_ARGS="--listen"/LIBVIRTD_ARGS="--listen"/g' /etc/sysconfig/libvirtd
```

Khởi động lại service
```
systemctl restart libvirtd
systemctl restart openstack-nova-compute.service
```

Nếu sử dụng block device, sửa file `nova.conf`
```
[libvirt]
block_migration_flag=VIR_MIGRATE_UNDEFINE_SOURCE, VIR_MIGRATE_PEER2PEER, VIR_MIGRATE_LIVE, VIR_MIGRATE_NON_SHARED_INC
```

Khởi động lại dịch vụ
```
systemctl restart openstack-nova-compute.service
```

### Live migrate VM
> #### Thực hiện trên node Controller
Kiểm tra danh sách các VM
```
openstack server list --long
+--------------------------------------+----------+--------+------------+-------------+------------------------------------------------+------------+----------+-------------+--------------------------------------+-------------------+----------+------------+
| ID                                   | Name     | Status | Task State | Power State | Networks                                       | Image Name | Image ID | Flavor Name | Flavor ID                            | Availability Zone | Host     | Properties |
+--------------------------------------+----------+--------+------------+-------------+------------------------------------------------+------------+----------+-------------+--------------------------------------+-------------------+----------+------------+
| 905d3d6a-8e5b-4a83-add8-88e6a24d13a3 | U20-v4   | ACTIVE | None       | Running     | private01=192.168.1.147; public01=10.10.32.174 |            |          | flv_1c1r    | c8e18df3-50f0-4e12-af58-6ea631da4915 | nova              | compute1 |            |
| 58607cad-aa47-4546-8531-4de7a1bdbb28 | vm02-ovc | ACTIVE | None       | Running     | public01=10.10.32.173                          |            |          | flv_ovc     | 54da83c2-c6cb-4b89-b8db-7c1cf7c2285c | nova              | compute1 |            |
+--------------------------------------+----------+--------+------------+-------------+------------------------------------------------+------------+----------+-------------+--------------------------------------+-------------------+----------+------------+
```

Xem danh sách các node có thể migrate tới
```
openstack compute service list
+----+----------------+------------+----------+---------+-------+----------------------------+
| ID | Binary         | Host       | Zone     | Status  | State | Updated At                 |
+----+----------------+------------+----------+---------+-------+----------------------------+
|  5 | nova-conductor | controller | internal | enabled | up    | 2020-08-05T07:39:44.000000 |
|  6 | nova-scheduler | controller | internal | enabled | up    | 2020-08-05T07:39:45.000000 |
|  7 | nova-compute   | compute2   | nova     | enabled | up    | 2020-08-05T07:39:40.000000 |
|  8 | nova-compute   | compute1   | nova     | enabled | up    | 2020-08-05T07:39:41.000000 |
+----+----------------+------------+----------+---------+-------+----------------------------+
```

Kiểm tra tài nguyên tại node `compute2`
```
openstack host show compute2
+----------+------------+-----+-----------+---------+
| Host     | Project    | CPU | Memory MB | Disk GB |
+----------+------------+-----+-----------+---------+
| compute2 | (total)    |   4 |      7821 |      47 |
| compute2 | (used_now) |   0 |       512 |       0 |
| compute2 | (used_max) |   0 |         0 |       0 |
+----------+------------+-----+-----------+---------+
```

Vào console của VM, đặt lệnh ping:
```
ping 8.8.8.8
```

Ta thực hiện migrate VM: `U20-v4` từ `compute1` sang `compute2`

- Lệnh này dùng với các VM dùng `shared storage`
    ```
    openstack server migrate 905d3d6a-8e5b-4a83-add8-88e6a24d13a3 --live-migration --host compute2
    ```

- Lệnh dùng với VM boot từ local: Cần thiết lập `QEMU-native TLS`
    Đọc thêm: [Configmigrate](https://docs.openstack.org/nova/train/admin/configuring-migrations.html#securing-live-migration-streams) và [Secure live migration with QEMU-native TLS](https://docs.openstack.org/nova/train/admin/secure-live-migration-with-qemu-native-tls.html)
    ```
    openstack server migrate 905d3d6a-8e5b-4a83-add8-88e6a24d13a3 --live-migration --block-migration --host compute2
    ```

Sau khi hoàn thành, kiểm tra lại:
```
openstack server show U20-v4
+-------------------------------------+----------------------------------------------------------+
| Field                               | Value                                                    |
+-------------------------------------+----------------------------------------------------------+
| OS-DCF:diskConfig                   | AUTO                                                     |
| OS-EXT-AZ:availability_zone         | nova                                                     |
| OS-EXT-SRV-ATTR:host                | compute2                                                 |
| OS-EXT-SRV-ATTR:hypervisor_hostname | compute2                                                 |
| OS-EXT-SRV-ATTR:instance_name       | instance-0000000b                                        |
| OS-EXT-STS:power_state              | Running                                                  |
| OS-EXT-STS:task_state               | None                                                     |
| OS-EXT-STS:vm_state                 | active                                                   |
| OS-SRV-USG:launched_at              | 2020-08-04T15:51:28.000000                               |
| OS-SRV-USG:terminated_at            | None                                                     |
| accessIPv4                          |                                                          |
| accessIPv6                          |                                                          |
| addresses                           | private01=192.168.1.147; public01=10.10.32.174           |
| config_drive                        |                                                          |
| created                             | 2020-08-04T15:51:07Z                                     |
| flavor                              | flv_1c1r (c8e18df3-50f0-4e12-af58-6ea631da4915)          |
| hostId                              | 51bdefb77b714d0ede0eb427e38003eb2abb927b9aa78079782f04b3 |
| id                                  | 905d3d6a-8e5b-4a83-add8-88e6a24d13a3                     |
| image                               |                                                          |
| key_name                            | None                                                     |
| name                                | U20-v4                                                   |
| progress                            | 0                                                        |
| project_id                          | 5b4c1d2155004acf849cd3aac03b8f36                         |
| properties                          |                                                          |
| security_groups                     | name='default'                                           |
|                                     | name='default'                                           |
| status                              | ACTIVE                                                   |
| updated                             | 2020-08-05T09:02:43Z                                     |
| user_id                             | 294c5c6181d442c68a13d5b615c4f031                         |
| volumes_attached                    | id='93ef3e86-f3e8-4019-a1a4-03a2b73dbb6e'                |
+-------------------------------------+----------------------------------------------------------+
```
Console của VM, vẫn thực hiện lệnh ping:

<img src="..\images\Screenshot_87.png">



# Tham khảo
- https://docs.openstack.org/nova/train/admin/migration.html
- https://docs.openstack.org/nova/train/admin/live-migration-usage.html
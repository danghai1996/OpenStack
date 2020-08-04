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



# Tham khảo
- https://docs.openstack.org/nova/train/admin/migration.html
- https://docs.openstack.org/nova/train/admin/live-migration-usage.html
# Resize instance 

Ta có thể thay đổi cấu hình của một instance bằng cách thay đổi flavor của nó. 

## Cú pháp:
```
openstack server resize --flavor FLAVOR SERVER
```

## 1. Ví dụ Resize máy ảo boot từ volume
List danh sách VM
```
openstack server list
+--------------------------------------+----------+--------+------------------------------------------------+--------+----------+
| ID                                   | Name     | Status | Networks                                       | Image  | Flavor   |
+--------------------------------------+----------+--------+------------------------------------------------+--------+----------+
| ...                                                                                                                           |
| 905d3d6a-8e5b-4a83-add8-88e6a24d13a3 | U20-v4   | ACTIVE | private01=192.168.1.147; public01=10.10.32.174 |        | flv_1c1r |
| ...                                                                                                                           |
+--------------------------------------+----------+--------+------------------------------------------------+--------+----------+
```

List các flavor
```
openstack flavor list
+--------------------------------------+----------+------+------+-----------+-------+-----------+
| ID                                   | Name     |  RAM | Disk | Ephemeral | VCPUs | Is Public |
+--------------------------------------+----------+------+------+-----------+-------+-----------+
| 54da83c2-c6cb-4b89-b8db-7c1cf7c2285c | flv_ovc  | 4096 |   10 |         0 |     3 | True      |
| 79b635c3-c0fe-453e-a503-13f1740b6cba | flv_01   |  512 |    5 |         0 |     1 | True      |
| 79ed0192-8a88-4463-ad1f-d1aec1be0af0 | flv_ovc2 | 2048 |    0 |         0 |     3 | True      |
| c8e18df3-50f0-4e12-af58-6ea631da4915 | flv_1c1r | 1024 |   20 |         0 |     1 | False     |
+--------------------------------------+----------+------+------+-----------+-------+-----------+
```

Tắt máy ảo
```
openstack server stop U20-v4
```

Xem danh sách volume:
```
openstack volume list
+--------------------------------------+---------------+-----------+------+-----------------------------------+
| ID                                   | Name          | Status    | Size | Attached to                       |
+--------------------------------------+---------------+-----------+------+-----------------------------------+
| ...                                                                                                         | 
| 93ef3e86-f3e8-4019-a1a4-03a2b73dbb6e | vlm-u20-blank | in-use    |   10 | Attached to U20-v4 on /dev/vda    |
| ...                                                                                                         |
+--------------------------------------+---------------+-----------+------+-----------------------------------+
```

Resize disk:
```
openstack volume set --state available 93ef3e86-f3e8-4019-a1a4-03a2b73dbb6e
openstack volume set 93ef3e86-f3e8-4019-a1a4-03a2b73dbb6e --size 20
```

Kiểm tra lại:
```
openstack volume list
+--------------------------------------+---------------+-----------+------+-----------------------------------+
| ID                                   | Name          | Status    | Size | Attached to                       |
+--------------------------------------+---------------+-----------+------+-----------------------------------+
| ...                                                                                                         | 
| 93ef3e86-f3e8-4019-a1a4-03a2b73dbb6e | vlm-u20-blank | in-use    |   20 | Attached to U20-v4 on /dev/vda    |
| ...                                                                                                         |
+--------------------------------------+---------------+-----------+------+-----------------------------------+
```


Resize VM: `U20-v4` - Flavor: `flv_1c1r` (1Core,1Ram) -> flv_ovc (3Core,4Ram)
```
openstack server resize --flavor flv_ovc2 U20-v4
```

Kiểm tra:
```
openstack server list
+--------------------------------------+----------+---------+------------------------------------------------+--------+----------+
| ID                                   | Name     | Status  | Networks                                       | Image  | Flavor   |
+--------------------------------------+----------+---------+------------------------------------------------+--------+----------+
| ...                                                                                                                            |
| 905d3d6a-8e5b-4a83-add8-88e6a24d13a3 | U20-v4   | RESIZE  | private01=192.168.1.147; public01=10.10.32.174 |        | flv_ovc  |
| ...                                                                                                                            |
+--------------------------------------+----------+---------+------------------------------------------------+--------+----------+
```

Đợi đến khi status chuyển thành `VERIFY_RESIZE`
```
+--------------------------------------+----------+---------------+------------------------------------------------+--------+----------+
| ID                                   | Name     | Status        | Networks                                       | Image  | Flavor   |
+--------------------------------------+----------+---------------+------------------------------------------------+--------+----------+
| ...                                                                                                                                  |
| 905d3d6a-8e5b-4a83-add8-88e6a24d13a3 | U20-v4   | VERIFY_RESIZE | private01=192.168.1.147; public01=10.10.32.174 |        | flv_ovc  |
| ...                                                                                                                                  |
+--------------------------------------+----------+---------------+------------------------------------------------+--------+----------|
```

Confirm resize
```
openstack server resize confirm U20-v4
```

Nếu không thì dùng lệnh sau để hủy bỏ resize:
```
openstack server resize --revert U20-v4
```

Kiểm tra lại:
```
openstack server list
+--------------------------------------+----------+---------+------------------------------------------------+--------+----------+
| ID                                   | Name     | Status  | Networks                                       | Image  | Flavor   |
+--------------------------------------+----------+---------+------------------------------------------------+--------+----------+
| ...                                                                                                                            |
| 905d3d6a-8e5b-4a83-add8-88e6a24d13a3 | U20-v4   | ACTIVE  | private01=192.168.1.147; public01=10.10.32.174 |        | flv_ovc  |
| ...                                                                                                                            |
+--------------------------------------+----------+---------+------------------------------------------------+--------+----------+
```
Kiểm tra trên VM:
```
# CPU
lscpu | egrep 'CPU\(s\)'
CPU(s):                          3
On-line CPU(s) list:             0-2
NUMA node0 CPU(s):               0-2

# RAM
free -m
              total        used        free      shared  buff/cache   available
Mem:           3935         149        3465           2         321        3559
Swap:             0           0           0

# Disk
lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0    7:0    0   55M  1 loop /snap/core18/1880
loop1    7:1    0 55.3M  1 loop /snap/core18/1885
loop2    7:2    0   69M  1 loop /snap/lxd/14804
loop3    7:3    0 71.5M  1 loop /snap/lxd/16530
loop4    7:4    0 27.1M  1 loop /snap/snapd/7264
loop5    7:5    0 29.9M  1 loop /snap/snapd/8542
vda    252:0    0   20G  0 disk
|-vda1 252:1    0    1M  0 part
`-vda2 252:2    0   20G  0 part /
```

## 2. Ví dụ resize máy ảo boot từ local
Thực hiện resize máy ảo bằng câu lệnh
```
openstack server resize --flavor <flavor-name> <vm-name>
```

Đợi đến khi trạng thái máy ảo chuyển về `VERIFY_RESIZE` (dùng câu lệnh `openstack server show` để xem). Ta tiến hành xác nhận hoặc loại bỏ kết quả của việc resize máy ảo. Tiến hành xác nhận bằng câu lệnh sau:
```
openstack server resize --confirm <vm-name>
```

Nếu muốn quay trở về sử dụng máy ảo cũ, sử dụng câu lệnh sau
```
openstack server resize --revert <vm-name>
```

**Lưu ý:** Đối với máy ảo boot từ local thì disk của flavor mới phải lớn hơn disk của flavor cũ
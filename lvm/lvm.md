# Ghi chép về LVM

# I. Logical Volume Management
Logical Volume Management (LVM) đã có sẵn trên hầu hết trên các bản phân phối Linux, có nhiều lợi thế so với quản lý phân vùng truyền thống. Logical Volume Management được sử dụng để tạo nhiều ổ đĩa logic. Nó cho phép các bộ phận logic được thay đổi kích thước (giảm hoặc tăng) theo ý muốn của người quản trị.

Cấu trúc của LVM bao gồm:

- Một hoặc nhiều đĩa cứng hoặc phân vùng được định cấu hình là physical volume(PV).
- Một volume group(VG) được tạo bằng cách sử dụng một hoặc nhiều khối vật lý.
- Nhiều logical volume có thể được tạo trong một volume group.

# II. Các lệnh quản lý và tạo LVM
## 1. Tạo Physical Volume

### Tạo PV (Physical volu)
```
pvcreate /dev/vda /dev/vdb
```
Kết quả khi tạo được:
```
Physical volume "/dev/vda" successfully created
Physical volume "/dev/vdb" successfully created
```

### Liệt kê các PV
```
psv
```
KQ:
```
  PV         VG     Fmt  Attr PSize   PFree
  /dev/vda2  centos lvm2 a--  <49.00g 4.00m
  /dev/vdb   vg0    lvm2 a--  <20.00g    0
  /dev/vdc   vg0    lvm2 a--  <10.00g    0
```
Ý nghĩa các trường của pvs:
- `PV`: Đĩa được sử dụng
- `PFree`: Kích thước đĩa vật lý (Kích thước PV)

### Xem thông tin chi tiết mỗi PV
```
pvdisplay /dev/vdb
```

```
  --- Physical volume ---
  PV Name               /dev/vdb
  VG Name               vg0
  PV Size               20.00 GiB / not usable 4.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              5119
  Free PE               0
  Allocated PE          5119
  PV UUID               jbtC5r-kstj-zufs-8tYb-I6Jq-rjqI-4X8vzL
```

**Lưu ý:** Nếu chúng ta có 2 ổ đĩa hay nhiều ổ đĩa để tạo một volume mà một đĩa ở volume bị mất thì dẫn tới volume đó mất luôn, vì thế phải chạy LVM trên RAID hoặc dùng tính năng RAID của LVM để có khả năng dung lỗi.

## 2. Tạo Volume Group
### Tạo VG từ `vdb` và `vdc` có tên là `vg0`
vdb = 20G, vdc = 10G
```
vgcreate vg0 /dev/vdb /dev/vdc
```
Output
```
Volume group "vg0" successfully created
```

### Xem thông tin VG
```
vgdisplay vg0
```
OP
```
  --- Volume group ---
  VG Name               vg0
  System ID
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               29.99 GiB
  PE Size               4.00 MiB
  Total PE              7678
  Alloc PE / Size       7678 / 29.99 GiB
  Free  PE / Size       0 / 0
  VG UUID               1nxTF3-URLK-2hQf-KLSE-uqi8-zIAZ-Y3hGu7
```

Ý nghĩa các thông tin của Volume group khi chạy lệnh vgdisplay:
- `VG Name`: Tên Volume Group.
- `Format`: Kiến trúc LVM được sử dụng.
- `VG Access`: Volume Group có thể đọc và viết và sẵn sàng để sử dụng.
- `VG Status`: Volume Group có thể được định cỡ lại, chúng ta có thể mở rộng thêm nếu cần thêm dung lượng.
- `PE Size`: Mở rộng Physical, Kích thước cho đĩa có thể được xác định bằng kích thước PE hoặc GB, 4MB là kích thước PE mặc định của LVM
- `Total PE`: Dung lượng Volume Group có
- `Alloc PE`: Tổng PE đã sử dụng
- `Free PE`: Tổng PE chưa được sử dụng

### Kiểm tra số lượng VG đã tạo:
```
vgs
```
OP
```
  VG     #PV #LV #SN Attr   VSize   VFree
  centos   1   2   0 wz--n- <49.00g 4.00m
  vg0      2   2   0 wz--n-  29.99g    0
```

Trong đó:
- `VG`: Tên Volume Group
- `#PV`: Physical Volume sử dụng trong Volume Group
- `VFree`: Hiển thị không gian trống có sẵn trong Volume Group
- `VSize`: Tổng kích thước của Volume Group
- `#LV`: Logical Volume nằm trong volume group
- `#SN`: Số lượng Snapshot của volume group
- `Attr`: Trạng thái của Volume group có thể ghi, có thể đọc, có thể thay đổi

Khi tạo ra các logical volume, chúng ta cần phải xem xét dung lượng khi tạo logical volume sau cho phù hợp với nhu cầu sử dụng.

## 3. Tạo Logical Volume
### Tạo LV
```
lvcreate -n data -L 15G vg0

Logical volume "data" created.
```

hoặc
```
lvcreate -n backups -l 100%FREE vg0
  
Logical volume "backups" created.
```
Trong đó:
- `n`: Sử dụng chỉ ra tên của logical volume cần tạo.
- `L`: Sử dụng chỉ một kích thước cố định.
- `l`: Sử dụng chỉ phần trăm của không gian còn lại trong volume group.

### Xem danh sách LV được tạo
```
lvs
```
OP
```
  LV      VG     Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root    centos -wi-ao---- <45.12g
  swap    centos -wi-ao----  <3.88g
  backups vg0    -wi-a-----  14.99g
  data    vg0    -wi-a-----  15.00g
```

Ý nghĩa các trường của lvs:
- `LV`: Tên logical volume
- `%Data`: Phần trăm dung lượng logical volume được sử dụng
- `Lsize`: Kích thước của logical volume

### Hiển thị chi tiết
```
lvdisplay vg0/data
```
OP
```
  --- Logical volume ---
  LV Path                /dev/vg0/data
  LV Name                data
  VG Name                vg0
  LV UUID                knH1w1-kWou-3Gxn-t34Q-OZzX-trTJ-YDPBmM
  LV Write Access        read/write
  LV Creation host, time 103.101.160.205, 2020-06-26 14:25:37 +0700
  LV Status              available
  # open                 0
  LV Size                15.00 GiB
  Current LE             3840
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:2
```

### File system
Chúng tôi sẽ sử dụng file system `ext4` vì nó cho phép chúng ta tăng và giảm kích thước của mỗi logical volume (với file system `xfs` chỉ cho phép tăng kích thước). Chúng ta thực hiện như sau:

```
mkfs.ext4 /dev/vg0/data
```
OP
```
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
983040 inodes, 3932160 blocks
196608 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=2151677952
120 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```

## 4. Mở rộng Volume Group và thay đổi kích thước các LV
Trong ví dụ dưới đây chúng ta sẽ thêm một physical volume có tên `/dev/vdd` với kích thước 10GB vào volume group `vg0`, sau đó chúng ta sẽ tăng kích thước của logical volume `/data` lên 10GB thực hiện như sau:

Tạo các thư mục và mount disk
```
mkdir /data
mkdir /backups

mount /dev/vg0/data /data/
mount /dev/vg0/backups /backups/
```

Chạy lệnh sau kiểm tra sử dụng không gian đĩa hệ thống tập tin:
```
df -TH
```
OP
```
Filesystem              Type      Size  Used Avail Use% Mounted on
devtmpfs                devtmpfs  2.0G     0  2.0G   0% /dev
tmpfs                   tmpfs     2.0G   25k  2.0G   1% /dev/shm
tmpfs                   tmpfs     2.0G  144M  1.9G   8% /run
tmpfs                   tmpfs     2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/mapper/centos-root xfs        49G  4.5G   44G  10% /
/dev/vda1               xfs       1.1G  197M  867M  19% /boot
tmpfs                   tmpfs     398M     0  398M   0% /run/user/0
tmpfs                   tmpfs     398M     0  398M   0% /run/user/1000
/dev/mapper/vg0-data    ext4       16G   42M   15G   1% /data
/dev/mapper/vg0-backups ext4       16G   42M   15G   1% /backups
```

### Mở rộng VG
Sử dụng lệnh sau để thêm `/dev/vdd` vào volume group `vg0`:
```
vgextend vg0 /dev/vdd
```
OP
```
  Physical volume "/dev/vdd" successfully created.
  Volume group "vg0" successfully extended
```

Kiểm tra xem kích thước của `vg0`
```
vgdisplay vg0
```
OP
```
  --- Volume group ---
  VG Name               vg0
  System ID
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               <39.99 GiB
  PE Size               4.00 MiB
  Total PE              10237
  Alloc PE / Size       7678 / 29.99 GiB
  Free  PE / Size       2559 / <10.00 GiB
  VG UUID               1nxTF3-URLK-2hQf-KLSE-uqi8-zIAZ-Y3hGu7
```

Bây giờ ta thấy VG Size đã là 40G

### Tăng kích thước LVM /data
Tăng kích thước /data từ 15G -> 25G
```
lvextend -L +10G /dev/vg0/data
```
OP
```
  Size of logical volume vg0/data changed from 15.00 GiB (3840 extents) to 24.00 GiB (6144 extents).
  Logical volume vg0/data successfully resized.
```

Sau đó chạy lệnh sau để xác nhận
```
resize2fs /dev/vg0/data
```
OP
```
resize2fs 1.42.9 (28-Dec-2013)
Filesystem at /dev/vg0/data is mounted on /data; on-line resizing required
old_desc_blocks = 2, new_desc_blocks = 3
The filesystem on /dev/vg0/data is now 6291456 blocks long.
```
Kiểm tra dung lượng:
```
df -TH

Filesystem              Type      Size  Used Avail Use% Mounted on
devtmpfs                devtmpfs  2.0G     0  2.0G   0% /dev
tmpfs                   tmpfs     2.0G   25k  2.0G   1% /dev/shm
tmpfs                   tmpfs     2.0G  144M  1.9G   8% /run
tmpfs                   tmpfs     2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/mapper/centos-root xfs        49G  4.5G   44G  10% /
/dev/vda1               xfs       1.1G  197M  867M  19% /boot
tmpfs                   tmpfs     398M     0  398M   0% /run/user/0
tmpfs                   tmpfs     398M     0  398M   0% /run/user/1000
/dev/mapper/vg0-data    ext4       26G   47M   24G   1% /data
/dev/mapper/vg0-backups ext4       16G   42M   15G   1% /backups
```

### Giảm kích thước LV
Khi chúng ta muốn giảm dung lượng các Logical Volume. Chúng ta cần phải chú ý vì nó rất quan trọng và có thể bị lỗi trong khi chúng ta giảm dung lượng Logical Volume. Để đảm bảo an toàn khi giảm Logical Volume cần thực hiện các bước sau:
- Trước khi bắt đầu, cần sao lưu dữ liệu vì vậy sẽ không phải đau đầu nếu có sự cố xảy ra.
- Để giảm dung lượng Logical Volume, cần thực hiện đầy đủ và cẩn thận 5 bước cần thiết.
- Khi giảm dung lượng Logical Volume chúng ta phải ngắt kết nối hệ thống tệp trước khi giảm.

Cần thực hiện cẩn thận 5 bước dưới đây:
- Ngắt kết nối file system.
- Kiểm tra file system sau khi ngắt kết nối.
- Giảm file system.
- Giảm kích thước Logical Volume hơn kích thước hiện tại.
- Kiểm tra lỗi cho file system.
- Mount lại file system và kiểm tra kích thước của nó.

**Ví dụ:** Giảm LV của /data từ 24G xuống 15G mà không làm mất dung lượng:
- Kiểm tra dung lượng LV và kiểm tra file system trước khi giảm
    ```
    lvs

    LV      VG     Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
    root    centos -wi-ao---- <45.12g
    swap    centos -wi-ao----  <3.88g
    backups vg0    -wi-ao----  14.99g
    data    vg0    -wi-ao----  24.00g
    ```
    ```
    df -TH

    Filesystem              Type      Size  Used Avail Use% Mounted on
    devtmpfs                devtmpfs  2.0G     0  2.0G   0% /dev
    tmpfs                   tmpfs     2.0G   25k  2.0G   1% /dev/shm
    tmpfs                   tmpfs     2.0G  144M  1.9G   8% /run
    tmpfs                   tmpfs     2.0G     0  2.0G   0% /sys/fs/cgroup
    /dev/mapper/centos-root xfs        49G  4.5G   44G  10% /
    /dev/vda1               xfs       1.1G  197M  867M  19% /boot
    tmpfs                   tmpfs     398M     0  398M   0% /run/user/0
    tmpfs                   tmpfs     398M     0  398M   0% /run/user/1000
    /dev/mapper/vg0-data    ext4       26G   47M   24G   1% /data
    /dev/mapper/vg0-backups ext4       16G   42M   15G   1% /backups
    ```

- **Bước 1:** Unmount file system
    ```
    umount /data/

    df -TH

    Filesystem              Type      Size  Used Avail Use% Mounted on
    devtmpfs                devtmpfs  2.0G     0  2.0G   0% /dev
    tmpfs                   tmpfs     2.0G   25k  2.0G   1% /dev/shm
    tmpfs                   tmpfs     2.0G  144M  1.9G   8% /run
    tmpfs                   tmpfs     2.0G     0  2.0G   0% /sys/fs/cgroup
    /dev/mapper/centos-root xfs        49G  4.5G   44G  10% /
    /dev/vda1               xfs       1.1G  197M  867M  19% /boot
    tmpfs                   tmpfs     398M     0  398M   0% /run/user/0
    tmpfs                   tmpfs     398M     0  398M   0% /run/user/1000
    /dev/mapper/vg0-backups ext4       16G   42M   15G   1% /backups
    ```

- **Bước 2:** Kiểm tra lỗi bằng lệnh `e2fsck`
    `-f` (force check)là để kiểm tra 
    ```
    e2fsck -f /dev/vg0/data
    ```
    OP
    ```
    e2fsck 1.42.9 (28-Dec-2013)
    Pass 1: Checking inodes, blocks, and sizes
    Pass 2: Checking directory structure
    Pass 3: Checking directory connectivity
    Pass 4: Checking reference counts
    Pass 5: Checking group summary information
    /dev/vg0/data: 11/1572864 files (0.0% non-contiguous), 142759/6291456 blocks
    ```

- **Bước 3:** Giảm kích thước xuống 15G ta thực hiện như sau:
    ```
    resize2fs /dev/vg0/data 10G
    ```
    OP
    ```
    resize2fs 1.42.9 (28-Dec-2013)
    Resizing the filesystem on /dev/vg0/data to 2621440 (4k) blocks.
    The filesystem on /dev/vg0/data is now 2621440 blocks long.
    ```

- **Bước 4:** Giảm kích thước bằng lệnh
    ```
    lvreduce -L 15G /dev/vg0/data
    ```
    OP
    ```
      WARNING: Reducing active logical volume to 15.00 GiB.
    THIS MAY DESTROY YOUR DATA (filesystem etc.)
    Do you really want to reduce vg0/data? [y/n]: y
    Size of logical volume vg0/data changed from 24.00 GiB (6144 extents) to 15.00 GiB (3840 extents).
    Logical volume vg0/data successfully resized.
    ```

- **Bước 5:** Để đảm bảo an toàn, bây giờ kiểm tra lỗi file system đã giảm
    ```
    mkfs -t ext4 /dev/vg0/data
    e2fsck -f /dev/vg0/data
    ```
    OP
    ```
    e2fsck 1.42.9 (28-Dec-2013)
    Pass 1: Checking inodes, blocks, and sizes
    Pass 2: Checking directory structure
    Pass 3: Checking directory connectivity
    Pass 4: Checking reference counts
    Pass 5: Checking group summary information
    /dev/vg0/data: 11/655360 files (0.0% non-contiguous), 83137/2621440 blocks
    ```

- **Bước 6:** Mount file system và kiểm tra kích thước của nó.
    ```
    mount /dev/vg0/data /data/

    lvs
    ```
    OP
    ```
      LV      VG     Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
    root    centos -wi-ao---- <45.12g
    swap    centos -wi-ao----  <3.88g
    backups vg0    -wi-ao----  14.99g
    data    vg0    -wi-ao----  15.00g
    ```
    Kiểm tra thêm bằng `df -TH`
    ```
    Filesystem              Type      Size  Used Avail Use% Mounted on
    devtmpfs                devtmpfs  2.0G     0  2.0G   0% /dev
    tmpfs                   tmpfs     2.0G   25k  2.0G   1% /dev/shm
    tmpfs                   tmpfs     2.0G  144M  1.9G   8% /run
    tmpfs                   tmpfs     2.0G     0  2.0G   0% /sys/fs/cgroup
    /dev/mapper/centos-root xfs        49G  4.5G   45G  10% /
    /dev/vda1               xfs       1.1G  197M  867M  19% /boot
    tmpfs                   tmpfs     398M     0  398M   0% /run/user/0
    tmpfs                   tmpfs     398M     0  398M   0% /run/user/1000
    /dev/mapper/vg0-backups ext4      5.2G   21M  4.9G   1% /backups
    /dev/mapper/vg0-data    ext4       16G   42M   15G   1% /data
    ```

## 5. Mounting LV
Chúng ta cần phải gắn kết vĩnh viễn. Để thực hiện gắn kết vĩnh viễn phải nhập trong tệp `/etc/fstab`. Bạn có thể sử dụng trình soạn thảo vi để nhập vào.

Để xác định UUID trên mỗi đĩa. Chạy lệnh:
```
blkid /dev/vg0/data

/dev/vg0/data: UUID="3e2b1e24-b675-4ac9-8e04-d52eb39f6a37" TYPE="ext4"
```

Sau đó nhập vào file `/etc/fstab` theo định dạng sau:
```
[Device] [Mount Point] [File System Type] [Options] [Dump] [Pass]
```

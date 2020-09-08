# Các bước cài đặt Openstack Queens với Local Repo

## Mục tiêu:
Cài đặt được OpenStack mà không cần sử dụng đến Internet


# Thực hiện:
## 
```
yum install yum-utils -y
```

```
yumdownloader --destdir=/root/backups-ops-queens/centos-release-openstack-queens/ --resolve centos-release-openstack-queens

yumdownloader --destdir=/root/backup-ops/tools/wget/ --resolve wget
yumdownloader --destdir=/root/backup-ops/tools/crudini/ --resolve crudini
yumdownloader --destdir=/root/backup-ops/tools/vim/ --resolve vim


yumdownloader --destdir=/root/backup-ops/python-openstackclient --resolve python-openstackclient

```

```
rpm -ivh *.rpm
```


1. centos-release-openstack-queens
- centos-release-ceph-luminous-1.1-2.el7.centos.noarch.rpm
- centos-release-openstack-queens-1-2.el7.centos.noarch.rpm
- centos-release-qemu-ev-1.0-4.el7.centos.noarch.rpm
- centos-release-storage-common-2-2.el7.centos.noarch.rpm
- centos-release-virt-common-1-1.el7.centos.noarch.rpm






https://buildlogs.centos.org/centos/7/extras/x86_64/centos-release-qemu-ev-1.0-3.el7.centos.noarch.rpm

## Các bước kiểm tra gói
1. Lấy thông tin các gói cài đặt để dựng OPS:
2. Tải các gói mới và các gói Dependency:
    ```
    yumdownloader --destdir=<đường_dẫn_lưu_file> --resolve <tên_gói_cài_đặt>
    ```
    Ví dụ:
    ```
    yumdownloader --destdir=/root/backups-ops-queens/centos-release-openstack-queens/ --resolve centos-release-openstack-queens
    ```

3. Lấy danh sách các file `.rpm`
    ```
    ls -1 > list_package
    ```

4. Check trên node cần backups bằng script:
    - Copy file `list_package` ở bước 3 sang
    - Chạy script:
    ```sh
    #!/bin/bash
    file="đường dẫn file list_package"
    A=""
    while IFS= read -r line
    do
            A=${line::-4}
            if rpm -qa | grep "$A" > /dev/null 2>&1; then {
                    echo "$A" >> file-package.txt
            } else {
                    echo "Not OK" >> file-package.txt
            } fi
    done <"$file"
    ```

5. Check lại `file-package.txt` và xem các gói không cùng phiên bản để xử lý.
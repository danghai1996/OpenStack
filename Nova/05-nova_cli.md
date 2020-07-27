# Các lệnh thường dùng trong Nova

## Flavor
### Lisst flavor
Cú pháp:
```
openstack flavor list

+--------------------------------------+----------+------+------+-----------+-------+-----------+
| ID                                   | Name     |  RAM | Disk | Ephemeral | VCPUs | Is Public |
+--------------------------------------+----------+------+------+-----------+-------+-----------+
| 05440949-b0ab-43df-9d12-2c968d74f69f | NH-Gói A | 1024 |   15 |         0 |     1 | False     |
| 3f9f6432-7868-4015-ad0b-9f41391fe790 | Gói A    |  512 |   15 |         0 |     1 | False     |
| 5535893e-d466-49df-ad05-327e57133e17 | NH-Gói B | 1536 |   20 |         0 |     2 | False     |
+--------------------------------------+----------+------+------+-----------+-------+-----------+
```

### Tạo mới 1 flavor
Cú pháp:
```
openstack flavor create
    [--id <id>]
    [--ram <size-mb>]
    [--disk <size-gb>]
    [--ephemeral-disk <size-gb>]
    [--swap <size-mb>]
    [--vcpus <num-cpu>]
    [--rxtx-factor <factor>]
    [--public | --private]
    [--property <key=value> [...] ]
    [--project <project>]
    [--project-domain <project-domain>]
    <flavor-name>
```

**Lưu ý:** Nên đặt tên flavor viết liền, không dấu

Ví dụ:
```
openstack flavor create --id auto \
--ram 2048 \
--disk 20 \
--vcpus 2 \
--public \
HaiDD-Test-A

+----------------------------+--------------------------------------+
| Field                      | Value                                |
+----------------------------+--------------------------------------+
| OS-FLV-DISABLED:disabled   | False                                |
| OS-FLV-EXT-DATA:ephemeral  | 0                                    |
| disk                       | 20                                   |
| id                         | d477398a-8691-4de4-a2c0-f3283b625f4f |
| name                       | HaiDD-Test-A                         |
| os-flavor-access:is_public | True                                 |
| properties                 |                                      |
| ram                        | 2048                                 |
| rxtx_factor                | 1.0                                  |
| swap                       |                                      |
| vcpus                      | 2                                    |
+----------------------------+--------------------------------------+
```

### Show chi tiết một flavor
Cú pháp
```
openstack flavor show <tên hoặc ID của flavor>
```

Ví dụ:
```
openstack flavor show HaiDD-Test-A
+----------------------------+--------------------------------------+
| Field                      | Value                                |
+----------------------------+--------------------------------------+
| OS-FLV-DISABLED:disabled   | False                                |
| OS-FLV-EXT-DATA:ephemeral  | 0                                    |
| access_project_ids         | None                                 |
| disk                       | 20                                   |
| id                         | d477398a-8691-4de4-a2c0-f3283b625f4f |
| name                       | HaiDD-Test-A                         |
| os-flavor-access:is_public | True                                 |
| properties                 |                                      |
| ram                        | 2048                                 |
| rxtx_factor                | 1.0                                  |
| swap                       |                                      |
| vcpus                      | 2                                    |
+----------------------------+--------------------------------------+
```

### Xóa flavor
Cú pháp:
```
openstack flavor delete <tên hoặc ID của flavor>
```

Ví dụ:
```
openstack flavor delete HaiDD-Test-B
```

### Update thuộc tính flavor
Cú pháp:
```
openstack flavor set
    [--no-property]
    [--property <key=value> [...] ]
    [--project <project>]
    [--project-domain <project-domain>]
    <flavor>
```

`--no-property` : Xóa tất cả các thuộc tính

Ví dụ:
```
openstack flavor set --project demo HaiDD-Test-B

openstack flavor show HaiDD-Test-B
+----------------------------+--------------------------------------+
| Field                      | Value                                |
+----------------------------+--------------------------------------+
| OS-FLV-DISABLED:disabled   | False                                |
| OS-FLV-EXT-DATA:ephemeral  | 0                                    |
| access_project_ids         | dd170f6234c14cf9992060c0f67a17cb     |
| disk                       | 10                                   |
| id                         | 08f5eb8f-eb48-41dc-8302-3fcc87efcf36 |
| name                       | HaiDD-Test-B                         |
| os-flavor-access:is_public | False                                |
| properties                 |                                      |
| ram                        | 1024                                 |
| rxtx_factor                | 1.0                                  |
| swap                       |                                      |
| vcpus                      | 1                                    |
+----------------------------+--------------------------------------+
```

`access_project_ids` : Id của project được gán

### Bỏ một thuộc tính
Cú pháp:
```
openstack flavor unset
    [--property <key> [...] ]
    [--project <project>]
    [--project-domain <project-domain>]
    <flavor>
```

Ví dụ:
```
openstack flavor unset --project demo HaiDD-Test-B

openstack flavor show HaiDD-Test-B
+----------------------------+--------------------------------------+
| Field                      | Value                                |
+----------------------------+--------------------------------------+
| OS-FLV-DISABLED:disabled   | False                                |
| OS-FLV-EXT-DATA:ephemeral  | 0                                    |
| access_project_ids         |                                      |
| disk                       | 10                                   |
| id                         | 08f5eb8f-eb48-41dc-8302-3fcc87efcf36 |
| name                       | HaiDD-Test-B                         |
| os-flavor-access:is_public | False                                |
| properties                 |                                      |
| ram                        | 1024                                 |
| rxtx_factor                | 1.0                                  |
| swap                       |                                      |
| vcpus                      | 1                                    |
+----------------------------+--------------------------------------+
```

## Server (VM)
### Tạo VM từ image hoặc volume
Cú pháp:
```
openstack server create
    (--image <image> | --volume <volume>)
    --flavor <flavor>
    [--security-group <security-group>]
    [--key-name <key-name>]
    [--property <key=value>]
    [--file <dest-filename=source-filename>]
    [--user-data <user-data>]
    [--availability-zone <zone-name>]
    [--block-device-mapping <dev-name=mapping>]
    [--nic <net-id=net-uuid,v4-fixed-ip=ip-addr,v6-fixed-ip=ip-addr,port-id=port-uuid,auto,none>]
    [--network <network>]
    [--port <port>]
    [--hint <key=value>]
    [--config-drive <config-drive-volume>|True]
    [--min <count>]
    [--max <count>]
    [--wait]
    <server-name>
```

**Ví dụ:** Ta tạo VM từ image với cloud-init config:
```
cat cloudconfig
```
```
#cloud-config
password: hai1996
chpasswd: {expire: False}
ssh_pwauth: True
```

Tạo VM với password truyền vào từ file cloud-config
```
openstack server create --flavor HaiDD-Test-A \
--image cirros \
--nic net-id=3650e62e-b6eb-4067-98e7-ff0fc7f90319 \
--security-group d2307a7d-f37d-404e-8270-d854aa5280dc \
--user-data cloudconfig \
HaiDD-Test-VM01
```

### List server
Cú pháp:
```
openstack server list
+--------------------------------------+-----------------+--------+-----------------------+-----------------+--------------+
| ID                                   | Name            | Status | Networks              | Image           | Flavor       |
+--------------------------------------+-----------------+--------+-----------------------+-----------------+--------------+
| 0415fc81-732a-4b5f-b881-b85912cab6a7 | HaiDD-Test-VM01 | ACTIVE | public01=10.10.32.173 | cirros          | HaiDD-Test-A |
| 034e57d8-05ca-46b1-9f60-ab3833ddfa88 | vm01u20         | ACTIVE | public01=10.10.32.180 | OPS_Ubuntu_2004 | NH-Gói A     |
+--------------------------------------+-----------------+--------+-----------------------+-----------------+--------------+
```

### Show thông tin 1 VM
Cú pháp:
```
openstack server show [--diagnostics] <tên hoặc ID của VM>
```

`--diagnostics` : Hiển thị thông tin chẩn đoán của VM

### start, stop, reboot, suspend, resume VM
Cú pháp:
```
openstack <start|stop|reboot|suspend|resume VM> <tên hoặc ID máy>
```

### Xóa VM
Cú pháp:
```
openstack server delete [--wait] <tên hoặc ID server>
```

`--wait` : Wait for delete to complete

### List danh sách Hypervisor
Cú pháp:
```
openstack hypervisor list
```

Ví dụ:
```
openstack hypervisor list
+----+---------------------+-----------------+--------------+-------+
| ID | Hypervisor Hostname | Hypervisor Type | Host IP      | State |
+----+---------------------+-----------------+--------------+-------+
|  1 | compute2            | QEMU            | 10.10.31.168 | up    |
|  2 | compute1            | QEMU            | 10.10.31.167 | up    |
+----+---------------------+-----------------+--------------+-------+
```

### Snapshot
Bản **Train** thay đổi câu lệnh snapshot, nó sẽ yêu cầu nhập thông tin volume chứ không còn là VM như các bản trước.

**Tham khảo:** [Train - Volume snapshot](https://docs.openstack.org/python-openstackclient/latest/cli/command-objects/volume-snapshot.html)

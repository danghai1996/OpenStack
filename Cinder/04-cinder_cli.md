# Các lệnh thường dùng với Cinder

## 1. create volume
### Tạo 1 volume no-source
```
openstack volume create --size <dung lượng volume- tính theo GB> <tên volume>
```
Ví dụ: 
```
openstack volume create --size 10 vlm_haidd

+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| attachments         | []                                   |
| availability_zone   | nova                                 |
| bootable            | false                                |
| consistencygroup_id | None                                 |
| created_at          | 2020-07-29T07:09:59.000000           |
| description         | None                                 |
| encrypted           | False                                |
| id                  | 8c6cfb4c-a969-48e0-9043-24630b176833 |
| migration_status    | None                                 |
| multiattach         | False                                |
| name                | vlm_haidd                            |
| properties          |                                      |
| replication_status  | None                                 |
| size                | 10                                   |
| snapshot_id         | None                                 |
| source_volid        | None                                 |
| status              | creating                             |
| type                | __DEFAULT__                          |
| updated_at          | None                                 |
| user_id             | 294c5c6181d442c68a13d5b615c4f031     |
+---------------------+--------------------------------------+
```

### Tạo 1 volume từ image
```
openstack volume create --size <dung lượng volume> --image <tên hoặc ID của image> <tên volume>
```
Ví dụ:
```
openstack volume create --size 5 --image cirros vlm01-cirros
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| attachments         | []                                   |
| availability_zone   | nova                                 |
| bootable            | false                                |
| consistencygroup_id | None                                 |
| created_at          | 2020-07-29T07:14:10.000000           |
| description         | None                                 |
| encrypted           | False                                |
| id                  | f9871b5a-4a73-4ed9-b934-b82b1659583a |
| migration_status    | None                                 |
| multiattach         | False                                |
| name                | vlm01-cirros                         |
| properties          |                                      |
| replication_status  | None                                 |
| size                | 5                                    |
| snapshot_id         | None                                 |
| source_volid        | None                                 |
| status              | creating                             |
| type                | __DEFAULT__                          |
| updated_at          | None                                 |
| user_id             | 294c5c6181d442c68a13d5b615c4f031     |
+---------------------+--------------------------------------+
```

### Tạo một volume từ 1 volume khác
```
openstack volume create --source <tên volume-source> --size <dung lượng volume> <tên volume>
```

Ví dụ:
```
openstack volume create --source vlm01-cirros --size 6 vlm02-cirros

+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| attachments         | []                                   |
| availability_zone   | nova                                 |
| bootable            | true                                 |
| consistencygroup_id | None                                 |
| created_at          | 2020-07-29T07:20:31.000000           |
| description         | None                                 |
| encrypted           | False                                |
| id                  | e085eb47-503b-4797-9859-7170c8a8c651 |
| migration_status    | None                                 |
| multiattach         | False                                |
| name                | vlm02-cirros                         |
| properties          |                                      |
| replication_status  | None                                 |
| size                | 6                                    |
| snapshot_id         | None                                 |
| source_volid        | f9871b5a-4a73-4ed9-b934-b82b1659583a |
| status              | creating                             |
| type                | __DEFAULT__                          |
| updated_at          | None                                 |
| user_id             | 294c5c6181d442c68a13d5b615c4f031     |
+---------------------+--------------------------------------+
```

### Tạo volume từ một bản snapshot
```
openstack volume create --snapshot <tên snapshot-source> --size <dung lượng volume> <tên volume>
```
Ví dụ:
```
openstack volume create --snapshot snap01-vlm02-cirros --size 7 vlm03-cirros
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| attachments         | []                                   |
| availability_zone   | nova                                 |
| bootable            | true                                 |
| consistencygroup_id | None                                 |
| created_at          | 2020-07-29T07:41:32.000000           |
| description         | None                                 |
| encrypted           | False                                |
| id                  | 4af6562f-31aa-4768-a209-37c276d893ac |
| migration_status    | None                                 |
| multiattach         | False                                |
| name                | vlm03-cirros                         |
| properties          |                                      |
| replication_status  | None                                 |
| size                | 7                                    |
| snapshot_id         | 016b27f8-dee6-4e63-b2af-02ecd23d2967 |
| source_volid        | None                                 |
| status              | creating                             |
| type                | __DEFAULT__                          |
| updated_at          | None                                 |
| user_id             | 294c5c6181d442c68a13d5b615c4f031     |
+---------------------+--------------------------------------+
```

## 2. List volume, show volume
### List volume
```
openstack volume list
```
Ví dụ:
```
+--------------------------------------+--------------+-----------+------+-------------+
| ID                                   | Name         | Status    | Size | Attached to |
+--------------------------------------+--------------+-----------+------+-------------+
| e085eb47-503b-4797-9859-7170c8a8c651 | vlm02-cirros | available |    6 |             |
| f9871b5a-4a73-4ed9-b934-b82b1659583a | vlm01-cirros | available |    5 |             |
| 8c6cfb4c-a969-48e0-9043-24630b176833 | vlm_haidd    | available |   10 |             |
+--------------------------------------+--------------+-----------+------+-------------+
```

### Show volume
```
openstack volume show <tên hoặc ID volume>
```
Ví dụ:
```
openstack volume show vlm_haidd
+--------------------------------+--------------------------------------+
| Field                          | Value                                |
+--------------------------------+--------------------------------------+
| attachments                    | []                                   |
| availability_zone              | nova                                 |
| bootable                       | false                                |
| consistencygroup_id            | None                                 |
| created_at                     | 2020-07-29T07:09:59.000000           |
| description                    | None                                 |
| encrypted                      | False                                |
| id                             | 8c6cfb4c-a969-48e0-9043-24630b176833 |
| migration_status               | None                                 |
| multiattach                    | False                                |
| name                           | vlm_haidd                            |
| os-vol-host-attr:host          | controller@lvm#LVM                   |
| os-vol-mig-status-attr:migstat | None                                 |
| os-vol-mig-status-attr:name_id | None                                 |
| os-vol-tenant-attr:tenant_id   | 5b4c1d2155004acf849cd3aac03b8f36     |
| properties                     |                                      |
| replication_status             | None                                 |
| size                           | 10                                   |
| snapshot_id                    | None                                 |
| source_volid                   | None                                 |
| status                         | available                            |
| type                           | __DEFAULT__                          |
| updated_at                     | 2020-07-29T07:10:00.000000           |
| user_id                        | 294c5c6181d442c68a13d5b615c4f031     |
+--------------------------------+--------------------------------------+
```

## 3. Snapshot volume
### Tạo snapshot
```
openstack volume snapshot create --volume <tên hoặc ID của volume để snapshot> <tên snapshot>
```
Ví dụ:
```
openstack volume snapshot create --volume vlm01-cirros snap01-vlm01-cirros

+-------------+--------------------------------------+
| Field       | Value                                |
+-------------+--------------------------------------+
| created_at  | 2020-07-29T07:35:38.694356           |
| description | None                                 |
| id          | a789ab04-105f-4195-92d0-383d7237515c |
| name        | snap01-vlm01-cirros                  |
| properties  |                                      |
| size        | 5                                    |
| status      | creating                             |
| updated_at  | None                                 |
| volume_id   | f9871b5a-4a73-4ed9-b934-b82b1659583a |
+-------------+--------------------------------------+
```

### List snapshot
```
openstack volume snapshot list
+--------------------------------------+---------------------+-------------+-----------+------+
| ID                                   | Name                | Description | Status    | Size |
+--------------------------------------+---------------------+-------------+-----------+------+
| 016b27f8-dee6-4e63-b2af-02ecd23d2967 | snap01-vlm02-cirros | None        | available |    6 |
| a789ab04-105f-4195-92d0-383d7237515c | snap01-vlm01-cirros | None        | available |    5 |
+--------------------------------------+---------------------+-------------+-----------+------+
```

### Xóa snapshot
```
openstack volume snapshot delete <tên snapshot>
```

## 4. Attach và detach volume cho máy ảo
### Attach volume
```
openstack server add volume <tên VM> <tên volume> --device <tên thiết bị add cho vm>
```
Ví dụ:
```
openstack server add volume vm01-cirros volume-empty2 --device /dev/vdb
```

### Detach volume
```
openstack server remove volume <tên VM> <tên volume>
```
Ví dụ:
```
openstack server remove volume vm01-cirros vlm_haidd
```

## 5. Resize a volume
Để resize volume thì nó cần ở trạng thái `available`. Tức là nếu volume đang được sử dụng thì ta cần detach nó ra khỏi server.

Để resize volume (đã ở trạng thái available): size của volume resize phải lớn hơn size ban đầu.
```
openstack volume set <ID_volume> --size <kích thước>
```

- Resize volume đang attack volume:
    - Với volume bootable (volume này không detach được): Không thực hiện được, nhưng khi chuyển trạng thái về `available` thì có thể resize (thêm dung lượng), thực hiện với volume LVM, sau khi resize, volume sẽ tự động chuyển về trạng thái `In-use`. Khi kiểm tra bằng lệnh `lsblk` thì disk đã thay đổi kích thước, nhưng trên file hệ thống thực chất là chưa. Do thay đổi kích thước volume chứ không phải partition, vậy nên sau khi extent thì cần vào server để mount lại.
    - Với volume non bootable: Tương tự với volume bootable nhưng có thể detach khỏi instance.
- Resize volume đã được deatach, đang trong trạng thái available -> ok
- Sau khi resize volume có phải reboot lại vm không? -> Có


# Tham khảo
- https://docs.openstack.org/cinder/train/cli/cli-manage-volumes.html
- https://docs.openstack.org/python-openstackclient/latest/cli/command-objects/volume-snapshot.html
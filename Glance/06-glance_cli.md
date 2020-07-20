# Các lệnh thường sử dụng với Glance

Có thể thao tác với Glance CLI bằng 2 cách:
- Glance CLI
- Openstack Client CLI


# Glance CLI
## List image
```
glance image-list
+--------------------------------------+---------+
| ID                                   | Name    |
+--------------------------------------+---------+
| 010ffd94-4d26-4dc6-be5b-1a7a31a3686a | cirros  |
| b7d0787f-b2da-4d68-8e7e-b137c2893ed1 | cirros2 |
| e94f97ef-58ec-4eb9-9613-beb02a0063b6 | cirros3 |
+--------------------------------------+---------+
```

## Tạo 1 image
Cú pháp:
```
glance image-create [--architecture <ARCHITECTURE>]
                           [--protected [True|False]] [--name <NAME>]
                           [--instance-uuid <INSTANCE_UUID>]
                           [--min-disk <MIN_DISK>] [--visibility <VISIBILITY>]
                           [--kernel-id <KERNEL_ID>]
                           [--tags <TAGS> [<TAGS> ...]]
                           [--os-version <OS_VERSION>]
                           [--disk-format <DISK_FORMAT>]
                           [--os-distro <OS_DISTRO>] [--id <ID>]
                           [--owner <OWNER>] [--ramdisk-id <RAMDISK_ID>]
                           [--min-ram <MIN_RAM>]
                           [--container-format <CONTAINER_FORMAT>]
                           [--property <key=value>] [--file <FILE>]
                           [--progress]
```

Ví dụ:
```
glance image-create --disk-format qcow2 --container-format bare --file cirros-0.4.0-x86_64-disk.img --name cirros-test

+------------------+----------------------------------------------------------------------------------+
| Property         | Value                                                                            |
+------------------+----------------------------------------------------------------------------------+
| checksum         | 443b7623e27ecf03dc9e01ee93f67afe                                                 |
| container_format | bare                                                                             |
| created_at       | 2020-07-20T02:25:21Z                                                             |
| disk_format      | qcow2                                                                            |
| id               | 12ec0a8a-5e9b-4a89-bcf6-458f6bca5b0b                                             |
| min_disk         | 0                                                                                |
| min_ram          | 0                                                                                |
| name             | cirros-test                                                                      |
| os_hash_algo     | sha512                                                                           |
| os_hash_value    | 6513f21e44aa3da349f248188a44bc304a3653a04122d8fb4535423c8e1d14cd6a153f735bb0982e |
|                  | 2161b5b5186106570c17a9e58b64dd39390617cd5a350f78                                 |
| os_hidden        | False                                                                            |
| owner            | 5b4c1d2155004acf849cd3aac03b8f36                                                 |
| protected        | False                                                                            |
| size             | 12716032                                                                         |
| status           | active                                                                           |
| tags             | []                                                                               |
| updated_at       | 2020-07-20T02:25:22Z                                                             |
| virtual_size     | Not available                                                                    |
| visibility       | shared                                                                           |
+------------------+----------------------------------------------------------------------------------+
```

## Show image
Cú pháp:
```
glance image-show [--human-readable] [--max-column-width <integer>] <IMAGE_ID>
```

Ví dụ:
```
glance image-show 12ec0a8a-5e9b-4a89-bcf6-458f6bca5b0b
+------------------+----------------------------------------------------------------------------------+
| Property         | Value                                                                            |
+------------------+----------------------------------------------------------------------------------+
| checksum         | 443b7623e27ecf03dc9e01ee93f67afe                                                 |
| container_format | bare                                                                             |
| created_at       | 2020-07-20T02:25:21Z                                                             |
| disk_format      | qcow2                                                                            |
| id               | 12ec0a8a-5e9b-4a89-bcf6-458f6bca5b0b                                             |
| min_disk         | 0                                                                                |
| min_ram          | 0                                                                                |
| name             | cirros-test                                                                      |
| os_hash_algo     | sha512                                                                           |
| os_hash_value    | 6513f21e44aa3da349f248188a44bc304a3653a04122d8fb4535423c8e1d14cd6a153f735bb0982e |
|                  | 2161b5b5186106570c17a9e58b64dd39390617cd5a350f78                                 |
| os_hidden        | False                                                                            |
| owner            | 5b4c1d2155004acf849cd3aac03b8f36                                                 |
| protected        | False                                                                            |
| size             | 12716032                                                                         |
| status           | active                                                                           |
| tags             | []                                                                               |
| updated_at       | 2020-07-20T02:25:22Z                                                             |
| virtual_size     | Not available                                                                    |
| visibility       | shared                                                                           |
+------------------+----------------------------------------------------------------------------------+
```

## Upload image
Cú pháp:
```
glance image-upload [--file <FILE>] [--size <IMAGE_SIZE>] [--progress]
                           <IMAGE_ID>
```

Để upload, ta cần có 1 image rỗng trước. Tạo image rỗng :
```
glance image-create --name cirros-upload --container-format bare --disk-format qcow2
+------------------+--------------------------------------+
| Property         | Value                                |
+------------------+--------------------------------------+
| checksum         | None                                 |
| container_format | bare                                 |
| created_at       | 2020-07-20T02:36:18Z                 |
| disk_format      | qcow2                                |
| id               | aa28146b-62a1-478f-99a1-44d558998b88 |
| min_disk         | 0                                    |
| min_ram          | 0                                    |
| name             | cirros-upload                        |
| os_hash_algo     | None                                 |
| os_hash_value    | None                                 |
| os_hidden        | False                                |
| owner            | 5b4c1d2155004acf849cd3aac03b8f36     |
| protected        | False                                |
| size             | None                                 |
| status           | queued                               |
| tags             | []                                   |
| updated_at       | 2020-07-20T02:36:18Z                 |
| virtual_size     | Not available                        |
| visibility       | shared                               |
+------------------+--------------------------------------+
```
Tiến hành upload image
```
glance image-upload --file cirros-0.4.0-x86_64-disk.img  --progress aa28146b-62a1-478f-99a1-44d558998b88
[=============================>] 100%
```

## Thay đổi trạng thái image
### Deactive image:
```
glance image-deactivate <IMAGE_ID>
```
Ví dụ:
```
glance image-deactivate e94f97ef-58ec-4eb9-9613-beb02a0063b6

openstack image list
+--------------------------------------+---------------+-------------+
| ID                                   | Name          | Status      |
+--------------------------------------+---------------+-------------+
| 010ffd94-4d26-4dc6-be5b-1a7a31a3686a | cirros        | active      |
| 12ec0a8a-5e9b-4a89-bcf6-458f6bca5b0b | cirros-test   | active      |
| aa28146b-62a1-478f-99a1-44d558998b88 | cirros-upload | active      |
| b7d0787f-b2da-4d68-8e7e-b137c2893ed1 | cirros2       | active      |
| e94f97ef-58ec-4eb9-9613-beb02a0063b6 | cirros3       | deactivated |
+--------------------------------------+---------------+-------------+
```

### Active image
```
glance image-reactivate <IMAGE_ID>
```

Ví dụ:
```
glance image-reactivate e94f97ef-58ec-4eb9-9613-beb02a0063b6

openstack image list
+--------------------------------------+---------------+--------+
| ID                                   | Name          | Status |
+--------------------------------------+---------------+--------+
| 010ffd94-4d26-4dc6-be5b-1a7a31a3686a | cirros        | active |
| 12ec0a8a-5e9b-4a89-bcf6-458f6bca5b0b | cirros-test   | active |
| aa28146b-62a1-478f-99a1-44d558998b88 | cirros-upload | active |
| b7d0787f-b2da-4d68-8e7e-b137c2893ed1 | cirros2       | active |
| e94f97ef-58ec-4eb9-9613-beb02a0063b6 | cirros3       | active |
+--------------------------------------+---------------+--------+
```

## Xóa image
```
glance image-delete <IMAGE_ID> [<IMAGE_ID> ...]
```

Ví dụ:
```
openstack image list
+--------------------------------------+---------------+--------+
| ID                                   | Name          | Status |
+--------------------------------------+---------------+--------+
| 010ffd94-4d26-4dc6-be5b-1a7a31a3686a | cirros        | active |
| 12ec0a8a-5e9b-4a89-bcf6-458f6bca5b0b | cirros-test   | active |
| aa28146b-62a1-478f-99a1-44d558998b88 | cirros-upload | active |
| b7d0787f-b2da-4d68-8e7e-b137c2893ed1 | cirros2       | active |
| e94f97ef-58ec-4eb9-9613-beb02a0063b6 | cirros3       | active |
+--------------------------------------+---------------+--------+

glance image-delete e94f97ef-58ec-4eb9-9613-beb02a0063b6 b7d0787f-b2da-4d68-8e7e-b137c2893ed1 aa28146b-62a1-478f-99a1-44d558998b88 12ec0a8a-5e9b-4a89-bcf6-458f6bca5b0b

openstack image list
+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| 010ffd94-4d26-4dc6-be5b-1a7a31a3686a | cirros | active |
+--------------------------------------+--------+--------+
```

# Openstack CLI
## List image
```
openstack image list
```
Có thể dùng kèm với `grep` để lọc

Ví dụ:
```
openstack image list | grep 'cirros'

| 010ffd94-4d26-4dc6-be5b-1a7a31a3686a | cirros   | active |
| 84d9441a-fab9-4ece-bd9f-14cb28764c67 | cirros-2 | active |
```

## Show image
```
openstack image show [--human-readable] <ID | image_name>
```

- `--human-readable` : định dạng kích thước của image dễ đọc

## Tạo image
Cú pháp
```
openstack image create
    [--id <id>]
    [--container-format <container-format>]
    [--disk-format <disk-format>]
    [--min-disk <disk-gb>]
    [--min-ram <ram-mb>]
    [--file <file> | --volume <volume>]
    [--force]
    [--sign-key-path <sign-key-path>]
    [--sign-cert-id <sign-cert-id>]
    [--protected | --unprotected]
    [--public | --private | --community | --shared]
    [--property <key=value>]
    [--tag <tag>]
    [--project <project>]
    [--import]
    [--project-domain <project-domain>]
    <image-name>
```

Ví dụ:
```
openstack image create "cirros-2" --file cirros-0.4.0-x86_64-disk.img --disk-format qcow2 --container-format bare --public
```

## Xóa image
```
openstack image delete <ID image | image_name>
```

## Update thông tin image
```
openstack image set
    [--name <name>]
    [--min-disk <disk-gb>]
    [--min-ram <ram-mb>]
    [--container-format <container-format>]
    [--disk-format <disk-format>]
    [--protected | --unprotected]
    [--public | --private | --community | --shared]
    [--property <key=value>]
    [--tag <tag>]
    [--architecture <architecture>]
    [--instance-id <instance-id>]
    [--kernel-id <kernel-id>]
    [--os-distro <os-distro>]
    [--os-version <os-version>]
    [--ramdisk-id <ramdisk-id>]
    [--deactivate | --activate]
    [--project <project>]
    [--project-domain <project-domain>]
    [--accept | --reject | --pending]
    <image>
```

## Thêm project cho image
```
openstack image add project
    [--project-domain <project-domain>]
    <image>
    <project>
```

- `--project-domain <project-domain>` : domain (tên hoặc ID)
- `<image>` : image ở trạng thái shared (tên hoặc ID)
- `<project>` : project (tên hoặc ID)

Để thêm project cho image, thì image phải ở trạng thái `shared`

- Tạo 1 image dạng `share`:
    ```
    openstack image create "cirros-2" --file cirros-0.4.0-x86_64-disk.img --disk-format qcow2 --container-format bare --shared
    ```

- Thêm project cho image:
    ```
    openstack image add project \
    cirros-2 \
    demo
    +------------+--------------------------------------+
    | Field      | Value                                |
    +------------+--------------------------------------+
    | created_at | 2020-07-20T04:05:09Z                 |
    | image_id   | c15fe381-0353-47f5-9bd7-b553c579c476 |
    | member_id  | dd170f6234c14cf9992060c0f67a17cb     |
    | schema     | /v2/schemas/member                   |
    | status     | pending                              |
    | updated_at | 2020-07-20T04:05:09Z                 |
    +------------+--------------------------------------+
    ```

Để remove, ta đổi `add` -> `remove`
```
openstack image remove project
    [--project-domain <project-domain>]
    <image>
    <project>
```

## Image member list
List những project được gán với image:
```
openstack image member list
    [--sort-column SORT_COLUMN]
    [--project-domain <project-domain>]
    <image>
```

Ví dụ:
```
openstack image member list cirros-2
+--------------------------------------+----------------------------------+---------+
| Image ID                             | Member ID                        | Status  |
+--------------------------------------+----------------------------------+---------+
| c15fe381-0353-47f5-9bd7-b553c579c476 | dd170f6234c14cf9992060c0f67a17cb | pending |
+--------------------------------------+----------------------------------+---------+
```
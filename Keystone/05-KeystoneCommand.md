# Command

# I. Khởi tạo biến môi trường
File này chứa thông tin credential (thông tin đăng nhập)

Tạo file khởi tạo biến môi trường:(tên file nên đặt theo user cho dễ nhớ)
```
vi admin-openrc
```
Nội dung:
```
export OS_USERNAME=admin
export OS_PASSWORD=Welcome123
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://10.10.31.166:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```

Khởi chạy môi trường:
```
source <đường_dẫn_thư_mục>/admin-openrc
```

Kiểm tra biến môi trường
```
env | grep OS
```

Check token
```
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                                                                                   |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2020-07-04T07:58:36+0000                                                                                                                                                                |
| id         | gAAAAABfACicZM9cL0bs1kXi3fVkgWmcJqY1DB8ryzdS6wC-wKLKXpdCSns-qf0uMBOVEJcSOFPi-U1lFY40qx_39iUZyhMStip_K29F4AzCoq0EApkWApknvaIbBIDpLZn55mL4FN7dPEarnQoMSh6b_gK3x7JOBBmoki_jkK3ifLJQbFN6ASU |
| project_id | 5b4c1d2155004acf849cd3aac03b8f36                                                                                                                                                        |
| user_id    | 294c5c6181d442c68a13d5b615c4f031                                                                                                                                                        |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

# II. Các lệnh làm việc với user, project, groups, roles, domain
## 1. User
### List users
Liệt kê tất cả user
```
openstack user list
```
```
+----------------------------------+-----------+
| ID                               | Name      |
+----------------------------------+-----------+
| 294c5c6181d442c68a13d5b615c4f031 | admin     |
| 8db90f3de7a54375aa16eb1d0626f1bb | demo      |
| afe0224c90f84b3b83a8d9788921ccef | glance    |
| f30b4ed7a5a54ea68d98aeee438bb80f | placement |
| 4be5d5ed900f4386a5f9268927c46ecb | nova      |
| 74cf804695e44ae4995f9579446ae813 | neutron   |
| 4f28c17b5ad7451b859b223e65755eff | haidd     |
| 147cd98e7a08481285ac72c2394c2c93 | hai       |
| 4c1e3821baea48dcbc314be7231ebbf1 | hai1      |
+----------------------------------+-----------+
```

List user trong 1 group:
```
openstack user list --group <group_name>
```

### Create user
```
openstack user create <user_name> --password <password>
```
Ví dụ: Tạo user `test`
```
openstack user create test --password Welcome123
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | f8b96ef15a6d4ff5b4f87dfdeeb96a8a |
| name                | test                             |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

### Update user
Ta có thể cập nhật tên, email, và enable status cho user
1. Show thông tin user
    ```
    openstack user show <user>
    ```

2. Tạm thời vô hiệu hóa user
    ```
    openstack user set <user> --disable
    ```

3. Enable user
    ```
    openstack user set <user> --enable
    ```

4. Thay đổi tên và mô tả cho tài khoản người dùng
    ```
    openstack user set <user> --name <user-new>
    ```

### Xóa user
```
openstack user delete <user>
```

## 2. Project
### List project
Liệt kê các project với lệnh sau:
```
openstack project list
```
```
+----------------------------------+---------+
| ID                               | Name    |
+----------------------------------+---------+
| 5b4c1d2155004acf849cd3aac03b8f36 | admin   |
| 9275d103960e432fa03e2644d03af368 | haidd   |
| af57453a686740f18e48a8c4cf4ac994 | service |
| dd170f6234c14cf9992060c0f67a17cb | demo    |
+----------------------------------+---------+
```

### Tạo project
```
openstack project create --description '<Description_Project>' <project_name> \
--domain <domain_name>
```
Ví dụ:
```
openstack project create --description 'Test Project' test --domain default
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Test Project                     |
| domain_id   | default                          |
| enabled     | True                             |
| id          | be0e711e08364a6291b5481388cddfaa |
| is_domain   | False                            |
| name        | test                             |
| options     | {}                               |
| parent_id   | default                          |
| tags        | []                               |
+-------------+----------------------------------+
```

### Update project
Để cập nhật project bạn cần có ID hoặc tên của project
1. Disable project
    ```
    openstack project set <project_ID || project_name> --disable
    ```

2. Enable project
    ```
    openstack project set <project_ID || project_name> --enable
    ```

3. Update tên cho project
    ```
    openstack project set <project_ID || project_name> --name <project_newname>
    ```

4. Xác nhận các thông tin của project
    ```
    openstack project show <PROJECT_ID || project_name>
    ```

### Delete project
```
openstack project delete <project_ID || project_name>
```

## 3. Group
### Tạo group
```
openstack group create
    [--domain <domain>]
    [--description <description>]
    [--or-show]
    <group-name>
```
Ví dụ:
```
openstack group create --domain default \
> --domain default \
> --description 'Group cua HaiDD' \
> --or-show \
> gr_haidd
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Group cua HaiDD                  |
| domain_id   | default                          |
| id          | e403dc73b63e4a1088ac6c31903ba2e1 |
| name        | gr_haidd                         |
+-------------+----------------------------------+
```

### List group
```
openstack group list
```

### Group add user
Thêm user vào group
```
openstack group add user
    [--group-domain <group-domain>]
    [--user-domain <user-domain>]
    <group>
    <user>
    [<user> ...]
```
Ví dụ:
```
openstack group add user \
--group-domain default \
--user-domain default \
gr_haidd \
test
```
Kiểm tra user:
```
openstack user list --group gr_haidd
+----------------------------------+------+
| ID                               | Name |
+----------------------------------+------+
| 1e0129b374e64cc5bf559f68c4b8dd6c | test |
+----------------------------------+------+
```

### Kiểm tra user có nằm trong group không
```
openstack group contains user
    [--group-domain <group-domain>]
    [--user-domain <user-domain>]
    <group>
    <user>
```
Ví dụ:
```
openstack group contains user \
--group-domain default \
--user-domain default \
gr_haidd \
test

test in group gr_haidd
```

### Remove user
Remove user ra khỏi group
```
openstack group remove user
    [--group-domain <group-domain>]
    [--user-domain <user-domain>]
    <group>
    <user> [<user> ...]
```

Ví dụ:
```
openstack group remove user \
> --group-domain default \
> --user-domain default \
> gr_haidd \
> test
```
Kiểm tra lại user `test`
```
openstack group contains user \
--group-domain default \
--user-domain default \
gr_haidd \
test

test not in group gr_haidd
```

### Group set
Ta có thể đặt lại các thuộc tính của Group
```
openstack group set
    [--domain <domain>]
    [--name <name>]
    [--description <description>]
    <group>
```

### Group show
Hiển thị thông tin chi tiết Group
```
openstack group show
    [--domain <domain>]
    <group>
```

## 4. Role
### List avaiable roles
```
openstack role list
```
Ví dụ
```
openstack role list
+----------------------------------+--------+
| ID                               | Name   |
+----------------------------------+--------+
| 6edcf71488424352817939a8505267b1 | admin  |
| b072025f81b0479eb6907dd47d5810d1 | user   |
| e62c83155a654bd4a31b4bc692b835f5 | reader |
| f1b4e02022d4444ba29769ec76146a23 | member |
+----------------------------------+--------+
```

### Create a role
Để gán người dùng cho nhiều project, hay xác định vai role cho user với các project
```
openstack role create <role_name>
```
Ví dụ:
```
openstack role create haidd
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | None                             |
| domain_id   | None                             |
| id          | 323e4ddaa63d4a1397fa466dfc864a21 |
| name        | haidd                            |
| options     | {}                               |
+-------------+----------------------------------+
```
**Chú ý:** Nếu sử dụng Identity v3 thì cần có option `--domain` với một domain cụ thể

### Assign a role
Để assingn user cho project, ta cần gán role theo cặp user-project. Để làm điều đó, ta cần user ID, role và projectID

1. List user và để ý userID mà ta muốn sử dụng
    ```
    openstack user list
    +----------------------------------+-----------+
    | ID                               | Name      |
    +----------------------------------+-----------+
    | 294c5c6181d442c68a13d5b615c4f031 | admin     |
    | 8db90f3de7a54375aa16eb1d0626f1bb | demo      |
    | afe0224c90f84b3b83a8d9788921ccef | glance    |
    | f30b4ed7a5a54ea68d98aeee438bb80f | placement |
    | 4be5d5ed900f4386a5f9268927c46ecb | nova      |
    | 74cf804695e44ae4995f9579446ae813 | neutron   |
    | 1e0129b374e64cc5bf559f68c4b8dd6c | test      |
    +----------------------------------+-----------+
    ```

2. List Role ID 
    ```
    openstack role list
    +----------------------------------+--------+
    | ID                               | Name   |
    +----------------------------------+--------+
    | 323e4ddaa63d4a1397fa466dfc864a21 | haidd  |
    | 6edcf71488424352817939a8505267b1 | admin  |
    | b072025f81b0479eb6907dd47d5810d1 | user   |
    | e62c83155a654bd4a31b4bc692b835f5 | reader |
    | f1b4e02022d4444ba29769ec76146a23 | member |
    +----------------------------------+--------+
    ```

3. List project ID
    ```
    openstack project list
    +----------------------------------+---------+
    | ID                               | Name    |
    +----------------------------------+---------+
    | 5b4c1d2155004acf849cd3aac03b8f36 | admin   |
    | 9275d103960e432fa03e2644d03af368 | haidd   |
    | af57453a686740f18e48a8c4cf4ac994 | service |
    | dd170f6234c14cf9992060c0f67a17cb | demo    |
    +----------------------------------+---------+
    ```

4. Assign role
    ```
    openstack role add --user <user> --project <project_name> <role_name>
    ```
    Ví dụ
    ```
    openstack role add --user test --project haidd haidd
    ```

5. Xác nhận lại
    ```
    openstack role assignment list --user USER_NAME \
    --project PROJECT_ID --names
    ```
    Ví dụ:
    ```
    openstack role assignment list --user test --project haidd --names             
    +-------+--------------+-------+---------------+--------+--------+-----------+
    | Role  | User         | Group | Project       | Domain | System | Inherited |
    +-------+--------------+-------+---------------+--------+--------+-----------+
    | haidd | test@Default |       | haidd@Default |        |        | False     |
    +-------+--------------+-------+---------------+--------+--------+-----------+
    ```

### Xem chi tiết role
```
openstack role show ROLE_NAME
```
Ví dụ:
```
openstack role show haidd
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | None                             |
| domain_id   | None                             |
| id          | 323e4ddaa63d4a1397fa466dfc864a21 |
| name        | haidd                            |
| options     | {}                               |
+-------------+----------------------------------+
```

### Remove role
1. Chạy lệnh sau:
    ```
    openstack role remove --user USER_NAME --project TENANT_ID ROLE_NAME
    ```
    Ví dụ:
    ```
    openstack role remove --user test --project haidd haidd
    ```

2. Kiểm tra việc remove
    ```
    openstack role list --user USER_NAME --project TENANT_ID
    ```
    Ví dụ
    ```
    
    ```
--------------

### Tạo domain mới
Cú pháp:
```
openstack domain create <tên_domain>
```
Ví dụ
```
openstack domain create haidd
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description |                                  |
| enabled     | True                             |
| id          | a27d8bfb8135423c80fffff287757733 |
| name        | haidd                            |
| options     | {}                               |
| tags        | []                               |
+-------------+----------------------------------+
```

### Xem các role user đang có
Cú pháp
```
openstack role assignment list --user <user> --name
```
Ví dụ:
```
openstack role assignment list --user haidd --name
+-------+---------------+-------+---------------+--------+--------+-----------+
| Role  | User          | Group | Project       | Domain | System | Inherited |
+-------+---------------+-------+---------------+--------+--------+-----------+
| admin | haidd@Default |       | haidd@Default |        |        | False     |
| user  | haidd@Default |       | haidd@Default |        |        | False     |
+-------+---------------+-------+---------------+--------+--------+-----------+
```
Xem thêm các tùy chọn [tại đây](https://docs.openstack.org/python-openstackclient/latest/cli/command-objects/role-assignment.html)

### Gán role
Cú pháp
```
openstack role add --project <tên project> --project-domain <tên domain> --user <tên user> --user-domain <tên domain> <tên role>
```




# Tham khảo
1. https://docs.openstack.org/python-openstackclient/latest/cli/command-objects/role-assignment.html
2. https://docs.openstack.org/python-openstackclient/latest/cli/command-objects/group.html
3. https://docs.openstack.org/keystone/pike/admin/cli-manage-projects-users-and-roles.html
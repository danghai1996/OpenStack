# Khái niệm Endpoint và Catalog 

# I. Endpoint
- Endpoint trong Keystone là một URL có thể được sử dụng để truy cập dịch vụ trong Openstack
- 1 Endpoint giống như một điểm liên lạc để người dùng sử dụng dịch vụ của Openstack.

## 3 loại Endpoint
1. `adminurl` : Dùng cho người dùng quản trị
2. `internalurl` : Là những gì các dịch vụ khác sử dụng để giao tiếp với nhau
3. `publicurl` : Là những gì mà người khác truy cập vào endpoint sử dụng dịch vụ

## Show list endpoint
```
openstack endpoint list

+----------------------------------+-----------+--------------+--------------+---------+-----------+-------------------------------+
| ID                               | Region    | Service Name | Service Type | Enabled | Interface | URL                           |
+----------------------------------+-----------+--------------+--------------+---------+-----------+-------------------------------+
| 01db1e80d3b540639861c1ee9d0a451f | RegionOne | nova         | compute      | True    | public    | http://10.10.31.166:8774/v2.1 |
| 0bae90c2cb9b4ca0a5f812da2273fb79 | RegionOne | glance       | image        | True    | admin     | http://10.10.31.166:9292      |
| 1354449c815d4f499d88cfc4f8f785b8 | RegionOne | keystone     | identity     | True    | public    | http://10.10.31.166:5000/v3/  |
| 2aad8b16f9534fa18fceaccc50fc927d | RegionOne | neutron      | network      | True    | internal  | http://10.10.31.166:9696      |
| 4004a42543744831971c53b85956450d | RegionOne | keystone     | identity     | True    | internal  | http://10.10.31.166:5000/v3/  |
| 454981cfd05041edb95971db63aec819 | RegionOne | nova         | compute      | True    | admin     | http://10.10.31.166:8774/v2.1 |
| 634db163e477401393da5675eb9454d9 | RegionOne | keystone     | identity     | True    | admin     | http://10.10.31.166:5000/v3/  |
| 743533c9a69f468590ff6b02a871d07f | RegionOne | glance       | image        | True    | public    | http://10.10.31.166:9292      |
| 8537fd3b514942bebb4e407554137d0f | RegionOne | placement    | placement    | True    | internal  | http://10.10.31.166:8778      |
| 890803754a454947ad2de61b8c89dd10 | RegionOne | placement    | placement    | True    | public    | http://10.10.31.166:8778      |
| 9e8b1c9d8e4e4085bc15ba5d7f91b1bb | RegionOne | nova         | compute      | True    | internal  | http://10.10.31.166:8774/v2.1 |
| acfaf8b4a9034821aad043464de64c29 | RegionOne | glance       | image        | True    | internal  | http://10.10.31.166:9292      |
| d6d5a5ecf36544b794dcd1021ff10769 | RegionOne | placement    | placement    | True    | admin     | http://10.10.31.166:8778      |
| e1d529ec9dc64f36ab3a4799c297ae17 | RegionOne | neutron      | network      | True    | public    | http://10.10.31.166:9696      |
| f0cd24d2520540b895efe81a3d00b437 | RegionOne | neutron      | network      | True    | admin     | http://10.10.31.166:9696      |
+----------------------------------+-----------+--------------+--------------+---------+-----------+-------------------------------+
```

Hiển thị chi tiết một endpoint
```
openstack endpoint show
    <endpoint>
```
`<endpoint>` : endpoint ID, service ID, service name, service type

Ví dụ:
```
openstack endpoint show 01db1e80d3b540639861c1ee9d0a451f
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 01db1e80d3b540639861c1ee9d0a451f |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | ec8b6081186f4922bc67a38b5f148ac2 |
| service_name | nova                             |
| service_type | compute                          |
| url          | http://10.10.31.166:8774/v2.1    |
+--------------+----------------------------------+
```

# II. Catalog
Catalog trong Openstack là tập hợp các danh mục sẵn sàng sử dụng mà khách hàng có thể sử dụng trong OPS.

Service catalog cung cấp cho người dùng về các dịch vụ có sẵn trong OPS, cùng với thông tin bổ sung về các vùng, phiên bản API và các project có sẵn.

Catalog làm cho việc tìm kiếm dịch vụ hiệu quả, chẳng hạn như cách định cấu hình liên lạc giữa các dịch vụ.

## List catalog
```
openstack catalog list
+-----------+-----------+-------------------------------------------+
| Name      | Type      | Endpoints                                 |
+-----------+-----------+-------------------------------------------+
| neutron   | network   | RegionOne                                 |
|           |           |   internal: http://10.10.31.166:9696      |
|           |           | RegionOne                                 |
|           |           |   public: http://10.10.31.166:9696        |
|           |           | RegionOne                                 |
|           |           |   admin: http://10.10.31.166:9696         |
|           |           |                                           |
| glance    | image     | RegionOne                                 |
|           |           |   admin: http://10.10.31.166:9292         |
|           |           | RegionOne                                 |
|           |           |   public: http://10.10.31.166:9292        |
|           |           | RegionOne                                 |
|           |           |   internal: http://10.10.31.166:9292      |
|           |           |                                           |
| placement | placement | RegionOne                                 |
|           |           |   internal: http://10.10.31.166:8778      |
|           |           | RegionOne                                 |
|           |           |   public: http://10.10.31.166:8778        |
|           |           | RegionOne                                 |
|           |           |   admin: http://10.10.31.166:8778         |
|           |           |                                           |
| keystone  | identity  | RegionOne                                 |
|           |           |   public: http://10.10.31.166:5000/v3/    |
|           |           | RegionOne                                 |
|           |           |   internal: http://10.10.31.166:5000/v3/  |
|           |           | RegionOne                                 |
|           |           |   admin: http://10.10.31.166:5000/v3/     |
|           |           |                                           |
| nova      | compute   | RegionOne                                 |
|           |           |   public: http://10.10.31.166:8774/v2.1   |
|           |           | RegionOne                                 |
|           |           |   admin: http://10.10.31.166:8774/v2.1    |
|           |           | RegionOne                                 |
|           |           |   internal: http://10.10.31.166:8774/v2.1 |
|           |           |                                           |
+-----------+-----------+-------------------------------------------+
```

### Show thông tin một mục trong catalog
```
openstack catalog show
    <service>
```
`<service>` : Dịch vụ và bạn muốn hiển thị

Ví dụ:
```
openstack catalog show keystone

+-----------+------------------------------------------+
| Field     | Value                                    |
+-----------+------------------------------------------+
| endpoints | RegionOne                                |
|           |   public: http://10.10.31.166:5000/v3/   |
|           | RegionOne                                |
|           |   internal: http://10.10.31.166:5000/v3/ |
|           | RegionOne                                |
|           |   admin: http://10.10.31.166:5000/v3/    |
|           |                                          |
| id        | 60d6403b4de346a59d92c707b2d4a437         |
| name      | keystone                                 |
| type      | identity                                 |
+-----------+------------------------------------------+
```


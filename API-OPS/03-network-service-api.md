# Phần 3 : Làm việc với Network service API

Làm việc với project neutron

## Yêu cầu
- Đã chứng thực scoped token với quyền trên system, hoặc project. 
- `X-Auth-Token` trên header

## Lấy danh sách network
```
GET: /v2.0/networks
```

Header:
- `Content-Type` : `application/json`
- `X-Auth-Token` : Token

<img src="..\images\api-ops\Screenshot_41.png">

<img src="..\images\api-ops\Screenshot_42.png">

## Xem chi tiết network
```
GET: /v2.0/networks/{network_id}
```

Header:
- `Content-Type` : `application/json`
- `X-Auth-Token` : Token

<img src="..\images\api-ops\Screenshot_43.png">

<img src="..\images\api-ops\Screenshot_44.png">

## Tạo network
```
POST: /v2.0/networks
```

Header:
- `Content-Type` : `application/json`
- `X-Auth-Token` : Token

Body:
- Xem thêm các tùy chọn tại [đây.](https://docs.openstack.org/api-ref/network/v2/index.html?expanded=create-network-detail#networks)

<img src="..\images\api-ops\Screenshot_45.png">

<img src="..\images\api-ops\Screenshot_46.png">

Kết quả:

<img src="..\images\api-ops\Screenshot_47.png">

## Xóa network
```
DELETE: /v2.0/networks/{network_id}
```

Header:
- `Content-Type` : `application/json`
- `X-Auth-Token` : Token

<img src="..\images\api-ops\Screenshot_48.png">

<img src="..\images\api-ops\Screenshot_49.png">

## Lấy danh sách subnet
```
GET: /v2.0/subnets
```

Header:
- `Content-Type` : `application/json`
- `X-Auth-Token` : Token

<img src="..\images\api-ops\Screenshot_50.png">

<img src="..\images\api-ops\Screenshot_51.png">

## Xem chi tiết subnet
```
GET: /v2.0/subnets/{subnet_id}
```

Header:
- `Content-Type` : `application/json`
- `X-Auth-Token` : Token

<img src="..\images\api-ops\Screenshot_52.png">

<img src="..\images\api-ops\Screenshot_53.png">

## Tạo subnet mới
```
POST: /v2.0/subnets
```

Header:
- `Content-Type` : `application/json`
- `X-Auth-Token` : Token

Body:
- `network_id` : ID của Network muốn tạo subnet
- `ip_version` : IP version
- `cidr` : Dải mạng
- [Xem thêm](https://docs.openstack.org/api-ref/network/v2/index.html?expanded=create-network-detail,create-subnet-detail#networks)

<img src="..\images\api-ops\Screenshot_54.png">

<img src="..\images\api-ops\Screenshot_55.png">

Kết quả:

<img src="..\images\api-ops\Screenshot_56.png">

## Xóa subnet
```
DELETE: /v2.0/subnets/{subnet_id}
```

Header:
- `Content-Type` : `application/json`
- `X-Auth-Token` : Token

<img src="..\images\api-ops\Screenshot_57.png">

Kết quả:

<img src="..\images\api-ops\Screenshot_58.png">

# Tham khảo
- https://docs.openstack.org/api-ref/network/v2/index.html
- https://github.com/lacoski/OpenStack-Note/blob/master/docs/ops-api/ops-api.md
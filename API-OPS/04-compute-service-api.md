# Phần 4: Làm việc với Compute Service API

Làm việc với project Nova

API Access: `http://<IP-address>:8774/v2.1`

## Yêu cầu 
- Đã chứng thực scoped token với quyền trên system, hoặc project. 
- `X-Auth-Token` trên Header

# 1. Thao tác với flavor
## Liệt kê danh sách flavor
```
GET: /v2.1/flavors
```

Header:
- `Content-Type` : `application/json`
- `X-Auth-Token` : Token

<img src="..\images\api-ops\Screenshot_59.png">

<img src="..\images\api-ops\Screenshot_60.png">

## Tạo mới flavor
```
POST: /v2.1/flavors
```

Header:
- `Content-Type` : `application/json`
- `X-Auth-Token` : Token

Body:
- [Xem thêm](https://docs.openstack.org/api-ref/compute/?expanded=list-flavors-detail,create-flavor-detail#create-flavor)

<img src="..\images\api-ops\Screenshot_61.png">

<img src="..\images\api-ops\Screenshot_62.png">

## Xem chi tiết 1 flavor
```
GET: /v2.1/flavors/{flavor_id}
```

Header:
- `Content-Type` : `application/json`
- `X-Auth-Token` : Token

<img src="..\images\api-ops\Screenshot_63.png">

<img src="..\images\api-ops\Screenshot_64.png">

## Xóa 1 flavor
```
DELETE: /v2.1/flavors/{flavor_id}
```

Header:
- `Content-Type` : `application/json`
- `X-Auth-Token` : Token

<img src="..\images\api-ops\Screenshot_65.png">

<img src="..\images\api-ops\Screenshot_66.png">

# 2. Thao tác với VM
## Liệt kê các VM
```
GET: /v2.1/servers
```

Header:
- `Content-Type` : `application/json`
- `X-Auth-Token` : Token

<img src="..\images\api-ops\Screenshot_70.png">

<img src="..\images\api-ops\Screenshot_71.png">

## Tạo VM
```
POST: /v2.1/servers
```

Header:
- `Content-Type` : `application/json`
- `X-Auth-Token` : Token

Body:
- [Xem thêm](https://docs.openstack.org/api-ref/compute/?expanded=create-server-detail#create-server)

<img src="..\images\api-ops\Screenshot_67.png">

<img src="..\images\api-ops\Screenshot_68.png">

Kết quả:

7d6e6bad-97f3-4cab-b909-21a472cd2178

## Xem chi tiết thông tin server
```
GET: /v2.1/servers/{server_id}
```

Header:
- `Content-Type` : `application/json`
- `X-Auth-Token` : Token

<img src="..\images\api-ops\Screenshot_72.png">

<img src="..\images\api-ops\Screenshot_73.png">

## Delete server
```
DELETE: /v2.1/servers/{server_id}
```

Header:
- `Content-Type` : `application/json`
- `X-Auth-Token` : Token

<img src="..\images\api-ops\Screenshot_74.png">

<img src="..\images\api-ops\Screenshot_75.png">

# Tham khảo
- https://docs.openstack.org/api-ref/compute/
- https://github.com/lacoski/OpenStack-Note/blob/master/docs/ops-api/ops-api.md
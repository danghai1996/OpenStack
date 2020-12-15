# Phần 2: Làm việc với image service API

- Làm việc với project glance

# Yêu cầu:
- Đã chứng thực scoped token với quyền trên system, hoặc project. X-Auth-Token trên header

## Liệt kê danh sách image
```
GET: /v2/images
```

Header:
- Tham số: `X-Auth-Token`

### Ví dụ:

<img src="..\images\api-ops\Screenshot_30.png">

Kết quả

<img src="..\images\api-ops\Screenshot_31.png">

## Xem thông tin chi tiết image
```
GET: /v2/images/{image_id}
```

Header:
- Tham số: `X-Auth-Token`

### Ví dụ:
<img src="..\images\api-ops\Screenshot_32.png">

Kết quả

<img src="..\images\api-ops\Screenshot_33.png">

## Delete image
```
DELETE: /v2/images/{image_id}
```

Header:
- Tham số: `X-Auth-Token`

### Ví dụ:
<img src="..\images\api-ops\Screenshot_34.png">

Kết quả

Không có nội dung trong phần `Body`

<img src="..\images\api-ops\Screenshot_35.png">

## Tạo mới image
2 bước để tạo image:
- Tạo image trống chưa có file data (Sau khi tạo status dạng queue)
- Upload file data (Sau khi upload image status chuyển sang dạng active)

### Bước 1: Tạo khung image
```
POST: /v2/images
```

**Header:**
- Cần `X-Auth-Token` tại header

**Body:**
- `container_format` : kiểu container của image
- `disk_format` : định dạng image
- `name` : tên image

```json
{
    "container_format": "bare",
    "disk_format": "qcow2",
    "name": "cirrosfromapi"
}
```

<img src="..\images\api-ops\Screenshot_36.png">

<img src="..\images\api-ops\Screenshot_37.png">

### Bước 2: Upload image:
```
PUT: /v2/images/{image_id}/file
```

Header:
- `Content-type` : Media type cho request body. Sử dụng `application/octet-stream`
- `X-auth-token` : Token

Body:
- Đường dẫn của image

<img src="..\images\api-ops\Screenshot_38.png">

<img src="..\images\api-ops\Screenshot_39.png">

<img src="..\images\api-ops\Screenshot_40.png">

# Tham khảo
- https://github.com/lacoski/OpenStack-Note/blob/master/docs/ops-api/ops-api.md
- https://docs.openstack.org/api-ref/image/v2/index.html
# Phần 1: Làm việc với Identity API v3 service

## Cơ bản về dịch vụ Identity
Dịch vụ Identity service sử dụng để sinh token. Token tượng trưng cho chứng thực định danh user, tổ chức, quyền hạn trên các project, domain và hệ thống

Có 2 phương thức chứng thực:
- Password
- Token

Trong các token sẽ chứa:
- Credential (Thông tin xác thực)
- Authorization scope (Phạm vi quyền hạn)

Token trả lại bao gồm:
- Token IDs và giá trị X-Subject-Token tại header response

Sau khi có token ta có thể:
- Tạo các REST API tới các dịch vụ OpenStack khác.
- Cần khai báo giá trị X-Auth-Token tại request header
- Validate token, liệt kê danh sách các domain, project, role, endpoint token cho phép truy cập
- Thu hồi token

## Chứng thực password dạng unscoped authorization
```
POST: <IP_CTL>:5000/v3/auth/tokens
```

**Lưu ý:**

- Token dạng unscoped, tức Token không có quyền sử dụng bất kỳ catalog, role, project, hoặc domain. Sử dụng token này đơn giản để chứng thực danh tính tại KeyStone tại 1 số thời điểm.
- Phương thức chứng thực dạng password, user cần khai báo id or name, password

### Ví dụ:
```
POST: http://10.10.34.170:5000/v3/auth/tokens
```

Body request raw dạng JSON:
```json
{
    "auth": {
        "identity": {
            "methods": [
                "password"
            ],
            "password": {
                "user": {
                    "name": "admin",
                    "domain": {
                        "name": "Default"
                    },
                    "password": "Welcome123"
                }
            }
        }
    }
}
```

Kết quả:

Body
```json
{
    "token": {
        "issued_at": "2020-12-11T10:10:51.000000Z",
        "audit_ids": [
            "p1a1lJwbT7i-S_QhO2kDBA"
        ],
        "methods": [
            "password"
        ],
        "expires_at": "2020-12-11T11:10:51.000000Z",
        "user": {
            "password_expires_at": null,
            "domain": {
                "id": "default",
                "name": "Default"
            },
            "id": "b3af407dfde94afd89b44598ae54eca0",
            "name": "admin"
        }
    }
}
```

<img src="..\images\api-ops\Screenshot_3.png">

Header

<img src="..\images\api-ops\Screenshot_4.png">



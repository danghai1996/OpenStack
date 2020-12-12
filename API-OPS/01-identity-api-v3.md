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

## Chứng thực password dạng scoped authorization
```
POST: <IP_CTL>:5000/v3/auth/tokens
```

Lưu ý:
- Phương thức chứng thực cho phép truy cập các project, domain, system
- Request body cần bao gồm pasword, thêm các thông tin về project, domain, system

### Các loại chứng thực cơ bản:
Chứng thực system scoped:
```json
{
   "auth": {
       "identity": {
           "methods": [
               "password"
           ],
           "password": {
               "user": {
                   "id": "b3af407dfde94afd89b44598ae54eca0",
                   "password": "Welcome123"
               }
           }
       },
       "scope": {
           "system": {
               "all": true
           }
       }
   }
}
```

Chứng thực Domain-scoped:
```json
{
    "auth": {
        "identity": {
            "methods": [
                "password"
            ],
            "password": {
                "user": {
                    "id": "b3af407dfde94afd89b44598ae54eca0",
                    "password": "Welcome123"
                }
            }
        },
        "scope": {
            "domain": {
                "id": "default"
            }
        }
    }
}
```

Chứng thực Project-Scoped:
```json
{
   "auth": {
       "identity": {
           "methods": [
               "password"
           ],
           "password": {
               "user": {
                   "id": "b3af407dfde94afd89b44598ae54eca0",
                   "password": "Welcome123"
               }
           }
       },
       "scope": {
           "project": {
               "domain": {
                   "id": "default"
               },
               "id": "67a922c3e7b24461950e8b13ad587773"
           }
       }
   }
}
```

**Ví dụ:**

Repuest:

<img src="..\images\api-ops\Screenshot_5.png">

Response:

<img src="..\images\api-ops\Screenshot_6.png">

<img src="..\images\api-ops\Screenshot_7.png">

## Chứng thực token dạng unscoped authorization
```
POST: <IP_CTL>:5000/v3/auth/tokens
```

**Lưu ý:**
- Token dạng unscoped, tức Token không có quyền sử dụng bất kỳ catalog, role, project, hoặc domain. Sử dụng token này đơn giản để chứng thực danh tính tại KeyStone tại 1 số thời điểm.
- Phương thức chứng thực dạng password, user cần khai báo id or name, password

Body request:
```json
{
    "auth": {
        "identity": {
            "methods": [
                "token"
            ],
            "token": {
                "id": "'$OS_TOKEN'"
            }
        }
    }
}
```

Ví dụ:

Request:
```json
{
    "auth": {
        "identity": {
            "methods": [
                "token"
            ],
            "token": {
                "id": "gAAAAABf1DU8QzmWks21jF3Z1ZY6mJKyxgArdaHnwbo5bFokbcIoXMmfCprX3wKCiu9aAgvE0O4X9Rc_ZhyFp0Qzuq1Syql7kX6jt4M_hzWxiXHftl145ItXicXqZSa8086DN6Mff8niBtk4y2WjCntR2jQU3liPw7ZYzmbYI5E_oezqgTEdIOY"
            }
        }
    }
}
```

Kết quả:

<img src="..\images\api-ops\Screenshot_8.png">

<img src="..\images\api-ops\Screenshot_9.png">

## Chứng thực token dạng scoped authorization
```
POST: <IP_CTL>:5000/v3/auth/tokens
```

**Lưu ý:**
- Phương thức chứng thực cho phép truy cập các project, domain, system
- Request body cần bao gồm token, thêm các thông tin về project, domain, system

Chứng thực dạng System-Scoped:
```json
{
    "auth": {
        "identity": {
            "methods": [
                "token"
            ],
            "token": {
                "id": "'$OS_TOKEN'"
            }
        },
        "scope": {
            "system": {
                "all": true
            }
        }
    }
}
```

Chứng thực dạng Domain-Scoped với Domain ID:
```json
{
    "auth": {
        "identity": {
            "methods": [
                "token"
            ],
            "token": {
                "id": "'$OS_TOKEN'"
            }
        },
        "scope": {
            "domain": {
                "id": "default"
            }
        }
    }
}
```

Chứng thực dạng Domain-Scoped với Domain name:
```json
{
    "auth": {
        "identity": {
            "methods": [
                "token"
            ],
            "token": {
                "id": "'$OS_TOKEN'"
            }
        },
        "scope": {
            "domain": {
                "name": "Default"
            }
        }
    }
}
```

Chứng thực Project-Scoped với Project ID:
```json
{
    "auth": {
        "identity": {
            "methods": [
                "token"
            ],
            "token": {
                "id": "'$OS_TOKEN'"
            }
        },
        "scope": {
            "project": {
                "id": "a6944d763bf64ee6a275f1263fae0352"
            }
        }
    }
}
```

Chứng thực Project-Scoped với Project Name
```json
{
    "auth": {
        "identity": {
            "methods": [
                "token"
            ],
            "token": {
                "id": "'$OS_TOKEN'"
            }
        },
        "scope": {
            "project": {
                "domain": {
                    "id": "default"
                },
                "name": "admin"
            }
        }
    }
}
```

Ví dụ:

Body request
```json
{
    "auth": {
        "identity": {
            "methods": [
                "token"
            ],
            "token": {
                "id": "gAAAAABf1DU8QzmWks21jF3Z1ZY6mJKyxgArdaHnwbo5bFokbcIoXMmfCprX3wKCiu9aAgvE0O4X9Rc_ZhyFp0Qzuq1Syql7kX6jt4M_hzWxiXHftl145ItXicXqZSa8086DN6Mff8niBtk4y2WjCntR2jQU3liPw7ZYzmbYI5E_oezqgTEdIOY"
            }
        },
        "scope": {
            "project": {
                "domain": {
                    "id": "default"
                },
                "name": "admin"
            }
        }
    }
}
```

Kết quả:

<img src="..\images\api-ops\Screenshot_10.png">

<img src="..\images\api-ops\Screenshot_11.png">

## Chứng thực token và show thông tin token
```
GET: <IP_CTL>:5000/v3/auth/tokens
```

Trả lại thông tin các token

Lưu ý: Cần 2 tham số
- Cần tham số `X-Auth-Token`: Token hiện tại
- Cần tham số `X-Subject-Token`: Token cần chứng thực

### Ví dụ:

**Token đúng:**

<img src="..\images\api-ops\Screenshot_12.png">

<img src="..\images\api-ops\Screenshot_13.png">

<img src="..\images\api-ops\Screenshot_14.png">

**Token sai:**

<img src="..\images\api-ops\Screenshot_15.png">

<img src="..\images\api-ops\Screenshot_16.png">

<img src="..\images\api-ops\Screenshot_17.png">

## Kiểm tra token
```
HEAD: <IP_CTL>:5000/v3/auth/tokens
```

Kiểm tra token giống chứng thực nhưng không có kết quả trả về.

Yêu cầu 2 tham số:
- Cần tham số `X-Auth-Token`: Token hiện tại
- Cần tham số `X-Subject-Token`: Token cần kiểm tra

### Ví dụ:
**Token đúng:**

<img src="..\images\api-ops\Screenshot_18.png">

**Token sai:**

<img src="..\images\api-ops\Screenshot_19.png">

## Thu hồi token
```
DELETE: <IP_CTL>:5000/v3/auth/tokens
```

Giống chứng thực, nhưng mục đích là thu hồi token

### Ví dụ:
**Token đúng:**

<img src="..\images\api-ops\Screenshot_20.png">

**Token sai:**

<img src="..\images\api-ops\Screenshot_21.png">

## Lấy catalog service được sử dụng
```
GET: <IP_CTL>:5000/v3/auth/catalog
```

**Lưu ý:** sử dụng token dạng scoped như project scoped.

Tham số: `X-Auth-Token`: Token hiện tại

Ví dụ:

<img src="..\images\api-ops\Screenshot_22.png">

Kết quả:

<img src="..\images\api-ops\Screenshot_23.png">

## Lấy project có thể sử dụng
```
GET: <IP_CTL>:5000/v3/auth/projects
```

**Lưu ý:** sử dụng token dạng scoped như project scoped

<img src="..\images\api-ops\Screenshot_24.png">

Kết quả:

<img src="..\images\api-ops\Screenshot_25.png">

## Liệt kê các service có thể sử dụng
```
GET: <IP_CTL>:5000/v3/services
```

**Lưu ý:** sử dụng token dạng scoped như project scoped

### Ví dụ
<img src="..\images\api-ops\Screenshot_26.png">

Kết quả:

<img src="..\images\api-ops\Screenshot_27.png">

## Liệt kê danh sách user
```
GET: <IP_CTL>:5000/v3/users
```

**Lưu ý:** sử dụng token dạng scoped như project scoped

Tham số: `X-Auth-Token`: Token hiện tại

### Ví dụ:
<img src="..\images\api-ops\Screenshot_28.png">

Kết quả:

<img src="..\images\api-ops\Screenshot_29.png">
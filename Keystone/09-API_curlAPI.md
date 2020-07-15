# Tìm hiểu về API và sử dụng `curl` để gọi API

# API
## API là gì ?
- API : viết tắt của **Application Programming Interface** - giao diện lập trình ứng dụng.

- API là các phương thức, giao thức kết nối với các thư viện và ứng dụng khác

- API cung cấp khả năng cung cấp khả năng truy xuất đến một tập các hàm hay dùng. Và từ đó có thể trao đổi dữ liệu giữa các ứng dụng.



# Sử dụng `curl` để gọi API
## 1. Lấy token
### Dùng curl
```
curl -i -H "Content-Type: application/json" -d '
{ "auth": {
    "identity": {
        "methods": ["password"],
        "password": {
            "user": {
              "name": "admin",
              "domain": { "name": "Default" },
              "password": "Welcome123"
            }
          }
        },
        "scope": {
          "project": {
            "name": "admin",
            "domain": { "name": "Default" }
          }
        }
  }
}' http://localhost:5000/v3/auth/tokens
```
OPtion sử dụng trong lệnh `curl`:
- `-i` (--include): Output sẽ chứa cả HTTP-header
- `-H` (--header): kết hợp với Content-Type để xác định kiểu dữ liệu truyền vào header. Tại đây là dạng json
- `-d` (--data) : Dữ liệu truyền vào

Kết quả:
```json
HTTP/1.1 201 CREATED
Date: Wed, 15 Jul 2020 02:54:06 GMT
Server: Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5
X-Subject-Token: gAAAAABfDm_OtSOIyIfHMTZFY_y4wht8_BOVNM06Jvhnfe2QpW-kP4KCm4N8IPA92TKLlX3DbFlD1hfUcWmtr79pH1889nzObvNy2sUFZ8O9mPp5eHF99mKoh_ODRSwbGgiB3XEyOZiC-q5SLiD_dmtTNAkx0He0M3BaWQE8i8H5xO0Zc4NYVaQ
Vary: X-Auth-Token
x-openstack-request-id: req-6a2d1a83-b4e5-4e2a-a8c3-ae5e81e99088
Content-Length: 3443
Content-Type: application/json

{
  "token": {
    "is_domain": false,
    "methods": [
      "password"
    ],
    "roles": [
      {
        "id": "6edcf71488424352817939a8505267b1",
        "name": "admin"
      },
      {
        "id": "e62c83155a654bd4a31b4bc692b835f5",
        "name": "reader"
      },
      {
        "id": "f1b4e02022d4444ba29769ec76146a23",
        "name": "member"
      }
    ],
    "expires_at": "2020-07-15T03:54:06.000000Z",
    "project": {
      "domain": {
        "id": "default",
        "name": "Default"
      },
      "id": "5b4c1d2155004acf849cd3aac03b8f36",
      "name": "admin"
    },
    "catalog": [
      {
        "endpoints": [
          {
            "region_id": "RegionOne",
            "url": "http://10.10.31.166:9696",
            "region": "RegionOne",
            "interface": "internal",
            "id": "2aad8b16f9534fa18fceaccc50fc927d"
          },
          {
            "region_id": "RegionOne",
            "url": "http://10.10.31.166:9696",
            "region": "RegionOne",
            "interface": "public",
            "id": "e1d529ec9dc64f36ab3a4799c297ae17"
          },
          {
            "region_id": "RegionOne",
            "url": "http://10.10.31.166:9696",
            "region": "RegionOne",
            "interface": "admin",
            "id": "f0cd24d2520540b895efe81a3d00b437"
          }
        ],
        "type": "network",
        "id": "01c45c0eefbf45f9b1005be92b5bd091",
        "name": "neutron"
      },
      {
        "endpoints": [
          {
            "region_id": "RegionOne",
            "url": "http://10.10.31.166:9292",
            "region": "RegionOne",
            "interface": "admin",
            "id": "0bae90c2cb9b4ca0a5f812da2273fb79"
          },
          {
            "region_id": "RegionOne",
            "url": "http://10.10.31.166:9292",
            "region": "RegionOne",
            "interface": "public",
            "id": "743533c9a69f468590ff6b02a871d07f"
          },
          {
            "region_id": "RegionOne",
            "url": "http://10.10.31.166:9292",
            "region": "RegionOne",
            "interface": "internal",
            "id": "acfaf8b4a9034821aad043464de64c29"
          }
        ],
        "type": "image",
        "id": "1cb30c4b8b304a0fb18f0e453ff2aa99",
        "name": "glance"
      },
      {
        "endpoints": [
          {
            "region_id": "RegionOne",
            "url": "http://10.10.31.166:8778",
            "region": "RegionOne",
            "interface": "internal",
            "id": "8537fd3b514942bebb4e407554137d0f"
          },
          {
            "region_id": "RegionOne",
            "url": "http://10.10.31.166:8778",
            "region": "RegionOne",
            "interface": "public",
            "id": "890803754a454947ad2de61b8c89dd10"
          },
          {
            "region_id": "RegionOne",
            "url": "http://10.10.31.166:8778",
            "region": "RegionOne",
            "interface": "admin",
            "id": "d6d5a5ecf36544b794dcd1021ff10769"
          }
        ],
        "type": "placement",
        "id": "30fe5529b60a4e859443051c7f650295",
        "name": "placement"
      },
      {
        "endpoints": [
          {
            "region_id": "RegionOne",
            "url": "http://10.10.31.166:5000/v3/",
            "region": "RegionOne",
            "interface": "public",
            "id": "1354449c815d4f499d88cfc4f8f785b8"
          },
          {
            "region_id": "RegionOne",
            "url": "http://10.10.31.166:5000/v3/",
            "region": "RegionOne",
            "interface": "internal",
            "id": "4004a42543744831971c53b85956450d"
          },
          {
            "region_id": "RegionOne",
            "url": "http://10.10.31.166:5000/v3/",
            "region": "RegionOne",
            "interface": "admin",
            "id": "634db163e477401393da5675eb9454d9"
          }
        ],
        "type": "identity",
        "id": "60d6403b4de346a59d92c707b2d4a437",
        "name": "keystone"
      },
      {
        "endpoints": [
          {
            "region_id": "RegionOne",
            "url": "http://10.10.31.166:8774/v2.1",
            "region": "RegionOne",
            "interface": "public",
            "id": "01db1e80d3b540639861c1ee9d0a451f"
          },
          {
            "region_id": "RegionOne",
            "url": "http://10.10.31.166:8774/v2.1",
            "region": "RegionOne",
            "interface": "admin",
            "id": "454981cfd05041edb95971db63aec819"
          },
          {
            "region_id": "RegionOne",
            "url": "http://10.10.31.166:8774/v2.1",
            "region": "RegionOne",
            "interface": "internal",
            "id": "9e8b1c9d8e4e4085bc15ba5d7f91b1bb"
          }
        ],
        "type": "compute",
        "id": "ec8b6081186f4922bc67a38b5f148ac2",
        "name": "nova"
      }
    ],
    "user": {
      "password_expires_at": null,
      "domain": {
        "id": "default",
        "name": "Default"
      },
      "id": "294c5c6181d442c68a13d5b615c4f031",
      "name": "admin"
    },
    "audit_ids": [
      "RkkiEdfCQ4WPxN7qwturcg"
    ],
    "issued_at": "2020-07-15T02:54:06.000000Z"
  }
}
```

Token nằm ở phía sau `X-Subject-Token`


## 2. List user
### Command
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
| 580b7509af5d48c6a152296a5f0c0137 | haidd     |
+----------------------------------+-----------+
```

### `curl` API
```
curl -s -H "X-Auth-Token: $OS_TOKEN" http://localhost:5000/v3/users | python -mjson.tool
```

- `python -mjson.tool` : định dạng output ra là dạng json
- `-s` (--silent) : chế độ im lặng, sử dụng option này sẽ không hiển thị thông báo tiến trình hoặc thông báo lỗi

**Kết quả:**
```json
{
    "links": {
        "next": null,
        "previous": null,
        "self": "http://localhost:5000/v3/users"
    },
    "users": [
        {
            "domain_id": "default",
            "enabled": true,
            "id": "294c5c6181d442c68a13d5b615c4f031",
            "links": {
                "self": "http://localhost:5000/v3/users/294c5c6181d442c68a13d5b615c4f031"
            },
            "name": "admin",
            "options": {},
            "password_expires_at": null
        },
        {
            "domain_id": "default",
            "enabled": true,
            "id": "8db90f3de7a54375aa16eb1d0626f1bb",
            "links": {
                "self": "http://localhost:5000/v3/users/8db90f3de7a54375aa16eb1d0626f1bb"
            },
            "name": "demo",
            "options": {},
            "password_expires_at": null
        },
        {
            "domain_id": "default",
            "enabled": true,
            "id": "afe0224c90f84b3b83a8d9788921ccef",
            "links": {
                "self": "http://localhost:5000/v3/users/afe0224c90f84b3b83a8d9788921ccef"
            },
            "name": "glance",
            "options": {},
            "password_expires_at": null
        },
        {
            "domain_id": "default",
            "enabled": true,
            "id": "f30b4ed7a5a54ea68d98aeee438bb80f",
            "links": {
                "self": "http://localhost:5000/v3/users/f30b4ed7a5a54ea68d98aeee438bb80f"
            },
            "name": "placement",
            "options": {},
            "password_expires_at": null
        },
        {
            "domain_id": "default",
            "enabled": true,
            "id": "4be5d5ed900f4386a5f9268927c46ecb",
            "links": {
                "self": "http://localhost:5000/v3/users/4be5d5ed900f4386a5f9268927c46ecb"
            },
            "name": "nova",
            "options": {},
            "password_expires_at": null
        },
        {
            "domain_id": "default",
            "enabled": true,
            "id": "74cf804695e44ae4995f9579446ae813",
            "links": {
                "self": "http://localhost:5000/v3/users/74cf804695e44ae4995f9579446ae813"
            },
            "name": "neutron",
            "options": {},
            "password_expires_at": null
        },
        {
            "domain_id": "default",
            "enabled": true,
            "id": "580b7509af5d48c6a152296a5f0c0137",
            "links": {
                "self": "http://localhost:5000/v3/users/580b7509af5d48c6a152296a5f0c0137"
            },
            "name": "haidd",
            "options": {},
            "password_expires_at": null
        }
    ]
}
```

## 3. List project
Tương tự list user, thay thế URL
```
curl -s -H "X-Auth-Token: $OS_TOKEN" \
http://localhost:5000/v3/projects | python -mjson.tool
```

## 4. List group
```
curl -s -H "X-Auth-Token: $OS_TOKEN" \
 http://localhost:5000/v3/groups | python -mjson.tool
```

## 5. List role
```
curl -s -H "X-Auth-Token: $OS_TOKEN" \
http://localhost:5000/v3/roles | python -mjson.tool
```

## 6. List domain
```
curl -s -H "X-Auth-Token: $OS_TOKEN" \
http://localhost:5000/v3/domains | python -mjson.tool
```
# File log của Glance

File log của Glance: `/var/log/glance/api.log`

Khi tạo thêm 1 image:
```log
2020-07-20 08:51:45.170 2041 INFO eventlet.wsgi.server [req-7d876b1c-b2bd-4370-a5e0-d30233e37d9d 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] 10.10.31.166 - - [20/Jul/2020 08:51:45] "GET /v2/schemas/image HTTP/1.1" 200 6283 0.515541
```

Các thông tin trong file log:
```
[req-7d876b1c-b2bd-4370-a5e0-d30233e37d9d 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default]
```
- `req-7d876b1c-b2bd-4370-a5e0-d30233e37d9d` : mã request
- `294c5c6181d442c68a13d5b615c4f031` : ID user
- `5b4c1d2155004acf849cd3aac03b8f36` : ID project
- `default default` : domain


Địa chỉ IP
```
10.10.31.166
```

Thông tin method và Status code của request
```
"GET /v2/schemas/image HTTP/1.1" 200 6283 0.515541
```
# File log của Keystone

File cấu hình log của Keystone:
```conf
cat /etc/keystone/logging.conf | egrep -v "^$|^#"

[loggers]
keys=root,access
[handlers]
keys=production,file,access_file,devel
[formatters]
keys=minimal,normal,debug
[logger_root]
level=WARNING
handlers=file
[logger_access]
level=INFO
qualname=access
handlers=access_file
[handler_production]
class=handlers.SysLogHandler
level=ERROR
formatter=normal
args=(('localhost', handlers.SYSLOG_UDP_PORT), handlers.SysLogHandler.LOG_USER)
[handler_file]
class=handlers.WatchedFileHandler
level=WARNING
formatter=normal
args=('error.log',)
[handler_access_file]
class=handlers.WatchedFileHandler
level=INFO
formatter=minimal
args=('access.log',)
[handler_devel]
class=StreamHandler
level=NOTSET
formatter=debug
args=(sys.stdout,)
[formatter_minimal]
format=%(message)s
[formatter_normal]
format=(%(name)s): %(asctime)s %(levelname)s %(message)s
[formatter_debug]
format=(%(name)s): %(asctime)s %(levelname)s %(module)s %(funcName)s %(message)s
```

**3 kiểu log:**

1. formatter_minimal
    ```
    format=%(message)s
    ```

2. formatter_normal
    ```
    format=(%(name)s): %(asctime)s %(levelname)s %(message)s
    ```
3. formatter_debug
    ```
    format=(%(name)s): %(asctime)s %(levelname)s %(module)s %(funcName)s %(message)s
    ```

File log: `/var/log/keystone/keystone.log`


# 1. Khi đăng nhập vào OPS
## Đúng user
### Đúng user-đúng pass
```log
2020-07-09 15:47:30.541 2378 WARNING keystone.server.flask.application [req-6570f6d3-399a-4564-82a6-666692b2bd5d 294c5c6181d442c68a13d5b615c4f031 - - default -] Authorization failed. The request you have made requires authentication. from 10.10.31.166: Unauthorized: The request you have made requires authentication.
```

`294c5c6181d442c68a13d5b615c4f031` -> ID của user đăng nhập. Ở đây là admin
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
+----------------------------------+-----------+
```

<img src="..\images\Screenshot_60.png">

### Đúng user-sai pass
```log
2020-07-09 16:04:21.195 2382 WARNING keystone.server.flask.application [req-54da289c-53be-4f52-a77e-8028f820ae5b - - - - -] Authorization failed. The request you have made requires authentication. from 10.10.31.166: Unauthorized: The request you have made requires authentication.
```

<img src="..\images\Screenshot_61.png">

## Sai user
```log
2020-07-09 16:06:55.123 2379 WARNING keystone.auth.plugins.core [req-9ca76628-2a93-4ab5-89c5-2328d3f430cd - - - - -] Could not find user: ad.: UserNotFound: Could not find user: ad.
2020-07-09 16:06:55.141 2379 WARNING keystone.server.flask.application [req-9ca76628-2a93-4ab5-89c5-2328d3f430cd - - - - -] Authorization failed. The request you have made requires authentication. from 10.10.31.166: Unauthorized: The request you have made requires authentication.
```

<img src="..\images\Screenshot_62.png">

# 2. Khởi tạo máy ảo
```log
2020-07-09 16:23:55.619 4859 WARNING keystone.common.rbac_enforcer.enforcer [req-25b17d71-277c-429b-b61c-658a7147d7a6 4be5d5ed900f4386a5f9268927c46ecb af57453a686740f18e48a8c4cf4ac994 - default default] Deprecated policy rules found. Use oslopolicy-policy-generator and oslopolicy-policy-upgrade to detect and resolve deprecated policies in your configuration.
2020-07-09 16:23:59.718 4858 WARNING keystone.common.rbac_enforcer.enforcer [req-226bdbba-ad99-4f32-ac53-3fcab64ca9a6 4be5d5ed900f4386a5f9268927c46ecb af57453a686740f18e48a8c4cf4ac994 - default default] Deprecated policy rules found. Use oslopolicy-policy-generator and oslopolicy-policy-upgrade to detect and resolve deprecated policies in your configuration.
2020-07-09 16:24:06.703 4861 WARNING keystone.common.rbac_enforcer.enforcer [req-e590d16b-6c4d-4be5-9d9b-b62391b5f769 f30b4ed7a5a54ea68d98aeee438bb80f af57453a686740f18e48a8c4cf4ac994 - default default] Deprecated policy rules found. Use oslopolicy-policy-generator and oslopolicy-policy-upgrade to detect and resolve deprecated policies in your configuration.
2020-07-09 16:24:09.897 4857 WARNING keystone.common.rbac_enforcer.enforcer [req-6c17b3be-f747-4838-9b98-12210c695637 294c5c6181d442c68a13d5b615c4f031 - - default -] Deprecated policy rules found. Use oslopolicy-policy-generator and oslopolicy-policy-upgrade to detect and resolve deprecated policies in your configuration.
2020-07-09 16:24:16.963 4860 WARNING keystone.common.rbac_enforcer.enforcer [req-937f1c63-022c-46a0-a229-6d767a3170ed 74cf804695e44ae4995f9579446ae813 af57453a686740f18e48a8c4cf4ac994 - default default] Deprecated policy rules found. Use oslopolicy-policy-generator and oslopolicy-policy-upgrade to detect and resolve deprecated policies in your configuration.
```

<img src="..\images\Screenshot_63.png">

```
[req-25b17d71-277c-429b-b61c-658a7147d7a6 4be5d5ed900f4386a5f9268927c46ecb af57453a686740f18e48a8c4cf4ac994 - default default]
```
**Trong đó:**
- `[req-25b17d71-277c-429b-b61c-658a7147d7a6]` : mã request
- `4be5d5ed900f4386a5f9268927c46ecb` : ID user (tại đây là nova)
- `af57453a686740f18e48a8c4cf4ac994` : ID project (tại đây là project service)
- `default default` : ID và tên domain


# 3. Rebuild, resize, suspend, start, ,create, delete VM
Keystone không có log


# File log Keystone trong http
File log: Ta có thể show các file log Keystone trong http :
```
ls -alh /var/log/httpd/ | grep keystone

-rw-r--r--   1 root root 458K Jul 11 08:27 keystone_access.log
-rw-r--r--   1 root root 150K Jul  9 16:22 keystone_access.log-20200709
-rw-r--r--   1 root root    0 Jun 25 22:59 keystone.log
```

## Khi login
Các log sẽ là các hành động gọi API để thực hiện hành động

<img src="..\images\Screenshot_64.png">

## Các hành động khác thì log của Keystone hoàn toàn tương tự
### Tạo VM
```log
10.10.31.166 - - [11/Jul/2020:08:37:20 +0700] "GET /v3/ HTTP/1.1" 200 252 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:20 +0700] "GET /v3/ HTTP/1.1" 200 252 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:20 +0700] "GET /v3/ HTTP/1.1" 200 252 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:20 +0700] "GET /v3/ HTTP/1.1" 200 252 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:20 +0700] "POST /v3/auth/tokens HTTP/1.1" 201 3478 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:20 +0700] "POST /v3/auth/tokens HTTP/1.1" 201 3478 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:21 +0700] "GET /v3/ HTTP/1.1" 200 252 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:20 +0700] "POST /v3/auth/tokens HTTP/1.1" 201 3478 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:20 +0700] "POST /v3/auth/tokens HTTP/1.1" 201 3478 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:21 +0700] "POST /v3/auth/tokens HTTP/1.1" 201 3478 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:22 +0700] "GET /v3/auth/tokens HTTP/1.1" 200 3478 "-" "python-keystoneclient"
10.10.31.166 - - [11/Jul/2020:08:37:22 +0700] "GET /v3/auth/tokens HTTP/1.1" 200 3478 "-" "python-keystoneclient"
10.10.31.166 - - [11/Jul/2020:08:37:23 +0700] "GET /v3/auth/tokens HTTP/1.1" 200 3478 "-" "python-keystoneclient"
10.10.31.166 - - [11/Jul/2020:08:37:23 +0700] "GET /v3/auth/tokens HTTP/1.1" 200 3478 "-" "python-keystoneclient"
10.10.31.166 - - [11/Jul/2020:08:37:23 +0700] "GET /v3/auth/tokens HTTP/1.1" 200 3478 "-" "python-keystoneclient"
10.10.31.166 - - [11/Jul/2020:08:37:24 +0700] "GET /v3/ HTTP/1.1" 200 252 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:24 +0700] "POST /v3/auth/tokens HTTP/1.1" 201 3478 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:24 +0700] "GET /v3/auth/tokens HTTP/1.1" 200 3478 "-" "python-keystoneclient"
10.10.31.166 - - [11/Jul/2020:08:37:25 +0700] "GET /v3/ HTTP/1.1" 200 252 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:25 +0700] "POST /v3/auth/tokens HTTP/1.1" 201 3478 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:25 +0700] "GET /v3/auth/tokens HTTP/1.1" 200 3478 "-" "python-keystoneclient"
::1 - - [11/Jul/2020:08:37:29 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
::1 - - [11/Jul/2020:08:37:30 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
::1 - - [11/Jul/2020:08:37:31 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
::1 - - [11/Jul/2020:08:37:32 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
::1 - - [11/Jul/2020:08:37:33 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
::1 - - [11/Jul/2020:08:37:34 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
::1 - - [11/Jul/2020:08:37:35 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
::1 - - [11/Jul/2020:08:37:36 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
::1 - - [11/Jul/2020:08:37:37 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
10.10.31.166 - - [11/Jul/2020:08:37:37 +0700] "GET /v3/ HTTP/1.1" 200 252 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:37 +0700] "POST /v3/auth/tokens HTTP/1.1" 201 3478 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
::1 - - [11/Jul/2020:08:37:38 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
10.10.31.166 - - [11/Jul/2020:08:37:38 +0700] "GET /v3/auth/tokens HTTP/1.1" 200 3478 "-" "python-keystoneclient"
10.10.31.166 - - [11/Jul/2020:08:37:38 +0700] "GET /v3/ HTTP/1.1" 200 252 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:38 +0700] "POST /v3/auth/tokens HTTP/1.1" 201 3478 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
::1 - - [11/Jul/2020:08:37:39 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
10.10.31.166 - - [11/Jul/2020:08:37:38 +0700] "GET /v3/auth/tokens HTTP/1.1" 200 3478 "-" "python-keystoneclient"
10.10.31.166 - - [11/Jul/2020:08:37:39 +0700] "GET /v3/auth/tokens HTTP/1.1" 200 3444 "-" "python-keystoneclient"
10.10.31.166 - - [11/Jul/2020:08:37:40 +0700] "POST /v3/auth/tokens HTTP/1.1" 201 3447 "-" "nova-api keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:41 +0700] "GET /v3/auth/tokens HTTP/1.1" 200 3447 "-" "python-keystoneclient"
10.10.31.166 - - [11/Jul/2020:08:37:42 +0700] "GET /v3/ HTTP/1.1" 200 252 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:42 +0700] "POST /v3/auth/tokens HTTP/1.1" 201 3478 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:42 +0700] "POST /v3/auth/tokens HTTP/1.1" 201 3449 "-" "nova-scheduler keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:42 +0700] "GET /v3/auth/tokens HTTP/1.1" 200 3478 "-" "python-keystoneclient"
10.10.31.166 - - [11/Jul/2020:08:37:43 +0700] "GET /v3/ HTTP/1.1" 200 252 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:43 +0700] "POST /v3/auth/tokens HTTP/1.1" 201 3478 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:43 +0700] "POST /v3/auth/tokens HTTP/1.1" 201 3449 "-" "placement/unknown keystonemiddleware.auth_token/7.0.1 keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:43 +0700] "GET /v3/auth/tokens HTTP/1.1" 200 3449 "-" "python-keystoneclient"
10.10.31.166 - - [11/Jul/2020:08:37:43 +0700] "GET /v3/auth/tokens HTTP/1.1" 200 3478 "-" "python-keystoneclient"
10.10.31.166 - - [11/Jul/2020:08:37:44 +0700] "POST /v3/auth/tokens HTTP/1.1" 201 3449 "-" "nova-scheduler keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:45 +0700] "GET /v3/users/294c5c6181d442c68a13d5b615c4f031/projects HTTP/1.1" 200 474 "-" "python-keystoneclient"
10.10.31.166 - - [11/Jul/2020:08:37:45 +0700] "GET /v3/ HTTP/1.1" 200 252 "-" "keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:45 +0700] "GET /v3/auth/tokens HTTP/1.1" 200 3449 "-" "python-keystoneclient"
10.10.31.166 - - [11/Jul/2020:08:37:46 +0700] "GET /v3/ HTTP/1.1" 200 252 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:46 +0700] "GET /v3/users/294c5c6181d442c68a13d5b615c4f031/projects HTTP/1.1" 200 474 "-" "python-keystoneclient"
10.10.31.166 - - [11/Jul/2020:08:37:46 +0700] "POST /v3/auth/tokens HTTP/1.1" 201 3478 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:46 +0700] "GET /v3/auth/tokens HTTP/1.1" 200 3478 "-" "python-keystoneclient"
10.10.31.166 - - [11/Jul/2020:08:37:47 +0700] "GET /v3/ HTTP/1.1" 200 252 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:47 +0700] "POST /v3/auth/tokens HTTP/1.1" 201 3478 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:48 +0700] "GET /v3/auth/tokens HTTP/1.1" 200 3478 "-" "python-keystoneclient"
::1 - - [11/Jul/2020:08:37:49 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
::1 - - [11/Jul/2020:08:37:50 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
10.10.31.166 - - [11/Jul/2020:08:37:50 +0700] "GET /v3/users/294c5c6181d442c68a13d5b615c4f031/projects HTTP/1.1" 200 474 "-" "python-keystoneclient"
::1 - - [11/Jul/2020:08:37:51 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
10.10.31.166 - - [11/Jul/2020:08:37:50 +0700] "POST /v3/auth/tokens HTTP/1.1" 201 3444 "-" "neutron-server keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:51 +0700] "GET /v3/auth/tokens HTTP/1.1" 200 3444 "-" "python-keystoneclient"
::1 - - [11/Jul/2020:08:37:52 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
10.10.31.166 - - [11/Jul/2020:08:37:53 +0700] "GET /v3/ HTTP/1.1" 200 252 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
::1 - - [11/Jul/2020:08:37:53 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
10.10.31.166 - - [11/Jul/2020:08:37:53 +0700] "POST /v3/auth/tokens HTTP/1.1" 201 3478 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:53 +0700] "GET /v3/auth/tokens HTTP/1.1" 200 3478 "-" "python-keystoneclient"
::1 - - [11/Jul/2020:08:37:54 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
10.10.31.166 - - [11/Jul/2020:08:37:53 +0700] "GET /v3/auth/tokens HTTP/1.1" 200 3444 "-" "python-keystoneclient"
10.10.31.166 - - [11/Jul/2020:08:37:54 +0700] "GET /v3/ HTTP/1.1" 200 252 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
10.10.31.166 - - [11/Jul/2020:08:37:54 +0700] "GET /v3/auth/tokens HTTP/1.1" 200 3449 "-" "python-keystoneclient"
10.10.31.166 - - [11/Jul/2020:08:37:54 +0700] "POST /v3/auth/tokens HTTP/1.1" 201 3478 "-" "python-novaclient keystoneauth1/3.17.2 python-requests/2.21.0 CPython/2.7.5"
::1 - - [11/Jul/2020:08:37:55 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
10.10.31.166 - - [11/Jul/2020:08:37:54 +0700] "GET /v3/auth/tokens HTTP/1.1" 200 3478 "-" "python-keystoneclient"
::1 - - [11/Jul/2020:08:37:56 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
10.10.31.166 - - [11/Jul/2020:08:37:56 +0700] "GET /v3/users/294c5c6181d442c68a13d5b615c4f031/projects HTTP/1.1" 200 474 "-" "python-keystoneclient"
::1 - - [11/Jul/2020:08:37:58 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
::1 - - [11/Jul/2020:08:37:59 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
::1 - - [11/Jul/2020:08:38:00 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
::1 - - [11/Jul/2020:08:38:01 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
::1 - - [11/Jul/2020:08:38:02 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
::1 - - [11/Jul/2020:08:38:03 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
::1 - - [11/Jul/2020:08:38:04 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
::1 - - [11/Jul/2020:08:38:05 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
::1 - - [11/Jul/2020:08:38:06 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
::1 - - [11/Jul/2020:08:38:07 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
::1 - - [11/Jul/2020:08:38:08 +0700] "OPTIONS * HTTP/1.0" 200 - "-" "Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5 (internal dummy connection)"
10.10.31.166 - - [11/Jul/2020:08:38:11 +0700] "GET /v3/auth/tokens HTTP/1.1" 200 3447 "-" "python-keystoneclient"
10.10.31.166 - - [11/Jul/2020:08:38:12 +0700] "GET /v3/auth/tokens HTTP/1.1" 200 3447 "-" "python-keystoneclient"
```
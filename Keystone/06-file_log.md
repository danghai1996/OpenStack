# File log cuar Keystone

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


# 3. Rebuild, resize, suspend, start, delete VM
Keystone không có log
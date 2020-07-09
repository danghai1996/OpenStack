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


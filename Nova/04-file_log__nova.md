# File log của Nova

**Các file log của Nova:**

## `/var/log/nova/nova-api.log`: 
Log khi ta thực hiện các request đến API như tạo, xóa, tắt bật các VM

Ví dụ:
```log
2020-07-27 16:11:37.249 2111 INFO nova.api.openstack.wsgi [req-a4b2aec3-add1-41c2-9e01-b4026690f481 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] HTTP exception thrown: Instance VM-test could not be found.
2020-07-27 16:11:37.254 2111 INFO nova.osapi_compute.wsgi.server [req-a4b2aec3-add1-41c2-9e01-b4026690f481 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] 10.10.31.166 "GET /v2.1/servers/VM-test HTTP/1.1" status: 404 len: 502 time: 0.5502350
2020-07-27 16:11:37.582 2111 INFO nova.osapi_compute.wsgi.server [req-ea56de1f-8b43-469e-b815-dcd90b29095e 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] 10.10.31.166 "GET /v2.1/servers?name=VM-test HTTP/1.1" status: 200 len: 695 time: 0.3203602
2020-07-27 16:11:37.920 2111 INFO nova.osapi_compute.wsgi.server [req-22d1ff5e-1c97-4225-9ba8-c3cbe0fbd4f1 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] 10.10.31.166 "GET /v2.1/servers/6c86e55c-e901-4681-a81c-f5c6e0db13f8 HTTP/1.1" status: 200 len: 2006 time: 0.3310719
2020-07-27 16:11:38.117 2111 INFO nova.osapi_compute.wsgi.server [req-cacd4e71-2a56-43c3-a99a-a0c6ea3b71e9 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] 10.10.31.166 "POST /v2.1/servers/6c86e55c-e901-4681-a81c-f5c6e0db13f8/action HTTP/1.1" status: 202 len: 403 time: 0.1901319
2020-07-27 16:11:42.371 2112 INFO nova.api.openstack.compute.server_external_events [req-3ee3f933-c8d0-4107-8363-cd7932c571c3 4be5d5ed900f4386a5f9268927c46ecb af57453a686740f18e48a8c4cf4ac994 - default default] Creating event network-vif-plugged:0af6273d-796d-4fb0-9e5f-cae7030e9c94 for instance 6c86e55c-e901-4681-a81c-f5c6e0db13f8 on compute1
2020-07-27 16:11:42.391 2112 INFO nova.osapi_compute.wsgi.server [req-3ee3f933-c8d0-4107-8363-cd7932c571c3 4be5d5ed900f4386a5f9268927c46ecb af57453a686740f18e48a8c4cf4ac994 - default default] 10.10.31.166 "POST /v2.1/os-server-external-events HTTP/1.1" status: 200 len: 582 time: 0.1204441
```

## `/var/log/nova/nova-conductor.log`
```log
nova-network is deprecated, as are any related configuration options.
).  Its value may be silently ignored in the future.
2020-07-22 14:55:23.917 2348 WARNING oslo_config.cfg [-] Deprecated: Option "firewall_driver" from group "DEFAULT" is deprecated for removal (
nova-network is deprecated, as are any related configuration options.
).  Its value may be silently ignored in the future.
2020-07-22 14:55:23.919 2348 WARNING oslo_config.cfg [-] Deprecated: Option "force_dhcp_release" from group "DEFAULT" is deprecated for removal (
nova-network is deprecated, as are any related configuration options.
).  Its value may be silently ignored in the future.
2020-07-22 14:55:23.924 2559 INFO nova.service [-] Starting conductor node (version 20.3.0-1.el7)
2020-07-22 14:55:23.927 2560 INFO nova.service [-] Starting conductor node (version 20.3.0-1.el7)
```

## `/var/log/nova/nova-manage.log`
```log
2020-06-26 08:18:07.591 3818 INFO migrate.versioning.api [req-65194828-0e81-433d-8f0a-b0b7755c4646 - - - - -] 397 -> 398...
2020-06-26 08:18:07.639 3818 INFO migrate.versioning.api [req-65194828-0e81-433d-8f0a-b0b7755c4646 - - - - -] done
2020-06-26 08:18:07.640 3818 INFO migrate.versioning.api [req-65194828-0e81-433d-8f0a-b0b7755c4646 - - - - -] 398 -> 399...
2020-06-26 08:18:07.695 3818 INFO migrate.versioning.api [req-65194828-0e81-433d-8f0a-b0b7755c4646 - - - - -] done
2020-06-26 08:18:07.696 3818 INFO migrate.versioning.api [req-65194828-0e81-433d-8f0a-b0b7755c4646 - - - - -] 399 -> 400...
2020-06-26 08:18:07.721 3818 INFO migrate.versioning.api [req-65194828-0e81-433d-8f0a-b0b7755c4646 - - - - -] done
2020-06-26 08:18:07.722 3818 INFO migrate.versioning.api [req-65194828-0e81-433d-8f0a-b0b7755c4646 - - - - -] 400 -> 401...
2020-06-26 08:18:07.790 3818 INFO migrate.versioning.api [req-65194828-0e81-433d-8f0a-b0b7755c4646 - - - - -] done
2020-06-26 08:18:07.791 3818 INFO migrate.versioning.api [req-65194828-0e81-433d-8f0a-b0b7755c4646 - - - - -] 401 -> 402...
2020-06-26 08:18:07.837 3818 INFO migrate.versioning.api [req-65194828-0e81-433d-8f0a-b0b7755c4646 - - - - -] done

```

## `/var/log/nova/nova-novncproxy.log`
```log
2020-07-27 14:52:51.101 7874 INFO nova.console.websocketproxy [-] 10.10.31.1 - - [27/Jul/2020 14:52:51] 10.10.31.1: Path: '/?token=1d9907ea-5b71-4704-b72d-bdeda3152945'
2020-07-27 14:52:51.903 7874 INFO nova.console.websocketproxy [req-4c8b505e-0b07-4f30-8779-4f93b8c5d223 - - - - -]  39: connect info: ConsoleAuthToken(access_url_base='http://10.10.31.166:6080/vnc_auto.html',console_type='novnc',created_at=2020-07-27T07:52:50Z,host='10.10.31.168',id=18,instance_uuid=0415fc81-732a-4b5f-b881-b85912cab6a7,internal_access_path=None,port=5900,token='***',updated_at=None)
2020-07-27 14:52:51.906 7874 INFO nova.console.websocketproxy [req-4c8b505e-0b07-4f30-8779-4f93b8c5d223 - - - - -]  39: connecting to: 10.10.31.168:5900
2020-07-27 14:52:51.928 7874 INFO nova.console.securityproxy.rfb [req-4c8b505e-0b07-4f30-8779-4f93b8c5d223 - - - - -] Finished security handshake, resuming normal proxy mode using secured socket
2020-07-27 14:54:13.676 7973 INFO nova.console.websocketproxy [-] 10.10.31.1 - - [27/Jul/2020 14:54:13] 10.10.31.1: Plain non-SSL (ws://) WebSocket connection
2020-07-27 14:54:13.680 7973 INFO nova.console.websocketproxy [-] 10.10.31.1 - - [27/Jul/2020 14:54:13] 10.10.31.1: Version hybi-13, base64: 'False'
2020-07-27 14:54:13.680 7973 INFO nova.console.websocketproxy [-] 10.10.31.1 - - [27/Jul/2020 14:54:13] 10.10.31.1: Path: '/?token=17ef65b6-c623-4046-a106-5937c0d451af'
2020-07-27 14:54:14.362 7973 INFO nova.console.websocketproxy [req-b3582c6e-ea2f-4f8f-a9af-73531cf64ca8 - - - - -]  40: connect info: ConsoleAuthToken(access_url_base='http://10.10.31.166:6080/vnc_auto.html',console_type='novnc',created_at=2020-07-27T07:54:13Z,host='10.10.31.168',id=19,instance_uuid=0415fc81-732a-4b5f-b881-b85912cab6a7,internal_access_path=None,port=5900,token='***',updated_at=None)
2020-07-27 14:54:14.364 7973 INFO nova.console.websocketproxy [req-b3582c6e-ea2f-4f8f-a9af-73531cf64ca8 - - - - -]  40: connecting to: 10.10.31.168:5900
2020-07-27 14:54:14.385 7973 INFO nova.console.securityproxy.rfb [req-b3582c6e-ea2f-4f8f-a9af-73531cf64ca8 - - - - -] Finished security handshake, resuming normal proxy mode using secured socket
```


## `/var/log/nova/nova-scheduler.log`
```log
2020-07-24 09:31:23.722 2583 INFO nova.scheduler.host_manager [req-29315026-80ee-48b4-ae5a-d6f2e596619d - - - - -] The instance sync for host 'compute2' did not match. Re-created its InstanceList.
2020-07-24 16:26:03.725 2582 INFO nova.scheduler.host_manager [req-4a80b728-3272-463d-a5d5-7684b35ad1a5 - - - - -] The instance sync for host 'compute2' did not match. Re-created its InstanceList.
2020-07-24 16:26:03.725 2583 INFO nova.scheduler.host_manager [req-4a80b728-3272-463d-a5d5-7684b35ad1a5 - - - - -] The instance sync for host 'compute2' did not match. Re-created its InstanceList.
2020-07-24 21:54:20.089 2582 INFO nova.scheduler.host_manager [req-31e4c43a-a065-4251-bbd2-a0162a744fc8 - - - - -] The instance sync for host 'compute1' did not match. Re-created its InstanceList.
2020-07-24 21:54:20.093 2583 INFO nova.scheduler.host_manager [req-31e4c43a-a065-4251-bbd2-a0162a744fc8 - - - - -] The instance sync for host 'compute1' did not match. Re-created its InstanceList.
2020-07-24 21:59:53.096 2582 INFO nova.scheduler.host_manager [req-56eaf5de-0a5f-42c2-bb9c-bf9d92dfea58 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Host filter ignoring hosts: compute1
2020-07-24 22:03:27.185 2583 INFO nova.scheduler.host_manager [req-01f4a2ea-4ed8-465a-ac48-110ba8602465 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Host filter ignoring hosts:
2020-07-24 22:04:17.691 2582 INFO nova.scheduler.host_manager [req-0364f43e-4b96-4666-9188-51c3cd4bfb32 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Host filter ignoring hosts:
2020-07-25 10:32:53.755 2583 INFO nova.scheduler.host_manager [req-7b01ed27-15cb-4e63-9680-5bd804a6c4d7 - - - - -] The instance sync for host 'compute2' did not match. Re-created its InstanceList.
2020-07-25 10:32:53.762 2582 INFO nova.scheduler.host_manager [req-7b01ed27-15cb-4e63-9680-5bd804a6c4d7 - - - - -] The instance sync for host 'compute2' did not match. Re-created its InstanceList.
```
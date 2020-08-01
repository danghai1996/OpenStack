# File log của Cinder

Cinder có thư mục chứa log mặc định: `/var/log/cinder` với các file log: `api.log`, `backup.log`, `cinder-manage.log`, `privsep-helper.log`, `scheduler.log`, `volume.log`


## Khi tạo volume
Ví dụ: Tạo volume từ image
```
openstack volume create --size 5 --image cirros vl01-cirros
```

File `volume.log`:
```log
2020-08-01 13:47:29.009 4520 INFO cinder.volume.flows.manager.create_volume [req-6f6a2905-7ee0-43dd-9102-f9f9a9049f40 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Volume 7b8068fd-01dc-44c6-a393-a11c483b99ad: being created as image with specification: {'status': u'creating', 'image_location': (None, None), 'volume_size': 5, 'volume_name': u'volume-7b8068fd-01dc-44c6-a393-a11c483b99ad', 'image_id': u'010ffd94-4d26-4dc6-be5b-1a7a31a3686a', 'image_service': <cinder.image.glance.GlanceImageService object at 0x7f639b6c0ad0>, 'image_meta': {u'status': u'active', u'file': u'/v2/images/010ffd94-4d26-4dc6-be5b-1a7a31a3686a/file', u'id': u'010ffd94-4d26-4dc6-be5b-1a7a31a3686a', u'name': u'cirros', u'tags': [], u'container_format': u'bare', u'created_at': datetime.datetime(2020, 6, 25, 16, 7, 50, tzinfo=<iso8601.Utc>), u'disk_format': u'qcow2', u'updated_at': datetime.datetime(2020, 6, 25, 16, 7, 50, tzinfo=<iso8601.Utc>), u'visibility': u'public', u'os_hash_algo': u'sha512', u'owner': u'5b4c1d2155004acf849cd3aac03b8f36', u'protected': False, u'os_hash_value': u'6513f21e44aa3da349f248188a44bc304a3653a04122d8fb4535423c8e1d14cd6a153f735bb0982e2161b5b5186106570c17a9e58b64dd39390617cd5a350f78', u'min_ram': 0, u'checksum': u'443b7623e27ecf03dc9e01ee93f67afe', u'min_disk': 0, u'os_hidden': False, u'virtual_size': None, 'properties': {}, u'size': 12716032}}
2020-08-01 13:47:29.333 4520 INFO cinder.image.image_utils [req-6f6a2905-7ee0-43dd-9102-f9f9a9049f40 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Image download 12.00 MB at 12.00 MB/s
2020-08-01 13:47:35.604 4520 INFO cinder.image.image_utils [req-6f6a2905-7ee0-43dd-9102-f9f9a9049f40 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Converted 44.00 MB image at 20.23 MB/s
2020-08-01 13:47:35.722 4520 INFO cinder.volume.flows.manager.create_volume [req-6f6a2905-7ee0-43dd-9102-f9f9a9049f40 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Volume volume-7b8068fd-01dc-44c6-a393-a11c483b99ad (7b8068fd-01dc-44c6-a393-a11c483b99ad): created successfully
2020-08-01 13:47:35.745 4520 INFO cinder.volume.manager [req-6f6a2905-7ee0-43dd-9102-f9f9a9049f40 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Created volume successfully.
```

Ta sẽ thấy các log với ý nghĩa:
- Khởi tạo thông số volume truyền vào như size, image, ...
- Download image từ Glance về volume
- Chuyển định dạng (Converted) image
- Thông báo tạo thành công

Tạo volume trắng tương tự nhưng sẽ không có thông tin liên quan đến image.


## Xóa volume
File `volume.log`
```log
2020-08-01 14:12:44.650 4520 INFO cinder.volume.targets.iscsi [req-b646caca-75fb-417e-a8cc-0a6a3de1b23f 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Skipping remove_export. No iscsi_target is presently exported for volume: c71d85ec-48d5-4eb3-a7f7-8490228caa19
2020-08-01 14:12:46.158 4520 INFO cinder.volume.drivers.lvm [req-b646caca-75fb-417e-a8cc-0a6a3de1b23f 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Successfully deleted volume: c71d85ec-48d5-4eb3-a7f7-8490228caa19
2020-08-01 14:12:48.229 4520 INFO cinder.volume.manager [req-b646caca-75fb-417e-a8cc-0a6a3de1b23f 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Deleted volume successfully.
```

## Tạo VM từ volume
File `volume.log`
```log
2020-08-01 14:16:23.725 4520 INFO cinder.volume.targets.lio [req-d07bbd0c-0dbc-4fb8-b815-140fa179d8e4 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Creating iscsi_target for volume: volume-7b8068fd-01dc-44c6-a393-a11c483b99ad
2020-08-01 14:16:28.870 4520 INFO cinder.volume.manager [req-d07bbd0c-0dbc-4fb8-b815-140fa179d8e4 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] attachment_update completed successfully.
```

## Tạo backup volume
Ví dụ: Tạo backup cho volume
```
openstack volume backup create --name bak-vl01-cirros vl01-cirros
```

File `volume.log`
```log
2020-08-01 14:34:43.124 4520 INFO cinder.volume.targets.lio [req-621845b1-7cb0-4b62-8c8e-2508992e6138 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Creating iscsi_target for volume: volume-7b8068fd-01dc-44c6-a393-a11c483b99ad
2020-08-01 14:34:47.026 4520 INFO cinder.volume.manager [req-621845b1-7cb0-4b62-8c8e-2508992e6138 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Initialize volume connection completed successfully.
2020-08-01 14:37:31.108 4520 INFO cinder.volume.manager [req-621845b1-7cb0-4b62-8c8e-2508992e6138 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Terminate volume connection completed successfully.
2020-08-01 14:37:31.990 4520 INFO cinder.volume.targets.lio [req-621845b1-7cb0-4b62-8c8e-2508992e6138 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Removing iscsi_target: 7b8068fd-01dc-44c6-a393-a11c483b99ad
2020-08-01 14:37:33.537 4520 INFO cinder.volume.manager [req-621845b1-7cb0-4b62-8c8e-2508992e6138 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Remove volume export completed successfully.
```

File `backup.log`
```log
2020-08-01 14:34:42.343 5085 INFO cinder.backup.manager [req-621845b1-7cb0-4b62-8c8e-2508992e6138 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Create backup started, backup: 7fef4195-f2e0-413b-8651-c445cf6533d3 volume: 7b8068fd-01dc-44c6-a393-a11c483b99ad.
2020-08-01 14:34:47.037 5085 INFO os_brick.initiator.connectors.iscsi [req-621845b1-7cb0-4b62-8c8e-2508992e6138 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Trying to connect to iSCSI portal 10.10.31.166:3260
2020-08-01 14:34:47.123 5085 WARNING os_brick.initiator.connectors.iscsi [req-621845b1-7cb0-4b62-8c8e-2508992e6138 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] iscsiadm stderr output when getting sessions: iscsiadm: No active sessions.

2020-08-01 14:37:31.203 5085 INFO cinder.backup.manager [req-621845b1-7cb0-4b62-8c8e-2508992e6138 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Create backup finished. backup: 7fef4195-f2e0-413b-8651-c445cf6533d3.
```

Khi backup thành công sẽ thấy dòng chữ `Create backup finished`

## Restore volume
Ví dụ: Restore volume

File `volume.log`
```log
2020-08-01 14:43:21.595 4520 INFO cinder.volume.targets.lio [req-52b04541-df42-494a-bd7d-44e19be26cd3 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Creating iscsi_target for volume: volume-7b8068fd-01dc-44c6-a393-a11c483b99ad
2020-08-01 14:43:25.460 4520 INFO cinder.volume.manager [req-52b04541-df42-494a-bd7d-44e19be26cd3 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Initialize volume connection completed successfully.
2020-08-01 14:46:19.845 4520 INFO cinder.volume.manager [req-52b04541-df42-494a-bd7d-44e19be26cd3 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Terminate volume connection completed successfully.
2020-08-01 14:46:20.747 4520 INFO cinder.volume.targets.lio [req-52b04541-df42-494a-bd7d-44e19be26cd3 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Removing iscsi_target: 7b8068fd-01dc-44c6-a393-a11c483b99ad
2020-08-01 14:46:22.326 4520 INFO cinder.volume.manager [req-52b04541-df42-494a-bd7d-44e19be26cd3 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Remove volume export completed successfully.
```

File `backup.log`
```log
2020-08-01 14:43:20.670 5085 INFO cinder.backup.manager [req-52b04541-df42-494a-bd7d-44e19be26cd3 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Restore backup started, backup: 7fef4195-f2e0-413b-8651-c445cf6533d3 volume: 7b8068fd-01dc-44c6-a393-a11c483b99ad.
2020-08-01 14:43:25.469 5085 INFO os_brick.initiator.connectors.iscsi [req-52b04541-df42-494a-bd7d-44e19be26cd3 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Trying to connect to iSCSI portal 10.10.31.166:3260
2020-08-01 14:43:25.587 5085 WARNING os_brick.initiator.connectors.iscsi [req-52b04541-df42-494a-bd7d-44e19be26cd3 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] iscsiadm stderr output when getting sessions: iscsiadm: No active sessions.

2020-08-01 14:46:19.984 5085 INFO cinder.backup.manager [req-52b04541-df42-494a-bd7d-44e19be26cd3 294c5c6181d442c68a13d5b615c4f031 5b4c1d2155004acf849cd3aac03b8f36 - default default] Finished restoring backup 7fef4195-f2e0-413b-8651-c445cf6533d3 to volume 7b8068fd-01dc-44c6-a393-a11c483b99ad.
```

Khi restore volume thành công, ta sẽ thấy dòng `Finished restoring backup`

## Cấu hình sai
Ví dụ: Thử đổi pass rabbit trong file cấu hình
```
[DEFAULT]
transport_url = rabbit://openstack:Welcome1234@10.10.31.166
```
Sau đó restart lại dịch vụ:
```
systemctl restart openstack-cinder-*
```

2 file volume.log và backup.log sẽ sinh log ERROR liên tục

File `volume.log`

<img src="..\images\Screenshot_77.png">


File `backup.log`

<img src="..\images\Screenshot_78.png">
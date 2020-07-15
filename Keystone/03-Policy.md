# Policy

## File `policy.json`
- Người dùng trên hệ thống được khởi tạo sẽ lưu vào database của keystone và được sử dụng để xác thực và làm việc với API. Điều đó là chuyện bình thường nhưng để phục vụ bài toán xa hơn Openstack sử dụng chức năng role để giải quyết bài toán role-based access, điều là nguyên nhân của policy trong openstack.

- Mỗi service trong Openstack, indenity, networking, compute... đều có một cơ chế quản lý quyền role-based access policies riêng của chúng. Các role-based này được chúng sử dụng để xác định một người dùng nào có thể truy cập vào một object nào trong chúng, và các policy này được định nghĩa tại file `policy.json`

- Khi một lời gọi tới API, các policy trên serivice sẽ được sử dụng để kiểm tra và chấp nhận các request này. Mỗi khi cập nhật một rule trên file policy.json thì sẽ có tác dụng ngay lập tức trong khi service đang chạy

- Một file `policy.json` được định dạng dưới dạng JSON. Mỗi policy sẽ được định nghĩa theo format sau:
    ```
    <target>" : "<rule>"
    ```

- Với target sẽ là một action, đại diện cho một API có thể là `os_compute_api:servers:create`. Các action name này sẽ tùy theo từng loại dịch vụ sẽ có các đặc tính đi kèm, và được đặt tại file "`/etc/{service_name}/policy.json`" để các service được hiểu đây là các action cần được kiểm tra.

- Với rule sẽ thường được sử dụng để xác định một role hay người dùng nào được làm việc hay không làm việc với API target

- Ví dụ : Cho phép người dùng khởi tạo một user mới , thì sẽ chỉ rõ role test sẽ được khởi tạo
    ```
    "identity:create_user" : "role: test"
    ```

- Ngoài ra còn có thể sử dụng chức năng alias để tạo một rule định nghĩa sẵn, rule này như một hằng được sử dụng nhiều lần trên các target có cùng rule giống nhau.
    ```
    "admin_required": "role:admin or is_admin:1",
    "owner" : "user_id:%(user_id)s",
    "admin_or_owner": "rule:admin_required or rule:owner",
    "identity:change_password": "rule:admin_or_owner"
    ```


**Bản Train:** Mặc định file policy

Để gen các policy mặc định, ta thực hiện như sau:
> ## Check lại -> gen ra dạng yaml chứ không phải json
```
oslopolicy-policy-generator --namespace keystone --output-file /etc/keystone/policy.yaml
oslopolicy-policy-generator --namespace glance --output-file /etc/glance/policy.yaml
oslopolicy-policy-generator --namespace nova --output-file /etc/nova/policy.yaml
oslopolicy-policy-generator --namespace neutron --output-file /etc/neutron/policy.yaml
oslopolicy-policy-generator --namespace cinder --output-file /etc/cinder/policy.yaml
```

Và để sử dụng file `.yaml` làm policy ta chỉnh trong file cấu hình keystone:
```
[oslo_policy]
policy_file = policy.yaml
```
Restart httpd
```
systemctl restart httpd
```

## Lab Thêm policy
- Thêm thử dòng sau vào file `policy.yaml` để cho role `test` có quyền list user:
```
"identity:list_users" : "role:test"
```

- Tạo user và gán role `test` cho user đó.

- Dùng user vừa tạo để list user:
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

**Tuy nhiên:**
- user admin lại không còn list được user
- Dashboard truy cập bị lỗi. Khoong login được:
    
    <img src="..\images\Screenshot_65.png">
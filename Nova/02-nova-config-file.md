# File cấu hình của Nova

File cấu hình Nova đặt tại `/etc/nova/nova.conf`


# 1. File cấu hình trên node Controller

### `[DEFAULT]`
- Ip managerent của node Controller:
    ```ini
    my_ip = 10.10.34.161
    ```

- Hỗ trợ neutron service:
    ```ini
    use_neutron = True
    firewall_driver = nova.virt.firewall.NoopFirewallDriver
    ```

### `[api]`
- Xác thực qua Keystone
    ```ini
    auth_strategy = keystone
    ```

### `[api_database]`

- Kết nối database của nova-api
    ```ini
    connection = mysql+pymysql://nova:Welcome123@10.10.34.161/nova_api
    ```

### `[cache]`
- Cấu hình cache sử dụng memcache:
    ```ini
    backend = oslo_cache.memcache_pool
    enabled = true
    memcache_servers = 10.10.34.161:11211
    ```

### `[database]`
- Cấu hình database của nova-service:
    ```ini
    connection = mysql+pymysql://nova:Welcome123@10.10.34.161/nova
    ```

### `[glance]`
- Đường dẫn dịch vụ glance:
    ```ini
    api_servers = http://10.10.34.161:9292
    ```

### `[keystone_authtoken]`
- URL xác thực với keyston
    ```ini
    auth_url = http://10.10.34.161:5000/v3
    ```

- Đường dẫn memachech server:
    ```ini
    memcached_servers = 10.10.34.161:11211
    ```

- Kiểu xác thực:
    ```ini
    auth_type = password
    ```

- Thông tin domain:
    ```ini
    project_domain_name = default
    user_domain_name = default
    ```

- Project name của Nova
    ```ini
    project_name = service
    ```

- Thông tin xác thực:
    ```ini
    username = nova
    password = Welcome123
    ```

- Region:
    ```ini
    region_name = RegionOne
    ```

### `[neutron]`
- Các cấu hình liên quan tới neutron-service:
    ```ini
    url = http://10.10.34.161:9696
    auth_url = http://10.10.34.161:35357
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    region_name = RegionOne
    project_name = service
    username = neutron
    password = Welcome123
    service_metadata_proxy = true
    metadata_proxy_shared_secret = Welcome123
    ```

### `[placement]`
- các thông tin xác thực, project với placement:
    ```ini
    os_region_name = RegionOne
    project_domain_name = Default
    project_name = service
    auth_type = password
    user_domain_name = Default
    auth_url = http://10.10.34.161:5000/v3
    username = placement
    password = Welcome123
    ```

### `[vnc]`
- Địa chỉ management IP của VNC proxy:
    ```ini
    novncproxy_host=10.10.34.161
    enabled = true
    vncserver_listen = 10.10.34.161
    vncserver_proxyclient_address = 10.10.34.161
    novncproxy_base_url = http://10.10.34.161:6080/vnc_auto.html
    ```

# 2. File cấu hình của Nova-compute
Các cấu hình khác so với Nova trên Controller

### `[libvirt]`
- Loại ảo hóa sử dụng: Có thể là `kvm`, `lxc`, `qemu`, `uml`, `xen`, `parallels`
    ```ini
    virt_type = kvm
    ```


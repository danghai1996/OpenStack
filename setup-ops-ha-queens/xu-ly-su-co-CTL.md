# Xử lý sự cố liên quan tới node Controller

Các node down theo 2 kiểu:
- Tắt an toàn: `init 0` hoặc node bị reboot
- Tắt nguy hiểm: mất điện, hay force off, tắt đột ngột node

# Case 1: 1 node Controller down
## Tắt an toàn
Mô phỏng trường hợp lab:
- Trên node CTL3, thực hiện tắt:
    ```
    init 0
    ```
    hoặc reboot
    ```
    init 6
    ```

**Cách xử lý:**

Thường sẽ không có lỗi gì, sau khi node online trở lại

- Sau khi node CTL3 khởi động trở lại, ta kiểm tra các service của OPS, Galera, Pacemaker corosyn, RabbitMQ
- Sau khi các service lên hết tiến hành tạo VM test
- Đồng thời kiểm tra trên trang stats pace HAproxy (http://10.10.34.170:8080/stats)

## Tắt nguy hiểm
Mô phỏng lab:
- Ta thực hiện Force off node CTL3 (Đối với WebvirtCloud KVM)

**Cách xử lý:**
- Trước khi bật lại node CTL3: Kiểm tra xem có VM tạo lỗi trong thời điểm CTL3 bị down hay không.

    Lưu ý: Nếu trong thời điểm CTL3 có VM đang khởi tạo thì VM đó có thể bị lỗi

- Sau khi bật node CTL3 online trở lại:
    - Ta kiểm tra các service của OPS, Galera, Pacemaker corosyn, RabbitMQ
    - Sau khi các service lên. Tạo VM test
    - Kiểm tra đồng thời các HAproxy trên trang stats page (http://10.10.34.170:8080/stats)

# Case 2: 2 node Controller down
## 2 node down an toàn
- Với mô hình 3 node, 2 node down an toàn thì dịch vụ chạy bình thường.

Mô phỏng lab:
- Thực hiện tắt bằng lệnh `init 0` trên 2 node CTL2 và CTL3

**Cách xử lý:**

Sau khi bật các node :

- Ta kiểm tra các service của OPS, Galera, Pacemaker corosyn, RabbitMQ. Nếu có lỗi thì thực hiện theo docs xử lý lỗi theo từng phần (Pacemaker, RabbitMQ, Galera)
- Sau khi các service lên. Tạo VM test
- Kiểm tra đồng thời các HAproxy trên trang stats page (http://10.10.34.170:8080/stats)

## 2 node down không an toàn
Nếu dịch vụ tại 2 trong 3 node down không an toàn, dịch vụ mariadb tại cả 3 node sẽ không thể hoạt động do cơ chế quorum nhằm tránh dữ liệu bị phân mảnh, khác nhau tại 3 node

Mô phỏng lab:
- Force off 2 CTL: CTL2, CTL3

-> Sau đó, trên COM sẽ có log dạng như sau:

File log: `/etc/nova/nova-compute.log
```
2020-11-20 13:56:37.128 19301 ERROR nova.servicegroup.drivers.db Traceback (most recent call last):
2020-11-20 13:56:37.128 19301 ERROR nova.servicegroup.drivers.db   File "/usr/lib/python2.7/site-packages/nova/servicegroup/drivers/db.py", line 91, in _report_state
2020-11-20 13:56:37.128 19301 ERROR nova.servicegroup.drivers.db     service.service_ref.save()
2020-11-20 13:56:37.128 19301 ERROR nova.servicegroup.drivers.db   File "/usr/lib/python2.7/site-packages/oslo_versionedobjects/base.py", line 210, in wrapper
2020-11-20 13:56:37.128 19301 ERROR nova.servicegroup.drivers.db     ctxt, self, fn.__name__, args, kwargs)
2020-11-20 13:56:37.128 19301 ERROR nova.servicegroup.drivers.db   File "/usr/lib/python2.7/site-packages/nova/conductor/rpcapi.py", line 245, in object_action
2020-11-20 13:56:37.128 19301 ERROR nova.servicegroup.drivers.db     objmethod=objmethod, args=args, kwargs=kwargs)
2020-11-20 13:56:37.128 19301 ERROR nova.servicegroup.drivers.db   File "/usr/lib/python2.7/site-packages/oslo_messaging/rpc/client.py", line 174, in call
2020-11-20 13:56:37.128 19301 ERROR nova.servicegroup.drivers.db     retry=self.retry)
2020-11-20 13:56:37.128 19301 ERROR nova.servicegroup.drivers.db   File "/usr/lib/python2.7/site-packages/oslo_messaging/transport.py", line 131, in _send
2020-11-20 13:56:37.128 19301 ERROR nova.servicegroup.drivers.db     timeout=timeout, retry=retry)
2020-11-20 13:56:37.128 19301 ERROR nova.servicegroup.drivers.db   File "/usr/lib/python2.7/site-packages/oslo_messaging/_drivers/amqpdriver.py", line 625, in send
2020-11-20 13:56:37.128 19301 ERROR nova.servicegroup.drivers.db     retry=retry)
2020-11-20 13:56:37.128 19301 ERROR nova.servicegroup.drivers.db   File "/usr/lib/python2.7/site-packages/oslo_messaging/_drivers/amqpdriver.py", line 616, in _send
2020-11-20 13:56:37.128 19301 ERROR nova.servicegroup.drivers.db     raise result
```

Khi thực hiện lệnh trên node CTL1 còn lại
```
[root@ctl01 ~]# openstack token issue
An unexpected error prevented the server from fulfilling your request. (HTTP 500) (Request-ID: req-2611f0fc-4e36-4d3c-9d23-368bacf925bb)
```

**Cách giải quyết:**

Chắc chắn 2 node CTL2, CTL3 đang tắt

Xử lý Galera
- Tại node không lỗi (CTL1) thực hiện:
```
mysql -uroot -pWelcome123 -e "SET GLOBAL wsrep_provider_options='pc.bootstrap=1';"
```
- Sau đó, bật 2 node lên, các node sẽ tự join cluster bình thường

Xử lý Pacemaker:
- Kiểm tra pacemaker. Nếu có vấn đề thì xử lý theo docs

Xử lý RabbitMQ:
- Kiểm tra cluster. Nếu có vấn đề thì xử lý theo docs

Kiểm tra:
- Theo dõi log trên các node compute mà máy ảo tạo vào
- Tạo 1 VM test
- Theo dõi log compute, nếu service nào lỗi thì restart lại các service đó

# Case 3: 3 node Controller down
## 3 node down không an toàn
Mô phỏng lab:
- Force off cả 3 node CTL

## 3 node down an toàn


# Case 4: Khởi tạo lại Cluster RabbitMQ
Thực hiện theo docs.

Lưu ý lúc khởi tạo: tạo user và phân quyền thêm

Xóa bỏ dữ liệu cũ của cluster
```
cd /var/lib/rabbitmq/mnesia/
rm -rf *
```

Loại bỏ tiến trình RabbitMQ
```
# Kiểm tra tiến trình
ps -ef | grep rabbitmq 

# Loại bỏ
pkill -KILL -u rabbitmq
```

Khởi tạo lại tiến trình
```
systemctl restart rabbitmq-server
```

Kiểm tra lại dịch vụ
```
systemctl status rabbitmq-server
```


Kiểm tra danh sách user hiện có
```
rabbitmqctl list_users
```

Kiểm tra trạng thái Cluster
```
rabbitmqctl cluster_status
```

Làm mới Cluster hiện có
```
rabbitmqctl reset
```

## Cấu hình Cluster RabbitMQ
> ## Node CTL1
Khởi tạo RabbitMQ Cluster:
```
rabbitmqctl add_user openstack Welcome123
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
rabbitmqctl set_user_tags openstack administrator
rabbitmqctl set_policy ha-all '^(?!amq\.).*' '{"ha-mode": "all"}'


scp -p /var/lib/rabbitmq/.erlang.cookie ctl02:/var/lib/rabbitmq/.erlang.cookie
scp -p /var/lib/rabbitmq/.erlang.cookie ctl03:/var/lib/rabbitmq/.erlang.cookie

rabbitmqctl start_app
rabbitmqctl cluster_status
```

Kiểm tra:
```
[root@ctl01 ~]# rabbitmqctl cluster_status
Cluster status of node rabbit@ctl01
[{nodes,[{disc,[rabbit@ctl01]}]},
 {running_nodes,[rabbit@ctl01]},
 {cluster_name,<<"rabbit@ctl01">>},
 {partitions,[]},
 {alarms,[{rabbit@ctl01,[]}]}]
```

> ## Node CTL2
Thực hiện join cluster
```
chown rabbitmq:rabbitmq /var/lib/rabbitmq/.erlang.cookie
chmod 400 /var/lib/rabbitmq/.erlang.cookie

systemctl restart rabbitmq-server.service

rabbitmqctl stop_app
rabbitmqctl join_cluster rabbit@ctl01
rabbitmqctl start_app
```

> ## Node CTL3
Thực hiện join cluster
```
chown rabbitmq:rabbitmq /var/lib/rabbitmq/.erlang.cookie
chmod 400 /var/lib/rabbitmq/.erlang.cookie

systemctl restart rabbitmq-server.service

rabbitmqctl stop_app
rabbitmqctl join_cluster rabbit@ctl01
rabbitmqctl start_app
```

# Case 5: Di chuyển VIP giữa các node
Kiểm tra VIP:
```
[root@ctl01 ~]# pcs status
Cluster name: ha_cluster
Stack: corosync
Current DC: ctl01 (version 1.1.23-1.el7-9acf116022) - partition with quorum
Last updated: Fri Nov 20 15:50:00 2020
Last change: Fri Nov 20 15:08:50 2020 by root via crm_resource on ctl01

3 nodes configured
2 resource instances configured

Online: [ ctl01 ctl02 ctl03 ]

Full list of resources:

 vip_public     (ocf::heartbeat:IPaddr2):       Started ctl01
 p_haproxy      (systemd:haproxy):      Started ctl01

Daemon Status:
  corosync: active/enabled
  pacemaker: active/enabled
  pcsd: active/enabled
```

VIP hiện đang trên node CTL1

Thực hiện chuyển VIP sang node CTL2

- Tạo ràng buộc di chuyển:
    ```
    pcs resource move vip_public ctl02
    ```

- Kiểm tra lại ràng buộc
    ```
    [root@ctl01 ~]# pcs constraint
    Location Constraints:
    Resource: vip_public
        Enabled on: ctl02 (score:INFINITY) (role: Started)
    Ordering Constraints:
    start vip_public then start p_haproxy (kind:Mandatory)
    Colocation Constraints:
    vip_public with p_haproxy (score:INFINITY)
    Ticket Constraints:
    ```

- Khởi động lại resource. Ta thực hiện ban VIP trên node CTL1
    ```
    pcs resource ban vip_public
    ```

- Kiểm tra lại:
    ```
    [root@ctl02 ~]# pcs status
    Cluster name: ha_cluster
    Stack: corosync
    Current DC: ctl01 (version 1.1.23-1.el7-9acf116022) - partition with quorum
    Last updated: Fri Nov 20 15:55:35 2020
    Last change: Fri Nov 20 15:54:40 2020 by root via crm_resource on ctl01

    3 nodes configured
    2 resource instances configured

    Online: [ ctl01 ctl02 ctl03 ]

    Full list of resources:

    vip_public     (ocf::heartbeat:IPaddr2):       Started ctl02
    p_haproxy      (systemd:haproxy):      Started ctl02

    Daemon Status:
    corosync: active/enabled
    pacemaker: active/enabled
    pcsd: active/enabled
    ```

- Kiểm tra IP trên node CTL2
    ```
    [root@ctl02 ~]# ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
        valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
        valid_lft forever preferred_lft forever
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 52:54:00:78:05:65 brd ff:ff:ff:ff:ff:ff
        inet 10.10.34.162/24 brd 10.10.34.255 scope global noprefixroute eth0
        valid_lft forever preferred_lft forever
        inet 10.10.34.170/24 brd 10.10.34.255 scope global secondary eth0
        valid_lft forever preferred_lft forever
        inet6 fe80::5054:ff:fe78:565/64 scope link noprefixroute
        valid_lft forever preferred_lft forever
    3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 52:54:00:ab:74:23 brd ff:ff:ff:ff:ff:ff
        inet 10.10.31.162/24 brd 10.10.31.255 scope global noprefixroute eth1
        valid_lft forever preferred_lft forever
        inet6 fe80::82ba:51e6:883e:5faa/64 scope link noprefixroute
        valid_lft forever preferred_lft forever
    4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 52:54:00:d8:d7:9c brd ff:ff:ff:ff:ff:ff
        inet 10.10.32.162/24 brd 10.10.32.255 scope global noprefixroute eth2
        valid_lft forever preferred_lft forever
        inet6 fe80::cebe:9ddb:bbc9:3c49/64 scope link noprefixroute
        valid_lft forever preferred_lft forever
    5: eth3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 52:54:00:d8:51:30 brd ff:ff:ff:ff:ff:ff
        inet 10.10.33.162/24 brd 10.10.33.255 scope global noprefixroute eth3
        valid_lft forever preferred_lft forever
        inet6 fe80::cd59:9ba0:1423:9939/64 scope link noprefixroute
        valid_lft forever preferred_lft forever
    ```

- Loại bỏ ràng buộc tạm thời để resource trở lại bình thường
    ```
    pcs resource clear vip_public
    ```
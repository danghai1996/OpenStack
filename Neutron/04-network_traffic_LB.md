# Network traffic trong Linux Bridge

# I. Provider network
## 1. North - South (Bắc - Nam): instance with IP Floating
- Instance nằm trên node Compute1 và sử dụng provider network 1
- Instance gửi packet đến một máy chủ khác trên Internet

<img src="..\images\Screenshot_101.png">

**Các bước thực hiện trên node compute 1:**

1. Instance interface (1) forward packet tới instance port tương ứng trên provider bridge (2) thông qua `veth` pair
2. Các rules của Security group (3) trên provider bridge sẽ xử lý firewall và theo dõi kết nối của packet
3. CÁc sub-interface (4) trên provider bridge sẽ forward packet tới physical interface (5)
4. Physical interface (5) sẽ thêm gắn thêm tag VLAN 101 (trong hình) vào packet và forward nó tới switch ngoài hạ tầng (6)

**Các bước thực hiện liên quan đến hạ tầng vật lý (physical network infrastructure):**

1. Switch sẽ bỏ tag VLAN 101 khỏi packet và forward tới router (7)
2. Router (7) sẽ định tuyến cho packet từ provider network (8) tới external network -mạng ngoài (9) và forward packet tới switch - mạng ngoài(10)
3. Switch (10) forward packet ra mạng ngoài- external network (11)
4. Mạng ngoài - external network (12) sẽ nhận packet để tiếp tục thực hiện gửi tới host đích.

## 2. East - West (Đông - Tây): Instances on the same network
Các instance trên cùng 1 mạng giao tiếp trực tiếp giữa các node compute chứa các instance đó:
- Instance 1 : nằm trên node Compute1 và sử dụng provider network 1
- Instance 2 : nằm trên node Compute1 và sử dụng provider network 2 
- Instance 1 gửi packet tới instance 2

<img src="..\images\Screenshot_102.png">

**Các bước liên quan đến node Compute1:**

1. Instance interface (1) forward packet tới instance port tương ứng trên provider bridge (2) thông qua `veth` pair
2. Các rules của Security group (3) trên provider bridge sẽ xử lý firewall và theo dõi kết nối của packet
3. CÁc sub-interface (4) trên provider bridge sẽ forward packet tới physical interface (5)
4. Physical interface (5) sẽ thêm gắn thêm tag VLAN 101 (trong hình) vào packet và forward nó tới switch ngoài hạ tầng (6)

**Các bước liên quan đến cơ sở hạ tầng vật lý:**

5. Switch forward packet từ compute1 tới compute2 (7)

**Các bước liên quan đến node Compute2:**

6. Physical interface của node Compute2 (8) bỏ VLAN tag 101 khỏi packet và forward nó đến sub-interface (9) trên provider bridge
7. Các rules của Security group (10) trên provider bridge sẽ xử lý firewall và theo dõi kết nối của packet
8. Instance port trên provider bridge (11) forward packet đến interface của instance 2 (12) thông qua `veth` pair

## 3. East -West (Đông - Tây): Instances on different networks
Các instance khác mạng sẽ giao tiếp thông qua Router trên hạ tầng mạng vật lý.
- Instance 1 nằm trên node Compute1 và sử dụng provider network 1
- Instance 2 nằm trên node Compute1 và sử dụng provider network 2
- Instamce 1 gửi packet tới instance 2

<img src="..\images\Screenshot_103.png">

**Các bước thực hiện trên node Compute:**

1. Instance interface (1) forward packet tới instance port tương ứng trên provider bridge (2) thông qua `veth` pair
2. Các rules của Security group (3) trên provider bridge sẽ xử lý firewall và theo dõi kết nối của packet
3. CÁc sub-interface (4) trên provider bridge sẽ forward packet tới physical interface (5)
4. Physical interface (5) sẽ thêm gắn thêm tag VLAN 101 (trong hình) vào packet và forward nó tới switch ngoài hạ tầng (6)

**Các bước liên quan đến cơ sở hạ tầng vật lý:**

5. Switch (6) bỏ tag VLAN 101 trên packet và forward nó đến Router (7)
6. Router (7) định tuyến đường đi cho packet từ provider network 1 (8) tới provider network 2 (9)
7. Router forward packet tới switch (10)
8. Switch (10) thêm tag VLAN 102 vào packet và forward tới node Compute1 (11)

**Các bước thực hiện trên node Compute:**

9. Physical interface (12) gỡ tag VLAN 102 khỏi packet và forward nó đến VLAN sub-interface (13) trên provider bridge
10. Các rules của Security group (14) trên provider bridge sẽ xử lý firewall và theo dõi kết nối của packet
11. Instance port tương wungs trên provider bridge (15) sẽ forward packet tới interface của instance 2 (16) thông qua `veth` pair

# II. Self service
Trong self-service các instance sẽ sử dụng IPv4 Private . Để truy cập được interface , networking service sẽ làm nhiệm vụ SNAT ( Source Network Addresss Translation ) để truy cập ra mạng external . . Để từ các mạng có thể truy cập , các instance yêu cầu có một floating IP . Networking service thực hiện DNAT ( desnation network address translation ) từ IP Floating sang IP self-service

## 1. North-south : Instance with a fixed IP address
Với các instance kèm IP v4 Floating, trên network node sẽ thực hiện SNAT để self-service có thể giao tiếp với mạng ngoài.

- Instance ở node Compute1 và sử dụng manjgn self-service 1
- Instance gửi packet tới host ngoài internet

<img src="..\images\Screenshot_104.png">

**Các bước liên quan đến node Compute1:**
1. Instance interface (1) forward packet tới port tương ứng trên self-service bridge (2) thông qua `veth` pair
2. Các rules của Security group (3) trên self-service bridge sẽ xử lý firewall và theo dõi kết nối của packet
3. Self-service bridge forward packet tới VXLAN interface trên bridge (4) kèm VNI
4. Physical interface (5) cho phép mạng VXLAN interface forward packet tới node Network thông qua overlay network (6)

**Các bước liên quan đến node Network:**
1. Physical network (7) nhận packet từ Overlay network VXLAN interface sau đó forward tới self-service bridge port (8)
2. Self-service bridge router port (9) forward packet tới self-service network interface (10) trong rourter namespace
    - Đối với IPv4, router thực hiện SNAT trên packet thay đổi IP nguồn thành địa chỉ IP của router trên provider network và gửi nó đến địa chỉ IP gateway  của provider network thông qua gateway interface trên provider network (11)
3. Router forward packet tới provider bridge router port (12)
4. VLAN sub-interface port (13) trên provider bridge sẽ forward packet tới physical network interface (14)
5. provider physical network interface (14) gán tag VLAN 101 vào packet và forward nó ra internet thông quan physical network infrastructure (15).

## 2. North-south: Instance with a floating IPv4 address
- Instance nằm trên node Compute1 và sử dụng self-service network 1
- Máy chủ trên internet gửi một packet tới instance

<img src="..\images\Screenshot_105.png">

**Các bước liên quan đến node Network**

1. Physical network infrastructure (1) forward packet tới provider physical network interface (2).
2. provider physical network interface(3) gỡ tag VLAN 101 và forward packet tới VLAN sub-interface (4) trên provider bridge
3. Provider bridge forward packet tới port gateway của self-service router trên provider network (5)
    - Đối với IPv4, router thực hiện DNAT trên packet thay đổi địa chỉ IP đích thành IP của instancetreen self-service network và gửi nó tới địa chỉ IP gateway trên self-service thông qua self-service interface (6).
4. Router forward packet tới self-service bridge router port (7)
5. self-service bridge forwards packet tới VXLAN interface (8) và kèm theo VNI 101
6.  physical interface (9) forward pacekt tới node network thông qua overlay network (10).


**Các bước liên quan đến node Compute:**
7. Physical interface (11) forward packet tới VXLAN interface (12) để mở packet
8. Các security group rules (13) trên self-service bridge xử lý firewall và theo dõi kết nối của packet
9. self-service bridge instance port (14) forwards packet tới interface của instance (15) thông qua `veth`


## 3. Instances on the same network
- Instance 1 nằm trên node Compute1 và sử dụng mạng self-service network 1
- Instance 2 nằm trên node Compute2 và sử dụng mạng self-service network 1
- Instance 1 gửi packet tới Instance 2

<img src="..\images\Screenshot_106.png">

**Các bước liên quan đến node Compute1:**

1. Instance interface (1) chuyển packet đến self-service port tương ứng (2)
2. Các security group rules (3) trên self-service bridge xử lý firewall và theo dõi kết nối của packet
3. self-service bridge forward packet tới VXLAN interface (4) kèm theo VNI 101
4.  physical interface (5) forward packet tới node Compute2 thông qua overlay network (6)

**Các bước liên quan đến node Compute2**

5.  physical interface (7) forward packet tới VXLAN interface (8) để mở packet
6. Security group rules (9) của self-service bridge sẽ xử lý với firewall và theo dõi kết nối của packet
7. self-service bridge instance port (10) forward packet tới interface của instance (11) thông qua `veth`

## 4. Instances on different networks
- Instance1 nằm trên node Compute1 và sử dụng mạng self-service1
- Instance2 nằm trên node Compute1 và sử dụng mạng self-service2
- Instance1 gửi packet tới Instance2

<img src="..\images\Screenshot_107.png">

**Các bước liên quan đến node Compute**

1. Instance interface (1) chuyển packet đến self-service port tương ứng (2)
2. Các security group rules (3) trên self-service bridge xử lý firewall và theo dõi kết nối của packet
3. self-service bridge forward packet tới VXLAN interface (4) kèm theo VNI 101
4. Physical interface (5) cho phép mạng VXLAN interface forward packet tới node Network thông qua overlay network (6)

**Các bước liên quan đến node Network**

5. physical interface (7) forward packet tới VXLAN interface (8) để mở packet
6. self-service bridge router port (9) forwards packet tới self-service network 1 interface (10) trên router namespace.
7. Router gửi packet tới IP tiếp theo, thường là IP gatewa của self-service network2 , thông qua self-service network2 interface (11)
8. Router forward pacekt tới  self-service network 2 bridge router port (12).
9. self-service network 2 bridge forward packet tới VXLAN interface (13) để mở pacekt sử dụng VNI 102
10. physical network interface (14) của VXLAN interface gửi packet tới node compute thông qua overlay network (15)

**Các bước liên quan đến node Compute**

11. physical interface (16) gửi packet tới VXLAN interface (17) để mở packet
12. Security group rules (18) trên  self-service bridge xử lý với firewall và theo dõi kết nối của packet
13.  self-service bridge instance port (19) forward paceket tới interface của instance2 (20) thông qua veth pair.



# Tham khảo
- https://docs.openstack.org/neutron/pike/admin/deploy-lb-provider.html
- https://docs.openstack.org/neutron/pike/admin/deploy-lb-selfservice.html

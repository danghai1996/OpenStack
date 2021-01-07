# Cài đặt Ceph AIO (Luminous)

# Phần 1: Chuẩn bị
## Phân hoạch
- vlan MNGT: eth0: 10.10.34.166
- vlan CephCOM: eth1: 10.10.33.166 
- vlan CephREP: eth2: 10.10.35.166

## Setup node
```
hostnamectl set-hostname cephaio

echo "Setup IP eth0"
nmcli con mod eth0 ipv4.addresses 10.10.34.166/24
nmcli con mod eth0 ipv4.gateway 10.10.34.1
nmcli con mod eth0 ipv4.dns 8.8.8.8
nmcli con mod eth0 ipv4.method manual
nmcli con mod eth0 connection.autoconnect yes

echo "Setup IP eth1"
nmcli con mod eth1 ipv4.addresses 10.10.33.166/24
nmcli con mod eth1 ipv4.method manual
nmcli con mod eth1 connection.autoconnect yes

echo "Setup IP eth2"
nmcli con mod eth2 ipv4.addresses 10.10.35.166/24
nmcli con mod eth2 ipv4.method manual
nmcli con mod eth2 connection.autoconnect yes

sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
systemctl stop firewalld
systemctl disable firewalld

curl -Lso- https://raw.githubusercontent.com/nhanhoadocs/scripts/master/Utilities/cmdlog.sh | bash

init 6
```

Thêm IP đường CEPH-COM vào file hosts:
```
echo "10.10.33.165 cephaio" >> /etc/hosts
```

Cấu hình chrony:
```
Lấy time từ server : 10.10.34.130
```

> ## Snapshot `env`

# Phần 2: Cài đặt Ceph
## Bổ sung user `cephuser`: pass: `Welcome123`
```
useradd -d /home/cephuser -m cephuser
passwd cephuser

echo "cephuser ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/cephuser
chmod 0440 /etc/sudoers.d/cephuser
```

## Bổ sung repo cài đặt Ceph
```
cat <<EOF> /etc/yum.repos.d/ceph.repo
[ceph]
name=Ceph packages for $basearch
baseurl=https://download.ceph.com/rpm-luminous/el7/x86_64/
enabled=1
priority=2
gpgcheck=1
gpgkey=https://download.ceph.com/keys/release.asc

[ceph-noarch]
name=Ceph noarch packages
baseurl=https://download.ceph.com/rpm-luminous/el7/noarch
enabled=1
priority=2
gpgcheck=1
gpgkey=https://download.ceph.com/keys/release.asc

[ceph-source]
name=Ceph source packages
baseurl=https://download.ceph.com/rpm-luminous/el7/SRPMS
enabled=0
priority=2
gpgcheck=1
gpgkey=https://download.ceph.com/keys/release.asc
EOF

yum update -y
```

## Cài đặt python-setuptools và Ceph deploy
```
yum install python-setuptools -y
yum install ceph-deploy -y
```

Kiểm tra version
```
ceph-deploy --version

2.0.1
```

> ## Snapshot `preceph`

## Cấu hình ssh-key
```
ssh-keygen

cat <<EOF> /root/.ssh/config
Host cephaio
    Hostname cephaio
    User cephuser
EOF

ssh-copy-id cephaio
```

## Cấu hình Ceph
Tạo các thư mục `ceph-deploy` để thao tác cài đặt vận hành Cluster
```
mkdir /ceph-deploy && cd /ceph-deploy
```

Khởi tại file cấu hình cho cụm với node quản lý là `cephaio`
```
ceph-deploy new cephaio
```

Cấu hình file: `ceph.conf`
```
cat << EOF >> ceph.conf
osd pool default size = 2
osd pool default min size = 1
osd pool default pg num = 128
osd pool default pgp num = 128

osd crush chooseleaf type = 0

public network = 10.10.33.166/24
cluster network = 10.10.35.166/24
EOF
```
Trong đó:
- Public network là đường Ceph-Com trong file quy hoạch
- Cluster network là đường Ceph-Replicate trong file quy hoạch

## Cài đặt Ceph qua Ceph deploy
Cài đặt
```
ceph-deploy install --release luminous cephaio
```

Kiểm tra version
```
ceph -v

ceph version 12.2.13 (584a20eb0237c657dc0567da126be145106aa47e) luminous (stable)
```

Khởi tạo cluster với các node mon (Monitor-quản lý) dựa trên file `ceph.conf`
```
ceph-deploy mon create-initial
```

Để node `cephaio` có thể thao tác với cluster chúng ta cần gán cho node `cephaio` với quyền admin bằng cách bổ sung cho node này `admin.keying`
```
ceph-deploy admin cephaio
```

Kiểm tra:
```
[root@cephaio ceph-deploy]# ceph -s
  cluster:
    id:     f81764ee-c551-4ea4-8dfc-356bce4dbbb5
    health: HEALTH_OK

  services:
    mon: 1 daemons, quorum cephaio
    mgr: no daemons active
    osd: 0 osds: 0 up, 0 in

  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0B
    usage:   0B used, 0B / 0B avail
    pgs:
```

## Khởi tạo MGR
Ceph-mgr là thành phần cài đặt yêu cầu cần khởi tạo từ bản Luminous, có thể cài đặt trên nhiều node hoạt động theo cơ chế Active-Passive

Cài đặt ceph-mgr trên cephaio
```
ceph-deploy mgr create cephaio
```

Ceph-mgr hỗ trợ dashboard để quan sát trạng thái của cluster, Enable mgr dashboard trên host cephaio
```
sudo ceph mgr module enable dashboard
sudo ceph mgr services
```

Truy cập
```
http://<ip-cephaio>:7000
```

## Bổ sung OSD
```
ceph-deploy disk zap cephaio /dev/vdb
ceph-deploy disk zap cephaio /dev/vdc
ceph-deploy disk zap cephaio /dev/vdd

ceph-deploy osd create --data /dev/vdb cephaio
ceph-deploy osd create --data /dev/vdc cephaio
ceph-deploy osd create --data /dev/vdd cephaio
```

Điều chỉnh Crushmap để có thể Replicate trên OSD thay vì trên HOST
```
cd /ceph-deploy
sudo ceph osd getcrushmap -o crushmap
sudo crushtool -d crushmap -o crushmap.decom
sudo sed -i 's|step choose firstn 0 type osd|step chooseleaf firstn 0 type osd|g' crushmap.decom
sudo crushtool -c crushmap.decom -o crushmap.new
sudo ceph osd setcrushmap -i crushmap.new
```

## Kiểm tra lại trạng thái và osd
TRạng thái
```
[root@cephaio ~]# ceph -s

  cluster:
    id:     f81764ee-c551-4ea4-8dfc-356bce4dbbb5
    health: HEALTH_OK

  services:
    mon: 1 daemons, quorum cephaio
    mgr: cephaio(active)
    osd: 3 osds: 3 up, 3 in

  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0B
    usage:   3.01GiB used, 147GiB / 150GiB avail
    pgs:
```

OSD
```
[root@cephaio ~]# ceph osd df tree

ID CLASS WEIGHT  REWEIGHT SIZE    USE     DATA    OMAP META AVAIL   %USE VAR  PGS TYPE NAME
-1       0.14600        -  150GiB 3.01GiB 5.44MiB   0B 3GiB  147GiB 2.00 1.00   - root default
-3       0.14600        -  150GiB 3.01GiB 5.44MiB   0B 3GiB  147GiB 2.00 1.00   -     host cephaio
 0   hdd 0.04900  1.00000 50.0GiB 1.00GiB 1.81MiB   0B 1GiB 49.0GiB 2.00 1.00   0         osd.0
 1   hdd 0.04900  1.00000 50.0GiB 1.00GiB 1.81MiB   0B 1GiB 49.0GiB 2.00 1.00   0         osd.1
 2   hdd 0.04900  1.00000 50.0GiB 1.00GiB 1.81MiB   0B 1GiB 49.0GiB 2.00 1.00   0         osd.2
                    TOTAL  150GiB 3.01GiB 5.44MiB   0B 3GiB  147GiB 2.00
MIN/MAX VAR: 1.00/1.00  STDDEV: 0
```
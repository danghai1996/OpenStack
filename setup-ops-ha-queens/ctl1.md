# Cấu hình trên Controler1

# Phần 1
Set hostname :
```
hostnamectl set-hostname ctl01
```

Cấu hình network:
```
nmcli con mod eth0 ipv4.address 10.10.34.161/24
nmcli con mod eth0 ipv4.gateway 10.10.34.1
nmcli con mod eth0 ipv4.dns 8.8.8.8
nmcli con mod eth0 ipv4.method manual
nmcli con mod eth0 connection.autoconnect yes

nmcli con mod eth1 ipv4.address 10.10.31.161/24
nmcli con mod eth1 ipv4.method manual
nmcli con mod eth1 connection.autoconnect yes

nmcli con mod eth2 ipv4.address 10.10.32.161/24
nmcli con mod eth2 ipv4.method manual
nmcli con mod eth2 connection.autoconnect yes

nmcli con mod eth3 ipv4.address 10.10.33.161/24
nmcli con mod eth3 ipv4.method manual
nmcli con mod eth3 connection.autoconnect yes
```

Selinux, Firewalld
```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
systemctl stop firewalld
systemctl disable firewalld
```

CMD log
```
curl -Lso- https://raw.githubusercontent.com/nhanhoadocs/scripts/master/Utilities/cmdlog.sh | bash
```


Cấu hình chronyd
```
Lấy time từ IP: 10.10.34.130
```

Reboot 
```
init 6
```

Chuẩn bị sysctl
```
echo 'net.ipv4.conf.all.arp_ignore = 1'  >> /etc/sysctl.conf
echo 'net.ipv4.conf.all.arp_announce = 2'  >> /etc/sysctl.conf
echo 'net.ipv4.conf.all.rp_filter = 2'  >> /etc/sysctl.conf
echo 'net.netfilter.nf_conntrack_tcp_be_liberal = 1'  >> /etc/sysctl.conf

cat << EOF >> /etc/sysctl.conf
net.ipv4.ip_nonlocal_bind = 1
net.ipv4.tcp_keepalive_time = 6
net.ipv4.tcp_keepalive_intvl = 3
net.ipv4.tcp_keepalive_probes = 6
net.ipv4.ip_forward = 1
net.ipv4.conf.all.rp_filter = 0
net.ipv4.conf.default.rp_filter = 0
EOF

sysctl -p
```

Cấu hình hostname
```
echo "10.10.34.161 ctl01" >> /etc/hosts
echo "10.10.34.162 ctl02" >> /etc/hosts
echo "10.10.34.163 ctl03" >> /etc/hosts
echo "10.10.34.164 com01" >> /etc/hosts
echo "10.10.34.165 com02" >> /etc/hosts
```

Setup keypair
```
ssh-keygen -t rsa -f /root/.ssh/id_rsa -q -P ""
ssh-copy-id -o StrictHostKeyChecking=no -i /root/.ssh/id_rsa.pub root@ctl02
ssh-copy-id -o StrictHostKeyChecking=no -i /root/.ssh/id_rsa.pub root@ctl03
```

> ## Tắt, snapshot `env`

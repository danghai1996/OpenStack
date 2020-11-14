# Cấu hình trên Compute02

# Phần 1
Set hostname :
```
hostnamectl set-hostname com02
bash
```

Cấu hình network:
```
nmcli con mod eth0 ipv4.address 10.10.34.165/24
nmcli con mod eth0 ipv4.gateway 10.10.34.1
nmcli con mod eth0 ipv4.dns 8.8.8.8
nmcli con mod eth0 ipv4.method manual
nmcli con mod eth0 connection.autoconnect yes

nmcli con mod eth1 ipv4.address 10.10.31.165/24
nmcli con mod eth1 ipv4.method manual
nmcli con mod eth1 connection.autoconnect yes

nmcli con mod eth2 ipv4.address 10.10.32.165/24
nmcli con mod eth2 ipv4.method manual
nmcli con mod eth2 connection.autoconnect yes

nmcli con mod eth3 ipv4.address 10.10.33.165/24
nmcli con mod eth3 ipv4.method manual
nmcli con mod eth3 connection.autoconnect yes

systemctl restart NetworkManager
```

Selinux, Firewalld
```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
systemctl stop firewalld
systemctl disable firewalld
```
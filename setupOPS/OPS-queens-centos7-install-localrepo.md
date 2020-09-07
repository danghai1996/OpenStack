# Các bước cài đặt Openstack Queens với Local Repo

## Mục tiêu:
Cài đặt được OpenStack mà không cần sử dụng đến Internet


# Thực hiện:
## 
```
yum install yum-utils -y
```

```
yumdownloader --destdir=/root/backups-ops-queens/centos-release-openstack-queens/ --resolve centos-release-openstack-queens
```

```
rpm -ivh *.rpm
```
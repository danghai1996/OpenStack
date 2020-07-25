# Các module phổ biến được dùng trong cloud-init

# 1. cloud_init_modules
## 1.1. `bootcmd`
Chạy câu lệnh ở ngay lúc quá trình boot mới diễn ra. 

**Lưu ý:** Nó chỉ nên được dùng cho những phần sẽ không làm được ở giai đoạn sau của quá trình boot.

**Config keys:**
```yaml
bootcmd:
    - echo 192.168.1.130 us.archive.ubuntu.com > /etc/hosts
    - [ cloud-init-per, once, mymkfs, mkfs, /dev/vdb ]
```

## 1.2. `set_hostname`
Đặt hostname và fqdn(Fully Qualified Domain). Nếu tùy chọn preserve_hostname được thiết lập thì hostname sẽ không thể bị thay đổi sau này.

**Distro hỗ trợ:** tất cả

**Config keys:**
```yaml
preserve_hostname: <true/false>
fqdn: <fqdn>
hostname: <fqdn/hostname>
```

## 1.3. `disk_setup`
Cấu hình partions và filesystems.

Các aliases được sinh ra để mapping đối với disks, có 2 aliases tự động đó là swap xcho phân vùng swap và ephemeral cho block device của ephemeral image.

Các cấu hình partions được khai báo trong `disk_setup`. `table_type` chỉ định loại partition table (mbr hoặc gpt). `layout` cho biết partions được sắp xếp như thế nào. Nếu tùy chọn này được set bằng true, 1 partion được tạo ra chứa tất cả các lưu lượng trống. Ngược lại nếu được set bằng `false` thì sẽ không có bất kì partion nào được tạo ra.

Partions có thể được chỉ định bằng việc cung cấp danh sách trong phần `layout`, mỗi một entry sẽ là size hoặc cả size cả mã partion. Size của partion sẽ được chỉ định bằng phần trăm (ví dụ 33 nghĩa là partion này chiếm khoảng 1/3 không gian disk).

Tùy chọn `overwrite` kiểm soát phần ghi đè lên và có thể làm mất dữ liệu. Nếu nó được set thành false, device sẽ check xem nó nó partion table và file system chưa. Nếu một trong hai cái được phát hiện thì câu lệnh sẽ được skip.

Cấu hình filesystem được khai báo trong phần `fs_setup`. Các partion có thể được set auto, module sẽ tìm kiếm filesystem match với `label`, `type`, `device` của `fs_setup` entry và skip quá trình tạo filesystem. Nếu nó được set thành `any` thì chỉ cần type và `device` match là được. Để tạo filesystem trực tiếp, sử dụng `partition: none`.

**Distro hỗ trợ:** tất cả

**Config keys:**
```yaml
device_aliases:
    <alias name>: <device path>
disk_setup:
    <alias name/path>:
        table_type: <'mbr'/'gpt'>
        layout:
            - [33,82]
            - 66
        overwrite: <true/false>
fs_setup:
    - label: <label>
      filesystem: <filesystem type>
      device: <device>
      partition: <"auto"/"any"/"none"/<partition number>>
      overwrite: <true/false>
      replace_fs: <filesystem type>
```

## 1.4. `ssh`
Cấu hình ssh và ssh keys. Có rất nhiều image đã có sẵn keys, bạn có thể xóa nó bằng tùy chọn `ssh_deletekeys`.

**Các loại keys hỗ trợ:**
- rsa
- dsa
- ecdsa
- ed25519

Root login có thể được kích hoạt hoặc hủy bỏ thông qua tùy chọn `disable_root`.

**Distro hỗ trợ:** tất cả

**Config keys:**
```yaml
ssh_deletekeys: <true/false>
ssh_keys:
    rsa_private: |
        -----BEGIN RSA PRIVATE KEY-----
        MIIBxwIBAAJhAKD0YSHy73nUgysO13XsJmd4fHiFyQ+00R7VVu2iV9Qco
        ...
        -----END RSA PRIVATE KEY-----
    rsa_public: ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAGEAoPRhIfLvedSDKw7Xd ...
    dsa_private: |
        -----BEGIN DSA PRIVATE KEY-----
        MIIBxwIBAAJhAKD0YSHy73nUgysO13XsJmd4fHiFyQ+00R7VVu2iV9Qco
        ...
        -----END DSA PRIVATE KEY-----
    dsa_public: ssh-dsa AAAAB3NzaC1yc2EAAAABIwAAAGEAoPRhIfLvedSDKw7Xd ...
ssh_genkeytypes: <key type>
disable_root: <true/false>
disable_root_opts: <disable root options string>
ssh_authorized_keys:
    - ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAGEA3FSyQwBI6Z+nCSjUU ...
    - ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA3I7VUf2l5gSn5uavROsc5HRDpZ ...
```

## 1.5. `users_groups`
Cấu hình user và group

Các entry hỗ trợ với `users`:
- `name` : Tên user login. (bắt buộc)
- `expiredate` : Ngày user không thể đăng nhập
- `gecos` : Comment về user, thường để lưu thông tin liên lạc
- `groups` : Groups để add users vào
- `homedir` : thư mục cho user, mặc định là /home/<username>
- `inactive` : Đánh dấu user chưa active
- `lock_passwd` : không cho phép login bằng password
- `no-create-home` : không tạo thư mục home
- `no-log-init` : không tạo lastlog và faillog cho user
- `no-user-group` : không tạo group cho user
- `passwd` : password cho user (bắt buộc)
- `ssh-authorized-keys` : danh sách các key để add vào user’s authkeys file
- `sudo` : Các rule để sử dụng sudo
- `system` : tạo system user không có thư mục home

**Distro hỗ trợ:** Tất cả

**Config keys:**
```yaml
groups:
    - <group>: [<user>, <user>]
    - <group>

users:
    - default
    - name: <username>
      expiredate: <date>
      gecos: <comment>
      groups: <additional groups>
      homedir: <home directory>
      inactive: <true/false>
      lock_passwd: <true/false>
      no-create-home: <true/false>
      no-log-init: <true/false>
      no-user-group: <true/false>
      passwd: <password>
      primary-group: <primary group>
      selinux-user: <selinux username>
      shell: <shell path>
      snapuser: <email>
      ssh-authorized-keys:
          - <key>
          - <key>
      ssh-import-id: <id>
      sudo: <sudo config>
      system: <true/false>
      uid: <user id>
```


# 2. cloud_config_modules
## 2.1. `set_password`
Cấu hình passwords và kích hoạt hoặc hủy ssh bằng password.

**Distro hỗ trợ:** tất cả

**Config keys:**
```yaml
ssh_pwauth: <yes/no/unchanged>

password: password1
chpasswd:
    expire: <true/false>

chpasswd:
    list: |
        user1:password1
        user2:RANDOM
        user3:password3
        user4:R

##
# or as yaml list
##
chpasswd:
    list:
        - user1:password1
        - user2:RANDOM
        - user3:password3
        - user4:R
        - user4:$6$rL..$ej...
```

Ví dụ: Truyền password cho user default
```yaml
#cloud-config
password: hai1996
chpasswd: {expire: False}
ssh_pwauth: True
```

## 2.2. `ntp`
Kích hoạt và cấu hình NTP

Nếu ntp chưa được cài đặt nhưng đã được cấu hình, nó sẽ được cài đặt sau. Nếu mặc định đã có file cấu hình ntp trên image hoặc trong distro, nó sẽ được copy tới `/etc/ntp.conf.dist` trước khi được thay đổi. Nếu không có pool hoặc server nào, 4 pools sẽ được sử dụng đó là `{0-3}.{distro}.pool.ntp.org`

**Distro hỗ trợ:** centos, debian, fedora, opensuse, ubuntu

**Config example:**
```yaml
ntp:
  pools:
  - 0.company.pool.ntp.org
  - 1.company.pool.ntp.org
  - ntp.myorg.org
  servers:
  - my.ntp.server.local
  - ntp.ubuntu.com
  - 192.168.23.2
```

## 2.3. `resolv_conf`
Cấu hình file `resolv.conf` nếu nó là cần thiết cho những tiến trình tiếp theo trong quá trình boot. 

**Lưu ý:** trên các distro Ubuntu/debian, người ta khuyến khích sử dụng file `/etc/network/interfaces`.

**Distro hỗ trợ:** fedora, rhel, sles

**Config keys:**
```yaml
manage_resolv_conf: <true/false>
resolv_conf:
    nameservers: ['8.8.4.4', '8.8.8.8']
    searchdomains:
        - foo.example.com
        - bar.example.com
    domain: example.com
    options:
        rotate: <true/false>
        timeout: 1
```

## 2.4. `runcmd`
Chạy các câu lệnh tùy ý. Tuy nhiên, các câu lệnh phải theo format của `yaml`

**Distro hỗ trợ:** Tất cả

**Ví dụ:**
```yaml
runcmd:
    - [ ls, -l, / ]
    - [ sh, -xc, "echo $(date) ': hello world!'" ]
    - [ sh, -c, echo "=========hello world'=========" ]
    - ls -l /root
    - [ wget, "http://example.org", -O, /tmp/index.html ]
```

## 2.5. `write_files`
Thêm nội dung vào file và có thể tùy ý thiết lập quyền.

Thư mục cha sẽ được tạo nếu chưa tồn tại.

**Các key trong `write_files`:**
- `path`: (string) Đường dẫn của tệp đến nội dung được giải mã và ghi

- `content`: (string) Option tùy chọn để ghi vào đường dẫn được cung cấp. Khi nội dung có và mã hóa không phải là text, rõ ràng. Hãy giải mã (decode) nội dung trước khi viết.. Mặc định: `''`

- `owner`: (string) owner:group. Default: `root:root`

- `permissions`: (string) quyền truy cập. Mặc định: `0644`

- `encoding`: (string) Tùy chọn kiểu mã hóa nội dung. Mặc định: `text/plain` và không có giải mã. Các loại mã hóa được hỗ trợ là: gz, gzip, gz+base64, gzip+base64, gz+b64, gzip+b64, b64, base64.

- `append`: (boolean) Có hay không nối nội dung vào tệp đã tồn tại. Mặc định: `false`

**Distro hỗ trợ:** tất cả

**Ví dụ:**
```yaml
write_files:
    - encoding: b64
      content: CiMgVGhpcyBmaWxlIGNvbnRyb2xzIHRoZSBzdGF0ZSBvZiBTRUxpbnV4...
      owner: root:root
      path: /etc/sysconfig/selinux
      permissions: '0644'
    - content: |
        # My new /etc/sysconfig/samba file

        SMDBOPTIONS="-D"
      path: /etc/sysconfig/samba
    - content: !!binary |
        f0VMRgIBAQAAAAAAAAAAAAIAPgABAAAAwARAAAAAAABAAAAAAAAAAJAVAAAAAA
        AEAAHgAdAAYAAAAFAAAAQAAAAAAAAABAAEAAAAAAAEAAQAAAAAAAwAEAAAAAAA
        AAAAAAAAAwAAAAQAAAAAAgAAAAAAAAACQAAAAAAAAAJAAAAAAAAcAAAAAAAAAB
        ...
      path: /bin/arch
      permissions: '0555'
```

## 2.6. `yum_repos`
Thêm cấu hình yum repository vào `/etc/yum.repos.d`

**Distro hỗ trợ:** fedora, rhel

**Config keys:**
```yaml
yum_repos:
    <repo-name>:
        baseurl: <repo url>
        name: <repo name>
        enabled: <true/false>
        # any repository configuration options (see man yum.conf)
```

## 2.7. `power_state`
Module này cho phép bạn shutdown/reboot máy ảo sau khi các module khác đã hoàn tất.

**Distro hỗ trợ:** tất cả

**Config keys:**
```yaml
power_state:
    delay: <now/'+minutes'>
    mode: <poweroff/halt/reboot>
    message: <shutdown message>
    timeout: <seconds>
    condition: <true/false/command>
```

# 3. `cloud_final_modules`
## 3.1. `package-update-upgrade-install`
CẬp nhật, nâng cấp và cài đặt các gói trong khi boot. Nếu có package nào được cài đặt hoặc upgrade thì package cache sẽ được update trước. Nếu yêu cầu reboot thì máy sẽ reboot khi phát hiện tùy chọn `package_reboot_if_required`.

**Distro hỗ trợ:** Tất cả

**Config keys:**
```yaml
packages:
    - pwgen
    - pastebinit
    - [libpython2.7, 2.7.3-0ubuntu3.1]
package_update: <true/false>
package_upgrade: <true/false>
package_reboot_if_required: <true/false>

apt_update: (alias for package_update)
apt_upgrade: (alias for package_upgrade)
apt_reboot_if_required: (alias for package_reboot_if_required)
```





# Tham khảo:

https://cloudinit.readthedocs.io/en/latest/topics/modules.html#
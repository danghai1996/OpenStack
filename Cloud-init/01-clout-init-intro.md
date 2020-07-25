# Giới thiệu về Cloud-init

## 1. Cloud-init
Cloud-init là một công cụ được sử dụng để thực hiện các thiếp lập ban đầu đối với các máy ảo hóa và cloud. Dịch vụ này sẽ chạy trước quá trình boot, nó lấy dữ liệu từ bên ngoài và thực hiện một số tác động tới máy chủ.

Các tác động mà cloud-init thực hiện phụ thuộc vào loại format thông tin mà nó tìm kiếm được. Các format hỗ trợ:

- Shell scripts (bắt đầu với #!)
- Cloud config files (bắt đầu với #cloud-config)
- MIME multipart archive.
- Gzip Compressed Content
- Cloud Boothook

Một trong những định dạng thông dụng nhất dành cho các scripts đó là `cloud-config`.

`cloud-config` là các file script được thiết kế để chạy trong các tiến trình cloud-init. Nó được sử dụng cho các cài đặt cấu hình ban đầu trên server như networking, SSH keys, timezone, user data injection...

**File config của cloud-init**

File cấu hình của cloud-init `/etc/cloud/cloud.cfg` mặc định chứa 3 module là Cloud_init_modules, Cloud_config_modules, Cloud_final_module

Ở trong 3 modules này chứa Jobs mặc định của Cloud- init, ta có thể thay đổi các Jobs này, định nghĩa ra các Jobs mới. Bạn cần phải viết thêm một file code python thực hiện các thông số đầu vào mà người dùng đặt và đặt nó trong thư mục chứa datasource. Các đầu mục trong /etc/cloud/cloud.cfg sẽ được map với code python.

**Cấu trúc thư mục của cloud-init**

Thông thường thư mục chính của cloud-init được đặt tại `/var/lib/cloud`. Cloud-init sẽ lấy thông tin từ datasource và lưu tại đây.

Datasource là các nguồn dữ liệu cấu hình cho cloud-init thường được lấy từ user (userdata) hoặc tới từ stack tạo ra configuration drive (metadata). userdata thường chứa files, yaml, và shell scripts. Trong khi đó, metadata lại chứa server name, instance id, display name và một số thông tin khác.

Đối với OpenStack, url để lấy metadata của nó thường là `http://169.254.169.254`

## 2. Chức năng của cloud-init
- Đặt nơi chứa máy ảo
- Thiết lập hostname
- Generate SSH private keys
- Chèn SSH key vào file .ssh/authorized_keys để user đó có thể đăng nhập
- Cấu hình các mount points
- Cấu hình các network devices

Hiện tại cloud-init đang được sử dụng rất rộng rãi trên hầu hết các distro của linux và các công nghệ cloud như OpenStack, Amazon EC2, RHEV, và VMware.

## 3. Cloud-config
Định dạng cloud-config thực thi các câu lệnh đã được khai báo cho rất nhiều các configuration items phổ biến, nó khiến việc thực hiện các task này trở nên dễ dàng hơn.

cloud-config sử dụng YAML data serialization format. Định dạng này được thiết kế để người sử dụng dễ hiểu và đơn giản hóa cho việc áp dụng vào các programs.

**Một số rules khi viết file định dạng YAML:**

- Các khoảng trống (space) cho biết cấu trúc và mối quan hệ giữa các items. Ví dụ các items là sub-items của item đầu tiên khi nó được viết thụt vào trong lề.
- Danh sách được xác định bằng các dấu gạch đầu dòng
- Các entries của mảng liên kết với nhau thể hiện qua dấu hai chấm (:), theo sau là khoảng trống (space) và các giá trị.
- Các khối văn bản với nhau được thụt vào lề dòng.

**Ví dụ:** 

```yaml
#cloud-config
password: hai1996
chpasswd: {expire: False}
ssh_pwauth: True
```

## 4. cloud-init sẽ thực hiện theo 5 giai đoạn
1. Generator (cloud-config.target): Đọc file cấu hình `cloud.cfg`. Mặc định sẽ kích hoạt dịch vụ cloud init.

2. Local (cloud-init-local.service): Xác định nguồn dữ liệu "local" và cấu hình network

3. Network (cloud-init.service): Đọc cấu hình đã chỉ định của module `cloud_init_modules`. Nó sẽ giải nén các module, get các dữ liệu về. Giai đoạn này sẽ chạy các module như bootcmd.

4. Config (cloud-config.service): Đọc cấu hình đã chỉ định của module `cloud_config_modules`. Các module không thể chạy trong giai đoạn khởi động.

5. Final (cloud-final.service) :Đọc cấu hình đã chỉ định của module cloud_final_modules. Thực hiện các tệp lệnh cơ bản người dùng. Như package installations, configuration management plugins (puppet, chef, alt-minion), user-scripts (including runcmd).

**Ví dụ:** image Ubuntu 20.04
```yaml
cloud_init_modules:
 - migrator
 - seed_random
 - bootcmd
 - write-files
 - growpart
 - resizefs
 - disk_setup
 - mounts
 - set_hostname
 - update_hostname
 - update_etc_hosts
 - ca-certs
 - rsyslog
 - users-groups
 - ssh

cloud_config_modules:
 - emit_upstart
 - snap
 - ssh-import-id
 - locale
 - set-passwords
 - grub-dpkg
 - apt-pipelining
 - apt-configure
 - ubuntu-advantage
 - ntp
 - timezone
 - disable-ec2-metadata
 - runcmd
 - byobu

cloud_final_modules:
 - package-update-upgrade-install
 - fan
 - landscape
 - lxd
 - ubuntu-drivers
 - puppet
 - chef
 - mcollective
 - salt-minion
 - rightscale_userdata
 - scripts-vendor
 - scripts-per-once
 - scripts-per-boot
 - scripts-per-instance
 - scripts-user
 - ssh-authkey-fingerprints
 - keys-to-console
 - phone-home
 - final-message
 - power-state-change
```
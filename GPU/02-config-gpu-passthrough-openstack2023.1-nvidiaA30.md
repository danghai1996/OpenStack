# Cấu hình GPU Passthrough trên OpenStack

💡 Đây chỉ là tài liệu tham khảo. Các thông số kỹ thuật hay lỗi có thể phát sinh.

## Tổng quan môi trường
| Thành phần | Thông tin |
|---|---|
| OpenStack | 2023.1 |
| Deployment | Kolla-Ansible |
| Compute OS | Ubuntu 22.04 |
| GPU | NVIDIA A30 |
| GPU `vendor_id` | `10de` |
| GPU `product_id` | `20b7` |
| GPU Driver | NVIDIA vGPU Driver 535.104.06 |

## 1. Cài đặt các gói hỗ trợ và driver
> Thực hiện trên node computeGPU

Update, Upgrade os:
```python
apt upgrade -y
apt update -y
```

Cài đặt các gói cần thiết:

```python
apt-get install linux-source -y
apt-get install linux-headers-generic -y
update-pciids
```

Enable IOMMU trên Host `/etc/default/grub`

- Nếu chip của Host là của Intel thì đổi dòng sau:
    ```python
    GRUB_CMDLINE_LINUX="intel_iommu=on"
    ```

- Nếu chip của Host là AMD thì đổi thành:
    ```python
    GRUB_CMDLINE_LINUX="amd_iommu=on"
    ```

Rebuild lại file grub:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

Cấu hình disable nouveau

```python
cat <<EOF | sudo tee /etc/modprobe.d/blacklist-nouveau.conf
blacklist nouveau
options nouveau modeset=0
EOF
```

Khởi tạo lại kernel initramfs:

```python
sudo update-initramfs -u
```

Khởi động lại máy chủ

```python
init 6
```

Kiểm tra KVM đã lấy được SRIOV chưa. Thực hiện exec vào nova_libvirt thực hiện lệnh:

```bash
virt-host-validate
```

Kết quả PASS là thành công:

```jsx
QEMU: Checking if IOMMU is enabled by kernel                               : PASS
```

Cài đặt GCC

```python
apt install build-essential -y
```

Download Driver (Loại Ubuntu KVM) → bản mới nhất. Sau đó thực hiện cài driver cho Host

```
https://ui.licensing.nvidia.com/software
```

Truy cập thư mục chứa driver cho Host:

```python
cd /root/GPU_dirver/Host_Drivers/

apt install ./nvidia-vgpu-ubuntu-*_amd64.deb -y
```

Reboot máy chủ:

```python
init 6
```

Kiểm tra các card GPU:

```python
nvidia-smi
```
```
root@compute-gpu:~#:~# nvidia-smi
+---------------------------------------------------------------------------------------+
| NVIDIA-SMI 535.104.06             Driver Version: 535.104.06   CUDA Version: N/A      |
|-----------------------------------------+----------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
|                                         |                      |               MIG M. |
|=========================================+======================+======================|
|   0  NVIDIA A30                     On  | 00000000:31:00.0 Off |                    0 |
| N/A   33C    P0              32W / 165W |      0MiB / 24576MiB |      0%      Default |
|                                         |                      |             Disabled |
+-----------------------------------------+----------------------+----------------------+
|   1  NVIDIA A30                     On  | 00000000:4B:00.0 Off |                    0 |
| N/A   39C    P0              32W / 165W |      0MiB / 24576MiB |      0%      Default |
|                                         |                      |             Disabled |
+-----------------------------------------+----------------------+----------------------+
|   2  NVIDIA A30                     On  | 00000000:98:00.0 Off |                    0 |
| N/A   36C    P0              33W / 165W |      0MiB / 24576MiB |      0%      Default |
|                                         |                      |             Disabled |
+-----------------------------------------+----------------------+----------------------+
|   3  NVIDIA A30                     On  | 00000000:CA:00.0 Off |                    0 |
| N/A   32C    P0              31W / 165W |      0MiB / 24576MiB |      0%      Default |
|                                         |                      |             Disabled |
+-----------------------------------------+----------------------+----------------------+

+---------------------------------------------------------------------------------------+
| Processes:                                                                            |
|  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
|        ID   ID                                                             Usage      |
|=======================================================================================|
|  No running processes found                                                           |
+---------------------------------------------------------------------------------------+
```

## 2. Cấu hình Compute GPU Passthrough
> Thực hiện trên node computeGPU

Check vendor and product ID của GPU:

```bash
lspci -nn | grep -i nvidia
```
```
root@compute-gpu:~# lspci -nn | grep -i nvidia
31:00.0 3D controller [0302]: NVIDIA Corporation GA100GL [A30 PCIe] [10de:20b7] (rev a1)
4b:00.0 3D controller [0302]: NVIDIA Corporation GA100GL [A30 PCIe] [10de:20b7] (rev a1)
98:00.0 3D controller [0302]: NVIDIA Corporation GA100GL [A30 PCIe] [10de:20b7] (rev a1)
ca:00.0 3D controller [0302]: NVIDIA Corporation GA100GL [A30 PCIe] [10de:20b7] (rev a1)
```

Cấu hình vendor/productID cho driver:

```bash
root@compute-gpu:~# echo options vfio-pci ids=10de:20b7 > /etc/modprobe.d/vfio.conf
```

Kiểm tra PCI lấy qua SR-IOV:

```bash
ls -ld /sys/kernel/iommu_groups/*/devices/*31:00.?/
=> drwxr-xr-x 7 root root 0 Mar  6 22:04 /sys/kernel/iommu_groups/34/devices/0000:31:00.0/

ls -ld /sys/kernel/iommu_groups/*/devices/*4b:00.?/
drwxr-xr-x 7 root root 0 Mar  6 22:04 /sys/kernel/iommu_groups/40/devices/0000:4b:00.0/

ls -ld /sys/kernel/iommu_groups/*/devices/*98:00.?/
drwxr-xr-x 7 root root 0 Mar  6 22:04 /sys/kernel/iommu_groups/162/devices/0000:98:00.0/

ls -ld /sys/kernel/iommu_groups/*/devices/*ca:00.?/
drwxr-xr-x 7 root root 0 Mar  6 22:04 /sys/kernel/iommu_groups/175/devices/0000:ca:00.0/
```

Ta sẽ lấy ra được PCI address của 4 card:
```bash
0000:31:00.0
0000:4b:00.0
0000:98:00.0
0000:ca:00.0
```

## 3. Cấu hình OpenStack Nova
### 3.1. Cấu hình trên node Compute GPU
> Thực hiện trên node computeGPU

Cấu hình PCI trong nova-compute `/etc/kolla/nova-compute/nova.conf`  

```bash
[pci]
device_spec = { "address": "0000:31:00.0" }
device_spec = { "address": "0000:4b:00.0" }
device_spec = { "address": "0000:98:00.0" }
device_spec = { "address": "0000:ca:00.0" }
alias = { "vendor_id":"10de", "product_id":"20b7", "device_type":"type-PF", "name":"a30" }
```

Trong đó:
- `vendor_id` : lấy kết quả từ lệnh `lspci -nn | grep -i nvidia` ở bước trên
- `product_id`: lấy kết quả từ lệnh `lspci -nn | grep -i nvidia` ở bước trên
- `device_type` : chọn `type-PF`
- `name` : đặt tên cho pci. Ở đây GPU A30 sẽ để `a30`

Khởi động lại nova-compute service:

```bash
docker restart nova_compute
```

### 3.2. Cấu hình trên các node Controller
> Thực hiện trên node controller

Thêm cấu hình `filter_schedule` và pci vào cấu hình của nova-api: `/etc/kolla/nova-api/nova.conf`

```bash
[filter_scheduler]
available_filters = nova.scheduler.filters.all_filters
enabled_filters = AggregateImagePropertiesIsolation,AvailabilityZoneFilter,ComputeFilter,ComputeCapabilitiesFilter,ImagePropertiesFilter,ServerGroupAntiAffinityFilter,ServerGroupAffinityFilter,AggregateInstanceExtraSpecsFilter
[pci]
alias = { "vendor_id":"10de", "product_id":"20b7", "device_type":"type-PF", "name":"a30", "numa_policy":"preferred" }
```

Thêm cấu hình `filter_scheduler` vào `/etc/kolla/nova-scheduler/nova.conf`

```bash
[filter_scheduler]
available_filters = nova.scheduler.filters.all_filters
enabled_filters = AggregateImagePropertiesIsolation,ComputeFilter,ComputeCapabilitiesFilter,ImagePropertiesFilter,ServerGroupAntiAffinityFilter,ServerGroupAffinityFilter,AggregateInstanceExtraSpecsFilter
```

Khởi động lại các service nova:

```bash
docker restart nova_novncproxy
docker restart nova_conductor
docker restart nova_api
docker restart nova_scheduler
```

### 3.3. Khởi tạo flavor GPU passthrough
> Thực hiện trên node controller
Tạo flavor theo các thông tin như sau:

```bash
openstack flavor create --vcpus 16 --ram 65536 --property "pci_passthrough:alias"="a30:1" 1GPU-A30.16C64R
```

**Lưu ý:** 
- `"pci_passthrough:alias"="a30:1" ` sẽ chuyển 1 GPU A30 tới VM
- `"pci_passthrough:alias"="a30:2" ` sẽ chuyển 2 GPU A30 tới VM
- `"pci_passthrough:alias"="a30:3" ` sẽ chuyển 3 GPU A30 tới VM
- `"pci_passthrough:alias"="a30:4" ` sẽ chuyển 4 GPU A30 tới VM

Tạo máy ảo từ flavor vừa tạo ở trên.

- Kiểm tra trên máy ảo ta sẽ thấy có GPU được passthrough vào VM:
    ```
    lspci -nn | grep -i nvidia
    00:06.0 3D controller [0302]: NVIDIA Corporation GA100GL [A30 PCIe] [10de:20b7] (rev a1)
    ```
    
- Kiểm tra trên host computeGPU sẽ thấy thiếu 1 GPU đã passthrough vào máy ảo:
    ```bash
    nvidia-smi
    ```
    ```bash
    root@compute-gpu:~# nvidia-smi
    +---------------------------------------------------------------------------------------+
    | NVIDIA-SMI 535.104.06             Driver Version: 535.104.06   CUDA Version: N/A      |
    |-----------------------------------------+----------------------+----------------------+
    | GPU  Name                 Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
    |                                         |                      |               MIG M. |
    |=========================================+======================+======================|
    |   0  NVIDIA A30                     On  | 00000000:31:00.0 Off |                    0 |
    | N/A   33C    P0              32W / 165W |      0MiB / 24576MiB |      0%      Default |
    |                                         |                      |             Disabled |
    +-----------------------------------------+----------------------+----------------------+
    |   1  NVIDIA A30                     On  | 00000000:4B:00.0 Off |                    0 |
    | N/A   39C    P0              32W / 165W |      0MiB / 24576MiB |      0%      Default |
    |                                         |                      |             Disabled |
    +-----------------------------------------+----------------------+----------------------+
    |   2  NVIDIA A30                     On  | 00000000:98:00.0 Off |                    0 |
    | N/A   36C    P0              33W / 165W |      0MiB / 24576MiB |      0%      Default |
    |                                         |                      |             Disabled |
    +-----------------------------------------+----------------------+----------------------+
    +---------------------------------------------------------------------------------------+
    | Processes:                                                                            |
    |  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
    |        ID   ID                                                             Usage      |
    |=======================================================================================|
    |  No running processes found                                                           |
    +---------------------------------------------------------------------------------------+
    ```
    ⇒ Trên host chỉ còn hiển thị 3/4 GPU

Khi hết GPU trên host computeGPU, thì kết quả lệnh `nvidia-smi` trên host sẽ thông báo như dưới đây:
```
root@compute-gpu:~# nvidia-smi
Failed to initialize NVML: No supported GPUs were found
Unable to determine the number of GPUs
```

## Tham khảo
- https://docs.openstack.org/nova/latest/admin/pci-passthrough.html
- https://github.com/vinhducnguyen1708/Tim-hieu-OpenStack/tree/master/docs/GPU-Openstack
# Nova-Scheduler

# 1. Host aggreate vs Availability zone

<img src="..\images\Screenshot_94.png">

## 1.1. Host aggregate
Là một cơ chế tạo ra một nhóm logic để phân vùng `availability zone`, `host aggregate` trong OPS tập hợp các Compute Node được chỉ định và liên kết với metadata. Trong khi các `availability zones` được hiển thị cho các users thì `host aggregate` chỉ hiển thị cho Administrator.

Host aggregates bắt đầu như cách sử dụng Xen hypervisor resource pools, nhưng đã được khái quát hóa để cung cấp cơ chế cho phép admin chỉ định key-value pairs cho các group. Mỗi node có thể có nhiều aggregates, mỗi aggregates có thể có nhiều key-value pairs, và các key-value giống nhau có thể chỉ định cho nhiều aggregate. Thông tin này có thể đươc sử dụng trong scheduler để cho phép scheduling nâng cao, để thiết lập các xen hypervisor resources pools hoặc định nghĩa các logical groups cho migration.

Các metadata trong các host aggregate thường được dùng để cung cấp thông tin cho quá trình nova-scheduler để xác định được được host đặt các mảy ảo . Metadata quy định trong một host aggretate sẽ chỉ định host chạy các instance mà có falvor cùng metadata

Người quản trị sử dụng host aggregate để xử lý cân bằng tải, dự phòng, resource pool, nhóm các server cùng thuộc tính. Các host aggregate sẽ không được public ra cho các end-user mà thay đó sẽ được gắn vào các flavor.

Ví dụ về Host aggregate : có thể tạo một tâp hợp các compute node tùy vào vị trí địa lý : "DC FPT HCM", hoặc các host trên rack 1 sử dụng disk SSD RACK 1 SSD

## 1.2. Availability Zones (AZs)
**Availability Zones** là visible logical abstraction của end-user cho việc phân vùng một đám mây mà không biết về kiến trúc hạ tầng vật lý của nó.

Availability zone là một metadata cụ thể được gắn vào một host aggregate. Việc thêm một metadata vào một host aggregate sẽ khiến cho các aggregate này bị nhìn thấy bởi các end-user, do đó cần nhờ nova-schedule làm việc với một host aggregate cụ thể

Availability zone cho phép các các end-user chọn một host aggregate để chạy máy ảo. Ví dụ: sử dụng availability zone , người dùng có thể khởi tạo một máy ảo chạy trên DC FPT ở HCM


## Giữa Aggregates and Availability zones:
- Một host có thể nằm trong nhiều aggregates, nhưng chỉ có thể thuộc về một availability zone
- Mặc định thì một host sẽ là thành viên của default availability zone ngay cả khi nó không thuộc aggregate nào (tùy chọn cấu hình là `default_availability_zone`)

## 1.3. Cấu hình scheduler để hỗ trợ host aggreate
Một trường hợp sử dụng chung cho host aggregates là khi bạn muốn hỗ trợ scheduling instances để một tập gồm một số các compute hosts có cùng khả năng cụ thể. Ví dụ, bạn có thể muốn cho phép người dùng request compute host có SSD driver nếu họ cần tốc độ disk I/O nhanh hơn, hoặc cho phép compute host có GPU cards.

Khi nhận được yêu cầu từ người dùng, nova-scheduler sẽ filter những host phù hợp để launch máy ảo, những host không phù hợp sẽ bị loại. Sau đó nó dùng tiếp weighting để xác định xem đâu là host phù hợp nhất. 

Để configure scheduler hỗ trợ host aggregates, thì `scheduler_default_filters` phải chứa `AggregateInstanceExtraSpecsFilter` ngoài các filters khác được sử dụng bởi scheduler. Thêm các dòng dưới đây vào `/etc/nova/nova.conf` trên host chạy nova-scheduler service để cho phép host aggregates filtering, cũng như các filter khác:
```
[filter_scheduler]
enabled_filters=AvailabilityZoneFilter,ComputeFilter,ComputeCapabilitiesFilter,ImagePropertiesFilter,ServerGroupAntiAffinityFilter,ServerGroupAffinityFilter,AggregateInstanceExtraSpecsFilter
```

# 2. Nova-scheduler
Nova-scheduler service xác định compute node nào sẽ thực hiện chạy instance.

- Scheduler sử dụng `filter scheduler` để lập kế hoạch cho việc khởi tạo máy ảo. Trong filter scheduler hỗ trợ 2 cơ chế filtering và weighting để có thể xác định được node compute chạy máy ảo.
- Khi filter scheduler nhận được một request sẽ thực hiện `filtering` các host compute, loại bỏ các host không phù hợp, sau đó sẽ thực hiện `weighting`, bằng cách dùng các thuật toán tính toán sẽ chọn ra được node compute phù hợp nhất (weight cao nhất) để tạo máy ảo.

## 2.1. Filtering
<img src="..\images\Screenshot_81.png">

Trong quá trính làm việc, Filter Scheduler lặp đi lặp lại trên nodes Compute được tìm thấy, mỗi lần lặp sẽ đánh giá lại các host, tìm ra danh sách kết quả các node đủ điều kiện, sau đó sẽ được sắp xếp theo thứ tự bởi `weighting`. Scheduler sẽ dựa vào đó đê chọn một host có weight cao nhất để lauch instance.

Nếu Scheduler không thể tìm thấy host phù hợp cho instance, nó có nghĩa là không có hosts thích hợp cho việc tạo instance.

Filter scheduler khá linh hoạt, hỗ trợ nhiều cách `filtering` và `weighting` cần thiết. Nếu bạn vẫn chưa thấy linh hoạt thì bạn có thể tự định nghĩa một giải thuật filtering cho chính mình.

### Có nhiều chuẩn filter classes có thể sử dụng (nova.scheduler.filters)**
- `AllHostsFilter` : không lọc. Chấp nhận tất cả các available host.
- `ImagePropertiesFilter` : Lọc dựa trên các thuộc tính (properties) xác định trên image. Chấp nhận các host có thể hỗ trợ các thuộc tính đó
- `AvailabilityZoneFilter` : Lọc theo availability zone. Chấp nhận các host phù hợp với availability zone được chỉ định. Sử dụng dấu `,` để chỉ định nhiều zone.
- `ComputeCapabilitiesFilter` : kiểm tra host compute service có thể đáp ứng bất kỳ thông số kỹ thuật nào liên quan đến type instance hay không. Nếu thỏa mãn thì host compute được chấp nhận

Nếu một extra specs key bao gồm dấu hai chấm (:) thì trước dấu (:) được coi là một namespace, sau đó sẽ là một khóa phù hợp. Ngược lại, nếu một key không bao gồm dấu (:) thì nội dung của key đó rất quan trọng. Nếu key này là một thuộc tính thể hiện trạng thái của host như `free_disk_mb`, thì filter sẽ coi extra specs key như một key phù hợp (matched). Nếu không filter sẽ bỏ qua key này

Một vài các thuộc tính được sử dụng như useful key và giá trị của chúng gồm:
```
* free_ram_mb (compared with a number, values like ">= 4096")
* free_disk_mb (compared with a number, values like ">= 10240")
* host (compared with a string, values like: "<in> compute","s== compute_01")
* hypervisor_type (compared with a string, values like: "s== QEMU", "s== powervm")
* hypervisor_version (compared with a number, values like : ">= 1005003", "== 2000000")
* num_instances (compared with a number, values like: "<= 10")
* num_io_ops (compared with a number, values like: "<= 5")
* vcpus_total (compared with a number, values like: "= 48", ">=24")
* vcpus_used (compared with a number, values like: "= 0", "<= 10")
```

- `AggregateInstanceExtraSpecsFilter` : Kiểm tra xem metadata có thỏa mãn các thông số kỹ thuật bổ sung liên quan đến type instance. Nếu thỏa mãn, host đó có thể tạo type instance được chỉ định. Các thông số kỹ thuật bổ sung có thể đi cùng là `ComputeCapabilitiesFilter`. Để chỉ định nhiều giá trị cho cùng 1 key, ta có thể sử dụng dấu phẩy (`,`)
- `ComputeFilter` : Lọc ra những host compute đang hoạt động

[Xem thêm các tùy chọn](https://docs.openstack.org/nova/latest/user/filter-scheduler.html)

### Cấu hình Filtering trong `/etc/nova/nova.conf`
Để sử dụng filter bạn sẽ có hai setting cụ thể:
- `filter_scheduler.available_filters` : Xác định các lớp bộ lọc có sẵn cho scheduler. Thiết lập này có thể được sử dụng nhiều lần
- `filter_scheduler.enabled_filters` : Trong các bộ lọc có sẵn, xác định những bộ lọc mà scheduler sử dụng trong mặc định.

Các giá trị mặc định trong `nova.conf` là:
```
--filter_scheduler.available_filters=nova.scheduler.filters.all_filters
--filter_scheduler.enabled_filters=ComputeFilter,AvailabilityZoneFilter,ComputeCapabilitiesFilter,ImagePropertiesFilter,ServerGroupAntiAffinityFilter,ServerGroupAffinityFilter
```
Với cấu hình như trên thì tất cả các filter trong `nova.scheduler.filters` đều sẵn sàng và mặc định sẽ là `ComputeFilter`, `AvailabilityZoneFilter`, `ComputeCapabilitiesFilter`, `ImagePropertiesFilter`, `ServerGroupAntiAffinityFilter` và `ServerGroupAffinityFilter` sẽ được sử dụng

[Xem thêm cấu hình Filtering](https://docs.openstack.org/nova/train/admin/configuration/schedulers.html)

## 2.2. Weights
Là cách chọn máy chủ phù hợp nhất từ một nhóm các máy chủ hợp lệ bằng cách tính toán và đưa ra trọng số (weights) cho tất cả các máy chủ trong danh sách.

Để ưu tiên 1 weigher so với với weigher khác, tất cả các weigher cần phải xác định multiplier sẽ được áp dụng trước khi tính toán weight cho node. Tất cả weights được chuẩn hóa trước khi multiplier có thể được áp dụng. Do đó, weight cuối dùng của object sẽ là:
```
weight = w1_multiplier * norm(w1) + w2_multiplier * norm(w2) + ...
```

Tất cả các trọng số được chuẩn hóa trước khi được tổng hợp. Host có trọng số lớn nhất được ưu tiên cao nhất.

<img src="..\images\Screenshot_82.png">

<img src="..\images\Screenshot_83.png">

[Xem thêm về Weight](https://docs.openstack.org/nova/train/admin/configuration/schedulers.html#weights)
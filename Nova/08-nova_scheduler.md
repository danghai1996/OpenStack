# Nova-Scheduler

## 1. Host aggregate
Là một cơ chế tạo ra một nhóm logic để phân vùng `availability zone`, `host aggregate` trong OPS tập hợp các Compute Node được chỉ định và liên kết với metadata. Trong khi các `availability zones` được hiển thị cho các users thì `host aggregate` chỉ hiển thị cho Administrator.

Host aggregates bắt đầu như cách sử dụng Xen hypervisor resource pools, nhưng đã được khái quát hóa để cung cấp cơ chế cho phép admin chỉ định key-value pairs cho các group. Mỗi node có thể có nhiều aggregates, mỗi aggregates có thể có nhiều key-value pairs, và các key-calue giống nhau có thể chỉ định cho nhiều aggregate. Thông tin này có thể đươc sử dụng trong scheduler để cho phép scheduling nâng cao, để thiết lập các xen hypervisor resources pools hoặc định nghĩa các logical groups cho migration.

Các metadata trong các host aggregate thường được dùng để cung cấp thông tin cho quá trình nova-scheduler để xác định được được host đặt các mảy ảo . Metadata quy định trong một host aggretate sẽ chỉ định host chạy các instance mà có falvor cùng metadata

Người quản trị sử dụng host aggregate để xử lý cân bằng tải, dự phòng, resource pool ,nhóm các server cùng thuộc tính. Các host aggregate sẽ không được public ra cho các end-user mà thay đó sẽ được gắn vào các flavor.

Ví dụ về Host aggregate : có thể tạo một tâp hợp các compute node tùy vào vị trí địa lý : "DC FPT HCM", hoặc các host trên rack 1 sử dụng disk SSD RACK 1 SSD
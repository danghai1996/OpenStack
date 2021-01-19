# Nova Placement và Nova Conductor

## 1. Nova placement
Dịch vụ API placement được giới thiệu trong bản phát hành Newton 14.0.0 trong kho lưu trữ nova và được trích xuất vào kho lưu trữ vị trí trong bản phát hành Stein 19.0.0.

Gồm có một REST API và data model sử dụng cho việc theo dõi các tài nguyên đã sử dụng và chưa được sử dụng giữa các loại tài nguyên khác nhau.

Ví dụ, một resource provider có thể là một compute node, storage pool hoặc là một dải IP. Placement service sẽ theo dõi tài nguyên dư thừa và tài nguyên đã được sử dụng trên mỗi resource provider. Khi một instance được tạo trên compute node, sẽ sử dụng tài nguyên RAM, CPU từ compute node resource provider, disk từ một external storage provider.

Các loại tài nguyên được theo dõi như classes. Dịch vụ này cung cấp một tập các chuẩn resource classes (ví dụ DISK_GB, MEMORY_MB và VCPU) và cung cấp khả năng định nghĩa tùy chọn các resource classes nếu cần.

Mỗi resource provider cũng có thể bao gồm nhiều tập hợp các đặc điểm mô tả từng khía cạnh của resource provider. Ví dụ available disk có thể không chỉ HDD mà còn có thể là SSD (Traits)


## 2. Nova-conductor
Conductor như một nơi điều phối các task. Rebuilt, resize/migrate và building một instance đều được quản lý ở đây. Điều này làm cho việc phân chia trách nhiệm tốt hơn giữa những gì compute nodes nên xử lý và những gì scheduler nên được xử lý, để dọn dẹp các path của execution.

**Ví dụ:** một old process để building một instance là:
- API nhận request để build một instance.
- API send một RPC cast để scheduler pick một compute
- Scheduler sends một RPC cast để compute build một instance, scheduler có thể sẽ cần giao tiếp với tất cả các compute

    - Nếu build thành công thì dừng ở đây
    - Nếu thất bại thì compute sẽ quyết định nếu max number của scheduler retries là hit. và dừng lại ở đó
        
        Nếu việc build được lên lịch lại thì compute sẽ send một RPC cast tới scheduler để pick một compute khác.

`Nova-conductor` là một RPC Server. Trong `nova-conductor` sẽ có hàng loạt các API, nhiệm vụ chính sẽ là một proxy line tới databse và tới các RPC server khác như `nova-api` và `nova-network`. RPC Client sẽ nằm trong `nova-compute`.

Khi muốn upstate của một VM trên `nova-compute`, thay vì kết nối trực tiếp đến DB thì `nova-compute` sẽ call đến `nova-conductor` trước, sau đó `nova-conductor` sẽ thực hiện kết nối tới DB và upstate VM trong DB.

### Lợi ích và hạn chế của Nova-conductor
**Bảo mật**

- **Lợi ích:** Nếu không có thành phần nova-conductor service, tất cả các compute node có nova-compute service sẽ có quyền truy cập trực tiếp vào database bằng việc sử dụng conductor API, khi compute-node bị tấn công thì attacker sẽ có toàn quyền để xâm nhập vào DB. Với nova-conductor, sự ảnh hưởng của các node tới DB sẽ được kiểm soát.

- **Hạn chế:** Nova-conductor API đã hạn chế quyền hạn kết nối tới database của nova-compute nhưng các service khác vẫn có quyền truy cập trực tiếp vào DB. Với một môi trường multi-host, nova-compute, nova-api-metadata, nova-network đều chạy trên compute node và vẫn tham chiếu trực tiếp đến DB.

**Nâng cấp**

Nova-conductor đứng giữa nova-compute và database. Nếu DB schema update thì sẽ không upgade trên nova-compute trên cùng một thời điểm, thay vào đó nova-conductor sẽ sử dụng những API tương thích để làm việc với DB.

## Tham khảo:
https://docs.openstack.org/placement/train/
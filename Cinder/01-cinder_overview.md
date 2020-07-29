# Tổng quan về Cinder

## 1. Tổng quan và một số khái niệm
Cinder là dịch vụ Block Storage trong Openstack. Nó được thiết kế để người dùng cuối có thể thực hiện việc lưu trữ bởi Nova. Có thể sử dụng cinder block storage bằng LVM hoặc các plugin driver cho các nền tảng lưu trữ khác.

Cinder ảo hóa việc quản lý các thiết bị block storage và cung cấp cho người dùng cuối một API đáp ứng được nhu cầu tự phục vụ cũng như tiêu thụ các tài nguyên đó mà không cần biết quá nhiều kiến thức chuyên sâu.

## Một số hình thức lưu trữ trong OPS
| |Lưu trữ tạm thời|Block Storage|Object Storage|
|-|----------------|-------------|--------------|
|**Hình thức sử dụng**	|Dùng để chạy hệ điều hành và scrath space|Thêm một persistent storage vào VM|lưu trữ các VM image , disk volume , snapshot VM,....|
|**Hình thức truy cập**	|Qua một file system	|Một Block device có thể là một partition, formated, mounted (giống như là : /dev/vdc)	|Thông qua RESTAPI|
|**Có thể truy cập từ**	|Trong một VM	|Trong một VM	|Bất kỳ đâu|
|**Quản lý bởi**	|NOVA	|Cinder	|Swift|
|**Những vấn đề tồn tại**	|VM được kết thúc	|Có thể được xóa bởi user	|Có thể được xóa bởi user|
|**Kích cỡ được xác định bởi**	|Người quản trị hệ thống cấu hình cài đặt kích cỡ, tương tự như là Flavors	|Dựa theo đặc điểm yêu cầu của người dùng	|Số lượng lưu trữ mà máy vật lý hiện có|

### Một số khái niệm
1. Share storage: là hệ thống lưu trữ được sử dụng chung bởi nhiều người hay máy tính. Nó lưu trữ tất cả các tệp trong một kho lưu trữ tập trung và cho phép nhiều người dùng truy cập chúng cùng một lúc.

2. Scale up: nâng cấp những thứ hiện có để có hiệu năng tốt hơn và xử lý nhiều tải hơn. Ví dụ: thay thế CPU 2 core thành CPU 4 core

3. Scale out: thêm nhiều thành phần tương tự hiện có để chia tải, đồng thời phải sử dụng các giải pháp cân bằng tải (load balencing)

4. File storage: lưu trữ cấp độ tệp hoặc lưu trữ dựa trên tệp, là một phương pháp lưu trữ phân cấp được sử dụng để tổ chức và lưu trữ dữ liệu trên ổ cứng máy tính hoặc trên NAS (network-attached storage). Thường được sử dụng cho dữ liệu có cấu trúc và dung lượng không quá lớn

5. Block storage: lưu trữ dựa trên các khối. Là một công nghệ được sử dụng để lưu trữ các file dữ liệu trên SANs (Storage Area Networks) hoặc cloud. Block storage chia dữ liệu thành các khối và sau đó lưu trữ các khối dưới dạng các phần riêng biệt, mỗi khối có mã định danh duy nhất. Điều đó có nghĩa là nó có thể lưu trữ các khối đó trên các hệ thống khác nhau và mỗi khối có thể được cấu hình (hoặc phân vùng) để hoạt động với các hệ điều hành khác nhau. Nó tách dữ liệu khỏi môi trường người dùng, cho phép dữ liệu đó được trải rộng trên nhiều môi trường. Điều này tạo ra nhiều đường dẫn đến dữ liệu và cho phép người dùng truy xuất dữ liệu nhanh chóng

6. Object storage: lưu trữ dựa trên đối tượng. Thường được sử dụng để xử lý khối lượng lớn dữ liệu phi cấu trúc. Đây là dữ liệu không phù hợp hoặc không thể được tổ chức dễ dàng vào cơ sở dữ liệu quan hệ truyền thống với các hàng và cột. Dữ liệu phi cấu trúc bao gồm: email, hình ảnh, video, ... Object storage không sử dụng thư mục hay hệ thống phân cấp phức tạp nào. Thay vào đó, mỗi object là một kho lưu trữ độc lập gồm dữ liệu, metadata, và ID xác thực để ứng dụng dùng để truy cập. Nó dùng để lưu trữ dữ liệu không thay đổi thường xuyên hoặc hoàn toàn (tệp tĩnh), chẳng hạn như hồ sơ giao dịch hoặc tệp nhạc, hình ảnh và video.

## 2. Kiến trúc và cơ chế Cinder
<img src="..\images\Screenshot_73.png">

- `cinder-api`: xác thực và định tuyến các yêu cầu xuyên suốt dịch vụ Cinder

- `cinder-scheduler`: Lên lịch và định tuyến các yêu cầu tới dịch vụ volume thích hợp. Tùy thuộc vào cách cấu hình, có thể chỉ là dùng round-robin để định ra việc sẽ dùng volume service nào, hoặc phức tạp hơn có thể sử dụng Filter Scheduler. Filter Scheduler là mặc định và bật các bộ lọc như Capacity(sức chứa), Avaibility Zone, Volume Type, và Capability(khả năng).

- `cinder-volume` : Quản lý thiết bị block storage, đặc biệt là các thiết bị back-end

- `cinder-backup` : Cung cấp phương thức để backup một Block Storage volume tới Openstack Object Storage (Swift)

- `Driver` : Chứa các mã back-end cụ thể để có thể liên lạc với các loại lưu trữ khác nhau.

- `Storage` : Các thiết bị lưu trữ từ các nhà cung cấp khác nhau.

- `SQL DB` : Cung cấp một phương tiện dùng để back up dữ liệu từ Swift/Celp, etc,....

## 3. Các thành phần trong Cinder
- **Back-end Storage Device** : Dịch vụ Block Storage yêu cầu một vài kiểu của back-end storage mà dịch vụ có thể chạy trên đó. Mặc định là sử dụng LVM trên một local volume group tên là "cinder-volumes"

- **User và Project** : Cinder được dùng bởi các người dùng hoặc khách hàng khác nhau (project trong một shared system), sử dụng chỉ định truy cập dưa vào role (role-based access). Các role kiểm soát các hành động mà người dùng được phép thực hiện. Trong cấu hình mặc định, phần lớn các hành động không yêu cầu một role cụ thể, nhưng sysad có thể cấu hình trong file `policy.json` để quản lý các rule. Một truy cập của người dùng có thể bị hạn chế bởi project, nhưng username và pass được gán chỉ định cho mỗi user. Key pairs cho phép truy cập tới một volume được mở cho mỗi user, nhưng hạn ngạch để kiểm soát sự tiêu thu tài nguyên trên các tài nguyên phần cứng có sẵn là cho mỗi project.

- **Volume, Snapshot và Backup**
    - `Volume` : Các tài nguyên block storage được phân phối có thể gán vào máy ảo như một ổ lưu trữ thứ 2 hoặc có thể dùng như là vùng lưu trữ cho root để boot máy ảo. Volume là các thiết bị block storage R/W bền vững thường được dùng để gán vào compute node thông qua iSCSI.
    - `Snapshot` : Một bản copy trong một thời điểm nhất định của một volume. Snapshot có thể được tạo từ một volume mà mới được dùng gaafnd ây trong trạng thái sẵn sàng. Snapshot có thể được dùng để tạo một volume mới thông qua việc tạo từ snapshot.
    - `Backup` : Một bản copy lưu trữ của một volume thông thường được lưu ở Swift.

## 4. Các phương thức boot máy ảo (từ góc nhìn đối với Cinder)
- **Image** : Tạo một ephameral disk từ image đã chọn

- **Volume** : Boot máy ảo từ một bootable volume đã có sẵn

- **Image (tạo một volume mới)** : Tạo một bootable volume mới từ image đã chọn và boot mấy ảo từ đó.

- **Volume snapshot (tạo một volume mới)** : Tạo một volume từ volume snapshot đã chọn và boot máy ảo từ đó

### Điểm khác nhau giữa Ephemeral và Volume boot disk
**Ephemeral Boot Disk**

- Ephemeral disk là disk ảo mà được tạo cho mục đích boot một máy ảo và nên được coi là nhất thời.
- Ephemeral disk hữu dụng trong trường hợp bạn không lo lắng về nhu cầu nhân đôi một máy ảo hoặc hủy một máy ảo và dữ liệu trong đó sẽ mất hết. Bạn vẫn có thể mount một volume trên một máy ảo được boot từ một ephemeral disk và đẩy bất kỳ data nào cần thiết để lưu lại trong volume.

Một số đặc tính :

- Không sử dụng hết volume quota: Nếu bạn có nhiều instance quota, bạn có thể boot chúng từ ephemeral disk ngay cả khi không có nhiều volume quota
- Bị xóa khi vm bị xóa: Dữ liệu trong emphemeral disk sẽ bị mất khi xóa mấy ảo

**Volume Boot Disk**

- Volume là dạng lưu trữ bền vững hơn ephemeral disk và có thể dùng để boot như là một block device có thể mount được.
- Volume boot disk hữu dụng khi bạn cần dupicate một vm hoặc backup chúng bằng cách snapshot, hoặc nếu bạn muốn dùng phương pháp lưu trữ đáng tin cậy hơn là ephemeral disk. Nếu dùng dạng này, cần có đủ quota cho các vm cần boot.

Một số đặc tính :

- Có thể snapshot
- Không bị xóa khi xóa máy ảo : Bạn có thể xóa máy ảo nhưng dữ liệu vẫn còn trong volume
- Sử dụng hết volume quota : volume quota sẽ được sử dụng hết khi dùng tùy chọn này.
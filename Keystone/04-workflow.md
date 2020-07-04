# Luồng làm việc của Keystone (Workflow)

<img src="..\images\Screenshot_59.png">


1. User muốn truy cập server vào hệ thống:
    - User gửi thông tin đăng nhập (username và mật khẩu) tới Keystone
    - Keystone kiểm tra thông tin đăng nhập. Đúng sẽ gửi lại một Temporary Token (được sinh ra từ thông tin đăng nhập của User) và Generic catalog (danh mục chung) cho User

2. User gửi yêu cầu danh sách các project hay service nó có quyền truy cập:
    - User gửi lại Temporay Token cùng với yêu cầu danh sách Project hay Service mà nó được quyền truy cập
    - Keystone sẽ gửi lại danh sách các Project và Service mà User có quyền truy cập

3. Keystone cung cấp danh sách service cho User
    - User gửi thông tin đăng nhập với Service muốn sử dụng.
    - Keystone sẽ gửi lại thông tin project hay service (Nếu User có quyền truy cập) kèm với Token sử dụng Service đó.
    - User xác định Endpoint chính xác để gọi đến và gửi request Token đến Endpoint đó

4. Service sẽ xác minh Token của User
    - Service sẽ gửi Token của User đến Keystone để kiểm tra xem Token có đúng User không
    - Keystone kiểm tra và gửi lại kết quả xác minh User.
    - Nếu đúng thì Service sẽ gửi request tới Keyston xem User này có được sử dụng Service hay không.

5. Keystone cung cấp thông tin về Token
    - Keystone sẽ xác nhận (validates) với Service là Token này đúng của User và khớp với request và xác nhận request đó từ User
    - Service sẽ xác nhận request với Policy với User của nó.

6. Service thực hiện yêu cầu của User
    - Service thực hiện request của User (Nếu Policy của User cho phép)

7. Service báo lại trạng thái và kết quả cho User
    - Service sẽ thông báo cho User có thể sử dụng.
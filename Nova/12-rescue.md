# Nova rescue

Openstack cung cấp chức năng để rescue các instnace trong các trường hợp lỗi hệ thống, mất mát filesystem, mất SSH key, cấu hình network bị sai hoặc dùng để khôi phục mật khẩu.

Lưu ý: Trong quá trình boot thì instance disk và recuse disk có thể trùng UUID , vì thế trong 1 một số trường hợp thì instance đã vào mode rescue nhưng vẫn boot từ local disk

Ví dụ một vài trường hợp bạn muốn sửa chữa một hệ thống bị bằng cách sử dụng rescue boot:
- Mất ssh key và muốn kích hoạt tàm thời chế độ đăng nhập bằng password
- Cấu hình mạng gặp lỗi
- Cấu hình boot lỗi


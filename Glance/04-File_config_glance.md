# Các file cấu hình của Glance

Glance gồm các file cấu hình cơ bản sau đây:
- **glance-api.conf** : File cấu hình cho API của image service.
- **glance-registry.conf** : File cấu hình cho glance image registry - nơi lưu trữ metadata về các images.
- **glance-scrubber.conf** : Được dùng để dọn dẹp các image đã được xóa
- **policy.json** : Bổ sung truy cập kiểm soát áp dụng cho các image service. Trong này, chúng tra có thể xác định vai trò, chính sách, làm tăng tính bảo mật trong Glane OpenStack.
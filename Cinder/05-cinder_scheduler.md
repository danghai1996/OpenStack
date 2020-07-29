# Cinder-scheduler

# 1. Giới thiệu về Cinder-scheduler
Giống với `nova-scheduler`, cinder cũng có một daemon chịu trách nhiệm cho việc quyết định xem sẽ tạo cinder volume ở đâu khi mô hình có hơn một backend storage. Mặc định nếu người dùng không chỉ rõ host để tạo máy ảo thì `cinder-scheduler` sẽ thực hiện filter và weight theo những opions sau:
```
# Which filter class names to use for filtering hosts when not specified in the
# request. (list value)
#scheduler_default_filters = AvailabilityZoneFilter,CapacityFilter,CapabilitiesFilter

# Which weigher class names to use for weighing hosts. (list value)
#scheduler_default_weighers = CapacityWeigher
```

Bạn sẽ buộc phải kích hoạt tùy chọn `filter_scheduler` để sử dụng multiple-storage back ends.

# 2. Cinder Scheduler Filters
- `AvailabilityZoneFilter`: Filter bằng availability zone

- `CapabilitiesFilter`: Filter theo tài nguyên (máy ảo và volume)

- `CapacityFilter`: Filter dựa vào công suất sử dụng của volume backend

- `DifferentBackendFilter`: Lên kế hoạch đặt các volume ở các backend khác nhau khi có 1 danh sách các volume

- `DriverFilter`: Dựa vào `filter function` và metrics.
# Module Write file

- Module cho phép tạo file trong quá trình tạo VM.
- Encodeding bằng base64, gzip hoặc kết hợp cả 2
- Không nên lưu file vào thư mục `/tmp` do khi boot có thể service `systemd-tmpfiles-clean` sẽ khiến các file thêm bị xóa

### B1: Chuẩn bị nội dung truyền vào để tạo file
1. Mã hóa base64. Có thể dùng lệnh sau trên linux để mã hóa
    ```
    cat <đường_dẫn_file> | base64
    ```
2. Nội dung file bình thường, không mã hóa
    ```
    Test write_files

    #Write_file
    ```

### B2: Tạo VM, truyền cloud-init
```yaml
#cloud-config
password: 'hai1996'
chpasswd: { expire: False }
ssh_pwauth: True
write_files:
- encoding: b64
  content: 
    IyEvYmluL2Jhc2gKCiMgU2NyaXB0IGNoYW5nZSBkb21haW4gSml0c2kKCk5FV19ET01BSU49JChj
    dXJsIGh0dHA6Ly8xNjkuMjU0LjE2OS4yNTQvbGF0ZXN0L21ldGEtZGF0YS9sb2NhbC1pcHY0KQoK
    IyBHZXQgb2xkLCBuZXcgZG9tYWl...
  owner: root:root
  path: /root/write_files
  permissions: '0755'
- content: |
    Test write_files

    #Write_file
  path: /root/test
  permissions: '0555'
```

# Kiểm chứng giới hạn nội dung truyền

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
3. Mã hóa gzip:
    ```
    cat <đường_dẫn_file> | gzip | base64 -w0 
    ```

### B2: Tạo VM, truyền cloud-init
#### Truyền văn bản dạng raw
```yaml
#cloud-config
password: 'password_VM'
chpasswd: { expire: False }
ssh_pwauth: True
write_files:
- content: |
    Test write_files

    #Write_file
  path: /root/test
  permissions: '0555'
```

### Truyền văn bản mã hóa base64
```yaml
#cloud-config
password: 'password_VM'
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
```


### Truyền văn bản mã hóa base64
```yaml
#cloud-config
#cloud-config
password: 'password_VM'
chpasswd: { expire: False }
ssh_pwauth: True
write_files:
- encoding: gzip
  content: !!binary |
      H4sIAAAAAAAA/81WbW/bNhD+rl9xc1zYTivRUtI0S+cOQVJsBdo0WzZgQN0atEjFxCRKJSk7QdL/viMl2XLipGn2gimBTB3vnrvjHR9y6zsyFZJMqZ55W3B8CIpLvoCCar3IFUPZEZWz4z/gZEblzzmFozQvGfzGaebh5E/cgJBJDtml/pxOVJ6bibNlQCUDRieUZUL2q2k3rucHKx95ymrhJBx1+zE1QEqtSJrHNCVMKB4bZ0p0rERhNNHclEVgLgxcw7niReV+hF9xacBn0Bv1wE8if9AGjx4J7qYtxp0ORPFIaDTchIklqDBFASovDYdzXOj9wP2hgeYMfAk9olVMrjQJtnEA2+P+h0/
      ....
  owner: root:root
  path: /root/write_files
  permissions: '0755'
```
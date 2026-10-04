# Báo Cáo Bài 1: Thay đổi cổng kết nối SSH (SSH Port Hardening)

- **Họ và tên:** Học viên DevOps
- **Đường dẫn nộp bài:** `homework/session_03/ex1/README.md`
- **Tài khoản thực hiện:** `devops` (Sử dụng quyền `sudo`)

---

## 1. Mục tiêu thực hiện
- Đổi cổng dịch vụ SSH mặc định từ **22** sang **2222** trên máy chủ ảo nhằm hạn chế các cuộc tấn công quét cổng tự động (port scanning/brute-force).
- Cấu hình tường lửa **UFW** mở cổng `2222/tcp` trước khi khởi động lại dịch vụ SSH để tránh nguy cơ bị khóa truy cập (Lockout).

---

## 2. Các bước tiến hành

### Bước 1: Cho phép cổng 2222/tcp qua tường lửa UFW
Mở cổng `2222/tcp` và kiểm tra lại trạng thái UFW để đảm bảo không bị ngắt kết nối đột ngột:

```bash
devops@vps:~$ sudo ufw allow 2222/tcp
Rules updated
Rules updated (v6)

devops@vps:~$ sudo ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere                  
2222/tcp                   ALLOW IN    Anywhere                  
22/tcp (v6)                ALLOW IN    Anywhere (v6)             
2222/tcp (v6)              ALLOW IN    Anywhere (v6)             
```

### Bước 2: Chỉnh sửa cấu hình SSH Daemon
Chỉnh sửa file cấu hình SSH `/etc/ssh/sshd_config` bằng tài khoản `devops` có quyền `sudo`:

```bash
sudo nano /etc/ssh/sshd_config
```

Thêm/thay đổi dòng chỉ thị cổng thành `Port 2222`:

```text
# Package-provided files regexp pattern: /etc/ssh/sshd_config.d/*.conf
Port 2222
# AddressFamily any
# ListenAddress 0.0.0.0
# ListenAddress ::
```

### Bước 3: Khởi động lại dịch vụ SSH
Khởi động lại SSH daemon để áp dụng cổng mới:

```bash
devops@vps:~$ sudo systemctl restart ssh
```

*Lưu ý: Không đóng cửa sổ Terminal hiện tại cho đến khi đã kiểm tra kết nối thành công ở cửa sổ mới.*

---

## 3. Nhật ký kiểm tra (Terminal Logs)

### Kiểm tra 1: Kết nối tới cổng mới 2222 (Thành công)
Từ máy tính cá nhân (Client), thực hiện lệnh kết nối SSH tới cổng `2222`:

```bash
$ ssh -p 2222 devops@103.xxx.xxx.xxx
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-88-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

Last login: Wed Oct 25 10:00:00 2023 from 118.xxx.xxx.xxx
devops@vps:~$ 
```

=> **Kết quả:** Kết nối thành công qua cổng **2222** bằng SSH Key/mật khẩu.

---

### Kiểm tra 2: Thử kết nối tới cổng mặc định 22 (Thất bại)
Thực hiện lệnh kết nối SSH qua cổng mặc định `22`:

```bash
$ ssh -p 22 devops@103.xxx.xxx.xxx
ssh: connect to host 103.xxx.xxx.xxx port 22: Connection refused
```

=> **Kết quả:** Kết nối tới cổng **22** bị từ chối hoàn toàn (`Connection refused`).

---

## 4. Dọn dẹp tường lửa (Tùy chọn nâng cao)
Sau khi xác nhận cổng `2222` hoạt động ổn định, xóa quy tắc cho phép cổng `22` trên UFW để tối ưu bảo mật:

```bash
devops@vps:~$ sudo ufw delete allow 22/tcp
Rule deleted
Rule deleted (v6)
```
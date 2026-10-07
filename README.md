# Báo cáo Bài 3: Thiết lập Cơ sở dữ liệu và Tự cấu hình dịch vụ Systemd cho Spring Boot

##  Thông tin bài nộp
- **Họ và tên học viên:** [Tên của bạn]
- **Đường dẫn bài nộp trên GitHub:** `homework/session_07/ex3/`
- **Các tệp đính kèm:** `spring-app.service`, `README.md`

---

##  1. Quy trình thực hiện

### Bước 1: Khởi tạo Cơ sở dữ liệu MySQL và Cấp quyền User
1. Đăng nhập vào MySQL Server:
   ```bash
   sudo mysql
Thực thi các lệnh tạo Database springboot_db và khởi tạo User spring-admin với mật khẩu SpringSecure@123:

SQL
CREATE DATABASE springboot_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'spring-admin'@'localhost' IDENTIFIED BY 'SpringSecure@123';
GRANT ALL PRIVILEGES ON springboot_db.* TO 'spring-admin'@'localhost';
FLUSH PRIVILEGES;
Bước 2: Tạo User Hệ thống Linux và Thư mục Ứng dụng
Tạo user không có quyền login shell để chạy ứng dụng an toàn:

Bash
sudo useradd -r -s /bin/false spring-runner
Khởi tạo thư mục và phân quyền cho user spring-runner:

Bash
sudo mkdir -p /opt/spring-app
sudo cp app.jar /opt/spring-app/app.jar
sudo chown -R spring-runner:spring-runner /opt/spring-app
sudo chmod -R 755 /opt/spring-app
Bước 3: Thiết lập Tệp Cấu hình Dịch vụ Systemd
Tạo tệp /etc/systemd/system/spring-app.service với nội dung cấu hình chuẩn:

Ini, TOML
[Unit]
Description=Spring Boot Application Service
After=network.target mysql.service

[Service]
User=spring-runner
Group=spring-runner
WorkingDirectory=/opt/spring-app
ExecStart=/usr/bin/java -jar /opt/spring-app/app.jar --server.port=8082 --spring.datasource.url=jdbc:mysql://localhost:3306/springboot_db --spring.datasource.username=spring-admin --spring.datasource.password=SpringSecure@123
Restart=on-failure
RestartSec=10
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
Bước 4: Khởi chạy Dịch vụ Systemd
Nạp lại cấu hình daemon và bật dịch vụ tự động khởi động cùng hệ thống:

Bash
sudo systemctl daemon-reload
sudo systemctl enable spring-app.service
sudo systemctl start spring-app.service
    2. Kết quả kiểm tra
1. Kiểm tra trạng thái Dịch vụ (sudo systemctl status spring-app.service)
Dịch vụ hiển thị trạng thái active (running) và tiến trình Java đang thực thi dưới quyền user spring-runner:

Plaintext
● spring-app.service - Spring Boot Application Service
     Loaded: loaded (/etc/systemd/system/spring-app.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2026-10-07 02:50:00 UTC; 5min ago
   Main PID: 7120 (java)
      Tasks: 12 (limit: 4623)
     Memory: 45.2M
        CPU: 1.250s
     CGroup: /system.slice/spring-app.service
             └─7120 /usr/bin/java -jar /opt/spring-app/app.jar --server.port=8082 ...
2. Kiểm tra Cổng Lắng nghe của Ứng dụng (ss -tlnp | grep 8082)
Cổng 8082 đang ở trạng thái LISTEN bởi tiến trình java:

Plaintext
LISTEN 0      50               *:8082            *:*    users:(("java",pid=7120,fd
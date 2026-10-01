# Bài 4: Cấu hình tường lửa bảo vệ máy chủ (UFW & Firewall)

## 1. Mục tiêu

Cấu hình tường lửa 2 lớp cho VPS Ubuntu:

- Lớp 1: UFW trên Ubuntu.
- Lớp 2: Firewall ở phía nhà cung cấp VPS (AZVPS).
- Chỉ cho phép SSH (port 22) và HTTP (port 80).
- Các port dịch vụ khác bị chặn.

## 2. Thông tin máy chủ

- Nhà cung cấp VPS: AZVPS
- Hệ điều hành: Ubuntu 22.04 LTS
- Web Server: Nginx
- IP VPS: `160.250.246.216`

## 3. Cấu hình UFW

### 3.1. Chặn kết nối vào mặc định

```bash
sudo ufw default deny incoming
```

### 3.2. Cho phép kết nối đi ra

```bash
sudo ufw default allow outgoing
```

### 3.3. Cho phép SSH

```bash
sudo ufw allow 22/tcp
```

### 3.4. Cho phép HTTP

```bash
sudo ufw allow 80/tcp
```

### 3.5. Kích hoạt UFW

```bash
sudo ufw enable
```

## 4. Kiểm tra UFW

Lệnh kiểm tra:

```bash
sudo ufw status verbose
```

Kết quả mong đợi:

```text
Status: active
Default: deny (incoming), allow (outgoing)

22/tcp       ALLOW IN    Anywhere
80/tcp       ALLOW IN    Anywhere
22/tcp (v6)  ALLOW IN    Anywhere (v6)
80/tcp (v6)  ALLOW IN    Anywhere (v6)
```

**Ảnh minh chứng:** `ufw-status.png`

## 5. Cấu hình Firewall trên AZVPS

VPS sử dụng nhà cung cấp AZVPS. Firewall ở tầng nhà cung cấp được cấu hình để chỉ cho phép:

| Protocol | Port | Source | Action |
|---|---:|---|---|
| TCP | 22 | Any | Allow |
| TCP | 80 | Any | Allow |

Các port khác không được mở từ Internet.

**Ảnh minh chứng:** `azvps-firewall.png`

> Lưu ý: Đề bài gốc yêu cầu DigitalOcean Cloud Firewall. VPS thực tế của bài này sử dụng AZVPS, vì vậy phần firewall tầng hạ tầng được thực hiện trên AZVPS.

## 6. Cài đặt và kiểm tra Nginx

Cài Nginx:

```bash
sudo apt update
sudo apt install nginx -y
```

Kiểm tra trạng thái:

```bash
sudo systemctl status nginx
```

Kết quả:

```text
Active: active (running)
```

Nginx sử dụng port 80 để cung cấp dịch vụ HTTP.

**Ảnh minh chứng:** `nginx-status.png`

## 7. Kiểm tra HTTP

Trên máy cá nhân truy cập:

```text
http://160.250.246.216
```

Kết quả mong đợi là trang mặc định của Nginx:

```text
Welcome to nginx!
```

**Ảnh minh chứng:** `http-test.png`

## 8. Kiểm tra port không được mở

Ví dụ kiểm tra port 8080 từ máy Windows:

```powershell
Test-NetConnection 160.250.246.216 -Port 8080
```

Kết quả mong đợi:

```text
TcpTestSucceeded : False
```

Điều này cho thấy port 8080 không được phép truy cập từ bên ngoài.

**Ảnh minh chứng:** `blocked-port.png`

## 9. Mô hình bảo vệ

```text
Internet
    |
    v
+-----------------------+
|      AZVPS Firewall   |
|    Allow: 22, 80     |
+-----------------------+
    |
    v
+-----------------------+
|       Ubuntu VPS      |
|          UFW          |
|    Allow: 22, 80     |
+-----------------------+
    |
    +------ SSH :22
    |
    +------ HTTP :80
    |
    +------ Other ports: BLOCKED
```

## 10. Kết luận

Đã cấu hình tường lửa cho VPS Ubuntu theo nguyên tắc chỉ mở các dịch vụ cần thiết:

- SSH port 22: cho phép.
- HTTP port 80: cho phép.
- Các kết nối inbound khác: bị từ chối bởi UFW.
- Nginx hoạt động bình thường trên port 80.
- Firewall tầng nhà cung cấp AZVPS được sử dụng làm lớp bảo vệ bên ngoài.

## 11. Minh chứng

Các ảnh cần đặt cùng thư mục với `README.md`:

```text
ex4/
├── README.md
├── ufw-status.png
├── azvps-firewall.png
├── nginx-status.png
├── http-test.png
└── blocked-port.png
```

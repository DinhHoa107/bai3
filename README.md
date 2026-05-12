# Bài Tập 03 - Triển khai WordPress bằng Docker + Cloudflare Tunnel

**Môn:** Phát triển ứng dụng với mã nguồn mở (TEE0421)  
**Sinh viên:** Tạ Phạm Đình Hòa  
**MSSV:** K225480106010  
**Lớp:** K58KTPM  
**Website:** https://wordpress.taphamdinhhoa.io.vn

---

---

## Yêu cầu

- Ubuntu Server chạy trên VirtualBox
- Docker + Docker Compose
- Domain: `taphamdinhhoa.io.vn` (Cloudflare)
- 3 service: MariaDB, phpMyAdmin, WordPress

---

## Bước 1 - Kiểm tra Docker

```bash
docker --version && docker compose version
```

<!-- Chụp terminal hiện: Docker version 29.4.0 và Docker Compose version v5.1.2 -->
![Kiểm tra Docker](images/b1-docker-version.png)

---

## Bước 2 - Tạo file docker-compose.yml

```bash
mkdir ~/wordpress-btvn03 && cd ~/wordpress-btvn03
cat > docker-compose.yml << 'EOF'
services:
  mariadb:
    image: mariadb:latest
    container_name: mariadb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpass123
      MYSQL_DATABASE: wordpress_db
      MYSQL_USER: wp_user
      MYSQL_PASSWORD: wp_pass123
    volumes:
      - mariadb_data:/var/lib/mysql

  phpmyadmin:
    image: phpmyadmin:latest
    container_name: phpmyadmin
    restart: always
    ports:
      - "8081:80"
    environment:
      PMA_HOST: mariadb
      MYSQL_ROOT_PASSWORD: rootpass123
    depends_on:
      - mariadb

  wordpress:
    image: wordpress:latest
    container_name: wordpress
    restart: always
    ports:
      - "8001:80"
    environment:
      WORDPRESS_DB_HOST: mariadb:3306
      WORDPRESS_DB_NAME: wordpress_db
      WORDPRESS_DB_USER: wp_user
      WORDPRESS_DB_PASSWORD: wp_pass123
    volumes:
      - wordpress_data:/var/www/html
    depends_on:
      - mariadb

volumes:
  mariadb_data:
  wordpress_data:
EOF
```

Kiểm tra nội dung file:

```bash
cat docker-compose.yml
```

<!-- Chụp terminal hiện nội dung file docker-compose.yml -->
![File docker-compose.yml](images/b2-docker-compose-file.png)

---

## Bước 3 - Chạy Docker Compose

```bash
docker compose up -d
docker compose ps
```

<!-- Chụp terminal hiện 3 container đang Up: mariadb, phpmyadmin, wordpress -->
![Docker Compose chạy thành công](images/b3-docker-compose-ps.png)

---

## Bước 4 - Cài đặt WordPress

Truy cập trang cài đặt tại `http://<IP_máy>:8001`

<!-- Chụp trình duyệt tại trang cài đặt WordPress (chọn ngôn ngữ) -->
![Trang cài đặt WordPress](images/b4-wordpress-install.png)

Điền thông tin cài đặt:

<!-- Chụp trang điền thông tin: Site Title, Username, Password, Email -->
![Điền thông tin cài đặt](images/b4-wordpress-setup.png)

<!-- Chụp thông báo "WordPress đã được cài đặt thành công" -->
![Cài đặt thành công](images/b4-wordpress-success.png)

Dashboard quản trị:

<!-- Chụp trang wp-admin Dashboard -->
![Dashboard WordPress](images/b4-wordpress-dashboard.png)

---

## Bước 5 - Cấu hình Cloudflare Tunnel

### 5.1 Cài cloudflared

```bash
curl -L https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb -o cloudflared.deb
sudo dpkg -i cloudflared.deb
```

<!-- Chụp terminal hiện "Setting up cloudflared (2026.3.0)" -->
![Cài cloudflared](images/b5-cloudflared-install.png)

### 5.2 Tạo tunnel

```bash
cloudflared tunnel create wordpress-btvn03
```

<!-- Chụp terminal hiện "Created tunnel wordpress-btvn03 with id 0bc72006-..." -->
![Tạo tunnel](images/b5-tunnel-create.png)

### 5.3 Tạo file cấu hình

```bash
cat > ~/.cloudflared/config.yml << 'EOF'
tunnel: 0bc72006-27c8-4985-bbf3-7a1069493cb8
credentials-file: /home/dinhhoa/.cloudflared/0bc72006-27c8-4985-bbf3-7a1069493cb8.json

ingress:
  - hostname: wordpress.taphamdinhhoa.io.vn
    service: http://localhost:8001
  - service: http_status:404
EOF
```

### 5.4 Tạo DNS record

```bash
cloudflared tunnel route dns wordpress-btvn03 wordpress.taphamdinhhoa.io.vn
```

<!-- Chụp terminal hiện "Added CNAME wordpress.taphamdinhhoa.io.vn which will route to this tunnel" -->
![Tạo DNS record](images/b5-dns-route.png)

### 5.5 Chạy tunnel nền

```bash
nohup cloudflared tunnel run wordpress-btvn03 > /dev/null 2>&1 &
```

Kiểm tra tunnel đang chạy:

```bash
ps aux | grep cloudflared
```

<!-- Chụp terminal hiện process cloudflared đang chạy -->
![Tunnel đang chạy](images/b5-tunnel-running.png)

---

## Bước 6 - Kết quả Website Public

Website WordPress đã được public tại:

🌐 **https://wordpress.taphamdinhhoa.io.vn**

<!-- Chụp trình duyệt hiện trang chủ WordPress tại URL public -->
![Website public - Trang chủ](images/b6-website-public.png)

Kiểm tra bằng curl:

```bash
curl -I https://wordpress.taphamdinhhoa.io.vn
# HTTP/2 200 ✅
```

<!-- Chụp terminal hiện HTTP/2 200 -->
![Curl kiểm tra](images/b6-curl-check.png)

---

## Bước 7 - Bài viết trên WordPress

### Bài viết 1: Giới thiệu bản thân

<!-- Chụp trang bài viết "Giới thiệu bản thân" trên website WordPress -->
![Bài viết 1 - Giới thiệu bản thân](images/b7-post1-gioithieu.png)

### Bài viết 2: Giới thiệu ngành Kỹ thuật Phần mềm tại TNUT

<!-- Chụp trang bài viết "Giới thiệu ngành học" trên website WordPress -->
![Bài viết 2 - Ngành học](images/b7-post2-nganhhoc.png)

---

## Nhận xét WordPress

### 1. Mức độ công sức
Triển khai WordPress bằng Docker tốn công sức với người mới: cấu hình 3 service trong `docker-compose.yml`, xử lý lỗi hết ổ đĩa, xung đột port, IP thay đổi khi đổi mạng. Tuy nhiên sau khi cài xong, việc quản lý nội dung rất dễ dàng.

### 2. Độ dễ/khó sử dụng
- ✅ Giao diện Dashboard thân thiện, hỗ trợ tiếng Việt
- ✅ Tạo bài viết, thêm ảnh/video trực quan, không cần biết code
- ✅ Kho plugin và theme miễn phí phong phú
- ⚠️ Cấu hình ban đầu với Docker + Cloudflare Tunnel đòi hỏi kiến thức kỹ thuật

### 3. Tài nguyên CPU/RAM
| Thành phần | RAM tiêu thụ |
|---|---|
| WordPress | ~150MB |
| MariaDB | ~100MB |
| phpMyAdmin | ~80MB |
| **Tổng** | **~330MB** |

- CPU ở mức thấp (~5-10%) khi ít người truy cập
- Cần tối thiểu **2GB RAM** và **3GB ổ đĩa trống** để chạy ổn định
- Máy thực hành gặp lỗi ổ đĩa đầy 100% vì các Docker image cũ chiếm ~10GB

### 4. Ưu / Nhược điểm
| Ưu điểm | Nhược điểm |
|---|---|
| Miễn phí, mã nguồn mở | Tốn RAM hơn CMS nhẹ khác |
| Cộng đồng hỗ trợ lớn | Cần quản lý bảo mật thường xuyên |
| Không cần biết code vẫn dùng được | Dễ bị đầy ổ đĩa khi dùng Docker |
| Dễ mở rộng với plugin | IP thay đổi cần cập nhật DB thủ công |

### 5. Kết luận
WordPress là công cụ tạo website mạnh mẽ, phù hợp cả người mới lẫn chuyên nghiệp. Kết hợp Docker + Cloudflare Tunnel giúp public website miễn phí mà không cần thuê hosting.

---

*Sinh viên: Tạ Phạm Đình Hòa — K225480106010 — K58KTPM — TNUT*

# ĐỒ ÁN BÁO CÁO MÔN LẬP TRÌNH WEB VÀ AN TOÀN BẢO MẬT THÔNG TIN

- **Họ và tên:** Nguyễn Trung Kiên
- **Domain triển khai:** `kiennguyen1404.id.vn`
- **Môi trường:** Ubuntu 24.04 trên WSL2 (Windows Subsystem for Linux)
- **Công nghệ sử dụng:** Docker Compose, Nginx, Node-RED, MariaDB, phpMyAdmin, Cloudflare Tunnel, Python, HTML5/CSS3/JavaScript (CryptoJS, JSEncrypt).

---

## DEMO HỆ THỐNG TRỰC TUYẾN (LIVE DEMO)

* **Website 1 (Xét học bổng & Gọi API):** [https://s1.kiennguyen1404.id.vn](https://s1.kiennguyen1404.id.vn)
* **API Endpoint (JSON):** [https://s1.kiennguyen1404.id.vn/api/sinh-vien](https://s1.kiennguyen1404.id.vn/api/sinh-vien)
* **Website 2 (Công cụ Mã hóa/Giải mã AES & RSA):** [https://s2.kiennguyen1404.id.vn](https://s2.kiennguyen1404.id.vn)

---

## PHẦN 1: BÀI TẬP MÔN LẬP TRÌNH WEB

### 1. Hạ tầng Docker Compose
Hệ thống sử dụng Docker Compose orchestrate 5 dịch vụ riêng biệt:
- **Nginx (`nginx_proxy`):** Web Server và Reverse Proxy điều hướng 2 Virtual Hosts và proxy endpoint `/api/` sang Node-RED.
- **Node-RED (`nodered_app`):** Xây dựng các luồng API backend nghiệp vụ.
- **MariaDB (`mariadb_db`):** Hệ quản trị cơ sở dữ liệu quan hệ.
- **phpMyAdmin (`phpmyadmin_tool`):** Trực quan hóa quản lý MariaDB qua web.
- **Cloudflared (`cloudflared_tunnel`):** Thiết lập đường truyền HTTPS an toàn từ localhost ra Internet qua Cloudflare Zero Trust.

### 2. Sơ đồ kiến trúc chuyển tiếp
```mermaid
graph TD
    Client[Client / Browser] -->|HTTPS| Cloudflare[Cloudflare Tunnel]
    Cloudflare -->|HTTP| Nginx[Nginx Reverse Proxy - Port 80]
    
    Nginx -->|s1.kiennguyen1404.id.vn| Site1[Website 1 - /html1]
    Site1 -->|/api/sinh-vien| NodeRED[Node-RED API - Port 1880]
    
    Nginx -->|s2.kiennguyen1404.id.vn| Site2[Website 2 - /html2]

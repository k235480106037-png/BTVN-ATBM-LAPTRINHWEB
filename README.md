# MÔN LẬP TRÌNH WEB VÀ AN TOÀN BẢO MẬT THÔNG TIN

- **Họ và tên:** Nguyễn Trung Kiên
- **Tên miền hệ thống:** `kiennguyen1404.id.vn`
- **Môi trường triển khai:** Ubuntu 24.04 (WSL2), Docker Compose, Cloudflare Tunnel

---

## 🌐 DEMO HỆ THỐNG TRỰC TUYẾN

* **Website 1 (Xét học bổng & API):** [https://s1.kiennguyen1404.id.vn](https://s1.kiennguyen1404.id.vn)
* **API Endpoint (JSON):** [https://s1.kiennguyen1404.id.vn/api/sinh-vien](https://s1.kiennguyen1404.id.vn/api/sinh-vien)
* **Website 2 (Công cụ Mã hóa/Giải mã AES & RSA):** [https://s2.kiennguyen1404.id.vn](https://s2.kiennguyen1404.id.vn)

---

## 📊 SƠ ĐỒ KIẾN TRÚC HỆ THỐNG

```mermaid
graph TD
    Client[Client / Browser] -->|HTTPS| Cloudflare[Cloudflare Tunnel]
    Cloudflare -->|HTTP| Nginx[Nginx Reverse Proxy - Port 80]
    
    Nginx -->|s1.kiennguyen1404.id.vn| Site1[Website 1 - /html1]
    Site1 -->|/api/sinh-vien| NodeRED[Node-RED API - Port 1880]
    
    Nginx -->|s2.kiennguyen1404.id.vn| Site2[Website 2 - /html2]

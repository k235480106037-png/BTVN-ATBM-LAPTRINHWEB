# LÝ THUYẾT VÀ KIẾN TRÚC MÔN LẬP TRÌNH WEB

- **Sinh viên thực hiện:** Nguyễn Trung Kiên
- **Tên miền hệ thống:** `kiennguyen1404.id.vn`

---

## 1. Hạ tầng Containerization (Docker Compose)
Hệ thống sử dụng Docker Compose orchestrate 5 dịch vụ độc lập trên cùng một mạng nội bộ (`web_net`):
1. **Nginx (`nginx_proxy`):** Đóng vai trò Reverse Proxy & Virtual Hosting, tiếp nhận request ở cổng 80/443 và phân phối tới các service nội bộ.
2. **Node-RED (`nodered_app`):** Xây dựng các luồng API backend nghiệp vụ (Xử lý logic, tính điểm GPA, cấp học bổng).
3. **MariaDB (`mariadb_db`):** Cơ sở dữ liệu quan hệ lưu trữ thông tin hệ thống.
4. **phpMyAdmin (`phpmyadmin_tool`):** Trực quan hóa quản trị database qua giao diện web.
5. **Cloudflared (`cloudflared_tunnel`):** Tạo đường truyền mã hóa Zero Trust đưa hệ thống ra Internet không cần mở Port Router.

---

## 2. Logic Thuật toán API Node-RED (`/api/sinh-vien`)
API được định tuyến qua 3 Node chính:
`HTTP In` (`/api/sinh-vien`) $\rightarrow$ `Function Node` (Xử lý dữ liệu) $\rightarrow$ `HTTP Response`.

### Thuật toán phân loại học bổng trong Function Node:
- Duyệt qua danh sách sinh viên.
- Nếu $GPA \ge 3.9$: Học bổng **Xuất sắc** ($5.000.000 \text{ VNĐ}$).
- Nếu $3.6 \le GPA < 3.9$: Học bổng **Giỏi** ($3.000.000 \text{ VNĐ}$).
- Trả về mảng JSON chứa các sinh viên đạt học bổng.

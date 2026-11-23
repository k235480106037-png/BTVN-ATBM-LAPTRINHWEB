BÀI TẬP VỀ NHÀ: LẬP TRÌNH WEB & AN TOÀN BẢO MẬT THÔNG TIN


Họ và tên: Nguyễn Trung Kiên


Tên miền hệ thống: `kiennguyen1404.id.vn`



Môi trường triển khai: Ubuntu 24.04 (WSL2), Docker Compose, Cloudflare Tunnel



Thời hạn nộp bài: 23h59 ngày 28/09/2026
---
🌐 DEMO HỆ THỐNG TRỰC TUYẾN
Website 1 (Xét học bổng & Gọi API): https://s1.kiennguyen1404.id.vn
API Endpoint (Node-RED JSON): https://s1.kiennguyen1404.id.vn/api/sinh-vien
Website 2 (Công cụ Mã hóa/Giải mã AES & RSA): https://s2.kiennguyen1404.id.vn
---
📊 SƠ ĐỒ KIẾN TRÚC HỆ THỐNG
```text
[ Client / Browser ]
        │ (HTTPS)
        ▼
[ Cloudflare Tunnel ]
        │ (HTTP)
        ▼
[ Nginx Reverse Proxy (Port 80) ]
        ├── s1.kiennguyen1404.id.vn ──► [ Website 1 - /html1 ] ──► /api/sinh-vien ──► [ Node-RED - Port 1880 ]
        └── s2.kiennguyen1404.id.vn ──► [ Website 2 - /html2 ]

```
PHẦN 1: BÀI TẬP MÔN LẬP TRÌNH WEB
1. Môi trường giả lập & Containerization (Docker Compose)
Hệ thống được khởi tạo trên môi trường giả lập Linux Ubuntu 24.04 (WSL2), sử dụng Docker Compose điều phối 5 dịch vụ chạy độc lập trên cùng mạng nội bộ `web_net`:
`nginx` (nginx_proxy): Tiếp nhận yêu cầu ở cổng `80/443`, phân phối traffic theo tên miền Virtual Host và làm Reverse Proxy cho Node-RED.
`nodered` (nodered_app): Xây dựng luồng backend xử lý logic nghiệp vụ và cung cấp API JSON.
`mariadb` (mariadb_db): Cơ sở dữ liệu quan hệ lưu trữ dữ liệu hệ thống.
`phpmyadmin` (phpmyadmin_tool): Trực quan hóa giao diện quản trị database trên cổng `8080`.
`cloudflared` (cloudflared_tunnel): Tạo đường truyền mã hóa Cloudflare Zero Trust public hệ thống ra Internet không cần mở port Router.
2. Cấu hình Nginx Virtual Host (2 Domain) & Reverse Proxy API
File cấu hình `vhosts.conf` phân chia lưu lượng cho 2 tên miền độc lập và chuyển tiếp kết nối API:
```nginx
# 1. Website 1 (s1.kiennguyen1404.id.vn) & Reverse Proxy API
server {
    listen 80;
    server_name s1.kiennguyen1404.id.vn site1.kiennguyen1404.id.vn;

    location / {
        root /usr/share/nginx/html/site1;
        index index.html index.htm;
    }

    # Proxy qua Node-RED container ở port 1880
    location /api/ {
        proxy_pass http://nodered:1880/api/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;

        # Cấu hình CORS để JS Frontend truy cập an toàn
        add_header 'Access-Control-Allow-Origin' '*' always;
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS' always;
    }
}

# 2. Website 2 (s2.kiennguyen1404.id.vn)
server {
    listen 80;
    server_name s2.kiennguyen1404.id.vn site2.kiennguyen1404.id.vn;

    location / {
        root /usr/share/nginx/html/site2;
        index index.html index.htm;
    }
}
```
3. Node-RED API & Mã JavaScript ở Frontend
Node-RED API (`/api/sinh-vien`): Sử dụng nút `http_in` $\rightarrow$ `Function Node` (xử lý logic phân loại học bổng theo điểm GPA) $\rightarrow$ `http_response`.
Cấu trúc JSON API trả về:
```json
{
  "status": 200,
  "message": "Lấy danh sách học bổng thành công",
  "timestamp": "2026-09-28T12:00:00.000Z",
  "students": [
    { "id": "SV01", "name": "Nguyễn Văn A", "major": "CNTT", "gpa": 3.95, "scholarship": 5000000 },
    { "id": "SV02", "name": "Trần Thị B", "major": "ATTT", "gpa": 3.72, "scholarship": 3000000 }
  ]
}
```
Mã JavaScript gọi API từ HTML (`index.html`):
```javascript
async function fetchScholarships() {
    try {
        const response = await fetch('/api/sinh-vien');
        const data = await response.json();
        if (data.status === 200) {
            // Render mảng JSON lên bảng HTML
            renderTable(data.students);
        }
    } catch (err) {
        console.error("Lỗi gọi API:", err);
    }
}
```
---
## PHẦN 2: BÀI TẬP MÔN AN TOÀN VÀ BẢO MẬT THÔNG TIN

### 1. Thuật toán Mã hóa Đối xứng DES và AES

* **Thuật toán DES (Data Encryption Standard):**
  * **Đặc tính:** Mã hóa khối đối xứng (Symmetric Block Cipher) theo cấu trúc mạng Feistel.
  * **Thông số:** Kích thước khối 64-bit, độ dài khóa thực tế 56-bit (8-bit dùng kiểm tra chẵn lẻ).
  * **Quy trình:** Khối rõ 64-bit đi qua Hoán vị ban đầu (IP), chia thành L₀ (32-bit) và R₀ (32-bit). Thực thi qua 16 vòng Feistel biến đổi: Lᵢ = Rᵢ₋₁, Rᵢ = Lᵢ₋₁ ⊕ f(Rᵢ₋₁, Kᵢ). Hàm f nén dữ liệu qua 8 hộp thế S-Box phi tuyến và P-Box. Hoán vị nghịch đảo (IP⁻¹) thu được bản mã.

* **Thuật toán AES (Advanced Encryption Standard):**
  * **Đặc tính:** Mã hóa khối đối xứng dựa trên Mạng thay thế - hoán vị (SPN).
  * **Thông số:** Kích thước khối 128-bit (ma trận trạng thái 4×4 byte). Độ dài khóa: AES-128 (10 vòng), AES-192 (12 vòng), AES-256 (14 vòng).
  * **Các bước biến đổi trong mỗi vòng:**
    * `SubBytes`: Thay thế byte phi tuyến qua bảng S-Box.
    * `ShiftRows`: Dịch vòng các hàng ma trận trạng thái.
    * `MixColumns`: Trộn dữ liệu theo cột trên trường Galois GF(2⁸).
    * `AddRoundKey`: Phép XOR ma trận trạng thái với khóa con của vòng.
  * **Cài đặt demo:** Được lập trình trực tiếp bằng JavaScript (thư viện CryptoJS) tại giao diện Website 2 (`s2.kiennguyen1404.id.vn`).

---

### 2. Thuật toán Mã hóa Bất đối xứng RSA

* **Nguyên lý Sinh cặp khóa (Key Generation):**
  1. Chọn 2 số nguyên tố ngẫu nhiên rất lớn `p` và `q` (`p ≠ q`).
  2. Tính mô-đun `n = p × q`.
  3. Tính hàm Euler `Φ(n) = (p - 1)(q - 1)`.
  4. Chọn số mũ công khai `e` sao cho `1 < e < Φ(n)` và `gcd(e, Φ(n)) = 1` (thường chọn `e = 65537`).
  5. Tính số mũ bí mật `d = e⁻¹ mod Φ(n)` (nghịch đảo nhân theo mod `Φ(n)`).
  6. Khởi tạo cặp khóa: **Khóa Công Khai** `PU = (e, n)`, **Khóa Bí Mật** `PR = (d, n)`.

* **Quy trình Mã hóa & Giải mã:**
  * **Mã hóa:** `C = Mᵉ mod n` (dùng Khóa công khai bên nhận).
  * **Giải mã:** `M = Cᵈ mod n` (dùng Khóa bí mật bên nhận).

---

### 3. Các Mô hình Áp dụng Thuật toán RSA & Mã hóa Lai

#### A. Mô hình Bảo mật (Xác thực người nhận)
* **Quy trình:** Bên gửi dùng Khóa công khai của bên nhận (`PU_nhận`) để mã hóa: `C = Mᵉ mod n`. Bên nhận dùng Khóa bí mật của mình (`PR_nhận`) để giải mã: `M = Cᵈ mod n`.
* **Mục đích:** Bảo vệ tính bí mật của thông điệp. Chỉ có người nhận sở hữu `PR_nhận` mới đọc được nội dung.

#### B. Mô hình Xác thực Nguồn gốc (Xác thực người gửi / Chữ ký số)
* **Quy trình:** Bên gửi dùng Khóa bí mật của mình (`PR_gửi`) để mã hóa/ký lên thông điệp: `S = Mᵈ mod n`. Bên nhận dùng Khóa công khai của bên gửi (`PU_gửi`) để giải mã/xác thực: `M = Sᵉ mod n`.
* **Mục đích:** Xác thực danh tính người gửi và chống từ chối trách nhiệm (non-repudiation), vì chỉ có bên gửi mới giữ `PR_gửi`.

#### C. Mô hình Kết hợp Cả hai (Vừa bảo mật vừa xác thực người gửi)
* **Quy trình:** 
  * *Bước 1 (Xác thực):* Bên gửi mã hóa thông điệp `M` bằng `PR_gửi` tạo ra bản ký `S = M^(d_gửi) mod n_gửi`.
  * *Bước 2 (Bảo mật):* Bên gửi tiếp tục mã hóa `S` bằng `PU_nhận` tạo bản mã gửi đi `C = S^(e_nhận) mod n_nhận`.
  * *Giải mã ở bên nhận:* Bên nhận dùng `PR_nhận` giải mã lớp ngoài lấy `S`, sau đó dùng `PU_gửi` giải mã lớp trong thu được thông điệp gốc `M`.
* **Mục đích:** Đạt được đồng thời 2 yêu cầu: Bảo mật dữ liệu trên đường truyền và Xác thực chính xác nguồn gốc người gửi.

#### D. So sánh Thời gian Mã hóa / Giải mã: RSA vs AES
* **Bản chất toán học:** AES dùng phép XOR, hoán vị, thay thế byte; RSA dùng Lũy thừa số nguyên lớn trên modulo `n`.
* **Tốc độ xử lý:** AES **Cực nhanh** (hàng trăm MB/s đến GB/s); RSA **Chậm** (chậm hơn AES từ 100 - 1000 lần).
* **Kích thước dữ liệu:** AES không giới hạn (tệp tin lớn, CSDL); RSA bị giới hạn bởi độ dài khóa (ví dụ 1024-bit mã hóa được tối đa 117 bytes).
* **Quản lý khóa:** AES khó phân phối khóa bí mật an toàn; RSA dễ dàng chia sẻ khóa công khai.

#### E. Mô hình Mã hóa Lai (Hybrid Encryption)
* **Nguyên lý:** Kết hợp tốc độ của AES và khả năng phân phối khóa an toàn của RSA.
* **Quy trình:**
  1. Bên gửi sinh ngẫu nhiên khóa phiên AES (`K_session`).
  2. Dùng AES mã hóa toàn bộ dữ liệu/file bằng `K_session`.
  3. Dùng RSA mã hóa `K_session` bằng Khóa công khai bên nhận (`PU_nhận`).
  4. Gửi file đã mã hóa cùng `K_session` (đã mã hóa) cho bên nhận.
  5. Bên nhận dùng `PR_nhận` giải mã lấy lại `K_session`, sau đó dùng `K_session` giải mã file bằng AES.
---
🛠 HƯỚNG DẪN KHỞI CHẠY HỆ THỐNG
Clone Repository:
```bash
   git clone https://github.com/k235480106037-png/BTVN-ATBM-LAPTRINHWEB.git
   cd BTVN-ATBM-LAPTRINHWEB
   ```
Khởi chạy toàn bộ 5 Dịch vụ bằng Docker Compose:
```bash
   docker compose up -d
   ```
Kiểm tra trạng thái các Container:
```bash
   docker compose ps
   ```

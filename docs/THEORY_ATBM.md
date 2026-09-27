# LÝ THUYẾT MÔN AN TOÀN VÀ BẢO MẬT THÔNG TIN

- **Sinh viên thực hiện:** Nguyễn Trung Kiên

---

## 1. Thuật toán Mã hóa Hiện đại DES và AES

### A. Thuật toán DES (Data Encryption Standard)
- **Đặc tính:** Mã hóa khối đối xứng (Symmetric Block Cipher) dựa trên cấu trúc mạng Feistel.
- **Thông số:** Kích thước khối 64-bit, độ dài khóa thực tế 56-bit (8-bit dùng kiểm tra chẵn lẻ).
- **Quy trình thực thi:**
  1. Khối rõ 64-bit đi qua Hoán vị ban đầu ($IP$), chia thành 2 nửa $L_0$ (32-bit) và $R_0$ (32-bit).
  2. Trải qua 16 vòng Feistel biến đổi: $L_i = R_{i-1}$, $R_i = L_{i-1} \oplus f(R_{i-1}, K_i)$. Hàm $f$ mở rộng $R_{i-1}$ lên 48-bit, XOR với khóa con $K_i$, nén qua 8 hộp thế S-Box phi tuyến về 32-bit và xáo trộn P-Box.
  3. Đảo vị trí $R_{16}L_{16}$ và đưa qua Hoán vị nghịch đảo ($IP^{-1}$) thu được bản mã.

### B. Thuật toán AES (Advanced Encryption Standard)
- **Đặc tính:** Mã hóa khối đối xứng dựa trên Mạng thay thế - hoán vị (SPN).
- **Thông số:** Khối cố định 128-bit ($4 \times 4$ byte matrix). Khóa: AES-128 (10 vòng), AES-192 (12 vòng), AES-256 (14 vòng).
- **Các phép biến đổi chính trong mỗi vòng:**
  1. `SubBytes`: Thay thế byte phi tuyến qua bảng S-Box.
  2. `ShiftRows`: Dịch vòng các hàng ma trận trạng thái.
  3. `MixColumns`: Trộn dữ liệu theo cột trên trường Galois $GF(2^8)$.
  4. `AddRoundKey`: Phép XOR ma trận trạng thái với khóa con của vòng.

---

## 2. Thuật toán Mã hóa Bất đối xứng RSA

### Nguyên lý Sinh cặp khóa (Key Generation)
1. Chọn 2 số nguyên tố ngẫu nhiên rất lớn $p$ và $q$ ($p \neq q$).
2. Tính mô-đun $n = p \times q$.
3. Tính hàm Phi-Euler $\phi(n) = (p - 1)(q - 1)$.
4. Chọn số mũ công khai $e$ sao cho $1 < e < \phi(n)$ và $\gcd(e, \phi(n)) = 1$ (thường chọn $e = 65537$).
5. Tính số mũ bí mật $d = e^{-1} \pmod{\phi(n)}$.
6. Thành phần khóa: **Khóa Công Khai** $PU = (e, n)$, **Khóa Bí Mật** $PR = (d, n)$.

### Quy trình Mã hóa & Giải mã
- **Mã hóa:** $C = M^e \pmod n$ (Dùng Khóa công khai $PU_{nhận}$).
- **Giải mã:** $M = C^d \pmod n$ (Dùng Khóa bí mật $PR_{nhận}$).

---

## 3. Mô hình Mã hóa Lai (Hybrid Encryption)

### So sánh AES và RSA
| Tiêu chí | AES (Mã hóa đối xứng) | RSA (Mã hóa bất đối xứng) |
| :--- | :--- | :--- |
| **Phép toán** | XOR, Thế S-Box, Trộn cột | Lũy thừa số nguyên lớn trên $mod \ n$ |
| **Tốc độ** | **Cực nhanh** (Hàng trăm MB/s) | **Chậm** (Chậm hơn AES 100 - 1000 lần) |
| **Dung lượng** | Bất kỳ (File, Video, DB lớn) | Giới hạn bởi độ dài khóa (< 245 bytes) |

### Quy trình Mã hóa Lai
1. Bên gửi sinh ngẫu nhiên **Khóa phiên AES (Session Key)** $K_{session}$.
2. Dùng AES mã hóa dữ liệu gốc bằng $K_{session}$.
3. Dùng RSA mã hóa $K_{session}$ bằng **Khóa Công Khai của bên nhận**.
4. Gửi cả bản mã dữ liệu và $K_{session}$ đã mã hóa sang bên nhận.
5. Bên nhận dùng **Khóa Bí Mật RSA** giải mã lấy lại $K_{session}$, sau đó dùng $K_{session}$ giải mã dữ liệu bằng AES.

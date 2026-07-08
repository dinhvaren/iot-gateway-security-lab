# Kịch bản demo 60 giây

**Môn học:** INT4410 - Bảo mật trong IoT  
**Sinh viên:** Lương Nguyễn Ngọc Đinh - 231A010001  
**Đề tài:** Vai trò của gateway trong bảo mật IoT  
**GitHub:** https://github.com/dinhvaren/iot-gateway-security-lab

---

## Chuẩn bị trước khi quay

- [ ] Docker Desktop đã khởi động.
- [ ] `docker compose up -d` đã chạy và Node-RED sẵn sàng (http://localhost:1880).
- [ ] Flow gateway đã được deploy (Inject Nodes + pipeline đầy đủ).
- [ ] Cửa sổ Node-RED Editor mở sẵn ở tab Debug.
- [ ] Terminal mở sẵn ở thư mục project, chạy `docker compose logs -f` để hiện audit log realtime.
- [ ] Công cụ quay màn hình đã sẵn sàng (OBS Studio / Xbox Game Bar).

---

## Kịch bản chi tiết

### 0:00 - 0:10 (10s) — Giới thiệu luồng

**Hành động:** Di chuyển chuột qua pipeline gateway, chỉ vào từng khối.

> **Lời thoại (10s):**
>
> "Chào thầy và các bạn. Đây là lab gateway bảo mật IoT của em. Pipeline gồm năm lớp: xác thực token, kiểm tra schema, kiểm tra phạm vi, chống replay, và giới hạn tần suất. Toàn bộ sự kiện được ghi vào audit log. Em sẽ demo ba kịch bản."

---

### 0:10 - 0:25 (15s) — Kịch bản 1: Dữ liệu hợp lệ → ACCEPT

**Hành động:**
1. Click Inject Node xanh "Valid Sensor".
2. Chỉ vào Debug panel hiển thị `ACCEPT`.
3. Chỉ vào terminal log: `ACCEPT | sensor-01 | temp=25.5`.

> **Lời thoại (15s):**
>
> "Kịch bản thứ nhất: thiết bị hợp lệ gửi dữ liệu cảm biến nhiệt độ 25.5 độ C, kèm token JWT đúng. Gateway kiểm tra tất cả các lớp — token OK, schema OK, nhiệt độ trong ngưỡng, nonce mới, chưa vượt rate limit — và cho phép đi qua. Audit log ghi nhận ACCEPT."

---

### 0:25 - 0:40 (15s) — Kịch bản 2: Nhiệt độ cao bất thường → REJECT

**Hành động:**
1. Click Inject Node đỏ "Malicious High Temp".
2. Chỉ vào Debug panel hiển thị `REJECT`.
3. Chỉ vào terminal log: `REJECT | sensor-02 | reason=TEMP_OUT_OF_RANGE | temp=120`.

> **Lời thoại (15s):**
>
> "Kịch bản thứ hai: thiết bị gửi nhiệt độ 120 độ C — giá trị bất thường vượt ngưỡng 80 độ. Dù token hợp lệ, lớp kiểm tra phạm vi phát hiện và từ chối ngay. Gateway không chuyển tiếp dữ liệu độc hại xuống backend. Audit log ghi rõ nguyên nhân TEMP_OUT_OF_RANGE."

---

### 0:40 - 0:50 (10s) — Kịch bản 3: Token giả mạo → REJECT

**Hành động:**
1. Click Inject Node đỏ "Attacker Spoof".
2. Chỉ vào Debug panel hiển thị `REJECT`.
3. Chỉ vào terminal log: `REJECT | attacker-01 | reason=INVALID_TOKEN`.

> **Lời thoại (10s):**
>
> "Kịch bản cuối: kẻ tấn công giả mạo thiết bị, gửi token sai. Lớp xác thực JWT phát hiện chữ ký không hợp lệ và từ chối ngay từ đầu. Dù dữ liệu có đúng đi nữa, gateway vẫn chặn vì thiết bị không được xác thực."

---

### 0:50 - 1:00 (10s) — Kết luận

**Hành động:** Phóng to audit log CSV (file `audit.csv`), chỉ vào các dòng ACCEPT và REJECT.

> **Lời thoại (10s):**
>
> "Kết quả: gateway xác thực, lọc dữ liệu và ghi nhật ký đầy đủ. Cả ba kịch bản cho thấy gateway đóng vai trò trung tâm trong bảo mật IoT — là điểm chặn duy nhất giữa thiết bị và hạ tầng. Em xin cảm ơn thầy và các bạn đã theo dõi."

---

## Tổng thời gian: 60 giây

| Phân đoạn | Thời gian | Nội dung | Inject Node |
|---|---|---|---|
| Giới thiệu | 0:00 - 0:10 | Pipeline gateway 5 lớp | — |
| Valid → ACCEPT | 0:10 - 0:25 | Dữ liệu cảm biến hợp lệ | "Valid Sensor" (xanh) |
| High Temp → REJECT | 0:25 - 0:40 | Nhiệt độ 120°C vượt ngưỡng | "Malicious High Temp" (đỏ) |
| Bad Token → REJECT | 0:40 - 0:50 | Token JWT sai | "Attacker Spoof" (đỏ) |
| Kết luận | 0:50 - 1:00 | Audit log + tổng kết | — |

---

## Phương án dự phòng (backup plan)

| Sự cố | Xử lý |
|---|---|
| Docker không khởi động được | Dùng Node-RED chạy local bằng `npx node-red` (bỏ qua Docker) |
| Node-RED flow lỗi khi deploy | Import file `flows-backup.json` từ thư mục `backup/` |
| Quên inject node nào đó | Làm lại phân đoạn đó — mỗi kịch bản độc lập |
| Audit log không hiển thị trên terminal | Đọc trực tiếp file `audit.csv` từ Notepad++ |
| Quay phim lỡ tay | Có thể cắt ghép hậu kỳ; mỗi đoạn quay riêng rẽ |
| Máy chạy chậm, giật hình | Giảm FPS quay xuống 30, tắt các ứng dụng không cần thiết |

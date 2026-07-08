# Hướng dẫn Quay Video Demo - IoT Security Gateway Lab

**Repository:** https://github.com/dinhvaren/iot-gateway-security-lab
**Thời lượng mục tiêu:** 60-90 giây

---

## Checklist Trước Khi Quay

- [ ] `docker compose ps` hiển thị service đang Up (Healthy)
- [ ] Node-RED mở được tại http://localhost:1880
- [ ] Flow "IoT Security Gateway" đã Deploy
- [ ] Debug Sidebar đã mở (Ctrl+G+D hoặc click icon con bọ)
- [ ] Clear toàn bộ log cũ trước khi quay (xóa file logs/audit.log và restart container)
- [ ] VALID - Normal Data cho kết quả ACCEPT
- [ ] INVALID - High Temperature cho kết quả REJECT RULE-04
- [ ] INVALID - Bad Token cho kết quả REJECT RULE-01
- [ ] Cloud Sink (debug node) chỉ hiển thị payload ACCEPT
- [ ] Audit log không chứa token đầy đủ (chỉ token_hint)
- [ ] Desktop không lộ dữ liệu riêng tư
- [ ] OBS ghi đúng cửa sổ (chỉ browser Node-RED)
- [ ] Microphone hoạt động
- [ ] Video dài khoảng 60-90 giây

---

## BƯỚC 1 - CHUẨN BỊ

### 1.1. Khởi động lab

Mở terminal tại thư mục repository:

```bash
cd iot-gateway-security-lab
docker compose up -d
```

Kiểm tra:

```bash
docker compose ps
# Phải thấy: iot-gateway-nodered ... Up ... (healthy)
```

### 1.2. Xóa log cũ để demo sạch

```bash
# Xóa audit log cũ
rm -f logs/audit.log

# Restart container để flow context được reset
docker compose restart
```

### 1.3. Mở Node-RED

Mở trình duyệt: **http://localhost:1880**

Đảm bảo:
- Tab "IoT Security Gateway" đang active
- Debug Sidebar đã mở (Ctrl+G+D)
- Click nút "Clear" trong Debug sidebar để xóa log cũ

### 1.4. Chuẩn bị OBS Studio

- **Nguồn:** Window Capture → chọn cửa sổ trình duyệt Node-RED
- **Độ phân giải:** 1920x1080
- **Frame rate:** 30fps
- **Audio:** Microphone

**QUAN TRỌNG - Chỉ quay:**
- Browser Node-RED (flow + debug sidebar)
- Terminal (khi cần hiển thị Docker/log)

**KHÔNG quay:**
- Mật khẩu
- Token đầy đủ
- Credential
- Thông tin cá nhân
- Màn hình desktop nếu có file/thông tin nhạy cảm

---

## BƯỚC 2 - BẮT ĐẦU QUAY

1. Bắt đầu OBS Recording.
2. Đảm bảo microphone đang thu âm.

---

## BƯỚC 3 - GIỚI THIỆU FLOW (0-10 giây)

**Thao tác:**
- Hiển thị toàn bộ Node-RED flow
- Zoom vừa đủ để thấy pipeline từ trái sang phải

**Hiển thị trên màn hình:**
- 8 Inject Node bên trái
- Chuỗi function node: Normalization → RULE-01 → RULE-02 → ... → RULE-06
- Audit Logger → Decision Router → Cloud Sink / Security Alert

**Lời nói:**
"Đây là mô hình IoT Security Gateway được xây dựng bằng Node-RED. Dữ liệu từ thiết bị IoT phải đi qua 6 luật bảo mật: xác thực token, kiểm tra thiết bị, schema validation, kiểm tra ngưỡng cảm biến, freshness và chống replay - trước khi được gửi lên cloud."

---

## BƯỚC 4 - DEMO PAYLOAD HỢP LỆ (10-25 giây)

**Thao tác:**
1. Click nút Inject (hình vuông nhỏ bên trái) của node **"VALID - Normal Data"**
2. Nhìn vào Debug sidebar

**Hiển thị trên màn hình:**
- Debug tab "Cloud Sink (ACCEPT)" hiển thị JSON với:
  - `"decision": "ACCEPT"`
  - `"rule_id": "PASS"`
  - `"reason": "All gateway checks passed"`

**Lời nói:**
"Đầu tiên tôi gửi một payload hợp lệ. Dữ liệu vượt qua toàn bộ 6 luật bảo mật, được gateway đánh dấu ACCEPT và chuyển tới Cloud Sink. Lưu ý dữ liệu chỉ đến được cloud sau khi đã vượt qua gateway."

---

## BƯỚC 5 - DEMO NHIỆT ĐỘ BẤT THƯỜNG (25-40 giây)

**Thao tác:**
Click nút Inject của node **"INVALID - High Temperature"**

**Hiển thị trên màn hình:**
- Debug tab "Security Alert (REJECT)" hiển thị:
  - `"decision": "REJECT"`
  - `"rule_id": "RULE-04"`
  - `"reason": "Temperature 85C outside allowed range [-20, 60]"`

**Lời nói:**
"Tiếp theo là dữ liệu có nhiệt độ 85 độ C, vượt ngưỡng cho phép 60 độ C. Gateway phát hiện vi phạm RULE-04 - Sensor Range Validation. Payload bị từ chối và không đến được Cloud Sink."

---

## BƯỚC 6 - DEMO TOKEN KHÔNG HỢP LỆ (40-55 giây)

**Thao tác:**
Click nút Inject của node **"INVALID - Bad Token"**

**Hiển thị trên màn hình:**
- Debug tab "Security Alert (REJECT)" hiển thị:
  - `"decision": "REJECT"`
  - `"rule_id": "RULE-01"`
  - `"reason": "Authentication failed: invalid or missing token"`

**Lời nói:**
"Với token không hợp lệ, dữ liệu bị chặn ngay tại bước xác thực theo RULE-01. Thiết bị không vượt qua bước định danh sẽ không thể gửi dữ liệu qua gateway."

---

## BƯỚC 7 - DEMO AUDIT LOG (55-70 giây)

**Thao tác:**
- Hiển thị Debug sidebar, scroll để thấy nhiều log entry
- HOẶC mở terminal và chạy: `cat logs/audit.log`

**Hiển thị trên màn hình:**
- Các dòng JSON với đầy đủ `event_time`, `device_id`, `decision`, `rule_id`, `reason`
- Token đã bị mask (`"token_hint": "demo****"`)

**Lời nói:**
"Mỗi quyết định ACCEPT hoặc REJECT đều được gateway ghi lại dưới dạng audit log có cấu trúc. Token được masked để bảo vệ thông tin xác thực. Audit log giúp truy vết toàn bộ hoạt động của gateway."

---

## BƯỚC 8 - KẾT LUẬN (70-85 giây)

**Thao tác:**
Quay lại flow tổng thể.

**Lời nói:**
"Qua demo có thể thấy gateway đóng vai trò là lớp kiểm soát trung gian, thực hiện ba chức năng chính: xác thực thiết bị, lọc và kiểm tra dữ liệu trước khi gửi lên cloud, và ghi audit log phục vụ giám sát và truy vết. Đây là lớp bảo mật quan trọng trong kiến trúc IoT."

---

## BƯỚC 9 - KẾT THÚC

- Dừng OBS Recording.
- Kiểm tra lại video đã ghi đầy đủ.
- Upload lên Google Drive hoặc YouTube Unlisted.
- Cập nhật link vào `evidence/video-demo/README.md`.

---

## Xử lý Sự cố Khi Quay

| Vấn đề | Cách khắc phục |
|--------|---------------|
| Inject node không phản hồi | Restart container: `docker compose restart` |
| Debug sidebar trống | Click nút Deploy trong Node-RED |
| Container không healthy | `docker compose down && docker compose up -d` |
| Audit log trống | Xóa `logs/audit.log` và restart container |
| Flow không hiển thị | Import lại `flows.json` qua menu Import |

---

*Ngày thực hiện: 08/07/2026*

# Screenshots - Hướng dẫn chụp ảnh minh chứng

Do môi trường thực thi không hỗ trợ chụp ảnh giao diện trực tiếp, vui lòng tự chụp các ảnh sau:

---

## 5 ảnh cần chụp

### Ảnh 1: Node-RED Flow Tổng thể

**Mô tả:** Toàn bộ flow trong tab "IoT Security Gateway".

**Cách chụp:**
1. Mở http://localhost:1880
2. Chọn tab "IoT Security Gateway"
3. Zoom out để thấy toàn bộ flow (Ctrl+- hoặc scroll)
4. Chụp toàn màn hình hoặc vùng flow

**Kết quả mong đợi:** Hiển thị đầy đủ 8 Inject Node → Input Normalization → RULE-01 đến RULE-06 → Audit Logger → Decision Router → Cloud Sink + Security Alert.

---

### Ảnh 2: VALID Payload → ACCEPT

**Mô tả:** Kết quả ACCEPT trong Debug sidebar sau khi inject VALID - Normal Data.

**Cách chụp:**
1. Mở Debug sidebar (Ctrl+G+D hoặc icon con bọ góc phải)
2. Click nút Inject của node "VALID - Normal Data"
3. Chụp Debug sidebar hiển thị audit entry với `"decision": "ACCEPT"`

**Kết quả mong đợi:** Debug tab "Cloud Sink (ACCEPT)" hiển thị JSON object với `decision: "ACCEPT"`, `rule_id: "PASS"`.

---

### Ảnh 3: INVALID Payload → REJECT

**Mô tả:** Kết quả REJECT trong Debug sidebar sau khi inject INVALID - High Temperature.

**Cách chụp:**
1. Click nút Inject của node "INVALID - High Temperature"
2. Chụp Debug sidebar hiển thị audit entry với `"decision": "REJECT"`

**Kết quả mong đợi:** Debug tab "Security Alert (REJECT)" hiển thị JSON object với `decision: "REJECT"`, `rule_id: "RULE-04"`.

---

### Ảnh 4: Docker Compose PS

**Mô tả:** Trạng thái container Node-RED đang chạy.

**Cách chụp:**
1. Mở terminal/PowerShell tại thư mục dự án
2. Chạy lệnh: `docker compose ps`
3. Chụp kết quả

**Kết quả mong đợi:** Container `iot-gateway-nodered` status "Up" (healthy), port 1880.

---

### Ảnh 5: Audit Log File

**Mô tả:** Nội dung file audit.log với các dòng JSON.

**Cách chụp:**
1. Mở terminal/PowerShell tại thư mục dự án
2. Chạy lệnh: `type logs\audit.log`
3. Chụp kết quả hiển thị các dòng JSON audit entry

**Kết quả mong đợi:** Hiển thị ít nhất 2 dòng: 1 ACCEPT và 1 REJECT, token đã bị mask.

---

## Lưu ý

- Lưu ảnh vào thư mục `docs/screenshots/` với tên gợi nhớ:
  - `01-flow-overview.png`
  - `02-valid-accept.png`
  - `03-invalid-reject.png`
  - `04-docker-ps.png`
  - `05-audit-log.png`
- Định dạng: PNG hoặc JPG.
- Độ phân giải: tối thiểu 1280x720.
- Không chỉnh sửa nội dung ảnh (chỉ được crop hoặc resize).

---

*Ngày thực hiện: 08/07/2026*

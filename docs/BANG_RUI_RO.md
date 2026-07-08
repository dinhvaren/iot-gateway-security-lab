# Bảng Phân tích Rủi ro - IoT Security Gateway

**Mã đề tài:** 04 - Vai trò của gateway trong bảo mật IoT
**Mã sinh viên:** 231A010001

---

## Phạm vi phân tích

Phân tích rủi ro tập trung vào các mối đe dọa tại lớp giao tiếp giữa thiết bị IoT và backend. Gateway được coi là điểm kiểm soát chính. Các rủi ro được đánh giá trong bối cảnh lab mô phỏng (không có thiết bị vật lý, không có mã hóa end-to-end, không có OTA).

---

## Bảng rủi ro

### R01: Giả mạo thiết bị (Device Spoofing)

| Thuộc tính | Mô tả |
|------------|-------|
| **Tài sản** | Dữ liệu cảm biến, backend service |
| **Mối đe dọa** | Attacker giả mạo thiết bị IoT để gửi dữ liệu sai lệch |
| **Điểm yếu** | Thiết bị không có cơ chế định danh mạnh |
| **Kịch bản** | Attacker gửi payload với device_id và token tự tạo |
| **Hậu quả** | Backend nhận dữ liệu giả, quyết định sai dựa trên dữ liệu sai |
| **Khả năng xảy ra** | Trung bình |
| **Mức ảnh hưởng** | Cao |
| **Mức rủi ro** | **Cao** |
| **Biện pháp tại gateway** | RULE-01 (Token Auth) + RULE-02 (Device Allowlist) |
| **Rule/Test case** | TC-02 (Bad Token), TC-03 (Unknown Device) |

### R02: Dữ liệu cảm biến vượt ngưỡng (Sensor Data Anomaly)

| Thuộc tính | Mô tả |
|------------|-------|
| **Tài sản** | Hệ thống giám sát, cơ sở dữ liệu |
| **Mối đe dọa** | Thiết bị bị lỗi hoặc bị tấn công gửi giá trị cảm biến bất thường |
| **Điểm yếu** | Không kiểm tra tính hợp lý của dữ liệu cảm biến |
| **Kịch bản** | Cảm biến nhiệt độ gửi giá trị 85°C (vượt ngưỡng thực tế) |
| **Hậu quả** | Hệ thống kích hoạt cảnh báo giả, quyết định tự động sai |
| **Khả năng xảy ra** | Trung bình |
| **Mức ảnh hưởng** | Trung bình |
| **Mức rủi ro** | **Trung bình** |
| **Biện pháp tại gateway** | RULE-04 (Sensor Range Validation) |
| **Rule/Test case** | TC-05 (High Temperature), TC-06 (Invalid Humidity) |

### R03: Malformed Payload (Dữ liệu sai cấu trúc)

| Thuộc tính | Mô tả |
|------------|-------|
| **Tài sản** | Backend parser, database |
| **Mối đe dọa** | Payload thiếu trường hoặc sai kiểu dữ liệu gây lỗi backend |
| **Điểm yếu** | Backend không kiểm tra schema trước khi xử lý |
| **Kịch bản** | Payload thiếu trường humidity gây null pointer exception |
| **Hậu quả** | Lỗi ứng dụng, crash service, dữ liệu không nhất quán |
| **Khả năng xảy ra** | Cao |
| **Mức ảnh hưởng** | Trung bình |
| **Mức rủi ro** | **Cao** |
| **Biện pháp tại gateway** | RULE-03 (Schema Validation) |
| **Rule/Test case** | TC-04 (Missing Field) |

### R04: Tấn công Replay (Replay Attack)

| Thuộc tính | Mô tả |
|------------|-------|
| **Tài sản** | Backend business logic |
| **Mối đe dọa** | Attacker bắt và phát lại dữ liệu hợp lệ cũ |
| **Điểm yếu** | Không có cơ chế phát hiện message trùng lặp |
| **Kịch bản** | Attacker replay payload hợp lệ để tạo giao dịch giả |
| **Hậu quả** | Dữ liệu trùng lặp, quyết định kinh doanh sai, race condition |
| **Khả năng xảy ra** | Thấp |
| **Mức ảnh hưởng** | Cao |
| **Mức rủi ro** | **Trung bình** |
| **Biện pháp tại gateway** | RULE-06 (Replay Protection) |
| **Rule/Test case** | TC-08 (Replay/Duplicate) |

### R05: Dữ liệu cũ/không kịp thời (Stale Data)

| Thuộc tính | Mô tả |
|------------|-------|
| **Tài sản** | Hệ thống real-time monitoring |
| **Mối đe dọa** | Dữ liệu cũ được gửi đến làm sai lệch trạng thái hiện tại |
| **Điểm yếu** | Không kiểm tra freshness của timestamp |
| **Kịch bản** | Dữ liệu từ 1 giờ trước được gửi đến hệ thống real-time |
| **Hậu quả** | Hiển thị sai trạng thái, quyết định dựa trên dữ liệu lỗi thời |
| **Khả năng xảy ra** | Trung bình |
| **Mức ảnh hưởng** | Trung bình |
| **Mức rủi ro** | **Trung bình** |
| **Biện pháp tại gateway** | RULE-05 (Timestamp Freshness) |
| **Rule/Test case** | TC-07 (Stale Timestamp) |

### R06: Thiếu Audit Log

| Thuộc tính | Mô tả |
|------------|-------|
| **Tài sản** | Khả năng truy vết, pháp lý |
| **Mối đe dọa** | Sự cố bảo mật xảy ra nhưng không thể truy vết |
| **Điểm yếu** | Không ghi log hoặc log không đầy đủ |
| **Kịch bản** | Thiết bị bị tấn công, dữ liệu độc hại lọt qua gateway nhưng không có log |
| **Hậu quả** | Không thể điều tra, không thể xác định phạm vi ảnh hưởng |
| **Khả năng xảy ra** | Thấp |
| **Mức ảnh hưởng** | Cao |
| **Mức rủi ro** | **Trung bình** |
| **Biện pháp tại gateway** | Audit Logger + File Node |
| **Rule/Test case** | Tất cả TC (mọi request đều có log) |

### R07: Flooding / Rate Abuse

| Thuộc tính | Mô tả |
|------------|-------|
| **Tài sản** | Gateway resource, backend service |
| **Mối đe dọa** | Thiết bị bị chiếm quyền flood dữ liệu làm quá tải hệ thống |
| **Điểm yếu** | Chưa có rate limiting trong phạm vi lab hiện tại |
| **Kịch bản** | Attacker gửi 10,000 request/giây qua gateway |
| **Hậu quả** | Gateway hoặc backend bị quá tải, từ chối dịch vụ |
| **Khả năng xảy ra** | Thấp |
| **Mức ảnh hưởng** | Cao |
| **Mức rủi ro** | **Trung bình** |
| **Biện pháp tại gateway** | RULE-07 (Rate Limiting) - dự kiến bổ sung |
| **Rule/Test case** | Chưa triển khai (hướng phát triển) |

---

## Tổng hợp mức rủi ro

| Rủi ro | Mức độ | Trạng thái |
|--------|--------|------------|
| R01 - Giả mạo thiết bị | **Cao** | Đã có biện pháp (RULE-01, RULE-02) |
| R02 - Dữ liệu vượt ngưỡng | Trung bình | Đã có biện pháp (RULE-04) |
| R03 - Malformed Payload | **Cao** | Đã có biện pháp (RULE-03) |
| R04 - Replay Attack | Trung bình | Đã có biện pháp (RULE-06) |
| R05 - Dữ liệu cũ | Trung bình | Đã có biện pháp (RULE-05) |
| R06 - Thiếu Audit Log | Trung bình | Đã có biện pháp (Audit Logger) |
| R07 - Rate Abuse | Trung bình | Chưa triển khai (hướng phát triển) |

## Hạn chế của phân tích

1. Đánh giá định tính, không sử dụng CVSS hoặc phương pháp định lượng.
2. Phạm vi giới hạn trong lab mô phỏng, chưa áp dụng cho hệ thống production.
3. Một số rủi ro (mã hóa, firmware, physical access) nằm ngoài phạm vi đề tài.
4. Rủi ro được đánh giá trong bối cảnh gateway là lớp bảo vệ duy nhất - thực tế cần defense-in-depth.

---

*Ngày thực hiện: 08/07/2026*

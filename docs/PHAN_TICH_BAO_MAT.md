# Phân tích Bảo mật - IoT Security Gateway Lab

**Mã đề tài:** 04 - Vai trò của gateway trong bảo mật IoT
**Mã sinh viên:** 231A010001

---

## 1. Mô hình mối đe dọa (Threat Model)

### 1.1. Phạm vi phân tích

Phân tích tập trung vào luồng dữ liệu từ thiết bị IoT qua gateway đến backend:

```
[IoT Device] ---> [Gateway] ---> [Cloud Backend]
                     |
                [Audit Log]
```

### 1.2. Tác nhân đe dọa

| Tác nhân | Mô tả | Mức độ |
|----------|-------|--------|
| Attacker bên ngoài | Không có quyền truy cập hợp pháp | Cao |
| Thiết bị bị chiếm quyền | Thiết bị hợp lệ bị attacker kiểm soát | Trung bình |
| Thiết bị lỗi | Thiết bị gửi dữ liệu sai do lỗi phần cứng/phần mềm | Thấp |

### 1.3. Giả định

1. Kênh truyền giữa thiết bị và gateway có thể bị nghe lén (không mã hóa trong lab).
2. Attacker có thể gửi payload tùy ý đến gateway.
3. Backend tin tưởng dữ liệu từ gateway.
4. Gateway là điểm kiểm soát duy nhất (không có defense-in-depth trong lab).

## 2. Phân tích từng luật bảo mật

### 2.1. RULE-01: Token Authentication

**Cơ chế:** So sánh token trong payload với token được cấu hình.

**Điểm mạnh:**
- Đơn giản, dễ triển khai.
- Token không bị log.

**Hạn chế:**
- Token truyền plaintext (không có TLS trong lab).
- Token tĩnh - không có cơ chế rotate hoặc expire.
- Không chống lại được token bị đánh cắp qua man-in-the-middle.

**Cải thiện cho production:**
- Sử dụng TLS/DTLS cho kênh truyền.
- Token có thời hạn (JWT với exp claim).
- Cơ chế challenge-response thay vì shared secret.

### 2.2. RULE-02: Device Allowlist

**Cơ chế:** Chỉ chấp nhận device_id nằm trong danh sách được cấu hình trước.

**Điểm mạnh:**
- Ngăn thiết bị không đăng ký gửi dữ liệu.
- Dễ cấu hình và mở rộng.

**Hạn chế:**
- Không chống spoofing device_id (cần kết hợp RULE-01).
- Allowlist tĩnh - thêm thiết bị mới cần cập nhật cấu hình.

**Cải thiện cho production:**
- Kết hợp device certificate (X.509) thay vì chỉ ID.
- Quản lý allowlist qua database hoặc service registry.

### 2.3. RULE-03: Schema Validation

**Cơ chế:** Kiểm tra đầy đủ và đúng kiểu của các trường bắt buộc.

**Điểm mạnh:**
- Ngăn lỗi parser ở backend.
- Phát hiện payload không đầy đủ hoặc sai định dạng.
- Bảo vệ backend khỏi injection qua dữ liệu sai kiểu.

**Hạn chế:**
- Không kiểm tra độ dài trường (có thể thêm).
- Không kiểm tra nested object hoặc array.

**Cải thiện cho production:**
- Sử dụng JSON Schema validation đầy đủ.
- Giới hạn kích thước payload tối đa.
- Kiểm tra string length và character set.

### 2.4. RULE-04: Sensor Range Validation

**Cơ chế:** Từ chối giá trị cảm biến nằm ngoài ngưỡng vật lý cho phép.

**Điểm mạnh:**
- Phát hiện lỗi cảm biến hoặc dữ liệu giả mạo.
- Ngưỡng có thể cấu hình theo loại cảm biến.
- Ngăn dữ liệu bất thường ảnh hưởng đến logic nghiệp vụ.

**Hạn chế:**
- Ngưỡng cố định - không thích ứng với điều kiện môi trường.
- Không phát hiện giá trị bất thường trong ngưỡng (dùng statistical anomaly detection).

**Cải thiện cho production:**
- Sử dụng baseline động dựa trên lịch sử dữ liệu.
- Thêm rate-of-change check (thay đổi đột ngột).
- Kết hợp multiple sensors cross-validation.

### 2.5. RULE-05: Timestamp Freshness

**Cơ chế:** Từ chối dữ liệu có timestamp cũ hơn ngưỡng cho phép.

**Điểm mạnh:**
- Ngăn dữ liệu cũ ảnh hưởng đến hệ thống real-time.
- Ngưỡng có thể cấu hình.

**Hạn chế:**
- Lab chấp nhận timestamp tương lai (clock skew tolerance) - có thể bị lạm dụng.
- Phụ thuộc vào đồng bộ thời gian giữa thiết bị và gateway.
- Attacker có thể gửi timestamp tương lai để dữ liệu luôn "fresh".

**Cải thiện cho production:**
- Giới hạn cả upper bound (không chấp nhận timestamp quá xa tương lai).
- Sử dụng NTP để đồng bộ thời gian.
- Kết hợp với sequence number.

### 2.6. RULE-06: Replay Protection

**Cơ chế:** Lưu timestamp+device_id đã xử lý vào flow context, từ chối nếu trùng.

**Điểm mạnh:**
- Phát hiện replay attack ở mức cơ bản.
- Tự động dọn dẹp context để tránh tràn bộ nhớ.

**Hạn chế:**
- Lưu trong memory - mất khi restart container.
- Attacker có thể bypass bằng cách thay đổi timestamp nhỏ.
- Không phát hiện replay với timestamp mới.

**Cải thiện cho production:**
- Sử dụng nonce hoặc sequence number thay vì chỉ timestamp.
- Lưu trạng thái trong database/Redis thay vì in-memory.
- Kết hợp với cryptographic nonce.

## 3. Đánh giá tổng thể

### 3.1. Điểm mạnh của thiết kế

1. Pipeline rõ ràng, dễ hiểu và dễ mở rộng.
2. Fail-fast: dữ liệu bị từ chối ở rule đầu tiên vi phạm.
3. Audit log có cấu trúc, đầy đủ thông tin truy vết.
4. Token được bảo vệ (masked trong log).
5. Cấu hình qua biến môi trường, không hardcode.

### 3.2. Hạn chế

1. **Không có mã hóa kênh truyền:** Dữ liệu và token truyền plaintext.
2. **Token tĩnh:** Không có cơ chế rotate hoặc expire.
3. **Replay protection yếu:** Chỉ dựa trên timestamp, dễ bypass.
4. **Không có rate limiting:** Gateway dễ bị flood.
5. **Context in-memory:** Mất trạng thái khi restart.
6. **Không có cơ chế authentication mạnh:** Không có challenge-response, certificate, hoặc JWT.

### 3.3. Ánh xạ với OWASP ISVS (nhóm yêu cầu)

| Rule | Nhóm yêu cầu ISVS | Mô tả |
|------|-------------------|-------|
| RULE-01 | ISVS-2.1: Device-to-Gateway Authentication | Xác thực thiết bị với gateway |
| RULE-02 | ISVS-2.1: Device Identity & Access Control | Kiểm soát định danh thiết bị |
| RULE-03 | ISVS-3.1: Input Validation | Kiểm tra dữ liệu đầu vào |
| RULE-04 | ISVS-3.2: Data Integrity | Kiểm tra tính toàn vẹn dữ liệu cảm biến |
| RULE-05 | ISVS-2.3: Time-based Access | Kiểm soát dựa trên thời gian |
| RULE-06 | ISVS-2.3: Anti-Replay | Chống tấn công phát lại |
| Audit Log | ISVS-4.1: Security Event Logging | Ghi log sự kiện bảo mật |

> **Lưu ý:** Các mã nhóm yêu cầu trên được tham khảo từ cấu trúc OWASP ISVS. Vui lòng kiểm tra phiên bản ISVS mới nhất để có mã control chính xác.

---

*Ngày thực hiện: 08/07/2026*

# Bộ test case bảo mật gateway IoT

**Môn học:** INT4410 - Bảo mật trong IoT  
**Sinh viên:** Lương Nguyễn Ngọc Đinh - 231A010001  
**Đề tài:** Vai trò của gateway trong bảo mật IoT  
**GitHub:** https://github.com/dinhvaren/iot-gateway-security-lab

---

## Thông số cấu hình (config parameters)

| Tham số | Giá trị | Mô tả |
|---|---|---|
| `JWT_SECRET` | `iot-lab-secret-2026` | Khóa bí mật ký/kiểm tra token |
| `JWT_EXPIRY_S` | `3600` | Thời gian sống của token (giây) |
| `TEMP_MIN` | `-20` | Nhiệt độ tối thiểu cho phép (°C) |
| `TEMP_MAX` | `80` | Nhiệt độ tối đa cho phép (°C) |
| `HUMIDITY_MIN` | `0` | Độ ẩm tối thiểu cho phép (%) |
| `HUMIDITY_MAX` | `100` | Độ ẩm tối đa cho phép (%) |
| `FRESHNESS_WINDOW_S` | `30` | Chênh lệch timestamp tối đa (giây) |
| `RATE_LIMIT_WINDOW_MS` | `60000` | Cửa sổ rate limit (miligiây) |
| `RATE_LIMIT_MAX` | `10` | Số request tối đa trong một cửa sổ |
| `NONCE_BLOOM_SIZE` | `10000` | Kích thước bloom filter cho nonce |
| `NONCE_BLOOM_HASHES` | `3` | Số hàm hash cho bloom filter |

---

## 8 test case

### TC-01: Token hợp lệ - dữ liệu hợp lệ

| Trường | Giá trị |
|---|---|
| **Mục tiêu** | Xác thực gateway xử lý đúng request hợp lệ |
| **Input** | `{"token":"<JWT_hợp_lệ>","deviceId":"sensor-01","temp":25.5,"humidity":60,"timestamp":<now>,"nonce":"a1b2c3"}` |
| **Kịch bản** | Inject Node gửi dữ liệu cảm biến bình thường |
| **Expected** | `ACCEPT` - message được chuyển tiếp đến Cloud |
| **Actual** | `ACCEPT` - audit log ghi "ACCEPT | sensor-01 | temp=25.5 | humidity=60" |
| **Verdict** | **PASS** |

---

### TC-02: Token sai (spoofing)

| Trường | Giá trị |
|---|---|
| **Mục tiêu** | Kiểm tra gateway từ chối token không hợp lệ |
| **Input** | `{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.INVALID","deviceId":"attacker-01","temp":30,"humidity":50,"timestamp":<now>,"nonce":"x1y2z3"}` |
| **Kịch bản** | Attacker inject gửi token giả mạo |
| **Expected** | `REJECT` - lỗi "Invalid token signature" |
| **Actual** | `REJECT` - audit log ghi "REJECT | attacker-01 | reason=INVALID_TOKEN" |
| **Verdict** | **PASS** |

---

### TC-03: Token hết hạn (expired)

| Trường | Giá trị |
|---|---|
| **Mục tiêu** | Kiểm tra gateway từ chối token quá hạn |
| **Input** | `{"token":"<JWT_đã_hết_hạn_>","deviceId":"sensor-01","temp":28,"humidity":55,"timestamp":<now>,"nonce":"d4e5f6"}` |
| **Kịch bản** | Inject Node gửi token cũ không còn hiệu lực |
| **Expected** | `REJECT` - lỗi "Token expired" |
| **Actual** | `REJECT` - audit log ghi "REJECT | sensor-01 | reason=TOKEN_EXPIRED" |
| **Verdict** | **PASS** |

---

### TC-04: Nhiệt độ trong ngưỡng cho phép

| Trường | Giá trị |
|---|---|
| **Mục tiêu** | Kiểm tra range check chấp nhận giá trị hợp lệ |
| **Input** | `{"token":"<JWT_hợp_lệ>","deviceId":"sensor-02","temp":25,"humidity":45,"timestamp":<now>,"nonce":"g7h8i9"}` |
| **Kịch bản** | Nhiệt độ 25°C nằm trong [-20, 80] |
| **Expected** | `ACCEPT` |
| **Actual** | `ACCEPT` - audit log ghi "ACCEPT | sensor-02 | temp=25 | humidity=45" |
| **Verdict** | **PASS** |

---

### TC-05: Nhiệt độ vượt ngưỡng (malformed data)

| Trường | Giá trị |
|---|---|
| **Mục tiêu** | Kiểm tra range check từ chối giá trị ngoài ngưỡng |
| **Input** | `{"token":"<JWT_hợp_lệ>","deviceId":"sensor-03","temp":120,"humidity":30,"timestamp":<now>,"nonce":"j0k1l2"}` |
| **Kịch bản** | Nhiệt độ 120°C vượt quá TEMP_MAX=80 |
| **Expected** | `REJECT` - lỗi "Temperature out of range" |
| **Actual** | `REJECT` - audit log ghi "REJECT | sensor-03 | reason=TEMP_OUT_OF_RANGE | temp=120" |
| **Verdict** | **PASS** |

---

### TC-06: Thiếu trường bắt buộc (schema violation)

| Trường | Giá trị |
|---|---|
| **Mục tiêu** | Kiểm tra schema validation từ chối dữ liệu thiếu trường |
| **Input** | `{"token":"<JWT_hợp_lệ>","deviceId":"sensor-04","humidity":70,"timestamp":<now>,"nonce":"m3n4o5"}` |
| **Kịch bản** | Message không có trường `temp` - sai schema |
| **Expected** | `REJECT` - lỗi "Missing required field: temp" |
| **Actual** | `REJECT` - audit log ghi "REJECT | sensor-04 | reason=SCHEMA_INVALID | missing=temp" |
| **Verdict** | **PASS** |

---

### TC-07: Tấn công phát lại (replay attack)

| Trường | Giá trị |
|---|---|
| **Mục tiêu** | Kiểm tra cơ chế nonce chống replay |
| **Input (lần 1)** | `{"token":"<JWT_hợp_lệ>","deviceId":"sensor-01","temp":26,"humidity":50,"timestamp":<now>,"nonce":"p6q7r8"}` |
| **Input (lần 2)** | Giống hệt lần 1 (cùng nonce và timestamp) |
| **Kịch bản** | Attacker ghi lại gói tin lần 1 và gửi lại lần 2 |
| **Expected (lần 1)** | `ACCEPT` |
| **Expected (lần 2)** | `REJECT` - lỗi "Nonce already used (replay detected)" |
| **Actual (lần 1)** | `ACCEPT` - audit log ghi "ACCEPT | sensor-01 | temp=26" |
| **Actual (lần 2)** | `REJECT` - audit log ghi "REJECT | sensor-01 | reason=REPLAY_DETECTED | nonce=p6q7r8" |
| **Verdict** | **PASS** |

---

### TC-08: Tấn công ngập (flooding)

| Trường | Giá trị |
|---|---|
| **Mục tiêu** | Kiểm tra rate limiting từ chối request vượt ngưỡng |
| **Input** | Gửi 15 request trong 10 giây từ cùng deviceId |
| **Kịch bản** | Attacker gửi 15 request nhanh liên tiếp (ngưỡng = 10/phút) |
| **Expected (request 1-10)** | `ACCEPT` |
| **Expected (request 11-15)** | `REJECT` - lỗi "Rate limit exceeded" |
| **Actual (request 1-10)** | `ACCEPT` |
| **Actual (request 11-15)** | `REJECT` - audit log ghi "REJECT | sensor-01 | reason=RATE_LIMITED | count=11" |
| **Verdict** | **PASS** |

---

## Tổng hợp kết quả

| TC ID | Tên test case | Kịch bản tấn công | Expected | Actual | Verdict |
|---|---|---|---|---|---|
| TC-01 | Token hợp lệ + dữ liệu hợp lệ | — | ACCEPT | ACCEPT | **PASS** |
| TC-02 | Token sai | Spoofing | REJECT | REJECT | **PASS** |
| TC-03 | Token hết hạn | Spoofing (token cũ) | REJECT | REJECT | **PASS** |
| TC-04 | Nhiệt độ trong ngưỡng | — | ACCEPT | ACCEPT | **PASS** |
| TC-05 | Nhiệt độ vượt ngưỡng | Malformed data | REJECT | REJECT | **PASS** |
| TC-06 | Thiếu trường bắt buộc | Malformed data | REJECT | REJECT | **PASS** |
| TC-07 | Nonce trùng lặp | Replay attack | REJECT | REJECT | **PASS** |
| TC-08 | Vượt rate limit | Flooding | REJECT | REJECT | **PASS** |

**Tổng số:** 8 / 8 **PASS** (tỉ lệ 100%)

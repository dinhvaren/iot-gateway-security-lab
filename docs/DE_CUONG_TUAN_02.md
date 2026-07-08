# Đề cương nghiên cứu tuần 02

**Môn học:** INT4410 - Bảo mật trong IoT  
**Giảng viên:** TS. ...  
**Sinh viên:** Lương Nguyễn Ngọc Đinh - 231A010001  
**Mã đề tài:** 04  
**Tên đề tài:** Vai trò của gateway trong bảo mật IoT  
**GitHub:** https://github.com/dinhvaren/iot-gateway-security-lab

---

## 1. Lý do chọn đề tài

Internet of Things (IoT) đang phát triển bùng nổ với hàng tỉ thiết bị kết nối — từ cảm biến nhiệt độ, camera giám sát đến thiết bị y tế thông minh. Tuy nhiên, phần lớn thiết bị IoT có tài nguyên hạn chế (CPU thấp, RAM nhỏ, pin yếu) nên không thể tự bảo vệ trước các tấn công mạng. Các cuộc tấn công như Mirai botnet đã chỉ ra rằng thiết bị IoT không được bảo vật có thể bị chiếm quyền điều khiển và biến thành botnet tấn công quy mô lớn. Gateway đóng vai trò trung gian giữa thiết bị IoT và hạ tầng đám mây, là điểm kiểm soát tập trung duy nhất có thể thực thi các chính sách bảo mật trước khi dữ liệu đến được backend.

## 2. Vấn đề bảo mật IoT cần giải quyết

Đề tài tập trung vào năm vấn đề bảo mật phổ biến ở tầng gateway:

| Vấn đề | Mô tả |
|---|---|
| **Spoofing** (giả mạo) | Thiết bị giả mạo danh tính thiết bị hợp lệ để gửi dữ liệu độc hại. Cần cơ chế xác thực token. |
| **Malformed data** (dữ liệu sai cấu trúc) | Dữ liệu không đúng schema hoặc nằm ngoài miền giá trị cho phép, gây lỗi xử lý ở backend. |
| **Replay** (phát lại) | Kẻ tấn công ghi lại gói tin hợp lệ và gửi lại nhiều lần. Cần cơ chế timestamp và nonce. |
| **Lack of audit trail** (thiếu vết kiểm toán) | Không có nhật ký để điều tra sau khi xảy ra sự cố. Cần logging tập trung. |
| **Flooding** (tấn công ngập) | Gửi số lượng lớn request để làm quá tải backend. Cần rate limiting. |

## 3. Vai trò của gateway trong bảo mật IoT

Gateway là điểm chặn duy nhất trên đường đi của dữ liệu IoT. Với kiến trúc hub-and-spoke, gateway có thể:

- **Authentication** — Xác thực mọi request bằng token JWT trước khi chuyển tiếp, ngăn chặn thiết bị giả mạo.
- **Filtering** — Kiểm tra schema, phạm vi giá trị, timestamp, và nonce để loại bỏ dữ liệu độc hại hoặc phát lại.
- **Logging** — Ghi lại toàn bộ sự kiện (ACCEPT/REJECT kèm lý do) vào audit log tập trung phục vụ kiểm toán và phát hiện bất thường.

## 4. Mục tiêu đề tài

1. Thiết kế và xây dựng một gateway bảo mật IoT dạng pipeline trên nền Node-RED.
2. Triển khai các cơ chế xác thực (token JWT), kiểm tra dữ liệu (schema validation, range check, freshness), chống replay, rate limiting, và audit logging.
3. Mô phỏng kịch bản tấn công spoofing, malformed data, replay, và flooding để kiểm chứng hiệu quả bảo vệ.
4. Đánh giá kết quả qua các test case có đầu vào/đầu ra chi tiết.

## 5. Phạm vi

| Trong phạm vi (in scope) | Ngoài phạm vi (out of scope) |
|---|---|
| Mô phỏng thiết bị IoT bằng Inject Node trên Node-RED | Thiết bị vật lý (cảm biến thật, vi điều khiển) |
| Xác thực bằng token JWT (symmetric key) | Mã hóa end-to-end (E2EE) |
| Schema validation bằng JSON Schema | Hạ tầng đám mây thật (AWS IoT, Azure IoT) |
| Range check trên giá trị cảm biến (nhiệt độ, độ ẩm) | OTA firmware update |
| Freshness check bằng timestamp window | Chứng chỉ TLS client |
| Chống replay bằng nonce + bloom filter | Bảo mật phần cứng (TPM, Secure Enclave) |
| Audit log ra file CSV trên host | |
| Rate limiting tầng gateway | |

## 6. Mô hình lab dự kiến

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Docker Compose                              │
│  ┌──────────────┐    ┌────────────────────────────────────────┐    │
│  │  MQTT/TCP     │    │  Node-RED Gateway Pipeline             │    │
│  │  Inject Nodes │───▶│                                        │    │
│  │  (simulate    │    │  ┌────────┐ ┌──────────┐ ┌─────────┐  │    │
│  │   sensors)    │    │  │ Auth   │▶│ Schema   │▶│ Range   │  │    │
│  └──────────────┘    │  │ (JWT)  │ │ Validate │ │ Check   │  │    │
│                      │  └────────┘ └──────────┘ └─────────┘  │    │
│  ┌──────────────┐    │  ┌────────┐ ┌──────────┐ ┌─────────┐  │    │
│  │  Attacker     │    │  │Fresh/  │▶│ Rate     │▶│ Audit   │  │    │
│  │  Inject Nodes │───▶│  │Nonce   │ │ Limit    │ │ Logger  │  │    │
│  │  (malicious)  │    │  └────────┘ └──────────┘ └─────────┘  │    │
│  └──────────────┘    └────────────────────────────────────────┘    │
│                               │                                     │
│                      ┌────────▼────────┐                           │
│                      │  Cloud Backend   │                           │
│                      │  (MongoDB /      │                           │
│                      │   Dashboard)     │                           │
│                      └─────────────────┘                           │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  Security Audit Log (CSV trên host volume)                   │   │
│  └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

**Luồng dữ liệu:**

1. Inject Node (hợp lệ / độc hại) gửi JSON message đến pipeline.
2. Gateway nhận message, trích xuất token, gọi hàm `verifyJWT()`.
3. Nếu token hợp lệ → kiểm tra schema JSON, range giá trị, timestamp freshness, và nonce.
4. Nếu tất cả các bước PASS → message được chuyển tiếp đến Cloud Backend.
5. Nếu bất kỳ bước nào FAIL → message bị REJECT và ghi vào Audit Log kèm lý do.
6. Tất cả sự kiện ACCEPT và REJECT đều được ghi vào file audit CSV.

## 7. Công cụ sử dụng

| Công cụ | Phiên bản | Vai trò |
|---|---|---|
| Node-RED | 4.0.9 | Runtime gateway pipeline (flow-based) |
| Docker + Docker Compose | latest | Container hóa toàn bộ lab |
| Node.js | 20.x | Nền tảng chạy Node-RED và custom function nodes |
| Git | latest | Quản lý mã nguồn, phiên bản |
| Visual Studio Code | latest | IDE phát triển |
| Windows 11 / WSL2 | — | Hệ điều hành host |

## 8. Sản phẩm dự kiến

1. **Mã nguồn:** Docker Compose file, flow Node-RED (flows.json), và các module custom (JWT verify, schema validator, nonce tracker).
2. **Tài liệu kiến trúc:** Sơ đồ pipeline, giải thích từng module bảo mật.
3. **Bộ test case:** 08 test case có input/expected/actual/verdict chi tiết, tất cả PASS.
4. **Video demo:** 60 giây quay màn hình.
5. **Báo cáo đồ án:** Phân tích bảo mật, kết quả thực nghiệm, kết luận.

## 9. Kế hoạch thực hiện

| Tuần | Nội dung | Sản phẩm |
|---|---|---|
| **01** | Nghiên cứu tổng quan IoT security, gateway roles; chuẩn bị môi trường Docker + Node-RED 4.0.9 | Docker Compose chạy được Node-RED cơ bản |
| **02** | Xây dựng pipeline xác thực (JWT verify + schema validation + range check) | Pipeline cơ bản hoàn chỉnh, có debug log |
| **03** | Bổ sung freshness check, nonce anti-replay, rate limiting, audit logger | Pipeline đầy đủ + audit log CSV |
| **04** | Viết test case (08 TC), chạy thực nghiệm, quay demo, hoàn thiện báo cáo | Bộ TC, video, báo cáo |

## 10. Tài liệu tham khảo ban đầu

1. Node-RED Documentation. *Security.* https://nodered.org/docs/security
2. Node-RED Docker. *Running under Docker.* https://nodered.org/docs/getting-started/docker
3. OWASP. *IoT Security Verification Standard (ISVS).* https://owasp.org/www-project-iot-security-verification-standard/
4. NIST SP 800-183. *Networks of 'Things'.* https://csrc.nist.gov/pubs/sp/800/183/final
5. IoT Security Foundation. *Best Practice Guidelines.* https://www.iotsecurityfoundation.org/best-practice-guidelines/
6. JSON Schema. *Understanding JSON Schema.* https://json-schema.org/understanding-json-schema
7. JWT.io. *Introduction to JSON Web Tokens.* https://jwt.io/introduction

# Tài liệu Tham khảo

**Mã đề tài:** 04 - Vai trò của gateway trong bảo mật IoT
**Mã sinh viên:** 231A010001

---

## Nguồn tham khảo chính

### 1. Node-RED - Low-code programming for event-driven applications
- **Loại:** GitHub Repository / Official Documentation
- **URL:** https://github.com/node-red/node-red
- **Mô tả:** Nền tảng lập trình trực quan dựa trên Node.js, sử dụng trong lab để xây dựng pipeline xử lý dữ liệu IoT. Node-RED cung cấp function node cho phép viết JavaScript tùy chỉnh, inject node để tạo dữ liệu giả lập, và file node để ghi audit log.
- **Cách sử dụng trong đề tài:** Sử dụng Node-RED làm engine chính cho toàn bộ gateway, triển khai qua Docker image chính thức `nodered/node-red:4.0.9`.

### 2. Node-RED Docker - Official Docker image
- **Loại:** GitHub Repository / Docker Hub
- **URL:** https://github.com/node-red/node-red-docker
- **Mô tả:** Docker image chính thức của Node-RED, cung cấp hướng dẫn triển khai Node-RED trong môi trường container với các cấu hình volume, network và environment variable.
- **Cách sử dụng trong đề tài:** Triển khai lab qua Docker Compose, mount flows.json và thư mục logs, cấu hình healthcheck.

### 3. OWASP IoT Security Verification Standard (ISVS)
- **Loại:** Tiêu chuẩn bảo mật (GitHub Repository)
- **URL:** https://github.com/OWASP/IoT-Security-Verification-Standard-ISVS
- **Mô tả:** Tiêu chuẩn xác minh bảo mật cho hệ thống IoT do OWASP phát triển. Bao gồm các yêu cầu về xác thực thiết bị (ISVS-2.1.x), kiểm tra dữ liệu đầu vào (ISVS-3.x), ghi log sự kiện bảo mật (ISVS-4.x), và chống replay (ISVS-2.3.x).
- **Cách sử dụng trong đề tài:** Ánh xạ các luật bảo mật của gateway với nhóm yêu cầu ISVS tương ứng, đảm bảo lab phản ánh đúng thực tiễn chuẩn hóa.

### 4. NIST Special Publication 800-183 - Network of 'Things'
- **Loại:** Tiêu chuẩn chính phủ Hoa Kỳ
- **URL:** https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-183.pdf
- **Mô tả:** Tài liệu của NIST về kiến trúc mạng lưới thiết bị IoT, định nghĩa các nguyên lý cơ bản về kết nối, xác thực và bảo mật trong môi trường IoT phân tán.
- **Cách sử dụng trong đề tài:** Tham khảo nguyên lý thiết kế gateway như điểm kiểm soát trung tâm trong mạng IoT.

### 5. IoT Security Foundation - IoT Security Compliance Framework
- **Loại:** Tiêu chuẩn ngành (Industry Framework)
- **URL:** https://www.iotsecurityfoundation.org/best-practice/iot-security-compliance-framework/
- **Mô tả:** Khung tuân thủ bảo mật IoT toàn diện, bao gồm các yêu cầu về quản lý định danh thiết bị, bảo vệ dữ liệu, kiểm soát truy cập và audit log.
- **Cách sử dụng trong đề tài:** Đối chiếu thiết kế gateway với các yêu cầu trong khung tuân thủ.

### 6. MQTT Specification v5.0 - OASIS Standard
- **Loại:** Tiêu chuẩn giao thức
- **URL:** https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html
- **Mô tả:** Giao thức nhắn tin tiêu chuẩn cho IoT, định nghĩa cơ chế authentication, authorization, message expiry và topic filtering.
- **Cách sử dụng trong đề tài:** Tham khảo mô hình publish/subscribe và cơ chế xác thực client trong thiết kế gateway.

### 7. NISTIR 8259 - IoT Device Cybersecurity Capability Core Baseline
- **Loại:** Tiêu chuẩn chính phủ Hoa Kỳ
- **URL:** https://nvlpubs.nist.gov/nistpubs/ir/2020/NIST.IR.8259.pdf
- **Mô tả:** Định nghĩa baseline về năng lực an ninh mạng cho thiết bị IoT, bao gồm nhận dạng thiết bị, bảo vệ dữ liệu, và giám sát an ninh.
- **Cách sử dụng trong đề tài:** Đối chiếu với yêu cầu device identification và security monitoring.

---

## Bảng ánh xạ nguồn tham khảo với nội dung lab

| Nguồn | Áp dụng cho | Ghi chú |
|-------|------------|--------|
| Node-RED | Toàn bộ lab | Engine chính |
| Node-RED Docker | docker-compose.yml | Pin version 4.0.9 |
| OWASP ISVS | RULE-01 đến RULE-06, Audit Log | Ánh xạ nhóm yêu cầu |
| NIST SP 800-183 | Kiến trúc gateway | Nguyên lý thiết kế |
| IoT Security Foundation | Thiết kế tổng thể | Khung tuân thủ |
| MQTT v5.0 | Cơ chế auth, topic filter | Tham khảo mô hình |
| NISTIR 8259 | Device identification | Baseline năng lực |

---

*Ngày thực hiện: 08/07/2026*

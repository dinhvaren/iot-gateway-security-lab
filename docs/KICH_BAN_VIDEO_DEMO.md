# Kịch bản Video Demo 60-90 Giây - IoT Security Gateway Lab

**Repository:** https://github.com/dinhvaren/iot-gateway-security-lab

---

## Kịch bản chi tiết từng giây

| Thời gian | Hành động | Lời nói | Minh chứng trên màn hình |
|-----------|-----------|---------|--------------------------|
| **0-10s** | Hiển thị toàn bộ Node-RED flow, zoom vừa đủ thấy pipeline | "Đây là mô hình IoT Security Gateway được xây dựng bằng Node-RED. Dữ liệu từ thiết bị IoT phải đi qua 6 luật bảo mật trước khi được gửi lên cloud." | Toàn bộ flow: Inject Nodes → Normalization → RULE-01 đến RULE-06 → Audit Logger → Decision Router → Cloud Sink / Security Alert |
| **10-17s** | Click Inject "VALID - Normal Data" | "Đầu tiên tôi gửi một payload hợp lệ từ sensor-001." | Debug sidebar hiển thị JSON với `"decision": "ACCEPT"`, `"rule_id": "PASS"` |
| **17-25s** | Chỉ vào Cloud Sink debug node | "Dữ liệu vượt qua toàn bộ kiểm tra và được chuyển tiếp lên Cloud Sink. Dữ liệu chỉ đến được cloud sau khi đã vượt qua gateway." | Cloud Sink debug panel hiển thị dữ liệu |
| **25-35s** | Click Inject "INVALID - High Temperature" | "Bây giờ là dữ liệu có nhiệt độ 85 độ C, vượt ngưỡng cho phép 60 độ C." | Debug hiển thị `"decision": "REJECT"`, `"rule_id": "RULE-04"`, reason về temperature |
| **35-42s** | Chỉ vào Security Alert debug, nhấn mạnh dữ liệu KHÔNG đến Cloud Sink | "Gateway phát hiện vi phạm RULE-04 và từ chối payload. Dữ liệu này không đến được Cloud Sink." | Security Alert panel, Cloud Sink trống (không có dữ liệu mới) |
| **42-52s** | Click Inject "INVALID - Bad Token" | "Với token không hợp lệ, dữ liệu bị chặn ngay tại bước xác thực." | Debug hiển thị `"rule_id": "RULE-01"`, reason về authentication |
| **52-62s** | Scroll Debug sidebar để thấy nhiều log entry | "Mỗi quyết định đều được ghi vào audit log với đầy đủ thông tin: thời gian, thiết bị, quyết định, mã luật và lý do. Token được masked để bảo vệ thông tin xác thực." | Các dòng audit log JSON, thấy `token_hint` đã mask |
| **62-75s** | Quay lại flow tổng thể | "Gateway thực hiện ba vai trò chính: xác thực thiết bị, lọc dữ liệu trước khi lên cloud và ghi log bảo mật. Đây là lớp kiểm soát quan trọng trong kiến trúc IoT." | Flow tổng thể |

---

## Tổng thời lượng: ~75 giây

---

## Lời thoại đầy đủ (để đọc khi quay)

> "Đây là mô hình IoT Security Gateway được xây dựng bằng Node-RED. Dữ liệu từ thiết bị IoT phải đi qua 6 luật bảo mật trước khi được gửi lên cloud.
>
> Đầu tiên tôi gửi một payload hợp lệ từ sensor-001. Dữ liệu vượt qua toàn bộ kiểm tra và được chuyển tiếp lên Cloud Sink. Dữ liệu chỉ đến được cloud sau khi đã vượt qua gateway.
>
> Bây giờ là dữ liệu có nhiệt độ 85 độ C, vượt ngưỡng cho phép 60 độ C. Gateway phát hiện vi phạm RULE-04 và từ chối payload. Dữ liệu này không đến được Cloud Sink.
>
> Với token không hợp lệ, dữ liệu bị chặn ngay tại bước xác thực.
>
> Mỗi quyết định đều được ghi vào audit log với đầy đủ thông tin: thời gian, thiết bị, quyết định, mã luật và lý do. Token được masked để bảo vệ thông tin xác thực.
>
> Gateway thực hiện ba vai trò chính: xác thực thiết bị, lọc dữ liệu trước khi lên cloud và ghi log bảo mật. Đây là lớp kiểm soát quan trọng trong kiến trúc IoT."

---

## Lưu ý khi quay

1. **Nói chậm, rõ ràng.** Không cần vội.
2. **Chỉ vào màn hình** khi nói về một thành phần cụ thể.
3. **Dừng 1-2 giây** sau mỗi thao tác inject để debug hiển thị.
4. **Không đọc lý thuyết dài.** Video tập trung vào demo thực tế.
5. **Kiểm tra audio** trước khi quay chính thức.
6. **Quay thử 1 lần** trước khi quay chính thức.

---

*Ngày thực hiện: 08/07/2026*

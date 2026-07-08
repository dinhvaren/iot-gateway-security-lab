# Kiến trúc Hệ thống - IoT Security Gateway Lab

**Mã đề tài:** 04 - Vai trò của gateway trong bảo mật IoT
**Mã sinh viên:** 231A010001

---

## 1. Tổng quan kiến trúc

Hệ thống được thiết kế theo mô hình pipeline tuần tự, mỗi tầng đảm nhiệm một chức năng bảo mật riêng biệt. Dữ liệu từ thiết bị IoT (giả lập qua Inject Node) đi qua chuỗi kiểm tra trước khi được chấp nhận hoặc từ chối.

```
┌─────────────────────────────────────────────────────────────┐
│                    DOCKER CONTAINER                          │
│                 (nodered/node-red:4.0.9)                     │
│                                                              │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐                │
│  │ Inject   │   │ Inject   │   │ Inject   │  ... (8 nodes)  │
│  │ TC-01    │   │ TC-02    │   │ TC-03    │                │
│  └────┬─────┘   └────┬─────┘   └────┬─────┘                │
│       └───────────────┼─────────────┘                       │
│                       ▼                                      │
│            ┌─────────────────────┐                           │
│            │ INPUT NORMALIZATION │                           │
│            │ Parse & normalize   │                           │
│            └─────────┬───────────┘                           │
│                       ▼                                      │
│            ┌─────────────────────┐                           │
│            │ RULE-01: TOKEN AUTH │                           │
│            │ Verify device token │                           │
│            └─────────┬───────────┘                           │
│                       ▼                                      │
│            ┌─────────────────────┐                           │
│            │ RULE-02: DEVICE LIST│                           │
│            │ Allowlist check     │                           │
│            └─────────┬───────────┘                           │
│                       ▼                                      │
│            ┌─────────────────────┐                           │
│            │ RULE-03: SCHEMA     │                           │
│            │ Required fields     │                           │
│            └─────────┬───────────┘                           │
│                       ▼                                      │
│            ┌─────────────────────┐                           │
│            │ RULE-04: RANGE      │                           │
│            │ Sensor thresholds   │                           │
│            └─────────┬───────────┘                           │
│                       ▼                                      │
│            ┌─────────────────────┐                           │
│            │ RULE-05: FRESHNESS  │                           │
│            │ Timestamp check     │                           │
│            └─────────┬───────────┘                           │
│                       ▼                                      │
│            ┌─────────────────────┐                           │
│            │ RULE-06: REPLAY     │                           │
│            │ Duplicate detection │                           │
│            └─────────┬───────────┘                           │
│                       ▼                                      │
│            ┌─────────────────────┐                           │
│            │ AUDIT LOGGER        │──────────────────┐        │
│            │ Format & log        │                  │        │
│            └─────────┬───────────┘                  │        │
│                       ▼                              ▼        │
│            ┌─────────────────────┐    ┌──────────────────┐   │
│            │ DECISION ROUTER     │    │ AUDIT LOG FILE   │   │
│            │ ACCEPT / REJECT     │    │ /data/logs/      │   │
│            └────┬───────────┬────┘    │ audit.log        │   │
│                 │           │         └──────────────────┘   │
│          ACCEPT │           │ REJECT                         │
│                 ▼           ▼                                │
│         ┌──────────┐ ┌──────────────┐                       │
│         │ Cloud    │ │ Security     │                       │
│         │ Sink     │ │ Alert        │                       │
│         │ (Debug)  │ │ (Debug)      │                       │
│         └──────────┘ └──────────────┘                       │
└─────────────────────────────────────────────────────────────┘
```

## 2. Thành phần hệ thống

### 2.1. Inject Nodes (8 nodes)

Cung cấp dữ liệu giả lập từ thiết bị IoT với 8 kịch bản test:

| # | Inject Node | Mục đích |
|---|------------|----------|
| 1 | VALID - Normal Data | Dữ liệu hợp lệ hoàn toàn |
| 2 | INVALID - Bad Token | Token sai |
| 3 | INVALID - Unknown Device | Thiết bị không trong danh sách |
| 4 | INVALID - Missing Field | Thiếu trường humidity |
| 5 | INVALID - High Temperature | Nhiệt độ vượt ngưỡng |
| 6 | INVALID - Invalid Humidity | Độ ẩm ngoài 0-100% |
| 7 | INVALID - Stale Timestamp | Timestamp quá cũ |
| 8 | INVALID - Replay/Duplicate | Payload trùng lặp |

### 2.2. Function Nodes (7 nodes)

| Node | Chức năng | Input | Output |
|------|-----------|-------|--------|
| Input Normalization | Parse JSON, trích xuất trường | Payload thô | msg.* fields |
| RULE-01 Token Auth | Xác thực token | msg.token | decision/REJECT |
| RULE-02 Device List | Kiểm tra allowlist | msg.device_id | decision/REJECT |
| RULE-03 Schema | Kiểm tra required fields | msg.* | decision/REJECT |
| RULE-04 Range | Kiểm tra ngưỡng cảm biến | msg.temp/hum | decision/REJECT |
| RULE-05 Freshness | Kiểm tra tuổi timestamp | msg.timestamp | decision/REJECT |
| RULE-06 Replay | Phát hiện trùng lặp | flow context | decision/REJECT |
| Audit Logger | Format log, quyết định cuối | msg.decision | audit_entry |

### 2.3. Routing & Output

- **Decision Router (Switch):** Phân luồng ACCEPT → Cloud Sink, REJECT → Security Alert.
- **Cloud Sink (Debug):** Hiển thị dữ liệu được chấp nhận.
- **Security Alert (Debug):** Hiển thị dữ liệu bị từ chối.
- **Audit Log File:** Ghi toàn bộ log ra `/data/logs/audit.log`.

## 3. Luồng dữ liệu

### 3.1. Payload định dạng

```json
{
  "device_id": "sensor-001",
  "token": "demo-secure-token-2024",
  "timestamp": 2000000000000,
  "temperature": 28.5,
  "humidity": 65
}
```

### 3.2. Trạng thái message trong pipeline

Mỗi function node bổ sung metadata vào `msg` object:

```javascript
msg.device_id       // ID thiết bị
msg.token           // Token gốc (chỉ dùng để kiểm tra)
msg.token_masked    // Token đã mask (dùng để log)
msg.temperature     // Nhiệt độ
msg.humidity        // Độ ẩm
msg.timestamp       // Timestamp epoch
msg.decision        // null → "ACCEPT" hoặc "REJECT"
msg.rule_id         // Mã luật từ chối (nếu REJECT)
msg.reason          // Lý do từ chối
msg.event_time      // Thời gian gateway nhận (ISO 8601)
```

### 3.3. Cơ chế bảo vệ token

Token KHÔNG được log đầy đủ:
- Input Normalization tạo `msg.token_masked` (4 ký tự đầu + `****`).
- Audit Logger chỉ ghi `token_hint`.
- Token gốc chỉ dùng trong RULE-01, không xuất hiện trong log.

## 4. Công nghệ sử dụng

| Thành phần | Công nghệ | Phiên bản |
|------------|-----------|-----------|
| Container Runtime | Docker | 29.5.3 |
| Orchestration | Docker Compose | 5.1.4 |
| Flow Engine | Node-RED | 4.0.9 |
| JavaScript Runtime | Node.js | 20.19.0 |
| OS (container) | Alpine Linux | - |

## 5. Quyết định thiết kế

1. **Pipeline tuần tự:** Mỗi rule là một function node riêng biệt để dễ đọc, dễ kiểm tra và dễ mở rộng.
2. **Fail-fast:** Khi một rule REJECT, các rule sau bỏ qua xử lý và chuyển tiếp message.
3. **Context-based replay detection:** Sử dụng Node-RED flow context (in-memory) để lưu timestamp đã thấy, phù hợp với phạm vi lab. Production nên dùng Redis hoặc database.
4. **File audit log:** Ghi log ra file trong container, mount ra host để dễ kiểm tra.
5. **Không hardcode secret trong code:** Token và cấu hình được đặt trong biến môi trường (docker-compose.yml), có file .env.example làm mẫu.

---

*Ngày thực hiện: 08/07/2026*

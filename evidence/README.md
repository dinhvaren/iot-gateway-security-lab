# Evidence - IoT Security Gateway Lab

Thư mục chứa minh chứng thực tế từ quá trình chạy lab.

## Danh sách file

| File | Mô tả |
|------|-------|
| `docker-compose-ps.txt` | Kết quả `docker compose ps` - trạng thái container |
| `gateway-test-results.txt` | Audit log đầy đủ cho 8 test case |
| `accepted-sample-log.json` | Mẫu log ACCEPT (TC-01 VALID) |
| `rejected-sample-log.json` | Mẫu log REJECT (TC-02 Bad Token) |

## Cách thu thập lại

```bash
# Trạng thái container
docker compose ps > evidence/docker-compose-ps.txt

# Audit log
docker exec iot-gateway-nodered sh -c "tail -8 /data/logs/audit.log" > evidence/gateway-test-results.txt

# Mẫu ACCEPT/REJECT
docker exec iot-gateway-nodered sh -c "tail -8 /data/logs/audit.log | head -1" > evidence/accepted-sample-log.json
docker exec iot-gateway-nodered sh -c "tail -8 /data/logs/audit.log | head -2 | tail -1" > evidence/rejected-sample-log.json
```

## Ngày thu thập

08/07/2026

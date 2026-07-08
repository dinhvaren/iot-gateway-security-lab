# Tổng kết Tiến độ - IoT Security Gateway Lab

**Repository:** https://github.com/dinhvaren/iot-gateway-security-lab
**Mã đề tài:** 04 - Vai trò của gateway trong bảo mật IoT
**Mã sinh viên:** 231A010001
**Ngày:** 08/07/2026

---

## Trạng thái hiện tại

| Hạng mục | Trạng thái | Ghi chú |
|----------|-----------|---------|
| Docker Compose | ✅ Chạy thành công | nodered/node-red:4.0.9, healthy |
| flows.json | ✅ Hoạt động | 23 nodes, 6 rules + audit |
| Test Cases (8/8) | ✅ PASS 100% | ACCEPT 1/8, REJECT 7/8 |
| Audit Log | ✅ Ghi file | logs/audit.log, JSON structured |
| Đề cương Tuần 02 | ✅ Hoàn thiện | docs/DE_CUONG_TUAN_02.md |
| Kiến trúc hệ thống | ✅ Hoàn thiện | docs/KIEN_TRUC_HE_THONG.md |
| Tài liệu tham khảo | ✅ 7 nguồn | docs/TAI_LIEU_THAM_KHAO.md |
| Bảng rủi ro | ✅ 7 rủi ro | docs/BANG_RUI_RO.md |
| Test Cases doc | ✅ Hoàn thiện | docs/TEST_CASES.md |
| Demo 60 giây | ✅ Hoàn thiện | docs/DEMO_60_GIAY.md |
| Phân tích bảo mật | ✅ Hoàn thiện | docs/PHAN_TICH_BAO_MAT.md |
| README | ✅ Hoàn thiện | README.md |
| Báo cáo Word | ✅ Đã điền | report/CapNhatTienDo_Tuan02_... |
| Video demo guide | ✅ Hoàn thiện | docs/HUONG_DAN_QUAY_VIDEO_DEMO.md |
| Video demo script | ✅ Hoàn thiện | docs/KICH_BAN_VIDEO_DEMO.md |
| GitHub Repository | ✅ Đã push | https://github.com/dinhvaren/iot-gateway-security-lab |
| Screenshots | ⏳ Chưa chụp | docs/screenshots/README.md |
| Video demo | ⏳ Chưa quay | evidence/video-demo/README.md |

---

## Cây thư mục

```
iot-gateway-security-lab/
├── docker-compose.yml
├── .env.example
├── .env
├── .gitignore
├── flows.json
├── README.md
├── PROGRESS_SUMMARY.md
├── node-red/data/
├── logs/
│   └── audit.log
├── docs/
│   ├── DE_CUONG_TUAN_02.md
│   ├── KIEN_TRUC_HE_THONG.md
│   ├── PHAN_TICH_BAO_MAT.md
│   ├── TAI_LIEU_THAM_KHAO.md
│   ├── BANG_RUI_RO.md
│   ├── TEST_CASES.md
│   ├── DEMO_60_GIAY.md
│   ├── HUONG_DAN_QUAY_VIDEO_DEMO.md
│   ├── KICH_BAN_VIDEO_DEMO.md
│   └── screenshots/README.md
├── evidence/
│   ├── README.md
│   ├── docker-compose-ps.txt
│   ├── gateway-test-results.txt
│   ├── accepted-sample-log.json
│   ├── rejected-sample-log.json
│   └── video-demo/README.md
└── report/
    └── CapNhatTienDo_Tuan02_LuongNguyenNgocDinh.docx
```

## Việc cần làm tiếp (Tuần 03)

1. ✅ Đã tạo GitHub repository và push source code.
2. ⏳ Chụp 5 ảnh minh chứng (theo docs/screenshots/README.md).
3. ⏳ Quay video demo 60-90 giây (theo docs/HUONG_DAN_QUAY_VIDEO_DEMO.md).
4. ⏳ Cập nhật link video vào evidence/video-demo/README.md.
5. ⏳ Chuẩn bị slide báo cáo/bảo vệ nếu cần.

## Hạn chế đã biết

1. Token tĩnh, không có cơ chế rotate.
2. Replay protection chỉ dựa trên timestamp.
3. Không có rate limiting.
4. Không có mã hóa kênh truyền.
5. Context in-memory (mất khi restart).
6. Dữ liệu cảm biến giả lập, không có thiết bị thật.

---

*Ngày tổng kết: 08/07/2026*

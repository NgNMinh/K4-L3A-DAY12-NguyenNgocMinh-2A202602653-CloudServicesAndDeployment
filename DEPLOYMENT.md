# Thông Tin Deploy — Checkpoint 5

> Ghi lại thông tin và kết quả kiểm tra deployment tại Railway.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Nguyen Ngoc Minh |
| Mã học viên | 2A202602653 |
| Repo | https://github.com/NgNMinh/K4-L3A-DAY12-NguyenNgocMinh-2A202602653-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://k4-l3a-day12-nguyenngocminh-2a202602653-cloudser-production.up.railway.app |
| Platform | Railway |
| Ngày deploy | 2026-09-28 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | Railway tự cấp |
| `AGENT_API_KEY` | ✅ | Đã đặt bằng Railway CLI; giá trị không lưu trong repo |
| `REDIS_URL` | ✅ | Tham chiếu tới service Redis của Railway |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

Public URL: `https://k4-l3a-day12-nguyenngocminh-2a202602653-cloudser-production.up.railway.app`

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i https://k4-l3a-day12-nguyenngocminh-2a202602653-cloudser-production.up.railway.app/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i https://k4-l3a-day12-nguyenngocminh-2a202602653-cloudser-production.up.railway.app/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST https://k4-l3a-day12-nguyenngocminh-2a202602653-cloudser-production.up.railway.app/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST https://k4-l3a-day12-nguyenngocminh-2a202602653-cloudser-production.up.railway.app/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST https://k4-l3a-day12-nguyenngocminh-2a202602653-cloudser-production.up.railway.app/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Dữ kiện đã xác nhận trên Railway ngày 2026-09-28:

```
Service: Online
Deployment ID: 2a1f9f79-5e63-4f15-8d97-482940f4e009
Redis: Online
```

Chưa có output HTTP từ các lệnh kiểm tra ở trên; chỉ ghi kết quả sau khi chạy thật.

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl

---


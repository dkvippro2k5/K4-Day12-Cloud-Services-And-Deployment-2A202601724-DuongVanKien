# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị token vào đây.**
> Repo này công khai — dán token vào là mất token.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Dương Văn Kiên |
| Mã học viên | 2A202601724 |
| Repo | https://github.com/dkvippro2k5/K4-Day12-Cloud-Services-And-Deployment-2A202601724-DuongVanKien |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-chat-lgyi.onrender.com |
| Platform | Render |
| Ngày deploy | 2026-08-10 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | platform tự gán |
| `API_TOKEN` | ✅ | đặt trong dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | Render Key Value (Redis add-on), tự nối qua `fromService` trong render.yaml |
| `BUCKET_CAPACITY` | ✅ | 10 |
| `REFILL_PER_MINUTE` | ✅ | 10 |
| `DAILY_BUDGET_USD` | ✅ | 1.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

Thay `<URL>` bằng Public URL ở trên:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i <URL>/healthz

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i <URL>/readyz

# 3. Không có token — mong đợi 401 kèm header WWW-Authenticate
curl -i -X POST <URL>/chat \
  -H "Content-Type: application/json" \
  -d '{"message":"Hello"}'

# 4. Có token — mong đợi 200 kèm câu trả lời
curl -i -X POST <URL>/chat \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $API_TOKEN" \
  -H "X-Client-Id: sv-test" \
  -d '{"message":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST <URL>/chat \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $API_TOKEN" \
    -H "X-Client-Id: sv-test" \
    -d '{"message":"test"}'
done; echo
```

## Kết Quả Chạy Thật

```
$ curl.exe -i https://day12-chat-lgyi.onrender.com/healthz
HTTP/1.1 200 OK
Content-Type: application/json
{"status":"ok","service":"day12-chat-service","version":"1.0.0"}

$ curl.exe -i https://day12-chat-lgyi.onrender.com/readyz
HTTP/1.1 200 OK
Content-Type: application/json
{"status":"ready","redis":true}

$ Invoke-WebRequest -Uri "https://day12-chat-lgyi.onrender.com/chat" -Method POST -ContentType "application/json" -Body '{"message":"Hello"}'
Invoke-WebRequest : The remote server returned an error: (401) Unauthorized.

$ Invoke-WebRequest -Uri "https://day12-chat-lgyi.onrender.com/chat" -Method POST -ContentType "application/json" -Headers @{Authorization="Bearer <API_TOKEN>"; "X-Client-Id"="sv-test"} -Body '{"message":"Deploy la gi?"}' -UseBasicParsing
StatusCode        : 200
StatusDescription : OK
Content           : {"reply":"...", "client_id":"sv-test", "turns_before":0, "usd_cost":..., "usage":{...}}
```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/healthz.png` — kết quả gọi `/healthz` từ trình duyệt hoặc curl


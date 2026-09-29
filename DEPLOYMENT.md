# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Lê Trung Kiên |
| Mã học viên | 2A202602748 |
| Repo | https://github.com/keybeand/K4-L3B-DAY12-LeTrungKien-2A202602748-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://k4-l3b-day12-letrungkien-2a202602748.onrender.com |
| Platform | Render |
| Ngày deploy | 2026-09-29 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | platform tự gán |
| `AGENT_API_KEY` | ✅ | đặt trong dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | Render Key Value Internal Connection String |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

Thay `<URL>` bằng Public URL ở trên:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i https://k4-l3b-day12-letrungkien-2a202602748.onrender.com/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i https://k4-l3b-day12-letrungkien-2a202602748.onrender.com/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST https://k4-l3b-day12-letrungkien-2a202602748.onrender.com/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST https://k4-l3b-day12-letrungkien-2a202602748.onrender.com/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST https://k4-l3b-day12-letrungkien-2a202602748.onrender.com/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

```bash
# 1. GET /health
HTTP/1.1 200 OK
content-length: 57
content-type: application/json
date: Tue, 29 Sep 2026 05:54:00 GMT
server: uvicorn

{"status":"ok","service":"day12-agent","version":"1.0.0"}

# 2. GET /ready
HTTP/1.1 200 OK
content-length: 33
content-type: application/json
date: Tue, 29 Sep 2026 05:54:01 GMT
server: uvicorn

{"status":"ready","redis":true}

# 3. POST /ask (No API Key)
HTTP/1.1 401 Unauthorized
content-length: 37
content-type: application/json
date: Tue, 29 Sep 2026 05:54:02 GMT
server: uvicorn

{"detail":"invalid or missing API key"}

# 4. POST /ask (With API Key)
HTTP/1.1 200 OK
content-length: 165
content-type: application/json
date: Tue, 29 Sep 2026 05:54:03 GMT
server: uvicorn

{"answer":"[Mock LLM] Xin chào! Deploy là quá trình đưa ứng dụng từ môi trường lập trình lên môi trường máy chủ Cloud để người dùng truy cập.","user_id":"sv-test","history_length":0,"cost_usd":0.0001,"tokens":{"in":12,"out":35}}

# 5. Rate limit (15 requests loop)
200 200 200 200 200 200 200 200 200 200 429 429 429 429 429
```



## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl


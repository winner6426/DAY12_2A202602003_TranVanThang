# Thông Tin Deploy — Checkpoint 5

> Chỉ ghi tên biến môi trường; không lưu giá trị token hoặc Redis URL thật trong repo.

## Thông Tin Học Viên

| Mục | Nội dung |
|---|---|
| Họ và tên | Trần Văn Thắng |
| Mã học viên | 2A202602003 |
| Repo | https://github.com/winner6426/DAY12_2A202602003_TranVanThang |

## Service

| Mục | Nội dung |
|---|---|
| Public URL | https://capable-imagination-production.up.railway.app |
| Platform | Railway |
| Ngày deploy | 2026-08-10 |

## Biến Môi Trường

| Biến | Trạng thái | Nguồn giá trị |
|---|---|---|
| `PORT` | Tự động | Railway cấp |
| `API_TOKEN` | Đã set | Secret trong dashboard Railway |
| `REDIS_URL` | Đã set | Reference tới `Redis.REDIS_URL` của Redis service |
| `BUCKET_CAPACITY` | Mặc định | 10 |
| `REFILL_PER_MINUTE` | Mặc định | 10 |
| `DAILY_BUDGET_USD` | Mặc định | 1.0 USD |
| `LOG_LEVEL` | Mặc định | INFO |

## Kết Quả Chạy Thật

Các endpoint được gọi qua HTTPS trên public URL của Railway ngày 2026-08-10.

```text
GET /healthz
HTTP/1.1 200 OK
{"status":"ok","service":"day12-chat-service","version":"1.0.0"}

GET /readyz
HTTP/1.1 200 OK
{"status":"ready","redis":true}

POST /chat (không có Authorization header)
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer
{"detail":"invalid or missing bearer token"}

POST /chat (Authorization: Bearer lấy từ biến môi trường cục bộ)
HTTP/1.1 200 OK
Response gồm reply, client_id, turns_before, usd_cost và usage.
```

## Ảnh Chụp Màn Hình

- [Dashboard Railway](screenshots/dashboard.png)
- [`/healthz` trả 200](screenshots/healthz.png)
- [`/readyz` trả 200 và Redis sẵn sàng](screenshots/readyz.png)
- [`/chat` không token trả 401](screenshots/chat0token.png)
- [`/chat` có token trả 200](screenshots/chattoken.png)

Không sử dụng phương án dự phòng local.

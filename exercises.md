# Phiếu Phản Ánh — K4 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Trần Văn Thắng  Mã học viên: 2A202602003

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `api_token` không có giá trị mặc định nên app chết ngay khi
khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà việc
"chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Khi deploy lên Railway, nếu tôi quên đặt `API_TOKEN` thì cấu hình không hợp lệ và
> service dừng ngay, nên tôi phát hiện lỗi trong deployment log trước khi mở traffic.
> Nếu dùng mặc định `"changeme"`, service vẫn chạy và bất kỳ ai biết hoặc đoán được
> token mẫu đều có thể gọi `/chat`, làm phát sinh chi phí mà tôi chỉ phát hiện sau đó.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/chat` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Một dòng log tôi quan sát được là:
> `{"event":"chat_completed","severity":"INFO","ts":"2026-08-10T10:39:05.312861+00:00","client_id":"sv-test","prompt_tokens":4,"completion_tokens":36,"usd_cost":0.0000222}`.
> Với log có cấu trúc này, tôi có thể (1) lọc và cộng `usd_cost` theo `client_id` để
> theo dõi chi phí, và (2) tìm kiếm/sắp xếp theo `event`, `severity`, `ts` để điều tra
> sự cố hoặc tạo cảnh báo. Một chuỗi `print("đã trả lời xong")` không có các trường
> ổn định để máy truy vấn hay tổng hợp.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t chat:single .
docker build -t chat:multi .
docker images | grep chat
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | 1.7 GB |
| Multi-stage | 270 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Tôi build thực tế được `chat:single-exercise` là 1.7 GB và
> `day12-chat:cp2-test` là 270 MB. Phần chênh lệch chủ yếu đến từ image
> `python:3.11` đầy đủ chứa nhiều công cụ và thư viện hệ thống, trong khi runtime
> dùng `python:3.11-slim`. Multi-stage cũng chỉ chuyển dependency đã cài sang stage
> runtime, không mang toàn bộ môi trường và layer phục vụ build sang image cuối.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi chỉ sửa `app/main.py`, stage builder vẫn dùng cache cho base image,
> `COPY requirements.txt` và `RUN pip install` vì requirements không đổi. Stage
> runtime dùng lại base và dependency từ builder; từ lớp `COPY app/ app/` trở đi
> phải tạo lại. Vì vậy bước tốn thời gian nhất là cài package vẫn được cache. Nếu
> đặt `COPY . .` trước `RUN pip install`, mọi thay đổi source đều làm lớp COPY đổi
> và buộc pip cài lại toàn bộ dependency dù `requirements.txt` không thay đổi.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Kẻ tấn công trước hết khai thác lỗ hổng trong Python để thực thi lệnh trong
> container. Nếu process chạy root, họ có toàn quyền trong container; khi container
> còn được cấu hình nguy hiểm như mount Docker socket, mount thư mục host hoặc có
> lỗ hổng container runtime, quyền đó có thể được dùng để tác động lên host với
> quyền cao. `USER appuser` cắt chuỗi tại bước thực thi trong container: mã bị chiếm
> quyền chỉ chạy bằng user ít quyền, giảm khả năng sửa file hệ thống và giảm phạm vi
> thiệt hại, dù vẫn phải tránh các mount/capability nguy hiểm.

---

### Câu 6 — Bearer token (CP3)

Vì sao 401 phải kèm header `WWW-Authenticate: Bearer`? Và vì sao ta trả **cùng
một** thông báo lỗi cho cả ba trường hợp (thiếu header, sai scheme, sai token)
thay vì nói rõ sai ở đâu cho người dùng dễ sửa?

> `WWW-Authenticate: Bearer` là tín hiệu chuẩn để client biết tài nguyên yêu cầu cơ
> chế Bearer và có thể xử lý 401 đúng cách. Tôi dùng cùng thông báo
> `invalid or missing bearer token` cho thiếu header, sai scheme và sai token để
> không biến API thành công cụ dò: nếu trả lỗi chi tiết, người tấn công biết mình đã
> đoán đúng phần nào và thu hẹp dần không gian thử.

---

### Câu 7 — Token bucket (CP3)

Với `capacity=10`, `refill_per_minute=10`: một client im lặng 10 phút rồi gửi
liên tiếp. Nó gửi được bao nhiêu request trước khi bị 429? Nếu bỏ đoạn
`min(capacity, ...)` trong `available()` thì con số đó thành bao nhiêu, và tại sao?

> Có `min(capacity, ...)`, sau 10 phút client vẫn chỉ có tối đa 10 token nên gửi
> được 10 request liên tiếp; request thứ 11 nhận 429. Nếu bỏ giới hạn và giả sử xô
> đã cạn trước lúc im lặng, tốc độ 10 token/phút sẽ tích thành 100 token nên gửi
> được 100 request. Nếu trước đó còn đủ 10 token thì phép tính thô thậm chí cho 110.
> Đây là lý do lượng refill luôn phải bị chặn trên ở `capacity`.

---

### Câu 8 — Ngân sách theo ngày (CP3)

So sánh hạn mức $30/tháng với hạn mức $1/ngày cho cùng một client. Giả sử có sự
cố khiến một client gọi liên tục từ 2h sáng. Với mỗi cách, thiệt hại tối đa là
bao nhiêu và service tự hồi phục khi nào?

> Với hạn mức 30 USD/tháng, sự cố có thể tiêu hết tối đa 30 USD và client chỉ tự có
> ngân sách lại khi sang kỳ/tháng mới (hoặc khi quản trị viên can thiệp). Với hạn
> mức 1 USD/ngày, thiệt hại trong ngày bị chặn ở khoảng 1 USD; nếu sự cố bắt đầu lúc
> 2 giờ sáng thì service tự cấp hạn mức mới vào nửa đêm UTC của ngày kế tiếp. Cách
> theo ngày giới hạn phạm vi thiệt hại nhỏ hơn, dù sự cố kéo dài qua nhiều ngày vẫn
> cần cảnh báo và xử lý.

---

### Câu 9 — /healthz khác /readyz (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Theo thứ tự: Redis mất kết nối → endpoint gộp của cả ba container trả 503 →
> orchestrator coi cả ba process không còn sống → đồng thời rút chúng khỏi cân bằng
> tải và restart → cụm không còn instance phục vụ nên người dùng nhận 502/503 → các
> container mới vẫn kiểm tra Redis lỗi và tiếp tục bị restart. Một sự cố Redis 30
> giây vì thế thành outage/restart storm toàn cụm. Tách `/healthz` giúp container
> vẫn được coi là sống, còn `/readyz` chỉ tạm ngừng nhận traffic đến khi Redis hồi
> phục.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Khi deploy Railway, `/healthz` đã chạy nhưng `/readyz` trả 500. Tôi mở deployment
> log và thấy `ValueError: Redis URL must specify one of the following schemes
> (redis://, rediss://, unix://)` tại `redis.from_url()`. Kiểm tra tab Variables cho
> thấy `REDIS_URL` đang là chuỗi rỗng. Tôi tạo Redis service, thay biến của app bằng
> reference `${{Redis.REDIS_URL}}`, bấm Deploy để áp dụng thay đổi rồi gọi lại;
> `/readyz` trả 200 với `{"status":"ready","redis":true}`.

# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Lê Trung Kiên  Mã học viên: 2A202602748

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu để mặc định là `"changeme"`, ứng dụng vẫn sẽ khởi động thành công trên môi trường Production khi ta quên cấu hình `AGENT_API_KEY`. Lúc này, hệ thống mở cổng công khai và ai cũng có thể gửi request bằng khóa mặc định `"changeme"` để lợi dụng API key / tài nguyên LLM của ta, khiến tài khoản bị tiêu tốn chi phí lớn mà ta không hề hay biết cho đến khi nhận hóa đơn. Việc "fail fast" khiến container sập ngay lập tức lúc deploy, buộc ta phát hiện lỗi thiếu secret ngay lập tức trên dashboard deployment.


---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Dòng log JSON: `{"event": "ask_completed", "level": "info", "timestamp": "2026-09-29T10:12:44.123456+00:00", "user_id": "sv01", "tokens_in": 15, "tokens_out": 25, "cost_usd": 0.0001}`
> 
> Hai việc làm được:
> 1. Truy vấn & Lọc tự động: Các hệ thống gom log (Datadog, Kibana, Grafana Loki) có thể parse các trường JSON để lọc riêng log của một `user_id` cụ thể hoặc tính tổng chi phí `cost_usd` theo thời gian.
> 2. Đặt Cảnh báo (Alerting): Có thể thiết lập luật cảnh báo tự động kích hoạt khi tổng `tokens_out` hoặc `cost_usd` vượt ngưỡng bất thường trong khoảng 5 phút.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | ~1.02 GB |
| Multi-stage | ~170 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Phần dung lượng chênh lệch hơn 800MB chính là hệ điều hành bản đầy đủ với bộ công cụ biên dịch (gcc, g++, make), thư viện C header (python3-dev), cache của pip và các file tạm tạo ra trong quá trình build package. Bản multi-stage loại bỏ toàn bộ các công cụ biên dịch này và chỉ copy các file wheel/package đã build sang base image `python:3.11-slim` tối giản.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> - Với Dockerfile hiện tại: Layer `COPY requirements.txt` và `RUN pip install` được dùng lại hoàn toàn từ Docker cache (vì file `requirements.txt` không đổi). Chỉ layer `COPY app /app/app` và các bước sau nó phải chạy lại. Thời gian build lại chỉ mất 1-2 giây.
> - Nếu đặt `COPY . .` lên trước `RUN pip install`: Mỗi lần sửa 1 ký tự trong `app/main.py`, Docker sẽ coi layer `COPY . .` bị thay đổi, dẫn tới toàn bộ layer `RUN pip install` phía sau bị mất cache và phải tải/cài lại toàn bộ thư viện từ đầu, làm tăng đáng kể thời gian build.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Chuỗi sự kiện: Kẻ tấn công lợi dụng lỗ hổng Remote Code Execution (RCE) trong code Python để thực thi lệnh shell bên trong container. Vì container chạy bằng user `root`, kẻ tấn công có quyền root trong container và tìm cách khai thác lỗ hổng thoát khỏi container (container escape) để truy cập hệ thống host. Khi thoát ra host, kẻ tấn công giữ nguyên UID=0 (root) và chiếm quyền kiểm soát máy host. Lệnh `USER appuser` cắt đứt chuỗi này ngay từ đầu vì kẻ tấn công chỉ có quyền của user thường (`appuser`), không thể sửa đổi hệ thống tệp và không thể lấy quyền root trên host.


---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Một người dùng có thể gửi tối đa **20 request** trong 2 giây liên tiếp. Cách đạt được: Người dùng gửi 10 request ở giây 10:00:59 (cuối phút thứ nhất) và gửi tiếp 10 request ở giây 10:01:01 (đầu phút thứ hai). Thuật toán đếm theo phút đồng hồ xem đây là 2 phút riêng biệt nên cho phép cả 20 request, nhưng về bản chất người dùng đã tạo burst 20 request chỉ trong 2 giây.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Khác nhau: Rate Limiter quản lý tần suất/số lượng request trong khoảng thời gian ngắn (max 10 req/phút). Cost Guard quản lý ngân sách/tổng chi phí tích lũy trong thời gian dài (max $10.0/tháng).
> 
> - Rate Limit cho qua nhưng Cost Guard chặn: User mới gửi request thứ 2 trong phút (chưa vượt 10 req/phút), nhưng request trước đó tiêu tốn số token cực lớn khiến tổng chi phí trong tháng chạm mốc $10.0.
> - Tình huống ngược lại: User mới tiêu $0.10 trong tháng (còn rất nhiều ngân sách), nhưng gửi 15 request liên tục trong 5 giây. Rate Limit lập tức chặn (HTTP 429) do vượt ngưỡng 10 req/phút.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Thứ tự sự kiện:
> 1. Redis gặp sự cố và mất kết nối trong 30 giây.
> 2. Endpoint liveness check thất bại (vì nó ném lỗi khi ping Redis).
> 3. Orchestrator (Docker/K8s) tưởng rằng các container app đã bị treo/chết process nên lập tức kill và khởi động lại (restart) cả 3 container agent.
> 4. Trong khi Redis đang khôi phục, 3 container khởi động lại tiếp tục fail health check và rơi vào vòng lặp restart liên tục (CrashLoopBackOff), biến một sự cố chốc quắc của Redis thành sự cố sập toàn bộ hệ thống web app.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Nếu lưu bằng dict Python trong RAM, `history_length` sẽ tăng giảm trồi sụt bất thường giữa các request thay vì tăng tiến liên tục (0, 1, 2, 3...). Lý do là Load Balancer chia đều các request vào 3 container agent khác nhau; mỗi container giữ một dict riêng trong bộ nhớ RAM của nó, khiến agent bị "mất trí nhớ" tùy thuộc vào request rơi vào instance nào.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Thông báo lỗi: Container bị crash với lỗi `pydantic_core._pydantic_core.ValidationError: 1 validation error for Settings agent_api_key Field required`.
> Nguyên nhân: Tìm thấy khi kiểm tra Runtime Logs trên Dashboard. Do `AGENT_API_KEY` trong `Settings` được thiết kế fail-fast không có giá trị mặc định, mà trên Environment Variables của Dashboard chưa thiết lập biến này.
> Cách sửa: Thêm biến `AGENT_API_KEY` kèm chuỗi bí mật tương ứng vào mục Environment Variables của Service trên Cloud Dashboard rồi trigger Redeploy.


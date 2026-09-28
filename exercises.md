# Phiếu Phản Ánh — K4 Level 3A, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng placeholder trong từng câu bằng nội dung của bạn.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Nguyễn Ngọc Minh  Mã học viên: 2A202602653

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

Nếu quên cấu hình `AGENT_API_KEY`, app dừng ngay khi khởi động để mình phát hiện lỗi cấu hình trước khi nhận traffic. Nếu mặc định là `changeme`, app có thể chạy với khóa ai cũng đoán được và bị gọi trái phép.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

Mình gọi `/ask` bằng TestClient, Redis giả và LLM giả của lab; request trả 200 và log thu được là:

```json
{"event": "ask_completed", "level": "info", "timestamp": "2026-09-28T16:46:21.327671+00:00", "user_id": "observation-user", "tokens_in": 3, "tokens_out": 37, "cost_usd": 2.265e-05}
```

Từ các trường này, mình có thể lọc log theo `user_id` để tìm request của một người dùng và cộng `tokens_in`/`tokens_out` hoặc `cost_usd` để theo dõi mức sử dụng. Một dòng `print("đã trả lời xong")` không có các trường dữ liệu để lọc và tổng hợp như vậy.

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
| 1 stage (bản đầu) | Chưa đo được: build dừng ở bước cài dependency với `exec /bin/sh: exec format error` (đã thử lại) |
| Multi-stage | 184 MB (`docker images`; tag `day12-agent:cp2-test`) |

Mình chưa thể kết luận mức chênh lệch vì image 1-stage không build xong trên Docker Engine này; lệnh build báo lỗi định dạng thực thi ở `/bin/sh`. Image multi-stage đã build thành công và hiển thị 184 MB. Theo Dockerfile, multi-stage giữ compiler trong builder còn image runtime chỉ nhận dependency đã cài và code.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

Khi chỉ sửa `app/main.py`, các layer cài dependency trước `COPY app` được dùng lại; layer copy app và các layer sau nó phải chạy lại. Nếu `COPY . .` đặt trước `pip install`, sửa code cũng làm mất cache của bước cài package và khiến cài lại dependency.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

Nếu có lỗ hổng, tiến trình bị chiếm quyền với quyền của user đang chạy trong container. Chạy bằng root cho kẻ tấn công nhiều quyền hơn trong container và tăng hậu quả nếu khai thác thêm cấu hình/mount yếu. `USER appuser` giảm quyền của tiến trình; nó không thay thế các lớp cô lập khác.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

Có thể gửi 10 request ngay trước ranh giới phút và thêm 10 request ngay sau đó: tối đa **20 request trong khoảng 2 giây**. Bộ đếm theo phút đồng hồ vừa reset nên không nhìn thấy cả hai nhóm như một cửa sổ liên tục.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

Rate limit giới hạn **số request theo thời gian**; cost guard giới hạn **tổng tiền đã tiêu**. 10 request ngắn vẫn có thể qua rate limit nhưng một request rất dài làm chạm ngân sách nên cost guard chặn. Ngược lại, nhiều request nhỏ có thể bị rate limit chặn dù ngân sách tháng còn nhiều.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

Nếu `/health` kiểm tra Redis, Redis mất kết nối có thể làm cả 3 container bị báo unhealthy và bị restart cùng lúc. Khi Redis hồi phục, cụm có thể vẫn chưa có instance phục vụ. Tách probe: `/health` chỉ kiểm tra process; `/ready` kiểm tra Redis để load balancer ngừng gửi traffic.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

Mình chạy ba replica agent cùng một Redis tạm và gửi lần lượt cùng một `X-User-Id` trực tiếp đến từng replica. `history_length` trả về lần lượt là **0, 2, 4**; mỗi câu hỏi và câu trả lời thêm hai message, và replica sau đọc được dữ liệu replica trước ghi vào Redis. Nếu thay Redis bằng dict trong RAM, mỗi process giữ lịch sử riêng nên replica mới có thể trả `history_length: 0` thay vì thấy các message trước đó.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

Trong lần GitHub Actions deploy đầu tiên, bước Deploy báo `Unauthorized. Please check that your RAILWAY_TOKEN is valid and has access to the resource you're trying to use.` Mình tìm nguyên nhân trong log của job `Deploy to Railway`; Railway từ chối token mà workflow đang dùng. Sau đó mình deploy lại service bằng Railway CLI và trang Railway hiện `Deployment successful`, service `Online`. Khi kiểm tra lại bài, badge GitHub Actions trả trạng thái `passing` (test bonus 13/13). Lỗi này cho mình biết token trong workflow phải được lưu ở GitHub Repository secrets với quyền truy cập đúng project và environment.

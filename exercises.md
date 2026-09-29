# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: ghi câu trả lời ngay dưới từng câu hỏi.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Phạm Quốc Đạt  Mã học viên: 2A202602384

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu quên cấu hình `AGENT_API_KEY` khi tạo service trên Render, tiến trình sẽ dừng ở bước khởi động với lỗi validation. Tôi sẽ phát hiện trong log deploy và thêm biến vào phần Secrets. Nếu code có khóa mặc định `"changeme"`, service vẫn lên mạng và bất kỳ ai biết khóa mẫu đều có thể gọi `/ask`.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Khi chạy Uvicorn tại máy và gọi `/ask`, tôi nhận được dòng thực tế: `{"event": "ask_completed", "level": "info", "timestamp": "2026-09-29T02:38:06.580899+00:00", "user_id": "smoke", "tokens_in": 3, "tokens_out": 41, "cost_usd": 2.505e-05}`. Tôi có thể lọc mọi event `ask_completed` của `smoke` và cộng `cost_usd` để xem chi phí; đồng thời nhóm theo `timestamp` để phát hiện số request tăng đột biến. Dòng `print("đã trả lời xong")` không chứa những trường đó.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f benchmarks/Dockerfile.single -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | Chưa đo: máy hiện không có Docker |
| Multi-stage | Chưa đo: máy hiện không có Docker |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Chưa có số đo thực tế vì `Get-Command docker` không tìm thấy Docker CLI trên máy này. Sau khi cài Docker, cần build cả hai bản để điền số MB; không nên suy đoán số đo. Về cấu trúc, bản một stage giữ base image đầy đủ, công cụ build và cache trong image cuối, còn bản multi-stage chỉ copy thư viện đã cài từ builder sang runtime `python:3.11-slim`.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Chưa chạy lại Docker build vì máy thiếu Docker. Theo thứ tự Dockerfile hiện tại, thay một ký tự ở `app/main.py` sẽ giữ cache của base image, lớp `COPY requirements.txt` và lớp `pip install`; lớp `COPY app` cùng các lớp sau nó phải chạy lại. Nếu đặt `COPY . .` trước `pip install`, thay đổi đó làm vô hiệu cache của lớp cài dependency và kéo dài mỗi lần build.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Giả sử code Python có lỗi cho phép thực thi lệnh tùy ý: kẻ tấn công chiếm quyền của tiến trình trong container. Nếu tiến trình chạy root và container có thêm quyền hoặc mount nhạy cảm, họ có thể sửa dữ liệu host hoặc tìm đường thoát container với đặc quyền cao. `USER appuser` khiến mã bị chiếm quyền chỉ chạy dưới UID 10001; đây là một lớp giảm thiểu, không phải bảo đảm tuyệt đối nếu container vẫn được cấp quyền nguy hiểm.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Có thể gửi 20 request trong khoảng 2 giây: 10 request ở giây 59 của phút trước và 10 request ở giây 00–01 của phút sau. Bộ đếm theo phút reset ở ranh giới nên nhận cả hai đợt. Cửa sổ trượt 60 giây vẫn nhìn thấy đợt trước và chặn đợt sau.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit đếm số lượt trong 60 giây, còn cost guard cộng chi phí theo user trong tháng UTC. Một request rất dài có thể vẫn là lượt đầu tiên trong phút nhưng bị chặn nếu chi phí ước lượng làm vượt ngân sách. Ngược lại, 11 request rẻ trong một phút có thể chưa chạm ngân sách nhưng lượt thứ 11 bị trả 429. Trong mã lab hiện tại `/ask` gọi `guard.check()` với chi phí ước lượng mặc định bằng 0, nên chặn trước khi gọi mock LLM chỉ khi tổng đã vượt mức; muốn bảo đảm không vượt ngân sách thực cần ước lượng và giữ chỗ chi phí trước khi gọi LLM.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Redis mất kết nối, cả ba container trả lỗi ở endpoint gộp. Nếu orchestrator dùng nó làm liveness probe, cả ba bị đánh dấu unhealthy rồi bị restart. Trong lúc restart không instance nào nhận request; khi Redis phục hồi, các instance còn phải khởi động lại. Tách `/health` chỉ kiểm tra process và `/ready` kiểm tra Redis giúp load balancer ngừng gửi request tới instance chưa sẵn sàng mà không khởi động lại cả cụm.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Chưa chạy ba container vì máy thiếu Docker. Khi gọi hai lượt qua Uvicorn cục bộ với cùng `X-User-Id`, tôi quan sát `history_length` là 0 rồi 2 khi dùng Redis giả lập. Với Redis thật dùng chung, các container sẽ thấy cùng dãy 0, 2, 4... đến khi lịch sử chạm giới hạn 20 message. Nếu dùng dict riêng trong RAM, khi request rơi sang instance khác con số có thể trở về 0 hoặc nhỏ hơn; kết quả không tăng ổn định.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Tôi không gặp lỗi trong lần deploy Render này, nên không thể ghi một thông báo lỗi hay cách sửa có thật. Để xác minh, tôi gọi Public URL: `/health` trả 200 với `status=ok`, `/ready` trả 200 với `redis=true`, `/ask` thiếu key trả 401 và có key trả 200; `pytest tests/test_cp5.py -v` đạt 9 test, 4 test fallback được bỏ qua. Nếu có lỗi ở lần deploy sau, tôi sẽ đọc build/runtime log trên Render, xác định nguyên nhân và bổ sung tình huống thực tế vào đây, không tự tạo lỗi giả.

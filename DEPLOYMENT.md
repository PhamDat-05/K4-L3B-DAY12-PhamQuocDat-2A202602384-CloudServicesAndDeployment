# Trạng thái triển khai — Checkpoint 5

## Thông tin học viên

| Mục | Nội dung |
| --- | --- |
| Họ và tên | Phạm Quốc Đạt |
| Mã học viên | 2A202602384 |
| Repo | https://github.com/PhamDat-05/K4-L3B-DAY12-PhamQuocDat-2A202602384-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
| --- | --- |
| Public URL | https://day12-agent-bveh.onrender.com |
| Platform | Render (`render.yaml`) |
| Ngày deploy | 2026-09-29 theo ảnh dashboard; xác minh HTTP cùng ngày |
| Trạng thái | Render hiển thị **Live**, commit `fd1fd11` (`Complete lab`), Docker Free, Blueprint managed |

## Biến môi trường trên cloud

Kết quả HTTP chứng minh service đã khởi động và nhận xác thực. Giá trị secret không được ghi vào repository. Các giá trị không bí mật được khai báo trong `render.yaml`; chưa đối chiếu trực tiếp màn hình Variables của Render.

| Biến | Trạng thái | Nguồn dự kiến |
| --- | --- | --- |
| `PORT` | Service đang phục vụ HTTP | Render cấp lúc chạy; giá trị chưa đối chiếu trên dashboard |
| `AGENT_API_KEY` | Đã xác minh qua `/ask` có key trả 200 | Secret của service Render; không ghi giá trị |
| `REDIS_URL` | `/ready` trả 200 với `"redis":true` | Blueprint tham chiếu Render Key Value; chưa xem giá trị thực trên dashboard |
| `RATE_LIMIT_PER_MINUTE` | Khai báo trong `render.yaml` | Chưa xác minh giá trị runtime |
| `MONTHLY_BUDGET_USD` | Khai báo trong `render.yaml` | Chưa xác minh giá trị runtime |
| `LOG_LEVEL` | Khai báo trong `render.yaml` | Chưa xác minh giá trị runtime |

## Kết quả gọi service công khai

Kiểm tra trực tiếp ngày 2026-09-29. Request có key dùng khóa từ `.env` cục bộ qua HTTPS; khóa không được in ra log kiểm tra hay file này.

```text
GET  https://day12-agent-bveh.onrender.com/health
HTTP 200  {"status":"ok","service":"day12-agent","version":"1.0.0"}

GET  https://day12-agent-bveh.onrender.com/ready
HTTP 200  {"status":"ready","redis":true}

POST https://day12-agent-bveh.onrender.com/ask  (không có X-API-Key)
HTTP 401  {"detail":"invalid or missing API key"}

POST https://day12-agent-bveh.onrender.com/ask  (có X-API-Key, X-User-Id: cp5-smoke)
HTTP 200  response có answer, cost_usd, history_length, tokens, user_id

POST /ask 15 lần liên tiếp (X-User-Id: cp5-ratecheck-20260929)
HTTP 200 x 10, sau đó HTTP 429 x 5
```

Ảnh minh chứng đã lưu trong repository: [dashboard Render](screenshots/dashboard.png) hiển thị trạng thái Live và commit `fd1fd11`; [kết quả `/health`](screenshots/health.png) là ảnh chụp trình duyệt từ Public URL.

Checkpoint chính thức đã chạy với URL trên và khóa trong `.env` cục bộ:

```text
pytest tests/test_cp5.py -v
9 passed, 4 skipped (4 test của LOCAL_FALLBACK không áp dụng)
```

`grade.py --no-bonus` báo 100/100 điểm tự động cho phần bắt buộc. Điểm này chưa thay thế phần kiểm tra thủ công của giảng viên về ảnh dashboard và các quan sát Docker.

## Kết quả đã kiểm chứng tại máy

```text
python -m pytest tests/test_cp1.py tests/test_cp2.py tests/test_cp3.py tests/test_cp4.py -q -m "not docker"
68 passed, 2 deselected
```

Hai test Docker bị bỏ qua vì máy kiểm tra hiện không có Docker CLI. Kết quả cục bộ dưới đây tách riêng với kiểm tra cloud ở trên.

Smoke test với Uvicorn cục bộ tại `127.0.0.1:8765` (Redis giả lập trong tiến trình):

```text
GET /health                 200
GET /ready                  200
POST /ask không có key      401
POST /ask có key, lượt 1   200  history_length=0
POST /ask có key, lượt 2   200  history_length=2
```

Đây là kết quả trên máy, chỉ dùng để kiểm tra tích hợp trước khi deploy.

## Kiểm tra thêm nếu có Docker

- Nếu cần hoàn tất câu 3 và 4 của `exercises.md`, cài Docker để đo kích thước image và quan sát cache khi build lại.

Render Key Value gói miễn phí không lưu bền dữ liệu khi instance khởi động lại. Vì vậy lịch sử, rate limit và tổng chi phí có thể bị reset; cần gói có persistence nếu muốn giới hạn chi phí đáng tin cậy trong production.

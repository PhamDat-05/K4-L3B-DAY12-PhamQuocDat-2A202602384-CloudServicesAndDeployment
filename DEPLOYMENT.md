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
| Public URL | Chưa có |
| Platform dự kiến | Render (`render.yaml`) |
| Ngày deploy | Chưa deploy |
| Trạng thái | Chưa hoàn thành CP5: chưa có tài khoản cloud; máy hiện cũng chưa có Docker để chạy phương án dự phòng |

## Biến môi trường trên cloud

Các biến sau **chưa được set trên cloud**. Khi tạo Render Blueprint, cung cấp secret qua dashboard; không chép giá trị vào repository.

| Biến | Trạng thái | Nguồn dự kiến |
| --- | --- | --- |
| `PORT` | Chưa kiểm tra | Render cấp lúc chạy |
| `AGENT_API_KEY` | Chưa set | Render dashboard, tạo khóa riêng |
| `REDIS_URL` | Chưa set | Render Key Value qua `fromService` trong Blueprint |
| `RATE_LIMIT_PER_MINUTE` | Khai báo trong `render.yaml` | Cấu hình không bí mật |
| `MONTHLY_BUDGET_USD` | Khai báo trong `render.yaml` | Cấu hình không bí mật |
| `LOG_LEVEL` | Khai báo trong `render.yaml` | Cấu hình không bí mật |

## Kết quả đã kiểm chứng tại máy

```text
python -m pytest tests/test_cp1.py tests/test_cp2.py tests/test_cp3.py tests/test_cp4.py -q -m "not docker"
68 passed, 2 deselected
```

Hai test Docker bị bỏ qua vì không có Docker CLI. Chưa gọi được `/health`, `/ready` hoặc `/ask` qua một URL công khai; chưa có ảnh dashboard hay ảnh health. Không dùng log của TestClient làm bằng chứng cloud.

Smoke test với Uvicorn cục bộ tại `127.0.0.1:8765` (Redis giả lập trong tiến trình):

```text
GET /health                 200
GET /ready                  200
POST /ask không có key      401
POST /ask có key, lượt 1   200  history_length=0
POST /ask có key, lượt 2   200  history_length=2
```

Đây là kết quả trên máy, chưa phải bằng chứng cho CP5.

## Các bước còn lại để hoàn thành CP5

1. Tạo tài khoản Render và kết nối repo GitHub này.
2. Tạo Blueprint từ `render.yaml`, nhập `AGENT_API_KEY` riêng trong dashboard, kiểm tra Render Key Value và `REDIS_URL`.
3. Deploy service; lấy Public URL thật và ghi vào bảng trên.
4. Gọi `/health` (200), `/ready` (200), `/ask` không có key (401), rồi `/ask` có key (200). Giữ khóa trong `.env` hoặc dashboard, không ghi vào file này.
5. Ghi output HTTP thực tế, chụp dashboard và kết quả `/health` vào `screenshots/`, sau đó chạy `pytest tests/test_cp5.py -v`.

Render Key Value gói miễn phí không lưu bền dữ liệu khi instance khởi động lại. Vì vậy lịch sử, rate limit và tổng chi phí có thể bị reset; cần gói có persistence nếu muốn giới hạn chi phí đáng tin cậy trong production.

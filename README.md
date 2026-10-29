# Exam Online — Backend

Backend FastAPI cho hệ thống thi online (https://dist-ophczzau.devinapps.com/).

- **Stack**: Python 3.12, FastAPI, SQLAlchemy (SQLite), `python-docx`, WebSocket cho leaderboard real-time.
- **Cấu trúc**:
  - `app/main.py` — FastAPI app, mount `/uploads` cho ảnh, đăng ký router admin/student/ws.
  - `app/routers/admin.py` — đăng nhập admin (JWT), quản lý đề/câu hỏi/mã thi/lượt làm/phân quyền.
  - `app/routers/student.py` — học sinh nhập mã, làm bài, nộp bài, xem kết quả.
  - `app/routers/ws.py` — WebSocket bảng xếp hạng theo từng đề.
  - `app/docx_parser.py` — parser `.docx` → câu hỏi (trắc nghiệm A/B/C/D, đúng/sai 4 ý a/b/c/d, **trả lời ngắn**), trích ảnh + công thức OMML.
  - `app/scoring.py` — chấm điểm tự động.
  - `app/seed.py` — seed admin mặc định.

## Chạy local

```bash
pip install uv
uv sync
EXAM_DB_PATH=./exam.db EXAM_UPLOAD_DIR=./uploads \
    uv run uvicorn app.main:app --reload --port 8080
```

Mở http://localhost:8080/healthz để kiểm tra.

## Biến môi trường

| Tên | Mặc định | Ý nghĩa |
|---|---|---|
| `EXAM_DB_PATH` | `/data/exam.db` | Đường dẫn DB SQLite |
| `EXAM_UPLOAD_DIR` | `/data/uploads` | Thư mục lưu ảnh upload |
| `EXAM_JWT_SECRET` | (random) | Secret ký JWT — đặt cố định cho prod |
| `PORT` | `8080` | Port HTTP |

## Deploy lên Fly.io

```bash
flyctl launch --copy-config --no-deploy   # bỏ qua nếu đã có app
flyctl volumes create exam_data --region sin --size 1
flyctl deploy
```

App đang chạy tại: https://exam-online-backend-fxiwahty.fly.dev/

## Loại câu hỏi hỗ trợ

| `type` | Mô tả | Đáp án |
|---|---|---|
| `mc` | Trắc nghiệm A/B/C/D, chọn 1 | chữ đỏ, highlight hoặc gạch chân ký hiệu/nội dung đáp án trong `.docx` |
| `tf` | 4 ý đúng/sai (a, b, c, d) | đọc từ bảng `Đúng/Sai` ngay sau câu |
| `sa` | Trả lời ngắn (số/chuỗi) | dòng `Đáp án: ...` ngay sau câu |

Chấm điểm tự động — `sa` so sánh không phân biệt hoa/thường và bỏ qua khoảng trắng.

### Đánh dấu đáp án trắc nghiệm trong Word

- Mỗi câu bắt đầu bằng `Câu N.`, có đủ bốn phương án `A.`, `B.`, `C.`, `D.` (hoặc `A)`, `B)`, `C)`, `D)`).
- Chỉ đánh dấu **một** phương án đúng: đổi màu chữ sang đỏ (kể cả đỏ đậm), dùng Text Highlight Color hoặc gạch chân. Có thể đánh dấu riêng chữ A/B/C/D hoặc phần nội dung đáp án.
- Hỗ trợ phương án trên cùng dòng, mỗi phương án một đoạn, hoặc bảng phương án A/B/C/D trong Word (đọc theo từng hàng). Định dạng từ character/paragraph style và màu theme cũng được xét.
- Nếu không có hoặc có nhiều phương án được đánh dấu, đáp án để trống kèm cảnh báo; giáo viên cần kiểm tra lại trong trang sửa đề. Tránh dùng các định dạng đánh dấu đáp án chỉ để trang trí phương án khác.
- Bản sửa áp dụng khi **tải DOCX mới**. Đề đã nhập trước đó cần sửa đáp án trong trang quản trị hoặc tải lại file để tạo đề mới.

Kiểm thử bộ đọc DOCX và API tải đề (SQLite tạm, không chạm dữ liệu đang chạy):

```bash
uv run --frozen python -m unittest discover -s tests -v
```

# NCKH: Vietnam Energy & Macro Forecasting

Nghiên cứu và dự báo điện năng cùng chỉ số kinh tế vĩ mô của Việt Nam bằng mô hình Temporal Fusion Transformer (TFT), kết hợp dữ liệu tải điện, thời tiết, và kinh tế để đánh giá xu hướng và lập dự báo.

## Tài liệu chính
- [docs/All_data.md](docs/All_data.md): Tổng quan dữ liệu thô và cấu trúc lưu trữ.
- [docs/Crawl_guide.md](docs/Crawl_guide.md): Hướng dẫn thu thập dữ liệu (API/Web/SQL).
- [docs/DATA_AVAILABILITY.md](docs/DATA_AVAILABILITY.md): Phạm vi và mức độ sẵn có của dữ liệu.

## Thư mục liên quan
- `data/`: dữ liệu raw và tổng hợp.
- `notebook/`: notebook theo quy tắc đặt tên (xem notebook/README.md).
- `src/`: mã nguồn thu thập, xử lý, và pipeline mô hình.

## Mục tiêu ngắn gọn
- Chuẩn hóa pipeline thu thập và làm sạch dữ liệu điện, thời tiết, kinh tế.
- Xây dựng đặc trưng và huấn luyện TFT cho bài toán dự báo.
- Đánh giá, so sánh và báo cáo kết quả trên bộ dữ liệu Việt Nam.

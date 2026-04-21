# V. Chiến lược Pre-training đa quốc gia

Tài liệu này trình bày các phương pháp và chiến lược được áp dụng trong giai đoạn huấn luyện mô hình cơ sở (Pre-training) trên dữ liệu của 20 quốc gia trước khi thực hiện Transfer Learning sang thị trường Việt Nam.

## 1. Phương pháp huấn luyện (Training Type)
- **Chiến lược**: **Train chung tất cả quốc gia (Joint Global Training)**.
- **Mô tả**: Thay vì huấn luyện các mô hình riêng biệt cho từng quốc gia hoặc sử dụng Multi-task Learning với các đầu ra khác nhau, chúng tôi gộp toàn bộ dữ liệu từ 20 quốc gia vào một tập dữ liệu hợp nhất duy nhất.
- **Mục tiêu**: 
    - Cho phép mô hình TFT học được các mẫu hình (patterns) chung về mối quan hệ giữa thời tiết và sản lượng phát điện trên quy mô toàn cầu.
    - Tận dụng sự đa dạng về các biến cố năng lượng (khủng hoảng giá, biến động thời tiết cực đoan) ở các quốc gia khác nhau để tăng tính bản lĩnh (robustness) cho mô hình.

## 2. Nhận diện quốc gia và Embedding (Country Identity)
- **Tính năng**: Sử dụng **Country Embedding** thông qua biến định danh `entity`.
- **Cấu hình trong TFT**: Biến `entity` (mã quốc gia) được đưa vào tham số `static_categoricals` của `TimeSeriesDataSet`.
- **Cơ chế**:
    - Mỗi quốc gia được đại diện bởi một vector embedding có kích thước cố định (tự động tính toán dựa trên `hidden_size`).
    - Vector này là đặc trưng tĩnh, giúp mô hình phân biệt được sự khác biệt về quy mô hạ tầng, chính sách năng lượng và đặc điểm địa lý riêng biệt của từng thực thể, ngay cả khi chúng cùng học chung một bộ tham số trọng số.

## 3. Chiến lược xáo trộn và tổ chức dữ liệu (Training Strategy)
- **Kỹ thuật**: **Shuffle toàn bộ dataset (Global Shuffling)**.
- **Thực hiện**: Trong `DataLoader`, tham số `shuffle=True` được kích hoạt cho tập huấn luyện.
- **Đặc điểm**:
    - Trong mỗi **Iteration**, các mẫu dữ liệu (window) từ các quốc gia khác nhau (ví dụ: một mẫu từ Đức năm 2020, một mẫu từ Việt Nam năm 2022) được xáo trộn và đưa vào cùng một Batch.
    - Việc này giúp Gradient của mô hình được cập nhật dựa trên mức độ lỗi trung bình của toàn bộ các quốc gia, ép mô hình phải tìm ra các trọng số có khả năng tổng quát hóa tốt nhất thay vì học thuộc lòng từng quốc gia một.

## 4. Lý do lựa chọn chiến lược này cho Bài báo (Paper Justification)
1. **Khắc phục giới hạn dữ liệu ngắn**: Dữ liệu Việt Nam chỉ có 72 tháng, quá ngắn để mô hình Deep Learning phức tạp như TFT có thể hội tụ mà không Overfitting. Việc học từ 151 nhóm chuỗi thời gian toàn cầu cung cấp "ngữ cảnh" phong phú hơn nhiều.
2. **Học đặc trưng chung (Shared Representations)**: Các đặc trưng như "tính mùa vụ của thủy điện" hay "hiệu suất pin mặt trời theo nhiệt độ" có tính tương đồng cao giữa các quốc gia. Huấn luyện chung giúp mô hình nắm bắt các quy luật vật lý này tốt hơn.
3. **Chuyển đổi tri thức (Knowledge Transfer)**: Tạo ra một "Global Brain" về năng lượng, giúp giai đoạn Fine-tuning tại Việt Nam diễn ra nhanh hơn và đạt độ chính xác cao hơn so với huấn luyện từ đầu (Train from scratch).

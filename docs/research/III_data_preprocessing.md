# III. Xử lý dữ liệu trước khi Pretrain

Phần này mô tả các kỹ thuật tiền xử lý dữ liệu được áp dụng để chuẩn bị đầu vào cho mô hình Temporal Fusion Transformer (TFT).

## 1. Xử lý giá trị thiếu và Trùng lặp
Việc xử lý dữ liệu được thực hiện qua các bước:
*   **Hợp nhất dữ liệu (Deduplication)**: Thực hiện `agg(mean)` khi hợp nhất dữ liệu thời tiết và sản lượng điện để xử lý vấn đề dư thừa dữ liệu phát hiện qua EDA.
*   **Xử lý giá trị thiếu (Missing Value)**:
    - **Zero Fill (`fillna(0)`)**: Áp dụng cho các đặc trưng trễ (Lag features) và đặc trưng thời tiết sau khi dịch chuyển thời gian (`shift(1)`).
*   **Loại bỏ (Drop)**: Các chuỗi thời gian (Time series) có độ dài < **30 tháng** sẽ bị loại bỏ để đảm bảo cửa sổ trượt Encoder-Decoder.
*   **Xử lý ngoại lệ (Outliers)**: Giữ nguyên các giá trị spike thực tế (như khủng hoảng năng lượng 2022) để mô hình học được các biến động thị trường.

## 2. Chuẩn hóa dữ liệu (Normalization)
Do EDA cho thấy sự chênh lệch quy mô cực lớn giữa các quốc gia và giữa các biến (ví dụ: GDP và tỷ giá), ta áp dụng:
*   **GroupNormalizer (với Transformation: Softplus)**: 
    - Thực hiện chuẩn hóa riêng biệt cho từng cặp (Quốc gia, Loại hình nguồn).
    - Sử dụng hàm `Softplus` trong quá trình biến đổi để đảm bảo các giá trị sản lượng điện được dự báo luôn không âm (phù hợp thực tế vật lý).

## 3. Mã hóa thời gian (Time Encoding)
Vì dữ liệu có tần suất hàng tháng, các đặc trưng thời gian sau đây đã được thêm vào:
*   **Mã hóa số nguyên:** `month` (1-12), `quarter` (1-4), `year`.
*   **Mã hóa vòng quay (Cyclical Encoding):** `month_sin` và `month_cos` được tính toán từ biến tháng để giúp mô hình học được tính chu kỳ mùa vụ (tháng 12 gần với tháng 1).
*   **Lưu ý:** Do đặc thù dữ liệu tháng, các biến như `day_of_week`, `holiday`, `weekend` không được áp dụng trong giai đoạn Pretrain đa quốc gia này.

## 4. Các đặc trưng tĩnh (Static Features)
TFT tận dụng các đặc trưng tĩnh để phân biệt đặc tính vận hành giữa các nhóm:
*   **country id (Thực thể):** Biến `entity` được đưa vào làm `static_categoricals`.
*   **energy type (Loại hình năng lượng):** Biến `series` (Coal, Gas, Hydro, Solar, Wind...) được đưa vào làm `time_varying_known_categoricals` (hoặc static tùy theo cách cấu hình metadata nhưng ở đây nó đóng vai trò định danh nhóm chính).
*   **region id:** Hiện tại chưa sử dụng biến vùng miền cụ thể, thay vào đó mô hình tập trung vào định danh quốc gia (`entity`).

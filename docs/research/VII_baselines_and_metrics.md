# VII. Mô hình đối chứng và Các chỉ số đánh giá

Tài liệu này xác định các mô hình so sánh và tập hợp các chỉ số định lượng được sử dụng để đánh giá hiệu quả của mô hình TFT trong nghiên cứu.

## 1. Mô hình đối chứng (Baseline Model)

Để làm nổi bật ưu thế của kiến trúc Temporal Fusion Transformer (TFT) kết hợp Transfer Learning, chúng tôi sử dụng mô hình sau làm đối chứng:

### 1.1. Long Short-Term Memory (LSTM)
- **Kiến trúc**: Sử dụng `RecurrentNetwork` từ thư viện `pytorch_forecasting`.
- **Đặc điểm**: Đây là mô hình học sâu truyền thống trong xử lý chuỗi thời gian, tập trung vào việc học các phụ thuộc dài hạn thông qua các cổng (gates).
- **Trạng thái**: Có file huấn luyện thực tế `LSTM_train.ipynb` và `LSTM_transfer_learning.ipynb` trong dự án.
- **Vai trò**: Dùng để so sánh độ chính xác dự báo (MAE, RMSE) so với TFT trên cùng một tập dữ liệu Việt Nam.

### 1.2. Mô hình Pre-trained TFT (Không Fine-tune)
- Sử dụng trực tiếp Checkpoint từ giai đoạn huấn luyện đa quốc gia để dự báo cho Việt Nam mà không qua bước cập nhật trọng số. Việc này giúp chứng minh giá trị của giai đoạn fine-tuning trong việc thích nghi với đặc thù năng lượng nội địa.

## 2. Các chỉ số đánh giá (Evaluation Metrics)

Chúng tôi sử dụng 4 chỉ số chính để đánh giá sai số và độ tin cậy của mô hình. Trong đó, đơn vị tính của sản lượng điện là **TWh**.

| Metric | Công thức / Ý nghĩa | Vai trò |
| :--- | :--- | :--- |
| **MAE** | Mean Absolute Error | Đánh giá sai số trung bình tuyệt đối theo đơn vị TWh. |
| **RMSE** | Root Mean Squared Error | Trừng phạt các sai số lớn, giúp đánh giá độ ổn định của dự báo. |
| **SMAPE** | Symmetric Mean Absolute Percentage Error | Đánh giá sai số theo tỷ lệ phần trăm (%), không bị ảnh hưởng bởi quy mô (scale). |
| **WAPE** | Weighted Absolute Percentage Error | Chỉ số quan trọng trong ngành điện để đánh giá tổng sai số trung bình trên tổng sản lượng. |

## 3. Kết quả thực nghiệm chi tiết

Kết quả dưới đây được trích xuất trực tiếp từ các notebook phiên bản V3, đại diện cho hiệu năng tốt nhất của mô hình.

### 3.1. Giai đoạn Pre-training (Mô hình Toàn cầu)
Dữ liệu đánh giá trên tập Validation của 20 quốc gia (Ember dataset). Mô hình học các đặc trưng chung về biến động sản lượng điện theo thời tiết.

| Chỉ số | Giá trị | Đơn vị |
| :--- | :--- | :--- |
| **Best Val Loss** | 0.4873 | QuantileLoss |
| **MAE** | 0.6513 | TWh |
| **RMSE** | 1.1450 | TWh |
| **SMAPE** | 25.98 | % |
| **WAPE** | 11.94 | % |

### 3.2. Giai đoạn Transfer Learning (Mô hình Việt Nam)
Dữ liệu Việt Nam được chia thành 3 tập: Train (54 tháng), Validation (12 tháng) và Test/Holdout (6 tháng cuối năm 2024).

**Kết quả trên tập Test Holdout (Tháng 7/2024 - 12/2024):**

| Chỉ số | TFT (Đề xuất) | LSTM (Baseline) | Ghi chú |
| :--- | :--- | :--- | :--- |
| **MAE** | **0.9406** | 0.9589 | Thấp hơn là tốt hơn |
| **RMSE** | **1.3959** | 4.9724 | TFT ổn định hơn đáng kể |
| **SMAPE** | **20.99%** | N/A | |
| **WAPE** | **17.62%** | N/A | |

### 3.3. Phân chia tập dữ liệu Việt Nam
Để đảm bảo tính khách quan, dữ liệu Việt Nam (72 tháng) được phân chia như sau:

| Split | Thời gian bắt đầu | Thời gian kết thúc | Số tháng |
| :--- | :--- | :--- | :--- |
| **Train** | 2019-01-01 | 2023-06-01 | 54 |
| **Validation** | 2023-07-01 | 2024-06-01 | 12 |
| **Holdout (Test)** | 2024-07-01 | 2024-12-01 | 6 |

## 4. Nhận xét
Mô hình TFT sau khi được chuyển đổi (Transfer Learning) từ bộ trọng số toàn cầu cho thấy khả năng thích nghi tốt với dữ liệu Việt Nam. Đặc biệt, sai số RMSE của TFT thấp hơn nhiều so với LSTM (1.39 so với 4.97), cho thấy TFT có khả năng xử lý các điểm biến động mạnh (outliers) và tính mùa vụ của năng lượng tái tạo hiệu quả hơn các kiến trúc RNN truyền thống.

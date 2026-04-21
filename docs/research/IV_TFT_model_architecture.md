# IV. Kiến trúc mô hình TFT

Tài liệu này chi tiết hóa cấu hình kiến trúc và các tham số huấn luyện của mô hình Temporal Fusion Transformer (TFT) cho cả hai giai đoạn: Huấn luyện cơ sở (Pre-training) và Học chuyển đổi (Transfer Learning).

## 1. Framework và Thư viện
- **Framework chính**: [PyTorch Forecasting](https://pytorch-forecasting.readthedocs.io/)
- **Nền tảng huấn luyện**: [PyTorch Lightning](https://www.pytorchlightning.ai/)
- **Lý do lựa chọn**: PyTorch Forecasting cung cấp triển khai chuẩn hóa của TFT với khả năng hỗ trợ biến tĩnh (static), biến động biểu kiến (known) và biến động không quan sát được (unknown), cùng các công cụ chuẩn hóa dữ liệu mạnh mẽ.

## 2. Thông số kiến trúc mô hình (Cấu hình dùng chung)
Kiến trúc cốt lõi được giữ nhất quán giữa hai giai đoạn để đảm bảo khả năng tương thích tham số:

| Tham số | Giá trị | Ý nghĩa |
| :--- | :--- | :--- |
| **hidden_size** | 128 | Kích thước của trạng thái ẩn (hidden state), quyết định năng lực biểu diễn của mô hình. |
| **lstm_layers** | 2 | Số lượng lớp LSTM trong thành phần xử lý chuỗi. |
| **attention_head_size** | 4 | Số lượng đầu Attention trong cơ chế Multi-head Attention. |
| **hidden_continuous_size** | 32 | Kích thước ẩn cho các biến số liên tục. |
| **output_size** | 5 | Tương ứng với 5 phân vị (quantiles) dự báo. |

## 3. Cấu hình huấn luyện cụ thể

### 3.1. Giai đoạn Pre-training (`trainTFT_v3.ipynb`)
Mục tiêu là học các đặc trưng chung từ 20 quốc gia.

- **Optimizer**: `AdamW` (Lựa chọn thay thế ổn định cho Adam khi có Weight Decay).
- **Loss Function**: `QuantileLoss` (Các phân vị: `0.1, 0.25, 0.5, 0.75, 0.9`).
- **Learning Rate**: `3e-4` (Thấp để đảm bảo sự hội tụ mịn).
- **Batch Size**: 64.
- **Dropout**: 0.15.
- **Max Epochs**: 80 (kết hợp Early Stopping với patience = 12).
- **Gradient Clipping**: 0.1 (ngăn chặn hiện tượng bùng nổ gradient).

### 3.2. Giai đoạn Transfer Learning (`transfer_learning_v3.ipynb`)
Sử dụng trọng số từ mô hình Pre-trained để hội tụ trên dữ liệu Việt Nam.

- **Chiến lược Fine-tuning**: Hai giai đoạn (Two-phase training).
    - **Giai đoạn 1 (Frozen Backbone)**: Đóng băng các lớp LSTM, chỉ huấn luyện các lớp Variable Selection Network (VSN) và Head.
        - **Learning Rate**: `1e-3`.
        - **Epochs**: 15.
    - **Giai đoạn 2 (Unfrozen)**: Giải phóng toàn bộ tham số để tinh chỉnh sâu.
        - **Learning Rate**: `3e-5` (Rất thấp để tránh phá hủy các đặc trưng đã học).
        - **Epochs**: 40.
- **Batch Size**: 32 (Giảm do tập dữ liệu Việt Nam nhỏ hơn).
- **Dropout**: 0.2 (Tăng nhẹ để tăng hiệu quả khả năng tổng quát hóa trên tập dữ liệu hẹp).

## 4. Đặc điểm nổi bật của kiến trúc
1. **Variable Selection Networks (VSN)**: Giúp mô hình tự động chọn lọc các đặc trưng quan trọng nhất (như nhiệt độ hoặc các biến trễ) tại mỗi bước thời gian.
2. **Gated Residual Networks (GRN)**: Cho phép mô hình bỏ qua các thành phần không cần thiết, giúp tránh tình trạng quá tải tham số đối với các chuỗi thời gian đơn giản.
3. **Multi-head Attention**: Học các mối quan hệ phụ thuộc dài hạn, giúp giải thích được tính mùa vụ trong dự báo sản lượng điện.
4. **Quantile Output**: Cung cấp khoảng tin cậy cho dự báo (P10, P50, P90), hỗ trợ các nhà quản lý hệ thống điện đánh giá rủi ro thiếu hụt hoặc dư thừa công suất.

# VI. Học chuyển đổi (Transfer Learning) sang Việt Nam

Tài liệu này mô tả chi tiết quy trình áp dụng kỹ thuật Transfer Learning để thích nghi mô hình TFT đã được huấn luyện cơ sở (Pre-trained) sang bài toán dự báo sản lượng điện tại Việt Nam.

## 1. Cơ sở dữ liệu Việt Nam
Dữ liệu được thu thập và tổng hợp từ nhiều nguồn uy tín để đảm bảo tính toàn diện:

- **Sản lượng điện**: Số liệu từ **Ember** và **Tổng cục Thống kê (GSO)** về sản lượng phát điện hàng tháng của các loại hình: Thủy điện, Nhiệt điện Than, Nhiệt điện Khí, Điện Mặt trời và Điện Gió.
- **Dữ liệu thời tiết**: Truy xuất từ **Open-Meteo** cho các trạm khí tượng đại diện tại Việt Nam, bao gồm: Nhiệt độ trung bình, Độ ẩm, Tốc độ gió và Bức xạ mặt trời.
- **Thời gian**: Từ tháng 01/2019 đến tháng 12/2024 (Tổng cộng 72 tháng/mẫu cho mỗi loại hình năng lượng).
- **Độ phân giải (Granularity)**: Theo tháng (Monthly).

## 2. Chiến lược Học chuyển đổi (Transfer Learning Strategy)
Do tập dữ liệu Việt Nam có kích thước nhỏ (72 dòng cho mỗi nhóm), chúng tôi áp dụng chiến lược **Fine-tuning hai giai đoạn** để đảm bảo mô hình không bị quá khớp (overfitting) và giữ lại được các tri thức quý giá từ mô hình toàn cầu.

### Giai đoạn 1: Đóng băng lớp trích xuất đặc trưng (Phase 1: Frozen Backbone)
- **Cơ chế**: Đóng băng (Freeze) toàn bộ trọng số của các lớp LSTM (Temporal Processing) và Attention. Chỉ cho phép cập nhật trọng số tại các lớp **Variable Selection Network (VSN)** và các lớp **Output Head**.
- **Mục tiêu**: Ép mô hình tập trung học cách lựa chọn và trọng số hóa các biến đầu vào dựa trên đặc thù khí hậu và cơ cấu điện của Việt Nam mà không làm thay đổi khả năng xử lý chuỗi thời gian đã học được.
- **Thông số**: 
    - `Learning Rate`: 1e-3.
    - `Epochs`: Tối đa 15.

### Giai đoạn 2: Tinh chỉnh toàn bộ (Phase 2: Full Fine-tuning)
- **Cơ chế**: Mở băng (Unfreeze) hoàn toàn các tham số của mô hình.
- **Mục tiêu**: Cho phép toàn bộ mạng lưới TFT tinh chỉnh sâu để đạt được độ chính xác tối ưu trên dữ liệu nội địa.
- **Thông số**: 
    - `Learning Rate`: 3e-5 (Sử dụng tốc độ học cực nhỏ để tránh làm hỏng các đặc trưng cốt lõi đã học được ở giai đoạn pre-train).
    - `Epochs`: Tối đa 40.

## 3. Quản lý quá trình huấn luyện
Để đảm bảo tính ổn định và hiệu quả, quy trình huấn luyện tích hợp các cơ chế kiểm soát:

- **Early Stopping**: Tự động dừng huấn luyện nếu hàm mất mát trên tập Validation không cải thiện trong một số chu kỳ nhất định (Patience: 6 cho giai đoạn 1 và 10 cho giai đoạn 2).
- **Learning Rate Scheduler**: Sử dụng tốc độ học cố định cho từng giai đoạn nhưng được tinh chỉnh thủ công để phù hợp với độ phức tạp của bài toán chuyển đổi.
- **Validation Cutoff**: Sử dụng 12 tháng cuối cùng (năm 2024) làm tập kiểm chứng và đánh giá cuối cùng để mô phỏng thực tế dự báo trong tương lai.

## 4. Ý nghĩa của phương pháp
Chiến lược này cho phép chuyển dịch thành công "kinh nghiệm" dự báo từ các nước phát triển (Mỹ, Đức, Úc...) sang bối cảnh Việt Nam, giúp giảm thiểu đáng kể sai số dự báo trong điều kiện dữ liệu lịch sử hạn chế, đặc biệt là đối với các loại hình năng lượng tái tạo mới như Điện Mặt trời và Điện Gió.

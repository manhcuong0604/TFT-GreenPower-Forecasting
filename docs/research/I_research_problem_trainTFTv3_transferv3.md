# I. Mục tiêu nghiên cứu và Bài toán (trainTFT_v3 + transfer_learning_v3)

## 1. Bài toán cụ thể
Bài toán được thực hiện trong hai notebook (v3) là **Dự báo sản lượng phát điện (Power Generation Forecasting)**, không phải dự báo phụ tải (Load Forecasting).

Đầu ra của mô hình là biến `generation_TWh` (Sản lượng điện tính theo TWh) cho từng nhóm nguồn điện cụ thể bao gồm: **Coal (Than), Gas, Hydro (NaN), Solar (Mặt trời), Wind (Gió)**. Dữ liệu được tổ chức theo cấu trúc `group entity` (Quốc gia) và `series` (Nguồn điện).

## 2. Phạm vi dự báo
Trong pipeline `transfer_learning_v3`, dữ liệu Việt Nam được sử dụng như một thực thể (entity) duy nhất và thực hiện dự báo chi tiết theo từng loại hình nguồn điện. 

Vì vậy, bài toán hiện tại là **Dự báo cấp quốc gia theo loại hình nguồn năng lượng**, chưa thực hiện phân rã theo cấp vùng miền.

#### 1. Dữ liệu Việt Nam
- **Dải dữ liệu**: Từ tháng 01/2019 đến tháng 12/2024 (72 tháng). Dữ liệu này bao gồm sản lượng phát điện theo loại hình nguồn, các chỉ số kinh tế vĩ mô (IPI, CPI, GDP, FDI, giá năng lượng) và dữ liệu thời tiết.
- **Tình trạng**: Dữ liệu sau xử lý không có giá trị thiếu (missing values) và đảm bảo tính liên tục theo tháng.

#### 2. Dữ liệu Đa quốc gia (Pretrain)
- **Quy mô**: 20 quốc gia trên thế giới được sử dụng để xây dựng mô hình cơ sở.
- **Nguồn**: Ember (dữ liệu điện) và Open-Meteo (dữ liệu thời tiết).
- **Vấn đề EDA**: Phát hiện và xử lý khoảng 58.9% dòng dữ liệu trùng lặp (duplicates) phát sinh trong quá trình merge thời tiết bằng phương pháp lấy trung bình.

## 4. Cửa sổ dự báo (Forecast Horizon) và Kiểu dự báo
- **Horizon (Đoạn dự báo):** 6 bước (tương đương 6 tháng).
- **Kiểu dự báo:** Dự báo đa bước (Multi-step forecasting) từ tháng t+1 đến t+6, không phải chỉ dự báo một bước duy nhất.
- **Cửa sổ lịch sử (Encoder length):** 18 tháng đối với pipeline Transfer Learning.

## 5. Phân loại bài toán
Đây là bài toán **Time-series Multi-step Forecasting** áp dụng cho lĩnh vực sản lượng phát điện.

## 6. Mục tiêu của Transfer Learning (Học chuyển đổi)
Mục tiêu chính trong phiên bản `transfer_learning_v3` bao gồm:
- Tận dụng Checkpoint đã được huấn luyện (pretrained) `tft_v3_best.ckpt` từ bộ dữ liệu đa quốc gia (20 nước).
- Thực hiện Fine-tune trên bộ dữ liệu Việt Nam vốn có độ dài ngắn hơn (72 tháng) và có sự biến động lớn giữa các nhóm nguồn điện.
- Cải thiện độ ổn định của quá trình huấn luyện và nâng cao khả năng tổng quát hóa của mô hình trong bối cảnh dữ liệu nội địa hạn chế.

## 7. Tại sao cần Transfer Learning?
Việc áp dụng Transfer Learning là cần thiết vì:

1. **Dữ liệu Việt Nam quá ngắn:** Với 72 tháng, cửa sổ Encoder (18) và Decoder (6) chiếm tỷ trọng lớn trên tổng dòng thời gian. Nếu huấn luyện từ đầu (train from scratch), mô hình dễ rơi vào tình trạng Overfitting hoặc có dao động lỗi rất cao.
2. **Đặc trưng vận hành hệ thống điện có tính phổ quát:** Mối quan hệ giữa tính mùa vụ, thời tiết (nhiệt độ, lượng mưa) và sản lượng điện tồn tại tương đồng ở nhiều quốc gia. Pre-training giúp mô hình học trước các dạng mẫu (pattern) chung này.
3. **Thích nghi với đặc thù Việt Nam:** Sự khác biệt về phân phối nguồn điện (ví dụ: Gió và Mặt trời tại VN biến động mạnh từ năm 2019) được xử lý thông qua chiến lược Fine-tune 2 giai đoạn (Freeze -> Unfreeze), giúp mô hình thích nghi mềm mại mà không gây sốc tham số.
4. **Hiệu quả thực nghiệm:** Kết quả trong notebook `transfer_learning_v3` cho thấy mức sai số thấp hơn so với các phương pháp truyền thống khi dữ liệu ít:
   - **MAE: 0.9406 TWh**
   - **RMSE: 1.3959 TWh**
   - **WAPE: 17.62%** (trên tập Validation/Holdout).
   - **SMAPE: 20.99%**

## 8. Ghi chú kết quả từ giai đoạn Pre-training (trainTFT_v3)
Mô hình Pre-trained đạt các chỉ số sau trên tập dữ liệu đa quốc gia:
- **Best val_loss:** 0.4873
- **MAE:** 0.6513 TWh
- **RMSE:** 1.1450 TWh
- **SMAPE:** 25.98%
- **WAPE:** 11.94%

Checkpoint này được sử dụng làm đầu vào trực tiếp cho giai đoạn Transfer Learning sang Việt Nam.

## 9. Kết luận cho phần mở đầu báo cáo
Nghiên cứu tập trung giải quyết bài toán dự báo đa bước sản lượng phát điện cấp quốc gia theo từng nhóm nguồn điện. Chiến lược cốt lõi là sử dụng mô hình Transformer (TFT) với phương pháp huấn luyện chuyển đổi (Pretrain đa quốc gia -> Fine-tune Việt Nam). Phương pháp này giúp vượt qua rào cản về giới hạn dữ liệu trong nước và tăng cường độ tin cậy cho kết quả dự báo hold-out.

## 10. Lưu ý về việc kiểm chứng kết quả
Theo ghi chép trong notebook `transfer_learning_v3`, các chỉ số MAE/RMSE/WAPE hiển thị trong phần tổng kết Validation và Holdout (Test) có xu hướng trùng lặp hoặc sát nhau trong một số phiên chạy. Cần thực hiện kiểm tra lại việc tách biệt dữ liệu giữa tập Validation (12 tháng cuối của tập huấn luyện) và tập Test Holdout (6 tháng thực tế mới nhất) để đảm bảo tính khách quan trước khi chốt số liệu cuối cùng cho báo cáo. 
*(Lưu ý: Kết quả hiện tại lấy từ phiên chạy cuối cùng của notebook v3)*.

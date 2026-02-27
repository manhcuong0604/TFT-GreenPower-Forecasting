# ĐỀ XUẤT QUY TRÌNH NGHIÊN CỨU: DỰ BÁO SẢN LƯỢNG ĐIỆN VIỆT NAM QUA TRANSFER LEARNING

## 1. TỔNG QUAN VẤN ĐỀ (EXECUTIVE SUMMARY)

Trong nghiên cứu dự báo năng lượng tại các quốc gia đang phát triển như Việt Nam, khó khăn lớn nhất là **Sự khan hiếm dữ liệu (Data Scarcity)**. Với tập dữ liệu khoảng 500 dòng (tần suất tháng), việc áp dụng trực tiếp các mô hình Deep Learning hiện đại (như TFT, Informer) thường dẫn đến hiện tượng quá khớp (overfitting) và độ chính xác thấp.

**Giải pháp đề xuất:** Sử dụng chiến lược **Transfer Learning (Học chuyển giao)**.

- **Nguồn dữ liệu bổ trợ:** Dữ liệu điện năng và thời tiết toàn cầu (Ember Data).
    
- **Mục tiêu:** Huấn luyện mô hình hiểu các "quy luật vật lý" của năng lượng từ thế giới, sau đó tinh chỉnh (fine-tune) để học các "quy luật kinh tế - xã hội" đặc thù của Việt Nam.
    

## 2. QUY TRÌNH THỰC HIỆN CHI TIẾT

Quy trình được chia thành hai giai đoạn chiến lược để tối ưu hóa việc chuyển giao tri thức.

### Giai đoạn 1: Pre-training (Huấn luyện nền tảng trên dữ liệu Ember)

- **Dữ liệu:** Sử dụng hàng chục nghìn dòng dữ liệu từ nhiều quốc gia (Mỹ, Châu Âu, Trung Quốc...) có trong tập Ember. Dữ liệu này được gộp (aggregate) theo tháng để đồng nhất tần suất.
    
- **Biến số (Common Features):** Chỉ sử dụng các biến chung (Sản lượng điện theo loại, Nhiệt độ, Độ ẩm, Bức xạ mặt trời).
    
- **Cơ chế:** Mô hình tập trung học các quan hệ phi tuyến giữa thời tiết và năng lượng. Ví dụ: _Bức xạ mặt trời tác động thế nào đến sản lượng điện mặt trời theo tháng._
    
- **Kết quả:** Một mô hình "Pre-trained Model" chứa các trọng số (weights) đã hiểu rõ các quy luật năng lượng cơ bản.
    

### Giai đoạn 2: Fine-tuning (Cá nhân hóa cho Việt Nam)

- **Dữ liệu:** 500 dòng dữ liệu thực tế tại Việt Nam (2019 - 2025).
    
- **Biến số bổ sung (Unique Features):** Thêm 10 - 15 biến đặc thù (GDP, Chỉ số sản xuất công nghiệp, Chính sách giá điện FIT, Ngày lễ Tết...).
    
- **Cơ chế:** * Giữ lại (Freeze) các lớp đã học quy luật thời tiết.
    
    - Mở khóa (Unfreeze) lớp **Variable Selection Network (VSN)** để tích hợp các biến kinh tế mới.
        
    - Huấn luyện với tốc độ học (Learning Rate) thấp để mô hình không quên kiến thức cũ mà vẫn cập nhật được kiến thức mới.
        

## 3. CƠ CHẾ KỸ THUẬT VÀ GIẢI PHÁP CHO BIẾN SỐ PHỨC TẠP

Một thách thức lớn là tập dữ liệu Việt Nam có nhiều biến số hơn tập thế giới. Chúng ta giải quyết bằng kiến trúc mô hình **Temporal Fusion Transformer (TFT)**:

### 3.1. Variable Selection Network (VSN)

Đây là "bộ lọc" thông minh của TFT. Khi chuyển sang dữ liệu Việt Nam, VSN sẽ tự động đánh giá tầm quan trọng của 10-15 biến mới. Nếu một biến kinh tế (ví dụ: GDP) có ảnh hưởng mạnh, VSN sẽ tăng trọng số cho nó mà không làm hỏng các quy luật thời tiết đã học từ trước.

### 3.2. Static Covariates vs. Time-dependent Features

- **Biến tĩnh (Static):** Các đặc thù vùng miền hoặc loại hình nguồn điện.
    
- **Biến động (Dynamic):** Các biến kinh tế thay đổi theo tháng (CPI, GDP). TFT cho phép tách biệt các loại biến này, giúp mô hình xử lý đa dạng thông tin mà không bị rối loạn về mặt toán học.
    

## 4. MINH CHỨNG KHOA HỌC VÀ TÍNH KHẢ THI

### 4.1. Các bài nghiên cứu tương tự (Evidence)

- **Nghiên cứu của Jung et al. (2020):** Đã chứng minh việc sử dụng Transfer Learning từ dữ liệu điện năng của các tòa nhà lớn sang các tòa nhà nhỏ giúp giảm sai số (MAPE) tới 15-20% so với huấn luyện thông thường.
    
- **Mô hình TFT trong cuộc thi M5 Forecasting:** TFT đã chứng tỏ khả năng xử lý hàng nghìn chuỗi thời gian cùng lúc với các biến bổ trợ (covariates) phức tạp, đạt top đầu về độ chính xác trong dự báo chuỗi thời gian.
    
- **Ứng dụng tại các nước đang phát triển:** Nhiều nghiên cứu dự báo điện tại các quốc gia như Ấn Độ hay Brazil đã sử dụng dữ liệu từ các nước IEA (Cơ quan Năng lượng Quốc tế) để pre-train do thiếu hụt dữ liệu nội địa.
    

### 4.2. Tính khả thi trên tập dữ liệu 500 dòng

Trong thống kê học sâu (Deep Learning Statistics), 500 dòng là rất ít nếu học từ đầu (from scratch). Tuy nhiên, trong **Fine-tuning**, 500 dòng là con số **lý tưởng** để điều chỉnh các lớp trên cùng (Top layers) của mô hình.

- **Cộng đồng AI (HuggingFace, PyTorch Forecasting):** Luôn khuyến nghị sử dụng Transfer Learning khi dữ liệu < 1000 mẫu để tránh hiện tượng "Vanishing Gradient" hoặc "Overfitting".
    

## 5. KẾT LUẬN

Việc kết hợp **Dữ liệu Ember (Quy mô lớn)** và **Dữ liệu Việt Nam (Đặc thù cao)** thông qua mô hình **TFT** là một hướng đi đột phá cho bài nghiên cứu. Nó không chỉ giải quyết được sự thiếu hụt dữ liệu mà còn tạo ra một mô hình có khả năng giải thích cao (Interpretability) - điều mà các hội đồng khoa học luôn đánh giá cao.

**Người đề xuất:** Nguyễn Văn Tiến  
**Ngày:** 21/01/2026

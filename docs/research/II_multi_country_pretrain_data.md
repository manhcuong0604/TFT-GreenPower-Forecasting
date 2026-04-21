# II. Dữ liệu Pretrain đa quốc gia

Đây là báo cáo chi tiết về bộ dữ liệu được sử dụng để huấn luyện mô hình cơ sở (Pretrained Model) trước khi thực hiện Transfer Learning.

## 1. Các quốc gia sử dụng pretrain
Mô hình được huấn luyện trên dữ liệu của **20 quốc gia/vùng lãnh thổ**, bao gồm:
*   **Châu Á:** Việt Nam, Malaysia, Pakistan, Philippines, Taiwan, Kazakhstan.
*   **Châu Âu:** Pháp, Đức, Áo, Cộng hòa Séc, Ba Lan, Serbia, Thụy Điển, Ukraine.
*   **Các khu vực khác:** Úc, Chile, Colombia, Nam Phi, Tây Ban Nha, Thổ Nhĩ Kỳ.

## 2. Nguồn dữ liệu
Dữ liệu được tích hợp từ hai nguồn chính đáng tin cậy:
*   **Ember (Electricity Data):** Cung cấp sản lượng điện chi tiết theo tháng và theo từng loại hình nguồn phát. (Nguồn: [ember-energy.org](https://ember-energy.org/data/monthly-electricity-data/))
*   **Open-Meteo (Weather Data):** Cung cấp các thông số thời tiết lịch sử (nhiệt độ, bức xạ, lượng mưa...) tương ứng với tọa độ các quốc gia. (Nguồn: [open-meteo.com](https://open-meteo.com))

## 3. Dữ liệu gồm những biến gì?
Bộ dữ liệu bao gồm 30 biến, tập trung vào các nhóm chính sau:

*   **Biến mục tiêu (Target Series):**
    *   `generation_TWh`: Sản lượng điện phát ra (đơn vị Terawatt-giờ).
*   **Biến phụ thuộc thời tiết (Covariates):**
    *   `temperature` (Nhiệt độ), `solar` (Bức xạ mặt trời), `humidity` (Độ ẩm), `precipitation` (Lượng mưa).
*   **Đặc trưng thời gian (Temporal Features):**
    *   Tháng (`month`), Quý (`quarter`), Năm (`year`), và các biến vòng quay `month_sin`, `month_cos`.
*   **Đặc trưng trễ và thống kê (Lags & Rolling Features):**
    *   Lag: 1, 3, 12 tháng (`gen_lag_1`, `gen_lag_3`, `gen_lag_12`).
    *   Rolling (3 và 6 tháng): Mean, Std, Max.
    *   Thay đổi so với cùng kỳ (`yoy_change`).

## 4. Granularity dữ liệu
*   **Theo tháng (Monthly).**

## 5. Khoảng thời gian dữ liệu
*   Từ **tháng 01/2018** đến **tháng 12/2024**.
    *(Dải dữ liệu 2018-2024 đảm bảo tính mùa vụ cho giai đoạn Pre-training).*

## 6. Quy mô dataset
*   **Số quốc gia:** 20.
*   **Số bản ghi (Records):** 34,614 dòng (Sau deduplication).
*   **Số đặc trưng (Features):** 30 cột dữ liệu.
*   **Số nhóm chuỗi thời gian (Groups):** 151 nhóm (Tổ hợp giữa Quốc gia và Loại hình nguồn điện).

## 7. Phân tích chất lượng dữ liệu (EDA Insights)
- **Vấn đề trùng lặp (Duplicates)**: EDA phát hiện tỷ lệ trùng lặp cao (~58.9%) do việc lấy dữ liệu thời tiết ở cấp độ vùng/trạm rồi gộp (merge) lại thành cấp quốc gia. Giải pháp tiền xử lý đã sử dụng giá trị trung bình (mean) để hợp nhất dữ liệu, đảm bảo tính duy nhất cho mỗi quốc gia tại mỗi thời điểm.
- **Giá trị Zero & Negative**:
    - `Net Imports`: Có thể chứa giá trị âm, phản ánh trạng thái xuất khẩu điện ròng.
    - `Other Fossil`: Có tỷ lệ zero rất cao (lên tới 73% ở một số chuỗi), được xác định là dữ liệu đúng do đặc thù hạ tầng của quốc gia đó.
- **Sự lệch quy mô (Scale Heterogeneity)**: Có sự chênh lệch cực lớn (hàng trăm lần) về sản lượng điện giữa các cường quốc kinh tế (như Đức, Pháp) so với các quốc gia nhỏ hơn. Đây là lý do cốt yếu để áp dụng chuẩn hóa theo nhóm (`GroupNormalizer`) thay vì chuẩn hóa toàn cục.

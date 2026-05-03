# Tổng hợp Nội dung EDA và Feature Engineering - Dữ liệu Việt Nam

**Nguồn:** `notebook/EDA_VN_data.ipynb`
**Đầu vào:** `data/processed/VN_data/full_vietnam_monthly_merger.csv`
**Đầu ra:** `data/processed/VN_data/vn_tft_ready.csv`
**Mục tiêu:** Khám phá (EDA), tiền xử lý và tạo đặc trưng (Feature Engineering) cho bộ dữ liệu Việt Nam chuẩn bị cho quá trình Transfer Learning mô hình TFT.

---

## 1. Tổng quan bộ dữ liệu
- **Kích thước:** 504 dòng x 18 cột.
- **Thời gian:** Tháng 1/2019 - Tháng 12/2024 (72 tháng).
- **Các chuỗi (Series):** 7 chuỗi bao gồm Coal (Than), Gas (Khí), Hydro (Thủy điện), Other fossil (Hóa thạch khác), Solar (Mặt trời), Wind (Gió), và Total generation (Tổng sản lượng).
- **Chất lượng cơ bản:** Không có giá trị thiếu (0 missing values), không có dữ liệu trùng lặp (0 duplicates), không bị gián đoạn thời gian.

---

## 2. Các vấn đề dữ liệu và Cách xử lý (Data Cleaning)
- **Sai định dạng dữ liệu:** Cột `castlecoal_vol` có chứa định dạng chuỗi với ký tự 'K' (ví dụ: '1.14K'). 
  - *Xử lý:* Hàm `parse_k` chuyển đổi giá trị thành số thực (nhân với 1000).
- **Dữ liệu bất thường (Giá trị âm):** Cột `FDI_Disbursed_Monthly(bilionUSD)` chứa một số giá trị âm (có thể là do điều chỉnh số liệu hồi tố). 
  - *Xử lý:* Cắt các giá trị âm về 0 bằng `clip(lower=0)`.
- **Nguy cơ Data Leakage từ "Total generation":** Total generation là chuỗi tổng hợp (tổng của các series thành phần). Việc dùng chung làm target để dự báo độc lập sẽ gây rò rỉ dữ liệu. 
  - *Xử lý:* Loại `Total generation` khỏi nhóm series mục tiêu (target), giữ lại giá trị này chuyển thành một cột covariate chung (`total_generation_TWh`).
- **Khác biệt về thang đo & Outliers:** Các biến kinh tế (GDP, IPI, Giá than, Giá khí) có thang đo chênh lệch lớn và bị nhiễu do khủng hoảng năng lượng toàn cầu. 
  - *Xử lý:* Áp dụng Log Transform ở phần tạo đặc trưng.
- **Zeros trong chuỗi Solar, Wind, Other fossil:** Giai đoạn 2019 các nguồn năng lượng tái tạo này chưa phát triển hoặc ít dùng nên có nhiều giá trị bằng 0. 
  - *Xử lý:* Giữ nguyên (dữ liệu thật).

---

## 3. Phân tích Xu hướng & Mùa vụ (EDA)
- **Mùa vụ của các nguồn năng lượng:** 
  - Nhiệt điện than (`Coal`) đạt đỉnh vào tháng 5-7 (mùa khô, lúc thủy điện cạn nước).
  - Thủy điện (`Hydro`) đạt đỉnh vào tháng 7-9 (mùa mưa ở miền Bắc/Trung).
  - Điện mặt trời (`Solar`) cao nhất vào tháng 3-5 (nhiều nắng nhất trước mùa mưa).
  - Điện gió (`Wind`) cao nhất vào tháng 10-1 (gió mùa Đông Bắc).
- **Tương quan (Correlation):** Phân tích cho thấy sự tương quan rõ rệt giữa các chỉ số kinh tế (như IPI, CPI, GDP) và diễn biến giá cả (giá dầu, giá than) đối với sản lượng điện tiêu thụ.

---

## 4. Kỹ thuật Tạo Đặc Trưng (Feature Engineering)
Nhằm tăng cường khả năng học của mô hình TFT, tập dữ liệu được bổ sung số lượng lớn các đặc trưng phân thành nhiều nhóm:

### 4.1. Đặc trưng Thời gian (Temporal Features)
- `month`, `quarter`, `year`.
- Mã hóa vòng lặp (Cyclical encoding): `month_sin`, `month_cos`, `quarter_sin`, `quarter_cos`.
- Chỉ báo mùa: `is_dry_season` (tháng 11-4), `is_flood_season` (tháng 7-10).
- Tạo `time_idx` (Chỉ mục thời gian liên tục - tham số bắt buộc cho TFT).

### 4.2. Đặc trưng Quá khứ & Thống kê Trượt (Lag & Rolling Features)
Được tính toán riêng cho từng series, đảm bảo sử dụng `shift` hợp lý để không gây leakage:
- Trễ tự hồi quy (Lags): `gen_lag_1`, `gen_lag_2`, `gen_lag_3`, `gen_lag_6`, `gen_lag_12`.
- Trung bình trượt (Rolling Mean): Trong 3, 6, 12 tháng.
- Độ lệch chuẩn (Rolling Std), Min, Max trượt.
- Hệ số biến thiên (Rolling CV): Đo lường sự biến động.
- Tốc độ thay đổi: `yoy_change` (So với cùng kỳ năm trước), `mom_change` (So với tháng trước).

### 4.3. Đặc trưng Thời tiết (Weather Features)
- **Chống Leakage tuyệt đối:** Toàn bộ dữ liệu thời tiết (`temperature`, `solar`, `humidity`, `precipitation`) được dịch trễ (`shift(1)`) 1 tháng về quá khứ (trong Production Pipeline).
- Tính độ lệch thời tiết (`anomaly`) so với mức trung bình của tháng đó.
- Lấy Logarít cho lượng mưa (`log_precipitation`) và tạo biến trung bình trượt lượng mưa.

### 4.4. Đặc trưng Kinh tế (Economic Features)
- Cân bằng thang đo bằng Log Transform: Giá dầu (`log_oil_price`), Giá than, Giá khí, Khối lượng than, GDP. Biến mục tiêu cũng được lấy log (`log_generation`).
- Tính toán giá trị quá khứ (`lag1`, `lag3`), trung bình trượt (`roll3`) và tốc độ tăng trưởng (`yoy`) cho các chỉ số kinh tế (IPI, GDP, CPI...).
- Tỷ lệ tăng trưởng IPI theo tháng (`ipi_mom`).

### 4.5. Đặc trưng Tương tác Chéo (Cross-series Features)
- Tính toán tỷ trọng của từng nguồn điện đóng góp vào tổng sản lượng: `coal_ratio`, `hydro_ratio`, `fossil_ratio`, `renew_ratio` (các giá trị này đều được tính từ quá khứ - shifted).
- Tín hiệu thay thế (`coal_hydro_sub`): Mức chênh lệch giữa điện than và thủy điện, mô tả sự bù đắp công suất giữa hai nguồn này.

---

## 5. Tổ chức Dữ liệu Đầu Vào cho TFT (TimeSeriesDataSet Config)
Dựa trên các đặc trưng đã tạo, biến số được phân nhóm chặt chẽ theo yêu cầu của Temporal Fusion Transformer:
- **`static_categoricals` (Biến tĩnh phân loại):** `entity`, `series`.
- **`time_varying_known_reals` (Biến biến đổi theo thời gian nhưng biết trước tương lai):** `time_idx`, thông tin thời gian/lịch (`month`, `quarter`, `year`, cyclical features, báo hiệu mùa), và các biến dự báo thời tiết (nếu đã shift 1 tháng thì xem như known ở thời điểm T).
- **`time_varying_unknown_reals` (Biến biến đổi nhưng KHÔNG biết trước):** Biến mục tiêu (`generation_TWh`), tất cả các giá trị Lag, Rolling, dữ liệu kinh tế, và Cross-series.

**Đánh giá:** Pipeline preprocessing cuối cùng (được thiết kế trong notebook) đã hoàn thành xuất sắc việc khắc phục lỗi Data Leakage, cung cấp bộ dữ liệu phong phú tín hiệu quá khứ (Lags/Rollings/Cross-series) và được lưu ra `vn_tft_ready.csv` một cách sạch sẽ để đưa vào huấn luyện mô hình.

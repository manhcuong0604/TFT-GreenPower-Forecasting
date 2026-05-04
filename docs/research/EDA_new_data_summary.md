# Tổng hợp Nội dung EDA và Feature Engineering - Dữ liệu Đa Quốc Gia

**Nguồn:** `notebook/EDA_new_data.ipynb`
**Đầu vào:** `data/processed/merged_electric_weather_data.csv`
**Đầu ra:** `data/processed/tft_premodel_dataset_EDA.csv`
**Mục tiêu:** Khám phá (EDA), phát hiện và xử lý lỗi dữ liệu đa quốc gia, tạo các đặc trưng (Feature Engineering) phục vụ huấn luyện mô hình Temporal Fusion Transformer (TFT) ở giai đoạn Pre-train.

---

## 1. Tổng quan bộ dữ liệu
- Dữ liệu chứa thông tin sản lượng điện và thời tiết của nhiều quốc gia (trong đó có Việt Nam).
- **Chuỗi dữ liệu (Series):** Bao gồm nhiều nguồn năng lượng khác nhau (Coal, Gas, Hydro, Solar, Wind, Bioenergy, Nuclear, Net Imports, Demand, Fossil, Renewables...).
- **Độ dài chuỗi:** Khác nhau ở mỗi quốc gia, cần lưu ý lọc các quốc gia có dữ liệu quá ngắn (như Ukraine ~57 tháng, vẫn đủ chuẩn tối thiểu).

---

## 2. Các vấn đề dữ liệu đa quốc gia & Cách xử lý (Data Cleaning)
Dữ liệu đa quốc gia có độ phức tạp cao, notebook đã chỉ ra 6 vấn đề lớn:

- **Vấn đề 1: Dữ liệu trùng lặp (Duplicates):** Khoảng 29% dữ liệu bị lặp dòng. Nguyên nhân do khi merge dữ liệu thời tiết ở nhiều vùng địa lý vào một quốc gia.
  - *Xử lý:* Dùng `groupby(['entity', 'entity code', 'date', 'series'])` và lấy trung bình (`agg('mean')`) để gộp các bản ghi trùng lặp.
- **Vấn đề 2: Giá trị 0 và số Âm (Zeros & Negatives):** Các nguồn "Other Fossil", "Wind", "Solar" có nhiều giá trị 0. Chuỗi "Net Imports" có số âm (xuất khẩu).
  - *Xử lý:* Giữ nguyên vì đây là hiện tượng vật lý/kinh tế hợp lý. Dùng phép biến đổi Logarit `log(1 + generation_TWh)` để mô hình học tốt hơn với các giá trị 0.
- **Vấn đề 3: Sai lệch thang đo lượng mưa (Precipitation Anomaly):** Lượng mưa có sự khác biệt rất lớn về đơn vị giữa các nước (VD: Colombia ~8000, Đức ~62).
  - *Xử lý:* Áp dụng chuẩn hóa Z-score theo từng quốc gia (`prec_zscore_entity`) và lấy Log lượng mưa (`log_precipitation`).
- **Vấn đề 4: Khoảng trống thời gian (Time Gaps):** TFT yêu cầu chuỗi thời gian liên tục, một số nước như Colombia bị thiếu dữ liệu nhiều tháng.
  - *Xử lý:* Đề xuất sử dụng Linear Interpolation (nội suy) cho các gap ngắn hoặc loại bỏ series nếu gap quá dài.
- **Vấn đề 5: Giá trị ngoại lai (Outliers):** Chiếm ~12.5% dữ liệu sản lượng (do chênh lệch quy mô giữa các quốc gia).
  - *Xử lý:* Không cắt bỏ (clip). Để mô hình TFT tự xử lý thông qua `GroupNormalizer` (chuẩn hóa theo từng nhóm).
- **Vấn đề 6: Dư thừa thông tin (Series Redundancy):** Các chuỗi tổng hợp (ví dụ Fossil = Coal + Gas) mang thông tin trùng lặp với chuỗi thành phần, gây hiện tượng đa cộng tuyến.
  - *Xử lý:* Chỉ định đúng chuỗi target, các chuỗi phụ trợ được dùng như `observed covariates`.

---

## 3. Phân tích Xu hướng & Mùa vụ (EDA)
- **Xu hướng tổng thể:** Từng quốc gia có cấu trúc nguồn năng lượng khác nhau, nhưng nhìn chung sản lượng tái tạo (Solar, Wind) đang tăng.
- **Tập trung vào Việt Nam:** 
  - Mùa vụ thể hiện rõ ràng: Thủy điện đạt đỉnh mùa mưa (Tháng 6-8), Điện than tăng mạnh vào mùa khô để bù đắp, Điện mặt trời mạnh nhất vào các tháng 3-5.
  - Phân tích độ trễ (Lag Correlation): Lượng mưa (Precipitation) có độ tương quan mạnh với Thủy điện (Hydro) sau một khoảng thời gian trễ nhất định. Do đó, việc sử dụng các lag lượng mưa (`prec_lag_1`, `prec_lag_2`) là rất quan trọng.

---

## 4. Kỹ thuật Tạo Đặc Trưng (Feature Engineering)
Kế hoạch Feature Engineering được thực hiện tương tự như bộ dữ liệu Việt Nam nhưng có mở rộng cho ngữ cảnh đa quốc gia:

### 4.1. Đặc trưng Thời gian (Temporal Features)
- `month`, `quarter`, `year`.
- Mã hóa vòng lặp (Cyclical encoding): `month_sin`, `month_cos`.
- Tính `time_idx` (đánh index liên tục theo nhóm quốc gia + chuỗi năng lượng).

### 4.2. Đặc trưng Quá khứ & Thống kê Trượt (Lag & Rolling)
Tính toán nhóm riêng cho từng `entity` và `series`:
- Trễ: `gen_lag_1`, `gen_lag_3`, `gen_lag_12` (quan trọng để bắt tính mùa vụ hằng năm).
- Trung bình trượt (Rolling Mean) 3 và 6 tháng; Độ lệch chuẩn trượt (Rolling Std) 3 tháng; Rolling Max 6 tháng.
- Tăng trưởng so với cùng kỳ (`yoy_change`).
- Trễ thời tiết: `prec_lag_1`, `prec_lag_2`.

### 4.3. Đặc trưng Thời tiết (Weather Transformations)
- Tính Z-score lượng mưa theo từng quốc gia (`prec_zscore`).
- Chuẩn hóa bức xạ mặt trời về khoảng [0,1] theo từng quốc gia (`solar_norm`).
- Tính độ lệch nhiệt độ so với trung bình tháng của quốc gia đó (`temp_anomaly`).

### 4.4. Biến đổi mục tiêu (Target Transformation)
- Tạo biến `log_generation = log1p(generation_TWh)` để xử lý phân phối bị lệch (skewness) của dữ liệu.

---

## 5. Tổ chức Dữ liệu Đầu Vào cho TFT (TimeSeriesDataSet Config)
Thiết lập bộ dữ liệu đa quốc gia cho TFT sử dụng `pytorch_forecasting`:
- **`group_ids`**: `['entity', 'series']` (Từng quốc gia và loại năng lượng là một chuỗi độc lập).
- **`static_categoricals`**: `entity` (giúp mô hình học được đặc điểm riêng của từng quốc gia).
- **`time_varying_known_reals`**: `time_idx`, `month`, `year`, `month_sin`, `month_cos`, `quarter`, và các biến thời tiết dự báo được (như `solar`, `temperature`, `humidity`).
- **`time_varying_unknown_reals`**: Target (`generation_TWh`), các biến thời tiết không dự báo xa được (`precipitation`, `prec_zscore`), độ lệch, các Lag và Rolling.
- **`target_normalizer`**: Sử dụng `GroupNormalizer(groups=['entity', 'series'], transformation='softplus')`. Tính năng này tự động Scale dữ liệu về cùng một phân phối bất chấp sự khác biệt quy mô giữa Mỹ/Trung Quốc với các nước nhỏ hơn, `softplus` giúp xử lý êm ái các giá trị Zeros.
- **Yêu cầu chuỗi:** Để dùng 24 tháng encoder và 6 tháng prediction, các chuỗi cần có `min_series_length = 30`.

**Đánh giá:** Notebook giải quyết triệt để các rủi ro của dữ liệu chéo quốc gia (Scale mismatch, Duplicates, Leakage) bằng các phép Scale theo cụm (`GroupNormalizer`, Z-score per entity). Kết quả tạo ra một bộ dữ liệu Pre-model hoàn chỉnh cho giai đoạn pre-train toàn cầu.

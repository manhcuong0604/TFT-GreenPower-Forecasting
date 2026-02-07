# Tổng quan dữ liệu thô

## Phạm vi và tần suất
- Thư mục lưu trữ: `data/raw`
- Dữ liệu được thu thập và sử dụng chủ yếu theo tháng; một số tệp đi kèm biến thể theo ngày/giờ để phục vụ tổng hợp.
- Phạm vi thời gian chính: từ tháng 1 năm 2019 đến hết năm 2025 (đối với dữ liệu Việt Nam)



## DATA VIỆT NAM (Dữ liệu chính)
### Điện (electricmap_data)
*(Nguồn chính: ElectricMap API - https://app.electricitymaps.com)*

>***Lưu ý: Dữ liệu này được tổ chức ElectricMap cung cấp và là dữ liệu mô phỏng dựa trên các mô hình nội bộ của họ, không phải dữ liệu thực tế đo đạc.***

- `load_data_VN_monthly.csv`: tải điện Việt Nam tổng hợp theo tháng.
- `load_data_VN_daily.csv`: tải điện Việt Nam theo ngày.
- `load_data_VN.csv`: tải điện Việt Nam chi tiết (theo giờ).

### Điện (ember_data)
*(Nguồn chính: Ember - https://ember-energy.org/data/monthly-electricity-data/)*

>***Lưu ý: Dữ liệu này được tổ chức Ember cung cấp, chủ yếu theo thời gian tháng và có số lượng dữ liệu rất ít ~ 500 dòng***

- `monthly_electric_data.csv`: dữ liệu điện Việt Nam theo tháng (sản lượng, nguồn phát theo danh mục Ember).

- `ember_monthly_electricity-generation - All electricity sources - Multiple regions - breakdown.csv`: bảng chi tiết sản lượng điện theo nguồn, nhiều vùng (dạng gốc trước khi chọn Việt Nam).

- `yearly_electric_data.csv`: tổng hợp điện Việt Nam theo năm (hỗ trợ kiểm thử xu hướng dài hạn).

### Thời tiết (weather_data)
*(Nguồn chính: Meteor API - https://archive-api.open-meteo.com/v1/archive)*

- `humidity_vietnam_tft.csv`: độ ẩm trung bình (tần suất tháng).
- `precipitation_vietnam_tft.csv`: lượng mưa (tần suất tháng).
- `solar_radiation_vietnam_tft.csv`: bức xạ mặt trời (tần suất tháng).
- `temp_vietnam_tft.csv`: nhiệt độ (tần suất tháng).



## ENTSO-E (Europe data - Dữ liệu dự phòng)
*(Đây là dữ liệu của châu âu dùng để dự phòng nếu dữ liệu việt nam không đủ)*

*** Nguồn chính: ENTSO-E Transparency Platform - https://transparency.entsoe.eu/***
### Năng lực (capacity)
- `installed_capacity.csv`: tổng công suất lắp đặt EU (gốc).
- `DE_LU_installed_capacity.csv`, `FR_installed_capacity.csv`: công suất lắp đặt theo quốc gia.
- `unavailable_capacity.csv`: công suất không khả dụng.

### Phát điện (generation)
- Actual: `DE_LU_biomass.csv`, `DE_LU_hydro_pumped_generation.csv`, `DE_LU_hydro_reservoir.csv`, `DE_LU_hydro_run_of_river.csv`, `DE_LU_solar.csv`, `DE_LU_wind_offshore.csv`, `DE_LU_wind_onshore.csv`, `FR_biomass.csv`, `FR_hydro_pumped_generation.csv`, `FR_hydro_reservoir.csv`, `FR_hydro_run_of_river.csv`, `FR_nuclear.csv`, `FR_solar.csv`, `FR_wind_offshore.csv`, `FR_wind_onshore.csv`.
- Forecast: `DE_LU_solar_forecast.csv`, `DE_LU_wind_forecast.csv`, `FR_solar_forecast.csv`, `FR_wind_forecast.csv`.

### Nhu cầu (load)
- Actual: `DE_LU_total_load_actual.csv`, `FR_total_load_actual.csv`, `total_load_actual.csv`.
- Forecast: `DE_LU_total_load_forecast.csv`, `FR_total_load_forecast.csv`, `total_load_forecast.csv`.

### Giá (prices)
- `day_ahead_price.csv`: giá day-ahead tổng hợp.
- `DE_LU_day_ahead_price.csv`, `FR_day_ahead_price.csv`: giá day-ahead theo quốc gia.

### Metadata
- `calendar.csv`: thông tin ngày/tuần/tháng để gắn nhãn thời gian.
- `DE_LU_info.csv`, `FR_info.csv`: meta mô tả khu vực, mã vùng và đơn vị.



## NHỮNG DỮ LIỆU CẦN THU THẬP BỔ SUNG
>***Dữ liệu này chưa có trong kho và cần thu thập bổ sung từ các nguồn bên ngoài. Lưu ý phải phù hợp với các dữ liệu hiện có về mặt thời gian***

### Dữ liệu kinh tế việt nam
- GDP theo tháng/quý/năm.

- Chỉ số sản xuất công nghiệp (IIP) theo tháng/quý/năm.

- Chỉ số giá tiêu dùng (CPI) theo tháng/quý/năm.

### Dữ liệu đặc trưng thời gian Việt Nam
- Lịch Nghỉ Lễ (Holidays): Đặc biệt là Tết Nguyên Đán (âm lịch), các ngày lễ 30/4-1/5, Quốc khánh. Phụ tải điện Việt Nam thường giảm sâu trong các dịp này.

- Số ngày làm việc trong tháng: Tổng số ngày trừ đi Thứ 7, Chủ nhật và ngày lễ.

### Dữ liệu ngành công nghiệp
- Vốn đầu tư trực tiếp nước ngoài (FDI) giải ngân theo tháng (OK)
(VBMA - Hiệp hội Thị trường Trái phiếu Việt Nam - https://vbma.org.vn/vi/market-data/fdi)

- Tổng mức bán lẻ hàng hóa và doanh thu dịch vụ tiêu dùng

- Kim ngạch Xuất - Nhập khẩu (OK)
(Nguồn chính: CƠ QUAN THỐNG KÊ QUỐC GIA CỤC THỐNG KÊ - BỘ TÀI CHÍNH - https://www.nso.gov.vn/xuat-nhap-khau/)

- Giá năng lượng đầu vào: Giá than thế giới (Newcastle), giá dầu Brent, giá khí (Henry Hub). (OK)
(Nguồn chính: Investing.com - https://www.investing.com/commodities)

### Dữ liệu dân số
- Dân số theo tháng/năm



## GHI CHÚ
### Sử dụng dữ liệu
- Mỗi danh mục trên tương ứng thư mục con cùng tên trong `data/raw`.

- Dữ liệu thu thập bổ sung tùy vào phân loại đã nêu mà đặt trong thư mục con tương ứng (Có thể tạo thư mục mới nếu cần thiết).

## Lưu ý khi thu thập dữ liệu bổ sung
- Đảm bảo dữ liệu bổ sung có phạm vi thời gian từ tháng 1 năm 2019 đến hết năm 2025 để phù hợp với dữ liệu hiện có *(Dữ liệu mới có thể ít hơn hoặc linh hoạt hơn tùy vào loại dữ liệu).*

- Ưu tiên các nguồn dữ liệu chính thức, đáng tin cậy như *Tổng cục Thống kê Việt Nam, Ngân hàng Thế giới (World Bank), Quỹ Tiền tệ Quốc tế (IMF) hoặc các tổ chức nghiên cứu uy tín.*

- Kiểm tra tính nhất quán và định dạng của dữ liệu bổ sung để đảm bảo dễ dàng tích hợp với các tập dữ liệu hiện có.

---
***Version:** 1.0.0*  
***Ngày tạo:** 2026-01-20*  
***Trạng thái:** Cần thu thập bổ sung dữ liệu kinh tế, dân số, đặc trưng thời gian và ngành công nghiệp Việt Nam.*

***Người soạn thảo: [Nguyễn Văn Tiến]***



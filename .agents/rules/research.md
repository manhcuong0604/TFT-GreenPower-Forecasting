---
trigger: always_on
---

Đây là dự án nghiên cứu về ứng dụng mô hình TFT vào bài toán dự báo sản lượng điện của việt nam dựa vào phương pháp transfer learning.
Cụ thể gồm các bước

- Xây dựng bộ dữ liệu việt nam: full_vietnam_monthly_merger.csv
- Xây dựng bộ dữ liệu các quốc gia: merged_electric_weather_data.csv
  Tiền xử lý dữ liệu:
- EDA cho bộ dữ liệu việt nam: EDA_VN_data.ipynb
- EDA cho bộ dữ liệu thế giới: EDA_new_data.ipynb
- Dữ liệu việt nam sau xử lý: vn_tft_ready.csv
- Dữ liệu thế giới sau xử lý: tft_premodel_dataset_EDA.csv
  Trainning mô hình TFT:
- pretrain mô hình với dữ liệu thế giới: trainTFT_v3.ipynb
- transfer learning với dữ liệu việt nam: transfer_learning_v3.ipynb

thư mục raw/ : có 1 đống file
-> Tạo ra 2 bộ dữ liệu: 1 bộ to cho train TFT ban đầu, 1 bộ nhỏ(VN) cho train transform learning
- bỏ qua electricmap data, entsoe data

# 1 Bộ dữ liệu to
- Chọn ra tầm 10(15, 20) quốc gia có đặc điểm chung với VN(VD: quy mô kinh tế, quy mô lưới điện, khí hậu,...)
- vào ember_data -> monthly electric data -> lọc ra toàn bộ dữ liệu của 10 quốc gia trên
- Thu thập dữ liệu thời tiết của 10 quốc gia đó: độ ẩm, lượng mưa, bức xạ mặt trời, nhiệt độ **Lưu ý: lấy dữ liệu trung bình theo tháng sao cho khớp với dữ liệu điện đã lọc của 10 quốc gia trên**
--> Kết hợp 2 bộ dữ liệu trên sẽ được bộ dữ liệu lớn để pretrain TFT.


# 2 Bộ dữ liệu VN
- Từ bộ ember_data/monthly electric data lọc ra toàn bộ dữ liệu của VN ra 1 file riêng
- Tìm cách kết hợp với toàn bộ data còn lại của VN đã tìm trong thời gian qua
--> Bộ dữ liệu đặc thù của VN

## Lưu ý: Bộ dữ liệu thứ 1 không được có những cái mà bộ thứ 2 không có

Ví dụ
Bộ 1: a, b, c, d
Bộ 2: a, b, c, d, e, f

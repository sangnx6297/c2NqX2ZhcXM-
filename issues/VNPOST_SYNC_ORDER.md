# Lỗi không tìm thấy mã Phường xã|Quận huyện khi gửi DVVC VNPost.
+ **Nguyên nhân**: Do sai lệch dữ liệu với VNPost.
+ **Mô hình**:
```
------------------       ----------------------------        --------------------
| Dữ liệu ERP (1) | ---> | Dữ liệu mspstransport (2) | ---> | Dữ liệu VNPost (3) |
------------------       -----------------------------      ---------------------
- (1): Tập table trong database erp: ward, district, province.
        + Mapping dùng cột external_code.
- (2): Table trong database msptransport: locality.
        + Dùng cột code.
- (3): File excel locality của VNPost.
        + Dùng cột mã.
```
+ **Hướng xử lý**:
```
1. Kiểm tra dữ liệu (1) có khớp với dữ liệu (2) - không khớp điều chỉnh dữ liệu (1).
2. Kiểm tra dữ liệu (2) có khớp với dữ liệu (3) - Không khớp điều chỉnh dữ liệu (2) - Kiểm tra, điều dữ liệu (1).
3. Cập nhật database cho các dữ liệu đồng nhất.
```

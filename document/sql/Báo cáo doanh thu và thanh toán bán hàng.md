# Query:
```SQL
WITH erp_warehouse AS ( SELECT `code`, `name` FROM scj_mcr_prod.whtb_place ) SELECT
FROM_UNIXTIME( oi.created_at, '%d-%m-%Y' ) AS 'Ngày', 
w.`name` AS 'Tên kho hàng',
oh.order_sell_chanel_name AS 'Kênh Bán Hàng',
oh.order_channel_name AS 'Tên kênh',
oh.order_customer_code AS 'Số khách hàng',
oh.order_customer_name AS 'Tên khách hàng',
oh.order_code AS 'Số đặt hàng',
oh.order_transporter_name AS 'Phương thức vận chuyển',
oh.product_sku AS 'Mã sản phẩm',
oh.product_name AS 'Tên sản phẩm',
oh.product_vendor_code AS 'Mã NCC',
oh.product_vendor_name AS 'Tên NCC',
oh.product_buy_cost AS 'Giá Mua chưa VAT',
oh.product_sale_cost AS 'Giá bán chưa VAT',
oh.product_sale_price_vat AS 'Giá Bán có VAT',
oh.product_quantity AS 'Số lượng',
oh.product_sale_price_vat * oh.product_quantity AS 'Số tiền',
oh.order_shipping_amount AS 'Phí dịch vụ',
0 AS 'Phí GH nhanh',
oh.order_total_amount AS 'Doanh số',
oh.order_total_exclude_vat * oh.product_quantity  AS 'Giá mua chưa VAT',
IF
	( oi.payment_method <> 'PAY001', oh.order_total_amount, 0 ) AS 'CMS',
IF
	( oi.payment_method = 'PAY001', oh.order_total_amount, 0 ) AS 'COD',
	0 AS 'Payment Online',
  oh.order_saving AS 'Điểm tích lũy',
	oif.deposit AS 'Tiền đặt cọc',
	oh.order_voucher AS 'Gift Voucher',
	oh.order_promotion_amount AS 'Số tiền giảm giá' 
FROM
	`erp_order_history` oh
	INNER JOIN erp_order_index oi ON oh.order_id = oi.order_id
	INNER JOIN erp_order_info oif ON oi.order_id = oif.order_id
	INNER JOIN erp_warehouse w ON w.`code` = oh.order_warehouse_code 
WHERE
	oh.order_updated_status_code = 'confirmed' 
	/* các mã trạng thái:
				new: Tạo mới
				confirmed: Callcenter xác nhận đ ặt hàng
				packaging: Chỉ thị vận chuyển
				delivering: Đang vận chuyển
				returning: chỉ thị thu hồi
				completed: Hoàn tất
				cancelled: Hủy
				returned: Đã thu hồi
	*/
	AND oh.type = 'detail'  -- detail: lấy chi tiết các món hàng; general: lấy thông tin tổng quan 
	AND oh.order_updated_date BETWEEN UNIX_TIMESTAMP('2022-01-01 00:00:00') AND  UNIX_TIMESTAMP(NOW()) 
	ORDER BY oi.order_date asc; 
```

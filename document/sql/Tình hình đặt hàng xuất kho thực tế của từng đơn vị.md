Query: Tình hình đặt hàng/ xuất kho thực tế của từng đơn vị
```SQL

SELECT
	`base_o`.`product_type_code` as 'Mã loại Sản phẩm',
	`base_o`.`product_type_name` as 'Tên Phân loại lớn',
	SUM( IF(new_o.total IS NULL, 0, new_o.total )) AS 'Số lượng đặt hàng',-- `new_total`,
	SUM( IF(new_o.order_total_exclude_vat is null, 0,  new_o.order_total_exclude_vat)) AS 'Số tiền đặt hàng', -- `new_order_total_exclude_vat`,

	SUM( IF (success_o.total IS NULL, 0,success_o.total ) ) AS 'Số lượng bán', -- `success_total`,
	SUM( IF (success_o.order_total_exclude_vat  IS NULL, 0, success_o.order_total_exclude_vat)) as 'Số tiền bán hàng' -- AS `success_order_total_exclude_vat`,
 
 
FROM
	(
	SELECT
		`product_type_code`,
		`product_type_name`,
		MD5(
		CONCAT_WS( '#', product_type_code, product_type_name )) AS keymap 
	FROM
		`erp_order_history` `oh` 
	WHERE
		( `type` = 'detail' ) 
		AND ( `order_updated_date` BETWEEN UNIX_TIMESTAMP('2022-01-01 00:00:00') AND  UNIX_TIMESTAMP(NOW()) ) -- Thay đổi ngày để lấy kết quả
	GROUP BY
		`product_type_code`,
		`product_type_name`,
		MD5(
		CONCAT_WS( '#', product_type_code, product_type_name ))) `base_o`
		
	LEFT JOIN (
	SELECT
		`product_type_code`,
		`product_type_name`,
		MD5(
		CONCAT_WS( '#', product_type_code, product_type_name )) AS keymap,
		count( DISTINCT order_code ) AS total,
		SUM( order_total_exclude_vat * product_quantity ) AS order_total_exclude_vat,
		SUM( order_total_amount ) AS order_total_vat,
		COUNT( DISTINCT order_customer_code ) AS total_customer_count,
		SUM( shipping_exclude_vat ) AS order_shipping_exclude_vat,
		SUM( product_sale_cost ) AS product_sale_cost,
		SUM( product_quantity ) AS product_quantity 
	FROM
		`erp_order_history` `oh`
		INNER JOIN (
		SELECT
			MAX( id ) AS `id`,
			`order_updated_status_code` AS `ousc`,
			`order_code` AS `ocd`,
			`product_sku` AS `psku`,
			`product_owner_code` AS `pwc` 
		FROM
			`erp_order_history` 
		WHERE
			( `order_updated_date` BETWEEN UNIX_TIMESTAMP('2022-01-01 00:00:00') AND  UNIX_TIMESTAMP(NOW()) ) -- Thay đổi ngày để lấy kết quả
			AND ( `type` = 'detail' ) 
		GROUP BY
			`order_updated_status_code`,
			`order_code`,
			`product_sku`,
			`product_owner_code` 
		) `oh_max` ON oh_max.id = oh.id 
	WHERE
		( `type` = 'detail' ) 
		AND ( `order_updated_date` BETWEEN UNIX_TIMESTAMP('2022-01-01 00:00:00') AND  UNIX_TIMESTAMP(NOW()) ) -- Thay đổi ngày để lấy kết quả
		AND ( `order_updated_date` BETWEEN UNIX_TIMESTAMP('2022-01-01 00:00:00') AND  UNIX_TIMESTAMP(NOW()) ) -- Thay đổi ngày để lấy kết quả
		AND ((
				`type` = 'detail' 
				) 
		AND ( `order_updated_status_code` = 'confirmed' )) 
	GROUP BY
		`product_type_code`,
		`product_type_name`,
		MD5(
		CONCAT_WS( '#', product_type_code, product_type_name ))) `new_o` ON new_o.keymap = base_o.keymap
	 
 
 
	LEFT JOIN (
	SELECT
		MD5(
		CONCAT_WS( '#', oh.product_type_code, oh.product_type_name )) AS keymap,
		COUNT( DISTINCT oh.order_code ) AS `total`,
		SUM( oh.order_total_exclude_vat ) AS order_total_exclude_vat,
		SUM( oh.order_total_amount ) AS order_total_vat,
		COUNT( DISTINCT order_customer_code ) AS total_customer_count,
		SUM( shipping_exclude_vat ) AS order_shipping_exclude_vat,
		SUM( product_sale_cost ) AS product_sale_cost,
		SUM( product_quantity ) AS product_quantity 
	FROM
		(
		SELECT
			`product_type_code`,
			`product_type_name`,
			MD5(
			CONCAT_WS( '#', product_type_code, product_type_name )) AS keymap,
			`order_code`,
			`order_id`,
			`order_updated_status_code`,
			`order_updated_date`,
			`order_customer_code`,
			`order_sell_chanel_code`,
			SUM( order_total_exclude_vat * product_quantity ) AS order_total_exclude_vat,
			SUM( order_total_amount ) AS order_total_amount,
			SUM(
				order_shipping_amount / (
				1 + ( 10 / 100 ))) AS shipping_exclude_vat,
			SUM(
			IF
			( order_sell_chanel_code = 'b2b', ( product_sale_price_vat /( 1+ ( product_sale_vat / 100 ))) * product_quantity, ( product_sale_price_vat /( 1+ ( product_sale_vat / 100 ))) )) AS product_sale_cost,
			SUM( product_quantity ) AS product_quantity,
			SUM( order_total_exclude_vat )/ ( COUNT(*)/ COUNT( DISTINCT CONCAT_WS( '#', `order_code`, product_type_code, product_type_name, `order_updated_status_code`, DATE_FORMAT( order_updated_date, '%Y%m%d%H%i' ), `product_sku` )) ) AS order_total_exclude_vat_tp 
		FROM
			`erp_order_history` `oh`
			INNER JOIN (
			SELECT
				MAX( id ) AS `id`,
				`order_updated_status_code` AS `ousc`,
				`order_code` AS `ocd`,
				`product_sku` AS `psku`,
				`product_owner_code` AS `pwc` 
			FROM
				`erp_order_history` 
			WHERE
				( `order_updated_date` BETWEEN UNIX_TIMESTAMP('2022-01-01 00:00:00') AND  UNIX_TIMESTAMP(NOW()) ) -- Thay đổi ngày để lấy kết quả
				AND ( `type` = 'detail' ) 
			GROUP BY
				`order_updated_status_code`,
				`order_code`,
				`product_sku`,
				`product_owner_code` 
			) `oh_max` ON oh_max.id = oh.id 
		WHERE
			( `type` = 'detail' ) 
			AND ( `order_updated_date` BETWEEN UNIX_TIMESTAMP('2022-01-01 00:00:00') AND  UNIX_TIMESTAMP(NOW()) ) -- Thay đổi ngày để lấy kết quả
			AND ((
					`order_updated_status_code` = 'completed' 
					) 
			AND ( `type` = 'detail' )) 
		GROUP BY
			`product_type_code`,
			`product_type_name`,
			MD5(
			CONCAT_WS( '#', product_type_code, product_type_name )),
			`order_code`,
			`order_id`,
			`order_updated_status_code`,
			`order_sell_chanel_code`,
			DATE_FORMAT( order_updated_date, '%Y%m%d%H%i' ),
			`order_updated_status_code`,
			`product_sku`,
		DATE_FORMAT( order_updated_date, '%Y%m%d%H%i' )) `oh` 
	WHERE
		`oh`.`order_updated_date` BETWEEN UNIX_TIMESTAMP('2022-01-01 00:00:00') AND  UNIX_TIMESTAMP(NOW()) -- Thay đổi ngày để lấy kết quả
	GROUP BY
		MD5(
		CONCAT_WS( '#', oh.product_type_code, oh.product_type_name ))) `success_o` ON success_o.keymap = base_o.keymap
 GROUP BY
	`base_o`.`product_type_code`,
```

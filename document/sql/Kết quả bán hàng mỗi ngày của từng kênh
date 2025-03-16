Query: Kết quả bán hàng mỗi ngày của từng kênh
```sql
SELECT
-- 	`base_o`.`order_sell_chanel_code`,
	`base_o`.`order_sell_chanel_name` as 'Tên kênh bàn hàng',
 
	SUM( new_o.order_total_exclude_vat ) AS 'Tiền đặt hàng',

	SUM( success_o.order_total_exclude_vat ) AS 'Doanh số'
 
FROM
	(
	SELECT
		`order_sell_chanel_code`,
		`order_sell_chanel_name`,
		`channel_code`,
		`channel_name`,
		MD5(
		CONCAT_WS( '#', order_sell_chanel_code, order_sell_chanel_name, channel_code, channel_name )) AS keymap 
	FROM
		`erp_order_history` `oh`
		LEFT JOIN (
		SELECT
			`o`.`order_id` AS `oc_id`,
			`c`.`MSALE_NAME` AS `channel_name`,
			`c`.`MSALE_CODE` AS `channel_code` 
		FROM
			`erp_order_index` `o`
			LEFT JOIN `erp_channel_tv` `c` ON o.channel_tv = c.id_channel_tv 
		) `oc` ON oc.oc_id = oh.order_id 
	WHERE
		( `type` = 'detail' ) 
		AND ( `order_updated_date` BETWEEN UNIX_TIMESTAMP('2022-01-01 00:00:00') AND  UNIX_TIMESTAMP(NOW())) -- Thay đổi ngày để lấy kết quả
		 
	GROUP BY
		`order_sell_chanel_code`,
		`order_sell_chanel_name`,
		`channel_code`,
		`channel_name`,
		MD5(
		CONCAT_WS( '#', order_sell_chanel_code, order_sell_chanel_name, channel_code, channel_name ))) `base_o`
	LEFT JOIN (
	SELECT
		`order_sell_chanel_code`,
		`order_sell_chanel_name`,
		`channel_code`,
		`channel_name`,
		MD5(
		CONCAT_WS( '#', order_sell_chanel_code, order_sell_chanel_name, channel_code, channel_name )) AS keymap,
		count( DISTINCT order_code ) AS total,
		SUM( order_total_exclude_vat * product_quantity ) AS order_total_exclude_vat,
		SUM( order_total_amount ) AS order_total_vat,
		COUNT( DISTINCT order_customer_code ) AS total_customer_count,
		SUM( shipping_exclude_vat ) AS order_shipping_exclude_vat,
		SUM( product_sale_cost ) AS product_sale_cost,
		SUM( product_quantity ) AS product_quantity 
	FROM
		`erp_order_history` `oh`
		LEFT JOIN (
		SELECT
			`o`.`order_id` AS `oc_id`,
			`c`.`MSALE_NAME` AS `channel_name`,
			`c`.`MSALE_CODE` AS `channel_code` 
		FROM
			`erp_order_index` `o`
			LEFT JOIN `erp_channel_tv` `c` ON o.channel_tv = c.id_channel_tv 
		) `oc` ON oc.oc_id = oh.order_id
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
			( `order_updated_date` BETWEEN UNIX_TIMESTAMP('2022-01-01 00:00:00') AND  UNIX_TIMESTAMP(NOW())) -- Thay đổi ngày để lấy kết quả
			AND ( `type` = 'detail' ) 
		GROUP BY
			`order_updated_status_code`,
			`order_code`,
			`product_sku`,
			`product_owner_code` 
		) `oh_max` ON oh_max.id = oh.id 
	WHERE
		( `type` = 'detail' ) 
		AND ( `order_updated_date` BETWEEN UNIX_TIMESTAMP('2022-01-01 00:00:00') AND  UNIX_TIMESTAMP(NOW())) -- Thay đổi ngày để lấy kết quả
		 
		AND ( `order_updated_date` BETWEEN UNIX_TIMESTAMP('2022-01-01 00:00:00') AND  UNIX_TIMESTAMP(NOW())) -- Thay đổi ngày để lấy kết quả
		AND ((
				`type` = 'detail' 
				) 
		AND ( `order_updated_status_code` = 'confirmed' )) 
	GROUP BY
		`order_sell_chanel_code`,
		`order_sell_chanel_name`,
		`channel_code`,
		`channel_name`,
		MD5(
		CONCAT_WS( '#', order_sell_chanel_code, order_sell_chanel_name, channel_code, channel_name ))) `new_o` ON new_o.keymap = base_o.keymap
	LEFT JOIN (
	SELECT
		MD5(
		CONCAT_WS( '#', oh.order_sell_chanel_code, oh.order_sell_chanel_name, oh.channel_code, oh.channel_name )) AS keymap,
		SUM( oh.order_total_exclude_vat ) AS order_total_exclude_vat,
		SUM( oh.order_total_amount ) AS order_total_vat,
		COUNT( DISTINCT order_customer_code ) AS total_customer_count,
		COUNT(*) AS `total`,
		SUM( shipping_exclude_vat ) AS order_shipping_exclude_vat,
		SUM( product_sale_cost ) AS product_sale_cost,
		SUM( product_quantity ) AS product_quantity 
	FROM
		`erp_order_delivery` `od`
		INNER JOIN (
		SELECT
			`order_sell_chanel_code`,
			`order_sell_chanel_name`,
			`channel_code`,
			`channel_name`,
			MD5(
			CONCAT_WS( '#', order_sell_chanel_code, order_sell_chanel_name, channel_code, channel_name )) AS keymap,
			`order_code`,
			`order_id`,
			`order_updated_status_code`,
			`order_updated_date`,
			`order_customer_code`,
			SUM( order_total_exclude_vat * product_quantity ) AS order_total_exclude_vat,
			SUM( order_total_amount ) AS order_total_amount,
			SUM(
				order_shipping_amount / (
				1 + ( 10 / 100 ))) AS shipping_exclude_vat,
			SUM(
			IF
			( order_sell_chanel_code = 'b2b', ( product_sale_price_vat /( 1+ ( product_sale_vat / 100 ))) * product_quantity, ( product_sale_price_vat /( 1+ ( product_sale_vat / 100 ))) )) AS product_sale_cost,
			SUM( product_quantity ) AS product_quantity 
		FROM
			`erp_order_history` `oh`
			LEFT JOIN (
			SELECT
				`o`.`order_id` AS `oc_id`,
				`c`.`MSALE_NAME` AS `channel_name`,
				`c`.`MSALE_CODE` AS `channel_code` 
			FROM
				`erp_order_index` `o`
				LEFT JOIN `erp_channel_tv` `c` ON o.channel_tv = c.id_channel_tv 
			) `oc` ON oc.oc_id = oh.order_id
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
				( `order_updated_date` BETWEEN UNIX_TIMESTAMP('2022-01-01 00:00:00') AND  UNIX_TIMESTAMP(NOW())) -- Thay đổi ngày để lấy kết quả
				AND ( `type` = 'detail' ) 
			GROUP BY
				`order_updated_status_code`,
				`order_code`,
				`product_sku`,
				`product_owner_code` 
			) `oh_max` ON oh_max.id = oh.id 
		WHERE
			( `type` = 'detail' ) 
			AND ( `order_updated_date` BETWEEN UNIX_TIMESTAMP('2022-01-01 00:00:00') AND  UNIX_TIMESTAMP(NOW())) -- Thay đổi ngày để lấy kết quả
			 
			AND ((
					`order_updated_status_code` = 'returning' 
					) 
			AND ( `type` = 'detail' )) 
		GROUP BY
			`order_sell_chanel_code`,
			`order_sell_chanel_name`,
			`channel_code`,
			`channel_name`,
			MD5(
			CONCAT_WS( '#', order_sell_chanel_code, order_sell_chanel_name, channel_code, channel_name )),
			`order_code`,
			`order_id`,
			`order_updated_status_code`,
			`order_sell_chanel_code`,
		DATE_FORMAT( order_updated_date, '%Y%m%d%H%i' )) `oh` ON oh.order_id = od.order_id 
	WHERE
		( `od`.`status` = 2 ) 
		AND ( `od`.`updated_at` BETWEEN UNIX_TIMESTAMP('2022-01-01 00:00:00') AND  UNIX_TIMESTAMP(NOW()))  -- Thay đổi ngày để lấy kết quả
	GROUP BY
		MD5(
		CONCAT_WS( '#', oh.order_sell_chanel_code, oh.order_sell_chanel_name, oh.channel_code, oh.channel_name ))) `return_failed_to_o` ON return_failed_to_o.keymap = base_o.keymap
	LEFT JOIN (
	SELECT
		MD5(
		CONCAT_WS( '#', oh.order_sell_chanel_code, oh.order_sell_chanel_name, oh.channel_code, oh.channel_name )) AS keymap,
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
			`order_sell_chanel_code`,
			`order_sell_chanel_name`,
			`channel_code`,
			`channel_name`,
			MD5(
			CONCAT_WS( '#', order_sell_chanel_code, order_sell_chanel_name, channel_code, channel_name )) AS keymap,
			`order_code`,
			`order_id`,
			`order_updated_status_code`,
			`order_updated_date`,
			`order_customer_code`,
			SUM( order_total_exclude_vat * product_quantity ) AS order_total_exclude_vat,
			SUM( order_total_amount ) AS order_total_amount,
			SUM(
				order_shipping_amount / (
				1 + ( 10 / 100 ))) AS shipping_exclude_vat,
			SUM(
			IF
			( order_sell_chanel_code = 'b2b', ( product_sale_price_vat /( 1+ ( product_sale_vat / 100 ))) * product_quantity, ( product_sale_price_vat /( 1+ ( product_sale_vat / 100 ))) )) AS product_sale_cost,
			SUM( product_quantity ) AS product_quantity,
			SUM( order_total_exclude_vat )/ (
				COUNT(*)/ COUNT(
				DISTINCT CONCAT_WS( '#', `order_code`, order_sell_chanel_code, order_sell_chanel_name, channel_code, channel_name, `order_updated_status_code`, DATE_FORMAT( order_updated_date, '%Y%m%d%H%i' ), `product_sku` )) 
			) AS order_total_exclude_vat_tp 
		FROM
			`erp_order_history` `oh`
			LEFT JOIN (
			SELECT
				`o`.`order_id` AS `oc_id`,
				`c`.`MSALE_NAME` AS `channel_name`,
				`c`.`MSALE_CODE` AS `channel_code` 
			FROM
				`erp_order_index` `o`
				LEFT JOIN `erp_channel_tv` `c` ON o.channel_tv = c.id_channel_tv 
			) `oc` ON oc.oc_id = oh.order_id
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
				( `order_updated_date` BETWEEN UNIX_TIMESTAMP('2022-01-01 00:00:00') AND  UNIX_TIMESTAMP(NOW())) -- Thay đổi ngày để lấy kết quả
				AND ( `type` = 'detail' ) 
			GROUP BY
				`order_updated_status_code`,
				`order_code`,
				`product_sku`,
				`product_owner_code` 
			) `oh_max` ON oh_max.id = oh.id 
		WHERE
			( `type` = 'detail' ) 
			AND ( `order_updated_date` BETWEEN UNIX_TIMESTAMP('2022-01-01 00:00:00') AND  UNIX_TIMESTAMP(NOW())) -- Thay đổi ngày để lấy kết quả
			 
			AND ((
					`order_updated_status_code` = 'completed' 
					) 
			AND ( `type` = 'detail' )) 
		GROUP BY
			`order_sell_chanel_code`,
			`order_sell_chanel_name`,
			`channel_code`,
			`channel_name`,
			MD5(
			CONCAT_WS( '#', order_sell_chanel_code, order_sell_chanel_name, channel_code, channel_name )),
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
		CONCAT_WS( '#', oh.order_sell_chanel_code, oh.order_sell_chanel_name, oh.channel_code, oh.channel_name ))) `success_o` ON success_o.keymap = base_o.keymap 
GROUP BY
	`base_o`.`order_sell_chanel_code`,
	`base_o`.`order_sell_chanel_name`;
```

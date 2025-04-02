---
title: DBeaver 数据探查
date created: 2024/7/31 11:15
date modified: 2024/8/15 14:5
---
# DBeaver进行数据探查

## 连接clickhouse

### **新建连接**

点击 上面的倒三角 -->other -->clickhouse

![[docs/01attachment/docs/Work/高科数聚/DBeaver/DBeaver/IMG-2025-03-26-11-58-37.png]]

![[docs/01attachment/docs/Work/高科数聚/DBeaver/DBeaver/IMG-2025-03-26-11-58-37-1.png]]

首次连接需要下载相关的驱动，点击“编辑驱动设置”

![[docs/01attachment/docs/Work/高科数聚/DBeaver/DBeaver/IMG-2025-03-26-11-58-37-2.png]]

点击“下载/更新”

![[docs/01attachment/docs/Work/高科数聚/DBeaver/DBeaver/IMG-2025-03-26-11-58-37-3.png]]

下载完成后，点击“确定” 

熟悉三张表

长久的原始数据：cj_csv_20191231_20240609

-- data20240609.cj_csv_20191231_20240609 definition

CREATE TABLE data20240609.cj_csv_20191231_20240609

(

	`id` String,

	`encode_vin` Nullable(String),

	`vin_id` String,

	`接车日期` Nullable(String),

	`释放日期` Nullable(String),

	`省` Nullable(String),

	`市` Nullable(String),

	`区县` Nullable(String)

)

ENGINE = MergeTree

ORDER BY id

SETTINGS index_granularity = 8192;

VIN码解析数据：cj_20191231_20240609_vin_original

-- data20240609.cj_20191231_20240609_vin_original definition

CREATE TABLE data20240609.cj_20191231_20240609_vin_original

(

	`id` String,

	`encoded_vin` String,

	`vin11` Nullable(String),

	`vin` Nullable(String),

	`arrival_date` Nullable(String) COMMENT '接车日期',

	`release_date` Nullable(String) COMMENT '释放日期',

	`province` Nullable(String) COMMENT '省',

	`city` Nullable(String) COMMENT '市',

	`district` Nullable(String) COMMENT '区',

	`province_gaode` Nullable(String) COMMENT '省_高德',

	`city_gaode` Nullable(String) COMMENT '市_高德',

	`district_gaode` Nullable(String) COMMENT '区_高德',

	`province_id_gaode` Nullable(String) COMMENT '省ID_高德',

	`city_id_gaode` Nullable(String) COMMENT '市ID_高德',

	`district_id_gaode` Nullable(String) COMMENT '区ID_高德',

	`brand` Nullable(String) COMMENT '品牌',

	`manufacturer_name` Nullable(String) COMMENT '厂商',

	`series_name` Nullable(String) COMMENT '车系',

	`brand_autohome` Nullable(String) COMMENT '品牌_汽车之家',

	`manufacturer_autohome` Nullable(String) COMMENT '厂商_汽车之家',

	`series_autohome` Nullable(String) COMMENT '车系_汽车之家',

	`brand_id` Nullable(String) COMMENT '品牌ID',

	`manufacturer_id` Nullable(String) COMMENT '厂商ID',

	`series_id` Nullable(String) COMMENT '车系ID',

	`model_name` Nullable(String) COMMENT '车型',

	`model_id` Nullable(String) COMMENT '车型ID',

	`country_of_origin` Nullable(String) COMMENT '国别',

	`brand_type` Nullable(String) COMMENT '品牌类型',

	`brand_tier` Nullable(String) COMMENT '品牌档次',

	`series_level_second` Nullable(String) COMMENT '车系级别_二级',

	`body_style` Nullable(String) COMMENT '车身结构',

	`energy_type` Nullable(String) COMMENT '能源类型',

	`drive_system` Nullable(String) COMMENT '驱动方式',

	`transmission` Nullable(String) COMMENT '变速箱',

	`msrp` Nullable(String) COMMENT '厂商指导价',

	`distribution_channel` Nullable(String) COMMENT '渠道',

	`launch_date` Nullable(String) COMMENT '上市时间',

	`on_sale` Nullable(String) COMMENT '是否在售',

	`series_level_first` Nullable(String) COMMENT '车系级别_一级',

	`energy_category` Nullable(String) COMMENT '能源类别',

	`average_msrp` Nullable(Float64) COMMENT '厂商指导价_均值',

	`price_range` Nullable(String) COMMENT '价格区间'

)

ENGINE = MergeTree

ORDER BY id

SETTINGS index_granularity = 8192;

库存指标数据：full_index_single_index_joint_index_data_statistics

-- data20240609.full_index_single_index_joint_index_data_statistics definition

CREATE TABLE data20240609.full_index_single_index_joint_index_data_statistics

(

	`province` String COMMENT '省',

	`city` String COMMENT '市',

	`district` Nullable(String) COMMENT '区',

	`brand` Nullable(String) COMMENT '品牌',

	`manufacturer` Nullable(String) COMMENT '厂商',

	`series_name` Nullable(String) COMMENT '系列',

	`date` Date COMMENT '当天日期',

	`cumulative_release_count` UInt64 COMMENT '前30天累计释放台数',

	`lwa_count` Nullable(UInt64) COMMENT '长库龄当天收车总数',

	`lwa_r_count` Nullable(UInt64) COMMENT '长库龄当天释放车总数',

	`lwa_total` Nullable(UInt64) COMMENT '长库龄当天累计收车总数',

	`lwa_r_total` Nullable(UInt64) COMMENT '长库龄当天累计释放总数',

	`lwa_inventory` Nullable(UInt64) COMMENT '长库龄库存',

	`total_days_in_stock` String COMMENT '在库车辆的总在库天数',

	`release_sum_turnover_days` Nullable(UInt64) COMMENT '释放车辆的总在库车数',

	`receive_count` UInt64 COMMENT '当天收车总数',

	`release_count` UInt64 COMMENT '当天释放车总数',

	`daily_inventory` UInt64 COMMENT '当日库存数',

	`inventory_depth` Nullable(Float32) COMMENT '库存深度',

	`turnover_days` Nullable(UInt64) COMMENT '流转天数',

	`lwa_percent` Nullable(Float32) COMMENT '长库龄百分比'

)

ENGINE = MergeTree

PRIMARY KEY date

ORDER BY date

SETTINGS index_granularity = 8192;

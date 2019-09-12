###sqoop实战###
一.任务:
   同步Oracle数据库交易数据到hive数据库
二.使用工具:
   sqoop1,azkaban,hive,shell
三.步骤:
   (1):同步oracle交易全量数据到hive
   (2):对hive的交易全量数据清洗整合
   (3):每天定时同步oracle交易增量数据到hive
   (4):每天定时对hive的交易增量数据清洗整合

四.同步全量数据
   (1)创建order.sh同步60个月的数据到hive,分区存储

	#!/bin/bash

	tableName=ORDER
	hiveTableName=s_order
	for(( i=1; i<=60; i++ ))
	do
	    lastDay=`date -d "${date} 1 day ago" +"%Y-%m-%d"`
		firstDay=`date -d "${date} 1 month ago" +"%Y-%m-%d"`
		path=`date -d "${date} 1 month ago" +"%Y%m"`
		date=`date -d "${date} 1 month ago" +"%Y-%m-%d"`
	 sqoop import  \
	 --connect jdbc:oracle:thin:@0.0.0.0:1111:TEST  \
	 --username TEST \
	 --password test \
	 --table ${tableName} \
	 --where "ORDER_TIME >= to_date('${firstDay}','yyyy-mm-dd') and ORDER_TIME <= to_date('${lastDay}','yyyy-mm-dd')" \
	 --hive-database stage \
	 --hive-table ${hiveTableName} \
	 --hive-overwrite          \
	 --hive-import  \
	 --hive-partition-key ymt \
	 --hive-partition-value ${path}  \
	 --delete-target-dir \
	 --null-string '\\N'  \
	 --null-non-string '\\N' \
	 --as-parquetfile \
	 --fields-terminated-by '≡' \
	 --lines-terminated-by '≠' \
	 --m 1
	done
	
(2)创建azkaban任务stg_order.job

	type=command
	command=sh order.sh

(3)创建清洗结果表t_order


	CREATE external TABLE `t_order`(	
    `Store_Id` varchar(20),                                                									
    `Trd_Dt`  timestamp,                                                        										
    `Setl_Account_Seq`   varchar(20),                                               												
    `Change_Price_Bill_Id`  varchar(20),                                               													
    `Account_Store_Id` varchar(20),                                                   													
    `Purch_Center_Id`   varchar(20),                                                 													
    `CHANGE_PRICE_CATEG_CD`   varchar(20),                                            													
    `Change_Price_Dt`   timestamp,                                                  												
    `Change_Price_Amt`   double,                                                 												
    `Confirm_Amt`         double,                                                												
    `Setl_Account_Dt`  timestamp,                                                   												
    `Acct_Statement_Dt`   timestamp,                                                												
    `Daily_Setl_Ind`	varchar(20),                                                 													
    `Record_Status_Cd`    varchar(20),                                                													
    `Src_Sysname`       varchar(20),                                                  														
    `Src_Table`        varchar(20),                                                   														
    `Etl_Job`        varchar(20),                                                     													
    `Etl_Tx_Dt`      timestamp,                                                     													
    `Etl_First_Dt`    timestamp,                                                    														
    `Etl_Proc_Dt`    timestamp,                                                     													
    `Etl_Batch_No`   bigint
	  )
	COMMENT '订单表'	
	PARTITIONED BY (`y` int,  `m` int,`d` int )	
	ROW FORMAT DELIMITED FIELDS TERMINATED BY '≡'
	STORED AS PARQUET
	LOCATION	'/databases/hive/order'

(4)创建azkaban任务order_all.job,清洗s_order数据到t_order

	type=command
	command=sh order_all.sh
	dependencies = stg_order

(5)创建order_all.sh脚本执行任务

	#!/bin/bash
	date=`date -d"$(date -d "1 month" +%Y-%m-01)" +%Y-%m-%d`
	for(( i=1; i<=60; i++ ))
	do
		path=`date -d "${date} 1 month ago" +"%Y%m"`
		date=`date -d "${date} 1 month ago" +"%Y-%m-%d"`
	hive --hiveconf hive.exec.dynamic.partition.mode=nonstrict -hiveconf path=${path} -f order_all.sql 
	done

(6)创建order_all.sql

	insert overwrite table pdm.t04_store_mat_change_price_mas 	partition(y,m,d)
	 select     Store_Id                                                            												
			   ,Trd_Dt                                                              												
			   ,Setl_Account_Seq                                                    												
			   ,Change_Price_Bill_Id                                                													
			   ,Account_Store_Id                                                    													
			   ,Purch_Center_Id                                                     													
			   ,CHANGE_PRICE_CATEG_CD                                               													
			   ,Change_Price_Dt                                                     												
			   ,Change_Price_Amt                                                    												
			   ,Confirm_Amt                                                         												
			   ,Setl_Account_Dt                                                     												
			   ,Acct_Statement_Dt                                                   												
			   ,Daily_Setl_Ind                                                      													
			   ,Daily_Setl_Dt                                                       													
			   ,AG_Input_Ind                                                        													
			   ,Record_Status_Cd                                                    													
			   ,Src_Sysname                                                         														
			   ,Src_Table                                                           														
			   ,Etl_Job                                                             													
			   ,Etl_Tx_Dt                                                           													
			   ,Etl_First_Dt                                                        														
			   ,Etl_Proc_Dt                                                         													
			   ,Etl_Batch_No  
					,y
			   ,m
			   ,d 		   
				FROM (															
	SELECT															
				SUBSTR((CASE WHEN LENGTH(P1.PCH_STORE_ID)=8 THEN P1.PCH_STORE_ID   ELSE '#'||P1.PCH_STORE_ID   END),1,6) AS Store_Id															
			   ,COALESCE(P1.PCH_BUSINESS_DATE,null) AS Trd_Dt															
			   ,COALESCE(P1.PCH_NO,'') AS Setl_Account_Seq                                             															
			   ,COALESCE(P1.PCH_SERIAL_NUMBER,'')  AS   Change_Price_Bill_Id                               														
			   ,CASE  WHEN LENGTH(P1.PCH_STORE_ID)=8 THEN P1.PCH_STORE_ID   ELSE '#'||P1.PCH_STORE_ID   END AS Account_Store_Id															
			   ,COALESCE(P1.PCH_COMPANY_GROUP_ID,'')  AS Purch_Center_Id                             														
			   ,COALESCE(P1.PCH_TYPE,'')     AS CHANGE_PRICE_CATEG_CD                                      														
			   ,COALESCE(P1.PCH_DATE,null) AS Change_Price_Dt 															
			   ,COALESCE(P1.PCH_CHANGE_AMOUNT,0) AS  Change_Price_Amt                                 															
			   ,COALESCE(P1.PCH_CONFIRM_AMOUNT,0)  AS   Confirm_Amt                              														
			   ,COALESCE(P1.PCH_ACCOUNT_DATE,null) AS Setl_Account_Dt 														
			   ,COALESCE(P1.PCH_REPORT_DATE,null) AS Acct_Statement_Dt														
			   ,CASE WHEN  P1.PCH_DAYPROCESS   ='01' THEN '1'                                   															
							WHEN P1.PCH_DAYPROCESS  = '00' THEN '0'                 															
							ELSE ''                                                 															
				END    AS Daily_Setl_Ind                                                             															
			   ,COALESCE(P1.PCH_DAYPROCESS_DATE,null) AS Daily_Setl_Dt															
			   ,CASE WHEN  P1.PCH_CENTER_INPUT   ='01' THEN '1'                                 															
							WHEN P1.PCH_CENTER_INPUT  = '00' THEN '0'               															
							ELSE ''                                                 															
				 END     AS  AG_Input_Ind                                                          														
			   ,COALESCE(P1.PCH_RECORD_STATUS,'')   AS Record_Status_Cd                               														
			   ,'店计系统'     AS  Src_Sysname                                       															
			   ,'PURCHASE_ORDER_HEADER'   AS  Src_Table                           												
			   ,'店铺变价头档' AS  Etl_Job                              															
			   ,unix_timestamp()   AS  Etl_Tx_Dt      														
			   ,CURRENT_TIMESTAMP        AS  Etl_First_Dt                         														
			   ,CURRENT_TIMESTAMP        AS  Etl_Proc_Dt                         														
			   ,0   AS  Etl_Batch_No   
			   ,year( from_unixtime(cast(P1.PCH_BUSINESS_DATE/1000 as bigint))) as y 
			   ,month( from_unixtime(cast(P1.PCH_BUSINESS_DATE/1000 as bigint))) as m
			   ,day( from_unixtime(cast(P1.PCH_BUSINESS_DATE/1000 as bigint)))as d	   
	FROM       stage.s09_price_change_header P1 
	where P1.PCH_BUSINESS_DATE is not null and P1.ym=${hiveconf:path}) s


五.同步增量数据
(1)创建sqoop job:job_s_order_add.job 同步Oracle增量数据到hive的

	#!/bin/bash

	tableName=ORDER
	hiveTableName=s_order
	 sqoop job --create job_s_order --\
	 import  \
	 --connect jdbc:oracle:thin:@0.0.0.0:1111:TEST  \
	 --username TEST \
	 --password-file file:///data/sqoop_job/.password \
	 --table ${tableName} \
	 --where "ORDER_TIME >= to_date('$(date -d '15 day ago' '+%Y-%m-%d')','yyyy-mm-dd') and ORDER_TIME <= to_date('$(date '+%Y-%m-%d')','yyyy-mm-dd')" \
	 --hive-database stg \
	 --hive-table ${hiveTableName} \
	 --hive-overwrite          \
	 --hive-import  \
	 --target-dir /user/TEST.ORDER \
	 --delete-target-dir \
	 --null-string '\\N'  \
	 --null-non-string '\\N' \
	 --as-parquetfile \
	 --fields-terminated-by '≡' \
	 --lines-terminated-by '≠' \
	 --m 1

创建.password文件存放Oracle密码:

	111111

(2)创建azkaban任务s_order_add.job

	type=command
	command=sqoop job --exec job_s_order_add

(4)创建azkaban任务t_order_add.job

	type=command
	command=hive --hiveconf hive.exec.dynamic.partition.mode=nonstrict -f t_order_add.sql
	dependencies =s_order_add

(5)创建t_order_add.sql

	 insert overwrite table pdm.t04_store_mat_change_price_mas 	partition(y,m,d)
 	 select     Store_Id                                                            												
		   ,Trd_Dt                                                              												
		   ,Setl_Account_Seq                                                    												
		   ,Change_Price_Bill_Id                                                													
		   ,Account_Store_Id                                                    													
		   ,Purch_Center_Id                                                     													
		   ,CHANGE_PRICE_CATEG_CD                                               													
		   ,Change_Price_Dt                                                     												
		   ,Change_Price_Amt                                                    												
		   ,Confirm_Amt                                                         												
		   ,Setl_Account_Dt                                                     												
		   ,Acct_Statement_Dt                                                   												
		   ,Daily_Setl_Ind                                                      													
		   ,Daily_Setl_Dt                                                       													
		   ,AG_Input_Ind                                                        													
		   ,Record_Status_Cd                                                    													
		   ,Src_Sysname                                                         														
		   ,Src_Table                                                           														
		   ,Etl_Job                                                             													
		   ,Etl_Tx_Dt                                                           													
		   ,Etl_First_Dt                                                        														
		   ,Etl_Proc_Dt                                                         													
		   ,Etl_Batch_No  
				,y
		   ,m
		   ,d 		   
			FROM (															
	SELECT															
			SUBSTR((CASE WHEN LENGTH(P1.PCH_STORE_ID)=8 THEN P1.PCH_STORE_ID   ELSE '#'||P1.PCH_STORE_ID   END),1,6) AS Store_Id															
		   ,COALESCE(P1.PCH_BUSINESS_DATE,null) AS Trd_Dt															
		   ,COALESCE(P1.PCH_NO,'') AS Setl_Account_Seq                                             															
		   ,COALESCE(P1.PCH_SERIAL_NUMBER,'')  AS   Change_Price_Bill_Id                               														
		   ,CASE  WHEN LENGTH(P1.PCH_STORE_ID)=8 THEN P1.PCH_STORE_ID   ELSE '#'||P1.PCH_STORE_ID   END AS Account_Store_Id															
		   ,COALESCE(P1.PCH_COMPANY_GROUP_ID,'')  AS Purch_Center_Id                             														
		   ,COALESCE(P1.PCH_TYPE,'')     AS CHANGE_PRICE_CATEG_CD                                      														
		   ,COALESCE(P1.PCH_DATE,null) AS Change_Price_Dt 															
		   ,COALESCE(P1.PCH_CHANGE_AMOUNT,0) AS  Change_Price_Amt                                 															
		   ,COALESCE(P1.PCH_CONFIRM_AMOUNT,0)  AS   Confirm_Amt                              														
		   ,COALESCE(P1.PCH_ACCOUNT_DATE,null) AS Setl_Account_Dt 														
		   ,COALESCE(P1.PCH_REPORT_DATE,null) AS Acct_Statement_Dt														
		   ,CASE WHEN  P1.PCH_DAYPROCESS   ='01' THEN '1'                                   															
						WHEN P1.PCH_DAYPROCESS  = '00' THEN '0'                 															
						ELSE ''                                                 															
			END    AS Daily_Setl_Ind                                                             															
		   ,COALESCE(P1.PCH_DAYPROCESS_DATE,null) AS Daily_Setl_Dt															
		   ,CASE WHEN  P1.PCH_CENTER_INPUT   ='01' THEN '1'                                 															
						WHEN P1.PCH_CENTER_INPUT  = '00' THEN '0'               															
						ELSE ''                                                 															
			 END     AS  AG_Input_Ind                                                          														
		   ,COALESCE(P1.PCH_RECORD_STATUS,'')   AS Record_Status_Cd                               														
		   ,'店计系统'     AS  Src_Sysname                                       															
		   ,'PURCHASE_ORDER_HEADER'   AS  Src_Table                           												
		   ,'店铺变价头档' AS  Etl_Job                              															
		   ,unix_timestamp()   AS  Etl_Tx_Dt      														
		   ,CURRENT_TIMESTAMP        AS  Etl_First_Dt                         														
		   ,CURRENT_TIMESTAMP        AS  Etl_Proc_Dt                         														
		   ,0   AS  Etl_Batch_No   
		   ,year( from_unixtime(cast(P1.PCH_BUSINESS_DATE/1000 as bigint))) as y 
		   ,month( from_unixtime(cast(P1.PCH_BUSINESS_DATE/1000 as bigint))) as m
		   ,day( from_unixtime(cast(P1.PCH_BUSINESS_DATE/1000 as bigint)))as d	   
	FROM       stage.s09_cre_price_change_header P1 
	where P1.PCH_BUSINESS_DATE is not null) s





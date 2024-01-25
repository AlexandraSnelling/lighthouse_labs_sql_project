What are your risk areas? Identify and describe them.

The #1 risk area that I found in the data is that there is no SKU or any other column that can be used to identify the product being sold in the analytics data. 
Product sku, name, category, etc. can be found in the the all_sessions data. 
Units_sold must come from the analytics data, since this information is missing from the Products data.
The only information that can be used to join these two data sets is visitId.
(or a combo of visitId+fullVisitorId, as was done in our analysis)

In the analytics data, a single instance of visitId corresponds to multiple instances of units_sold and unit_price.
This could make sense if say, a customer had purchased different quantities of different products in one visit.

In the all_sessions data, a single instance of visitId corresponds to multiple instances of productSKU, ProductName, etc. 
This could again make sense sense if a customer had purchased different products in one visit.

However, when visitId is used to join the two data sets, it is not possible to identify the correct sku connected to each row in the analytics data so information must be lost from one of the two tables.
(Since product_price and unit_price do not align, this is no help either.)

QA Process:
Describe your QA process and include the SQL queries used to execute it.


--let's try joining our analytics_temp_table_2 to our all_sessions_temp_table_2 using both the visit_key 
WITH all_sessions_temp_table_2 AS(
	SELECT 	DISTINCT(CONCAT("visitId"::varchar, "fullVisitorId"::varchar)) AS visit_key,	
			"productSKU" AS product_sku,
			"productPrice" AS product_price
	FROM public.all_sessions
	ORDER BY visit_key
	)--Total rows: 15129

   ,analytics_temp_table_2 AS(
	SELECT 	CONCAT("visitStartTime"::varchar, "fullvisitorId"::varchar) AS visit_key, 
			units_sold, 
			unit_price
	FROM public.analytics
	WHERE NOT units_sold ISNULL
	GROUP BY visit_key, units_sold, unit_price
	)--Total rows: 50176

   ,join_temp_table_2s AS(
	SELECT 	analytics_temp_table_2.visit_key, 
			units_sold, 
			unit_price,
			product_price,
			product_sku
	FROM analytics_temp_table_2
	FULL OUTER JOIN all_sessions_temp_table_2
	ON analytics_temp_table_2.visit_key = all_sessions_temp_table_2.visit_key
	WHERE NOT units_sold ISNULL
	AND NOT product_sku ISNULL
	ORDER BY visit_key
	)--Total rows: 208

--SELECT DISTINCT(visit_key)
--FROM join_temp_table_2s
--Total rows: 137

   ,join_temp_table_2s_duplicate_visit_ids AS(
	SELECT "visit_key", SUM(1) AS "count"
	FROM join_temp_table_2s
	GROUP BY 1
	HAVING SUM(1) >1
	)
    
	,all_sessions_temp_table_2_with_duplicate_visit_ids_in_join_table AS(
	SELECT *
	FROM all_sessions_temp_table_2
	--FROM join_temp_table_2s
	--FROM public.all_sessions
	--WHERE CONCAT("visitId"::varchar, "fullVisitorId"::varchar) IN (SELECT visit_key FROM join_temp_table_2s_duplicate_visit_ids)
	--ORDER BY CONCAT("visitId"::varchar, "fullVisitorId"::varchar)
	--ORDER BY "visitId"
	WHERE visit_key IN (SELECT visit_key FROM join_temp_table_2s_duplicate_visit_ids)
	ORDER BY visit_key
	--ORDER BY "visitId"
	--ORDER BY product_sku
	)--Total rows: 38
	
--SELECT DISTINCT(visit_key)
--FROM all_sessions_temp_table_2_with_duplicate_visit_ids_in_join_table
--Total rows: 36 so only two instances

	,last_count_table AS(
	SELECT "visit_key", SUM(1) AS "count"
	FROM all_sessions_temp_table_2_with_duplicate_visit_ids_in_join_table
	GROUP BY 1
	HAVING SUM(1) >1
	)
	
	--X,all_sessions_temp_table_2_with_duplicate_visit_ids_in_join_table AS(
	SELECT *
	FROM all_sessions_temp_table_2_with_duplicate_visit_ids_in_join_table
	--FROM join_temp_table_2s
	--FROM public.all_sessions
	--WHERE CONCAT("visitId"::varchar, "fullVisitorId"::varchar) IN (SELECT visit_key FROM join_temp_table_2s_duplicate_visit_ids)
	--ORDER BY CONCAT("visitId"::varchar, "fullVisitorId"::varchar)
	--ORDER BY "visitId"
	
	FULL OUTER JOIN analytics_temp_table_2
	ON all_sessions_temp_table_2_with_duplicate_visit_ids_in_join_table.visit_key = analytics_temp_table_2.visit_key
	
	WHERE analytics_temp_table_2.visit_key IN (SELECT visit_key FROM last_count_table)
	ORDER BY analytics_temp_table_2.visit_key

--Here we see that ther are two instances of visit keys over which SKUs will be lost
--visit_key = 14965538013401942741070364584
--visit_key = 14964127528946840755894418080
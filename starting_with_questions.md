Answer the following questions and provide the SQL queries used to find the answer.

**************************************************************************************************************
>>>>>>>>>>>>>>>>>>>--QUESTIONS ANSWERED USING ANALYTICS AND ALL_SESSIONS DATA COMBINED--<<<<<<<<<<<<<<<<<<<<<<
**************************************************************************************************************
    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:
/*
CTE Legend for new tables created as cleaned ecommerce database: 

analytics_temp_table_1 >>> (shown as ---- analytics_revenue ---- in ERD)
analytics_temp_table_2 >>> (shown as ---- analytics_units ---- in ERD)
analytics_temp_table_3 >>> (shown as ---- analytics_visit_key ---- in ERD)

all_sessions_temp_table_1 >>> (shown as ---- all_sessions_location ---- in ERD)
all_sessions_temp_table_2 >>> (shown as ---- all_sessions_product_info ---- in ERD)
all_sessions_temp_table_3 >>> (shown as ---- all_sessions_product_category ---- in ERD)

products >>> (shown as ---- products ---- in ERD)
*/ 
WITH analytics_temp_table_2 AS(
	SELECT 	CONCAT("visitStartTime"::varchar, "fullvisitorId"::varchar) AS visit_key, 
			units_sold, 
			unit_price
	FROM public.analytics
	WHERE NOT units_sold ISNULL
	GROUP BY visit_key, units_sold, unit_price
	)

    ,all_sessions_temp_table_1 AS(
	SELECT 	DISTINCT(CONCAT("visitId"::varchar, "fullVisitorId"::varchar)) AS visit_key,	
			country,
			city
	FROM public.all_sessions
	)
	
--Create CTE showing revenue for each visit_key (calculated as the sum of units_sold * unit_price) 
--/1000000 to give logical revenue values	
   ,revenue_per_visit AS(
	SELECT visit_key, ROUND(SUM(units_sold * unit_price)/1000000, 2) AS revenues
	FROM analytics_temp_table_2
	GROUP BY visit_key
	)


-->>>TRANSACTION REVENUES BY COUNTRY<<<--
--Calculate sum of revenue by country and arrange in descending order
SELECT country, SUM(revenues) AS "transaction revenue"
FROM revenue_per_visit 
JOIN all_sessions_temp_table_1
ON revenue_per_visit .visit_key = all_sessions_temp_table_1.visit_key
GROUP BY country
ORDER BY "transaction revenue" DESC


-->>>TRANSACTION REVENUES BY COUNTRY<<<--
--Calculate sum of revenue by country and arrange in descending order
SELECT city, country, SUM(revenues) AS "transaction revenue"
FROM revenue_per_visit 
JOIN all_sessions_temp_table_1
ON revenue_per_visit .visit_key = all_sessions_temp_table_1.visit_key
--remove illogical city values
WHERE NOT city IN ('(not set)', 'not available in demo dataset') 
GROUP BY city, country
ORDER BY "transaction revenue" DESC

Answer:

Top 5 countries:United States, Mexico, Sweden, Canada, United Kingdom
Top 5 cities: Sunnyvale US, Mountain View US, Chicago US, San Francisco US, Palo Alto US


**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:
/*
CTE Legend for new tables created as cleaned ecommerce database: 

analytics_temp_table_1 >>> (shown as ---- analytics_revenue ---- in ERD)
analytics_temp_table_2 >>> (shown as ---- analytics_units ---- in ERD)
analytics_temp_table_3 >>> (shown as ---- analytics_visit_key ---- in ERD)

all_sessions_temp_table_1 >>> (shown as ---- all_sessions_location ---- in ERD)
all_sessions_temp_table_2 >>> (shown as ---- all_sessions_product_info ---- in ERD)
all_sessions_temp_table_3 >>> (shown as ---- all_sessions_product_category ---- in ERD)

products >>> (shown as ---- products ---- in ERD)
*/
WITH analytics_temp_table_2 AS(
	SELECT 	CONCAT("visitStartTime"::varchar, "fullvisitorId"::varchar) AS visit_key, 
			units_sold, 
			unit_price
	FROM public.analytics
	WHERE NOT units_sold ISNULL
	GROUP BY visit_key, units_sold, unit_price
	)
	
   ,analytics_temp_table_3 AS(
	SELECT 	DISTINCT(CONCAT("visitStartTime"::varchar, "fullvisitorId"::varchar)) AS visit_key,	
			"visitNumber" AS visit_number,
			"visitStartTime" AS visit_start_time,
			date,
			"fullvisitorId" AS full_visitor_id,
			"channelGrouping" AS channel_grouping,
			pageviews AS page_views,
			timeonsite AS time_on_site,
			bounces
	FROM public.analytics
	WHERE NOT units_sold ISNULL
	)

    ,all_sessions_temp_table_1 AS(
	SELECT 	DISTINCT(CONCAT("visitId"::varchar, "fullVisitorId"::varchar)) AS visit_key,	
			country,
			city
	FROM public.all_sessions
	)
	
--Create CTE showing average order by visitor id 
	,average_products_ordered_per_visitor AS(
	SELECT full_visitor_id, country, city, AVG(units_sold) AS average_products_per_visitor
	FROM analytics_temp_table_2
	JOIN analytics_temp_table_3
	ON analytics_temp_table_2.visit_key = analytics_temp_table_3.visit_key
	JOIN all_sessions_temp_table_1
	ON analytics_temp_table_2.visit_key = all_sessions_temp_table_1.visit_key
	GROUP BY full_visitor_id, country, city
	)

-->>>AVERAGE PRODUCTS ORDERED BY VISITORS IN EACH COUNTRY<<<---
--Take the average of all visitor averages, grouping by country 
SELECT 	country,
		AVG(average_products_per_visitor)::numeric(10,2) AS "average products ordered from visitors"
FROM average_products_ordered_per_visitor
GROUP BY country
ORDER BY "average products ordered from visitors" DESC
	
 
-->>>AVERAGE PRODUCTS ORDERED BY VISITORS IN EACH CITY<<<---
--Take the average of all visitor averages, grouping by city 
SELECT 	city, country,
		AVG(average_products_per_visitor)::numeric(10,2) AS "average products ordered from visitors"
FROM average_products_ordered_per_visitor
--remove illogical city values
WHERE NOT city IN ('(not set)', 'not available in demo dataset') 
GROUP BY city, country
ORDER BY "average products ordered from visitors" DESC



Answer:
For averages by country, Canada has top average products ordered of 3.17 , US next with 1.89, Mexico and Bulgaria in 3rd with 1.5
Top 2 cities are Chicago and Pittsburg US with 5.39 and 4 Avg orders, 3rd is Toronto Canada with 3
Too long to list all, please run -->>>AVERAGE PRODUCTS ORDERED BY VISITORS IN EACH COUNTRY<<<--- 
and -->>>AVERAGE PRODUCTS ORDERED BY VISITORS IN EACH CITY<<<--- with CTEs for full lists



**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:

/*
CTE Legend for new tables created as cleaned ecommerce database: 

analytics_temp_table_1 >>> (shown as ---- analytics_revenue ---- in ERD)
analytics_temp_table_2 >>> (shown as ---- analytics_units ---- in ERD)
analytics_temp_table_3 >>> (shown as ---- analytics_visit_key ---- in ERD)

all_sessions_temp_table_1 >>> (shown as ---- all_sessions_location ---- in ERD)
all_sessions_temp_table_2 >>> (shown as ---- all_sessions_product_info ---- in ERD)
all_sessions_temp_table_3 >>> (shown as ---- all_sessions_product_category ---- in ERD)

products >>> (shown as ---- products ---- in ERD)
*/
WITH analytics_temp_table_2 AS(
	SELECT 	CONCAT("visitStartTime"::varchar, "fullvisitorId"::varchar) AS visit_key, 
			units_sold, 
			unit_price
	FROM public.analytics
	WHERE NOT units_sold ISNULL
	GROUP BY visit_key, units_sold, unit_price
	)
	
   ,analytics_temp_table_3 AS(
	SELECT 	DISTINCT(CONCAT("visitStartTime"::varchar, "fullvisitorId"::varchar)) AS visit_key,	
			"visitNumber" AS visit_number,
			"visitStartTime" AS visit_start_time,
			date,
			"fullvisitorId" AS full_visitor_id,
			"channelGrouping" AS channel_grouping,
			pageviews AS page_views,
			timeonsite AS time_on_site,
			bounces
	FROM public.analytics
	WHERE NOT units_sold ISNULL
	)

    ,all_sessions_temp_table_1 AS(
	SELECT 	DISTINCT(CONCAT("visitId"::varchar, "fullVisitorId"::varchar)) AS visit_key,	
			country,
			city
	FROM public.all_sessions
	)
	
--Create CTE showing average order by visitor id 
	,average_products_ordered_per_visitor AS(
	SELECT full_visitor_id, country, city, AVG(units_sold) AS average_products_per_visitor
	FROM analytics_temp_table_2
	JOIN analytics_temp_table_3
	ON analytics_temp_table_2.visit_key = analytics_temp_table_3.visit_key
	JOIN all_sessions_temp_table_1
	ON analytics_temp_table_2.visit_key = all_sessions_temp_table_1.visit_key
	GROUP BY full_visitor_id, country, city
	)

-->>>AVERAGE PRODUCTS ORDERED BY VISITORS IN EACH COUNTRY<<<---
--Take the average of all visitor averages, grouping by country 
SELECT 	country,
		AVG(average_products_per_visitor)::numeric(10,2) AS "average products ordered from visitors"
FROM average_products_ordered_per_visitor
GROUP BY country
ORDER BY "average products ordered from visitors" DESC
	
--Canada has top average products ordered of 3.17 , US next with 1.89, Mexico and Bulgaria in 3rd with 1.5
 
-->>>AVERAGE PRODUCTS ORDERED BY VISITORS IN EACH CITY<<<---
--Take the average of all visitor averages, grouping by city 
SELECT 	city, country,
		AVG(average_products_per_visitor)::numeric(10,2) AS "average products ordered from visitors"
FROM average_products_ordered_per_visitor
--remove illogical city values
WHERE NOT city IN ('(not set)', 'not available in demo dataset') 
GROUP BY city, country
ORDER BY "average products ordered from visitors" DESC

--Top 2 cities are Chicago and Pittsburg US with 5.39 and 4 Avg orders, 3rd is Toronto Canada with 3
WITH analytics_temp_table_2 AS(
	SELECT 	CONCAT("visitStartTime"::varchar, "fullvisitorId"::varchar) AS visit_key, 
			units_sold, 
			unit_price
	FROM public.analytics
	WHERE NOT units_sold ISNULL
	GROUP BY visit_key, units_sold, unit_price
	)

    ,all_sessions_temp_table_1 AS(
	SELECT 	DISTINCT(CONCAT("visitId"::varchar, "fullVisitorId"::varchar)) AS visit_key,	
			country,
			city
	FROM public.all_sessions
	)
	
   ,all_sessions_temp_table_3 AS(
	SELECT 	DISTINCT(CONCAT("visitId"::varchar, "fullVisitorId"::varchar)) AS visit_key,	
			"v2ProductCategory" AS product_category
	FROM public.all_sessions
	)

--SELECT DISTINCT(product_category)
--Tital rows: 38
SELECT SUM(units_sold) AS "total products ordered", country, product_category
FROM analytics_temp_table_2
JOIN all_sessions_temp_table_1
ON analytics_temp_table_2.visit_key = all_sessions_temp_table_1.visit_key
JOIN all_sessions_temp_table_3
ON analytics_temp_table_2.visit_key = all_sessions_temp_table_3.visit_key
GROUP BY product_category, country
ORDER BY country, "total products ordered" DESC

Answer:
Too long to list, please run Too long to list, please run with CTEs to view lists.




**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:

/*
CTE Legend for new tables created as cleaned ecommerce database: 

analytics_temp_table_1 >>> (shown as ---- analytics_revenue ---- in ERD)
analytics_temp_table_2 >>> (shown as ---- analytics_units ---- in ERD)
analytics_temp_table_3 >>> (shown as ---- analytics_visit_key ---- in ERD)

all_sessions_temp_table_1 >>> (shown as ---- all_sessions_location ---- in ERD)
all_sessions_temp_table_2 >>> (shown as ---- all_sessions_product_info ---- in ERD)
all_sessions_temp_table_3 >>> (shown as ---- all_sessions_product_category ---- in ERD)

products >>> (shown as ---- products ---- in ERD)
*/ 

WITH analytics_temp_table_2 AS(
	SELECT 	CONCAT("visitStartTime"::varchar, "fullvisitorId"::varchar) AS visit_key, 
			units_sold, 
			unit_price
	FROM public.analytics
	WHERE NOT units_sold ISNULL
	GROUP BY visit_key, units_sold, unit_price
	)

    ,all_sessions_temp_table_1 AS(
	SELECT 	DISTINCT(CONCAT("visitId"::varchar, "fullVisitorId"::varchar)) AS visit_key,	
			country,
			city
	FROM public.all_sessions
	)
	
   	,all_sessions_temp_table_2 AS(
	SELECT 	DISTINCT(CONCAT("visitId"::varchar, "fullVisitorId"::varchar)) AS visit_key,	
			"productSKU" AS product_sku,
			"productPrice" AS product_price
	FROM public.all_sessions
	)
	
	,all_sessions_temp_table_3 AS(
	SELECT 	DISTINCT(CONCAT("visitId"::varchar, "fullVisitorId"::varchar)) AS visit_key,	
			"v2ProductCategory" AS product_category
	FROM public.all_sessions
	)

-->>>TOP SELLING PRODUCTS BY COUNTRY<<<<<--
--Create CTE displaying units sold by country with product name and product_sales_rank 
--product_sales_rank calculated as dense rank of total_units_sold over country
--Uses data from inner joins of analytics_temp_table_2, all_sessions_temp_table_1,2,3 and products table 
	,product_rank_by_country AS(
	SELECT country, name, SUM(units_sold) AS total_units_sold,
			DENSE_RANK() OVER (
			PARTITION BY country
			ORDER BY SUM(units_sold) DESC) product_sales_rank
	FROM analytics_temp_table_2
	JOIN all_sessions_temp_table_1
	ON analytics_temp_table_2.visit_key = all_sessions_temp_table_1.visit_key
	JOIN all_sessions_temp_table_2
	ON analytics_temp_table_2.visit_key = all_sessions_temp_table_2.visit_key
	JOIN all_sessions_temp_table_3
	ON analytics_temp_table_2.visit_key = all_sessions_temp_table_3.visit_key
	JOIN public.products
	ON all_sessions_temp_table_2.product_sku = public.products."SKU"
	GROUP BY name, country
	ORDER BY country, total_units_sold DESC
	)

--Select top-selling products by filtering for rows with product_sales_rank = 1
SELECT country, name AS product, total_units_sold As "total units sold"
FROM product_rank_by_country
WHERE product_sales_rank = 1


-->>>TOP SELLING PRODUCTS BY CITY<<<<<--
--Create CTE displaying units sold by city with product name and product_sales_rank 
--product_sales_rank calculated as dense rank of total_units_sold over city
--Uses data from inner joins of analytics_temp_table_2, all_sessions_temp_table_1,2,3 and products table 
	,product_rank_by_city AS(
	SELECT city, country, name, SUM(units_sold) AS total_units_sold,
			DENSE_RANK() OVER (
			PARTITION BY city, country
			ORDER BY SUM(units_sold) DESC) product_sales_rank
	FROM analytics_temp_table_2
	JOIN all_sessions_temp_table_1
	ON analytics_temp_table_2.visit_key = all_sessions_temp_table_1.visit_key
	JOIN all_sessions_temp_table_2
	ON analytics_temp_table_2.visit_key = all_sessions_temp_table_2.visit_key
	JOIN all_sessions_temp_table_3
	ON analytics_temp_table_2.visit_key = all_sessions_temp_table_3.visit_key
	JOIN public.products
	ON all_sessions_temp_table_2.product_sku = public.products."SKU"
	GROUP BY name, city, country
	ORDER BY city, country, total_units_sold DESC
	)

--Select top-selling products by filtering for rows with product_sales_rank = 1
SELECT city, country, name AS product, total_units_sold As "total units sold"
FROM product_rank_by_city
WHERE product_sales_rank = 1
--remove illogical city values
AND NOT city IN ('(not set)', 'not available in demo dataset') 


Answer:

Too long to list, please run >>>TOP SELLING PRODUCTS BY COUNTRY<<<<< or >>>TOP SELLING PRODUCTS BY CITY<<<<< with CTEs to view lists.
2 products were unisually popular: 
SPF-15 Slim & Slender Lip Balm in Sunnyvale US
Alpine Style Backpack in  Chicago US


**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:

--Question 5: Can we summarize the impact of revenue generated from each city/country?
/*
CTE Legend for new tables created as cleaned ecommerce database: 

analytics_temp_table_1 >>> (shown as ---- analytics_revenue ---- in ERD)
analytics_temp_table_2 >>> (shown as ---- analytics_units ---- in ERD)
analytics_temp_table_3 >>> (shown as ---- analytics_visit_key ---- in ERD)

all_sessions_temp_table_1 >>> (shown as ---- all_sessions_location ---- in ERD)
all_sessions_temp_table_2 >>> (shown as ---- all_sessions_product_info ---- in ERD)
all_sessions_temp_table_3 >>> (shown as ---- all_sessions_product_category ---- in ERD)

products >>> (shown as ---- products ---- in ERD)
*/ 
WITH analytics_temp_table_2 AS(
	SELECT 	CONCAT("visitStartTime"::varchar, "fullvisitorId"::varchar) AS visit_key, 
			units_sold, 
			unit_price
	FROM public.analytics
	WHERE NOT units_sold ISNULL
	GROUP BY visit_key, units_sold, unit_price
	)

    ,all_sessions_temp_table_1 AS(
	SELECT 	DISTINCT(CONCAT("visitId"::varchar, "fullVisitorId"::varchar)) AS visit_key,	
			country,
			city
	FROM public.all_sessions
	)

--Create CTE showing revenue for each visit_key (calculated as the sum of units_sold * unit_price) 
--/1000000 to give logical revenue values
   ,revenue_per_visit AS(
	SELECT visit_key, ROUND(SUM(units_sold * unit_price)/1000000, 2) AS revenues
	FROM analytics_temp_table_2
	GROUP BY visit_key
	)
	
--SELECT SUM(revenues)
--FROM revenue_per_visit 
--JOIN all_sessions_temp_table_1
--ON revenue_per_visit .visit_key = all_sessions_temp_table_1.visit_key


-->>>TRANSACTION REVENUES BY COUNTRY AS % TOTAL SALES<<<--
--Calculate sum of revenue by country
--Use nested query to calculate the percentage of total sales revenue represented by each country
SELECT 	country, SUM(revenues) AS "transaction revenue", 
		((SUM(revenues)/(SELECT SUM(revenues)
						 FROM revenue_per_visit 
						 JOIN all_sessions_temp_table_1
						 ON revenue_per_visit .visit_key = all_sessions_temp_table_1.visit_key))
		*100)::NUMERIC(10,2) AS "percent total sales revenue"
FROM revenue_per_visit 
JOIN all_sessions_temp_table_1
ON revenue_per_visit .visit_key = all_sessions_temp_table_1.visit_key
GROUP BY country
ORDER BY "transaction revenue" DESC


-->>>TRANSACTION REVENUES BY CITY AS % TOTAL SALES<<<--
--Calculate sum of revenue by country
--Use nested query to calculate the percentage of total sales revenue represented by each country
SELECT 	city, country, SUM(revenues) AS "transaction revenue", 
		((SUM(revenues)/(SELECT SUM(revenues)
						 FROM revenue_per_visit 
						 JOIN all_sessions_temp_table_1
						 ON revenue_per_visit .visit_key = all_sessions_temp_table_1.visit_key))
		*100)::NUMERIC(10,2) AS "percent total sales revenue"
FROM revenue_per_visit 
JOIN all_sessions_temp_table_1
ON revenue_per_visit .visit_key = all_sessions_temp_table_1.visit_key
--remove illogical city values
WHERE NOT city IN ('(not set)', 'not available in demo dataset') 
GROUP BY city, country
ORDER BY "transaction revenue" DESC

Answer:

Too long to list, please run Too long to list, please run >>>TRANSACTION REVENUES BY CITY AS % TOTAL SALES<<< 
or >>>TRANSACTION REVENUES BY COUNTRY AS % TOTAL SALES<<< with CTEs to view lists.

All but 1 top selling city are in the US and the US is the top selling country by a large margin


**************************************************************************************************************
>>>>>>>>>>>>>>>>>>>>>>>>>>>--QUESTIONS ANSWERED USING ALL_SESSIONS DATA ONLY--<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
**************************************************************************************************************

**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**

SQL Queries:
-->>>TRANSACTION REVENUE BY CITY-USING ALL_SESSIONS DATA ONLY<<<--
WITH transaction_revenues_CTE AS(
	SELECT *
	FROM public.all_sessions 
	WHERE "visitId" IN
		(SELECT DISTINCT("visitId")
		FROM public.all_sessions 
		WHERE NOT "totalTransactionRevenue" ISNULL))

SELECT country, city, ROUND(SUM("totalTransactionRevenue")/1000000)::NUMERIC(10,2) AS "transaction revenue ($)"
FROM transaction_revenues_CTE
GROUP BY city, country
ORDER BY "transaction revenue ($)" DESC

-->>>TRANSACTION REVENUE BY COUNTRY-USING ALL_SESSIONS DATA ONLY<<<--
WITH transaction_revenues_CTE AS(
	SELECT *
	FROM public.all_sessions 
	WHERE "visitId" IN
		(SELECT DISTINCT("visitId")
		FROM public.all_sessions 
		WHERE NOT "totalTransactionRevenue" ISNULL))

SELECT country, ROUND(SUM("totalTransactionRevenue")/1000000)::NUMERIC(10,2) AS "transaction revenue ($)"
FROM transaction_revenues_CTE
GROUP BY country
ORDER BY "transaction revenue ($)" DESC






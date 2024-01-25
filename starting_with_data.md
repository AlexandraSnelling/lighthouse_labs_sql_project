Question 1: Which month and years had the highest sales?

SQL Queries:
/*
CTE Legend for new tables created as cleaned ecommerce database: 

analytics_temp_table_1 >>> (shown as ---- analytics_revenue ---- in ERD)
analytics_temp_table_2 >>> (shown as ---- analytics_units ---- in ERD)
analytics_temp_table_3 >>> (shown as ---- analytics_visit_key ---- in ERD)
*/ 

--List CTEs containing tables created from analytics data
WITH analytics_temp_table_2 AS(
	SELECT 	CONCAT("visitStartTime"::varchar, "fullvisitorId"::varchar) AS visit_key, 
			units_sold, 
			unit_price
	FROM public.analytics
	WHERE NOT units_sold ISNULL
	GROUP BY visit_key, units_sold, unit_price
	)

    ,analytics_temp_table_3 AS(
	SELECT DISTINCT(CONCAT("visitStartTime"::varchar, "fullvisitorId"::varchar)) AS visit_key,	
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

--	create CTE containing revenue, calculated using units_sold by unit_price 
   ,revenue_per_visit AS(
	SELECT visit_key, ROUND(SUM(units_sold * unit_price)/1000000, 2) AS revenues
	FROM analytics_temp_table_2
	GROUP BY visit_key
	)

--EXTRACT year from date and use TO_CHAR to cast the month from date as month in text
--SUM total revenue (units sold x unit price), grouping by month, then year
SELECT 	EXTRACT(YEAR FROM date) AS year, TO_CHAR(date, 'Month') AS Month,--EXTRACT(MONTH FROM date) AS month, 
		SUM(revenues) AS "total revenue"
FROM revenue_per_visit 
JOIN analytics_temp_table_3
ON revenue_per_visit .visit_key = analytics_temp_table_3.visit_key
GROUP BY MONTH, YEAR
ORDER BY "total revenue" DESC

Answer: July 2017


Question 2: Which path to the site (channelgrouping) leads to the most orders?


SQL Queries:
/*
CTE Legend for new tables created as cleaned ecommerce database: 

analytics_temp_table_1 >>> (shown as ---- analytics_revenue ---- in ERD)
analytics_temp_table_2 >>> (shown as ---- analytics_units ---- in ERD)
analytics_temp_table_3 >>> (shown as ---- analytics_visit_key ---- in ERD)
*/ 

--List CTE containing tables created from analytics 
WITH analytics_temp_table_3 AS(
	SELECT DISTINCT(CONCAT("visitStartTime"::varchar, "fullvisitorId"::varchar)) AS visit_key,	
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

--COUNT orders made, Grouping by channel_grouping
SELECT channel_grouping AS "path to website", COUNT(visit_key) AS "orders made"
FROM analytics_temp_table_3
GROUP BY channel_grouping
ORDER BY "orders made" DESC

Answer: Referral



Question 3: Which product price categories perform best?

SQL Queries:
/*
CTE Legend for new tables created as cleaned ecommerce database: 

analytics_temp_table_1 >>> (shown as ---- analytics_revenue ---- in ERD)
analytics_temp_table_2 >>> (shown as ---- analytics_units ---- in ERD)
analytics_temp_table_3 >>> (shown as ---- analytics_visit_key ---- in ERD)
*/ 

WITH analytics_temp_table_2 AS(
	SELECT 	CONCAT("visitStartTime"::varchar, "fullvisitorId"::varchar) AS visit_key, 
			units_sold, 
			unit_price
	FROM public.analytics
	WHERE NOT units_sold ISNULL
	GROUP BY visit_key, units_sold, unit_price
	)
--Look at the Max and Min values to decide which price categories should be used 
/*
SELECT (MIN(unit_price)/1000000) AS min, 
	   (MAX(unit_price)/1000000) AS max
FROM analytics_temp_table_2 
*/

--Use CASE statement to assign consumer-relevant categories
--SUM units sold, grouping by price category
--divide price by 1000000 to give logical numbers
SELECT	SUM (units_sold) AS "units sold",
		CASE
		WHEN unit_price/1000000 < 10 THEN 'Under $10'
		WHEN unit_price/1000000 BETWEEN 10 AND 25 THEN 	'$10 to $25'
		WHEN unit_price/1000000 BETWEEN 25 AND 50 THEN 	'$25 to $50'
		WHEN unit_price/1000000 BETWEEN 50 AND 100 THEN '$50 to $100'
		WHEN unit_price/1000000 BETWEEN 100 AND 250 THEN '$100 to $250'
		WHEN unit_price/1000000 BETWEEN 250 AND 500 THEN '$250 to 500'
		ELSE '$500+'
		END AS "price category"
FROM analytics_temp_table_2
GROUP BY "price category"
ORDER BY "units sold" DESC

Answer: Prices under $10



Question 4: 

SQL Queries:

Answer:



Question 5: 

SQL Queries:

Answer:

What issues will you address by cleaning the data?

ANALYTICS TABLE  DATA CLEANING: 

Column Elimination:
-------------------
-userid eliminated from the table because the entire column was NULL 

-socialEngagementType column eliminated from the table because it retuned only 1 Distinct category ‘Not Socially Engaged’, so all data was the same

New Table Creation:
-------------------
-multiple instances of revenue for a unique visit_id which is not logical therefore… 
-assume that the revenue column is not reliable and move this to it’s own separate table

-use units_sold * unit_price to calculate revenue in our queries 
-pull out units_sold and unit_price to a separate table with visit_id as a foreign key

-fix capitalization in column titles and cast to new data types where necessary

Filter Relevant Data:
---------------------
-only interested in rows with units_sold and/or revenue NOT NULL

Investigate Primary Key Duplication:
-----------------------------------
-the remaining columns from the analytics table are made into a separate table which should be normalized with visitId as primary key however, instances of duplication are present…

Fix Primary Key Duplication:
----------------------------
-visit_id and visit start time are exactly the same and this works most of the time but when two different visitors have the same start time, the visitid is duplicated!!!!

-This means that visit_id cannot be our primary key but we can make a new combined primary key using visit_start_time +full_visitor_id 
-call this visit_key

ALL_SESSIONS TABLE  DATA CLEANING: 

Fix Primary Key Duplication:
----------------------------
-make a new combined primary key using visitId +fullvisitorid to correspond with visit_key from analytics table

New Table Creation:
-------------------
-pull out city and country into seperate table with visit_key as primary key
-pull out productSKU and product_price into seperate table with visit_key 
-pull out product_category into seperate table with visit_key 

-fix capitalization in column titles and cast to new data types where necessary


Note: PRODUCTS TABLE
-can be used as is to retrieve product name by SKU due to name as there are instances of name 'variation' in the all_sessions data


Queries:
Below, provide the SQL queries you used to clean your data.

-----------------------Cleaning analytics Table----------------------------

SELECT *
FROM public.analytics
limit 1000
--highly repetitive data, check cases of NOT NULL data types to investigate 

SELECT * 
FROM public.analytics
WHERE NOT userid ISNULL
--returned Total rows: 0 therefore it could be eliminated from the table

SELECT DISTINCT("socialEngagementType")
FROM public.analytics
--retuned only 1 Distinct category ‘Not Socially Engaged’, since all data is the same, this can be removed

--create new table without the userid and socialEngagementType
--only interested in rows with units_sold and/or revenue NOT NULL
WITH analytics_temp_table_0 AS(
	SELECT 	"visitId"::varchar AS visit_id, "visitNumber" AS visit_number, "visitStartTime" AS visit_start_time, 
			date, "fullvisitorId" AS full_visitor_id, "channelGrouping" AS channel_grouping, units_sold, 
			pageviews AS page_views, timeonsite AS time_on_site, bounces, revenue, unit_price
	FROM public.analytics
	WHERE NOT units_sold ISNULL
	ORDER BY "visitId"
	)
--every column appears visually to be duplicated accross visit number other than units_sold, revenue and unit_price

--revenue is not in every row so let's separate reveue into it's own table with visit_id  
WITH analytics_temp_table_1 AS(
	SELECT "visitId" AS visit_id, revenue
	FROM public.analytics
	WHERE NOT revenue ISNULL
	GROUP BY "visitId", revenue
	)
--Total rows: 13412	
--We see multiple instances of revenue for a unique visit_id.
--This is not logical so we will assume that the revenue column is not reliable and eliminate this from the data 
--(Note: revenue could have been attached to date or visitorid and then accidentally applied across the data set.)
--We can use units_sold * unit_price to calculate revenue in our queries 

--we can now pull out units_sold and unit_price to a seperate table with visit_id as a foreign key 
WITH analytics_temp_table_2 AS(
	SELECT "visitId" AS visit_id, units_sold, unit_price
	FROM public.analytics
	WHERE NOT units_sold ISNULL
	GROUP BY "visitId", units_sold, unit_price
	)
--This should leave us with the following table where visit_id should be the primary key(without duplicates)
WITH analytics_temp_table_3 AS(
	SELECT 	DISTINCT("visitId")::varchar AS visit_id, 
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
--Total rows: 20369

--Let's check for duplicates by looking at the total rows with just distinct visitid values
SELECT DISTINCT(visit_id)
FROM analytics_temp_table_3
--Total rows: 20291 so there do appear to be 78 instances of primary key duplication

--Let's pull out these non-unique instances as follows:
WITH analytics_temp_table_3 AS(
	SELECT 	DISTINCT("visitId")::varchar AS visit_id, 
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
	),

	analytics_temp_table_3b AS(
	SELECT "visit_id", SUM(1) AS "count"
	FROM analytics_temp_table_3
	GROUP BY 1
	HAVING SUM(1) >1
	)

SELECT * 
FROM analytics_temp_table_3
WHERE visit_id IN (SELECT visit_id FROM analytics_temp_table_3b)
ORDER BY visit_id

--Aha! visit_id and visit start time are the same and this works most of the time but...
--when two visitors have the same start time, the visitid is duplicated!!!!
--This means that visit_id cannot be our primary key (and is relatively useless)
--visit_start_time +full_visitor_id combined can act as our new primary key



-----------------------3 Tables (CTEs) Created from Analytics----------------------------

--Let's remake our temp_table_3 with visit start time"+"fullvisitorId" as visit_key in a new column
WITH analytics_temp_table_3 AS(
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
--Total rows: 20369

--let's check for instances of duplication again with our new combined primary key 
SELECT DISTINCT(visit_key)
FROM analytics_temp_table_3
--Now Total rows: 20369 remains the same, so I beleive we have created a table in proper in 3rd normal form

--We must now remake the other tables that were created from analytics using our new combined primary key
WITH analytics_temp_table_1 AS(
	SELECT 	CONCAT("visitStartTime"::varchar, "fullvisitorId"::varchar) AS visit_key, 
			revenue
	FROM public.analytics
	WHERE NOT revenue ISNULL
	GROUP BY visit_key, revenue
	)
	
WITH analytics_temp_table_2 AS(
	SELECT 	CONCAT("visitStartTime"::varchar, "fullvisitorId"::varchar) AS visit_key, 
			units_sold, 
			unit_price
	FROM public.analytics
	WHERE NOT units_sold ISNULL
	GROUP BY visit_key, units_sold, unit_price
	)




-----------------------Cleaning all_sessions table----------------------------

SELECT * 
FROM all_sessions
ORDER BY "visitId"
LIMIT 100

--We should be able to join the data from analytics_temp_table_2 (unit_sold, unit_price) using a primary key
--Since we created a new combo key for the analytics table >>visit_key = ("visitStartTime"+"fullvisitorId")
--We need to create the corresponding foreign key in the analytics table >>visit_key = ("visitId"+"fullvisitorId")
--This is completed using the CONCAT command below and casting each column type to character vaying to allow concatenation 
--(Note: "visitId" and "visitStartTime" are identicle values in the analytics table and therefore interchangeable)

--For Questions 1-5 we are interested in city, country, product category and 'which products' or product SKU 
--(can retreive name later from products table as there are instances of name 'variation' in the all_sessions data)
WITH all_sessions_temp_table_0 AS(
	SELECT 	DISTINCT(CONCAT("visitId"::varchar, "fullVisitorId"::varchar)) AS visit_key,	
			country,
			city,
			"productSKU" AS product_sku,
			"v2ProductCategory" AS product_category,
			"productPrice" AS product_price
	FROM public.all_sessions
	)
	
--country and city should be unique based on each visit_key so we can pull these columns out as their own table
WITH all_sessions_temp_table_1 AS(
	SELECT 	DISTINCT(CONCAT("visitId"::varchar, "fullVisitorId"::varchar)) AS visit_key,	
			country,
			city
	FROM public.all_sessions
	)
--Total rows: 14561

--let's check to see if there is duplication of the primary key 
SELECT DISTINCT(visit_key)
FROM all_sessions_temp_table_1
--Total rows: 14561
--No duplicates so this table is in proper in 3rd normal form

--Now lets look at the remaining columns of interest (SKU, category and price)
WITH all_sessions_temp_table_2 AS(
	SELECT 	DISTINCT(CONCAT("visitId"::varchar, "fullVisitorId"::varchar)) AS visit_key,	
			"productSKU" AS product_sku,
			"v2ProductCategory" AS product_category,
			"productPrice" AS product_price
	FROM public.all_sessions
	)
--Total rows: 15131

--let's check to see if there is duplication of the primary key 
SELECT DISTINCT(visit_key)
FROM all_sessions_temp_table_2
--Total rows: 14561
--There appear to be 570 instances of primary key duplication
					--(Interestingly, the Total rows is the same as our temp_table_1 for city and country

					--Try adding product_category to 
					WITH all_sessions_temp_table_1 AS(
						SELECT 	DISTINCT(CONCAT("visitId"::varchar, "fullVisitorId"::varchar)) AS visit_key,	
								country,
								city,
								"v2ProductCategory" AS product_category
						FROM public.all_sessions
						)
					--Total rows: 14822

					--let's check to see if there is duplication of the primary key 
					SELECT DISTINCT(visit_key)
					FROM all_sessions_temp_table_1
					--Yes, there is so product category should not be joined to this table)

--Let's pull out these non-unique visit_key instances from the SKU,category,price table to investigate as follows:
WITH all_sessions_temp_table_2 AS(
	SELECT 	DISTINCT(CONCAT("visitId"::varchar, "fullVisitorId"::varchar)) AS visit_key,	
			"productSKU" AS product_sku,
			"v2ProductCategory" AS product_category,
			"productPrice" AS product_price
	FROM public.all_sessions
	),

	all_sessions_temp_table_2b AS(
	SELECT "visit_key", SUM(1) AS "count"
	FROM all_sessions_temp_table_2
	GROUP BY 1
	HAVING SUM(1) >1
	)

SELECT * 
FROM all_sessions_temp_table_2
WHERE visit_key IN (SELECT visit_key FROM all_sessions_temp_table_2b)
--ORDER BY visit_key, (No insight gained here. Try ORDER BY SKU...)
ORDER BY product_sku
--seeing a single product_sku in multiple product_categories

--let's try pulling product_category from the table and see where this gets us
WITH all_sessions_temp_table_2 AS(
	SELECT 	DISTINCT(CONCAT("visitId"::varchar, "fullVisitorId"::varchar)) AS visit_key,	
			"productSKU" AS product_sku,
			"productPrice" AS product_price
	FROM public.all_sessions
	)
--Total rows: 15129

--let's check to see if there is duplication of the primary key 
SELECT DISTINCT(visit_key)
FROM all_sessions_temp_table_2
--Total rows:14561 so still lots of duplication but I think this makes sense because
--customers will have ordered more than one product(sku) in the same visit

--let's put ProductCategory in it's own seperate table with only our key
WITH all_sessions_temp_table_3 AS(
	SELECT 	DISTINCT(CONCAT("visitId"::varchar, "fullVisitorId"::varchar)) AS visit_key,	
			"v2ProductCategory" AS product_category
	FROM public.all_sessions
	)

SELECT DISTINCT visit_key 
FROM all_sessions_temp_table_3

-----------------------3 Tables (CTEs) Created from all_sessions----------------------------
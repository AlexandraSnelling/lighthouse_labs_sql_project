# Final-Project-Transforming-and-Analyzing-Data-with-SQL
# Alexandra Snelling
# 2024-01-24


## Project/Goals
Goal 1. Import the raw data to tables in a database
Goal 2. Clean the data and create new tables from the raw data
Goal 3. Answer the given questions and create new questions to probe different areas of the data.
Goal 4. Create project files with readable verions of code, assumptions and explanations 
Goal 5. Create PPT and present project 
Goal 6. Submit project via Github repository


## Process
Step 1: -Create ecommerce database
	-Open .csv files to determine how to look at the tables 
	-note the numer of columns, names of each column and determine what data type to set each column to
	-create tables corresponding to the above
	-import .csv files into created tables

Step 2: -start by attempting to answer questions directly from only the all_sessions table

(## Challenge 1: Column names were not behaving as anticipated. I had to list with "column name" for almost all columns.
		 I realized that column names included capitalization. I later fixed this when creating my own tables.)

(## Challenge 2: I realized that there were only 4 instances where "transactionRevenue" was NOT NULL.
		 I decided to work with "totalTransactionRevenue" instead.
		 I answered some of the questions but the realized that...
		 There are multiple instances of "totalTransactionRevenue" for a single visitid. 
		 Also, the data I was getting seemed to be missing a lot of countries.
		 See example of question 1 answered using only the all_sessions table at the end of starting_with_questions.md)


Step 3: -Cleaning data
	-For description of data cleaning and table creation, please see:
	-Please see ANALYTICS TABLE  DATA CLEANING and ALL_SESSIONS TABLE  DATA CLEANING sections at the top of cleaning_dat.md 

(## Challenge 3: I realized that there were again, multiple instances of revenue for a unique visit_id in the analytics table.
		 This is not logical therefore I assume that the revenue column is not reliable and move this to a separate table.
		 I instead used the sum of units_sold * unit_price for each visitid to calculate revenue in our queries.)

(## Challenge 4: I realized that with visitId as primary key there were instances of multiple visitors being attached to the same key
		 visit_id and visit start time are exactly the same (almost always) and this works most of the time 
		 However, when two different visitors have the same start time, the visitid is duplicated!!!!
		 This means that visit_id cannot be our primary key so...
		 I decided to make a new combined primary key using visit_start_time +full_visitor_id called visit_key)


Step 4: -Answering and creating questions
	-Please see ## Results for description

Step 5: -Create new ecommerce2 database and set up tablesbased on created CTEs used to answer the questions
	-run CTE for each table in pgadmin, then download the resulting table as a .csv file
	-import those .csv files into my tables in ecommerce2
	-create ERD for ecommerce2, download image file

Step 6: -Build power point presentation 
	-Generate images using a combination of:
		1) table snips from pgAdmin
		2) copied graphic visualizer images from pgadmin 
		3) copying data to excel, graphing in excel and copying images to PPT 
	-Present using PPT in class

Step 7: -fill in required .md project files
	-push files and commit to my Github repository and submit repo link as project


## Results:

See starting_with_quetsions.md and starting_with_data.md for queries, query explanation and results

*Please note: questions were answered using the following CTEs rather than database as the database was created after*
---------------------------------------------------------------------------------------------------------------------
CTE Legend for new tables created as cleaned ecommerce database: 

analytics_temp_table_1 >>> (shown as ---- analytics_revenue ---- in ERD)
analytics_temp_table_2 >>> (shown as ---- analytics_units ---- in ERD)
analytics_temp_table_3 >>> (shown as ---- analytics_visit_key ---- in ERD)

all_sessions_temp_table_1 >>> (shown as ---- all_sessions_location ---- in ERD)
all_sessions_temp_table_2 >>> (shown as ---- all_sessions_product_info ---- in ERD)
all_sessions_temp_table_3 >>> (shown as ---- all_sessions_product_category ---- in ERD)

products >>> (shown as ---- products ---- in ERD)
---------------------------------------------------------------------------------------------------------------------
Please also note: questions were answered using joined data for the all_sessions and analytics tables 
		  example of question 1 answered using only the all_sessions table can be found at the end of starting_with_questions.md 



## Challenges 
Please see above for challenges faced through process steps
Also see QA.md file for overview of major risk area (aka.challenge)



## Future Goals
Next Step 1: -I would rewrite code to reflect nexly created ecommerce2 database, rather than using CTEs
	     -This would simplify the code and make it much easier to read

Next Step 2: -I would dive deeper into the all_sessions table to see if I could create more logical table relationships.

Next Step 3: -I would explore categories from question 3 further as I was unable to pull out any strong relationships yet.

Next Step 4: -I would try to contact the company to ensure that they fix their data collecion
	     -At a minimum, SKU should be included in the analytics table
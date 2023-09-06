What are your risk areas? Identify and describe them.

1. There are neither primary foreign or unique keys in the tables all_sessions and analytics. I had to join the two tables with a common non-unique column. The only way I could maintain a data dependency between the two tables was to use a non-unique column having the same description and datatype.

2. The visitid or fullvisitorid were supposed to serve as primary keys or at least unique identifiers (unique keys). But this was not the case. Probably, because the visitors visited a site due to interest and then visited another site to view another product of interest. Or the same visitor used different channels to gain access to the website.

3. There is a huge amount of repititions in the all_sessions table and analytics table. It would be difficult to control this potential issue because those tables contain generalized information collected in bulk.

4. For the business context, several columns in the analytics table is critical for operations and analytics. But most of those columns contains a huge amount of NULL values. I dropped them and deemed them unfit (missing values) for analytics until I saw questions that required using the few values present in those columns.


QA Process:
Describe your QA process and include the SQL queries used to execute it.

1. I started with data quality issues by checking for total counts of missing values in the all_sessions and analytics tables using the following codes:

```sql
SELECT
				COUNT(*) AS total_count,
				(SELECT 
								COUNT(*) AS missing_values_count
				FROM			all_sessions
				WHERE			channelgrouping IS NULL OR time IS NULL OR country IS NULL OR city IS NULL
								OR timeonsite IS NULL OR pageviews IS NULL OR date IS NULL
								OR visitid IS NULL OR productprice IS NULL OR productsku IS NULL
								OR productname IS NULL OR productcategory IS NULL OR currencycode IS NULL
								OR pagetitle IS NULL OR pagepathlevel IS NULL OR ecommerceaction_type IS NULL
								OR ecommerceaction_step IS NULL)
FROM			all_sessions;


SELECT
				COUNT(*) AS total_count,
				(SELECT 
								COUNT(*) AS missing_values_count
				FROM			analytics
				WHERE			visitnumber IS NULL OR visitid IS NULL OR date IS NULL OR fullvisitorid IS NULL
								OR channelgrouping IS NULL OR units_sold IS NULL OR pageviews IS NULL
								OR timeonsite IS NULL OR revenue IS NULL OR unit_price IS NULL)
FROM			analytics;
```

2. Continuing with data quality checks, I examined the duplicate records across all five tables using the following codes:

```sql

SELECT		
				COUNT(*) AS total_record,
				(SELECT
								COUNT(*)
				FROM			(SELECT 
												COUNT(*)
								FROM 			public.all_sessions
								GROUP BY		channelgrouping, time, country, city, timeonsite, pageviews,
												date, visitid, productprice, productsku, productname,
												productcategory, currencycode, pagetitle, pagepathlevel,
								 				ecommerceaction_type, ecommerceaction_step
									HAVING 		COUNT(*) > 1) AS duplicate_records
					) AS duplicates
FROM			all_sessions;


SELECT		
				COUNT(*) AS total_record,
				(SELECT
								COUNT(*)
				FROM			(SELECT 
												COUNT(*)
								FROM 			public.analytics
								GROUP BY		visitnumber, visitid, date, 
								 				fullvisitorid, channelgrouping,	units_sold, 
								 				pageviews, timeonsite, revenue, unit_price
									HAVING 		COUNT(*) > 1) AS duplicate_records
					) AS duplicates
FROM			analytics;


SELECT		
				COUNT(*) AS total_record,
				(SELECT
								COUNT(*)
				FROM			(SELECT 
												COUNT(*)
								FROM 			public.products
								GROUP BY		sku, name, orderedquantity, stocklevel, 
								 				restockingleadtime, sentimentscore,
								 				sentimentmagnitude
									HAVING 		COUNT(*) > 1) AS duplicate_records
					) AS duplicates
FROM			products;


SELECT		
				COUNT(*) AS total_record,
				(SELECT
								COUNT(*)
				FROM			(SELECT 
												COUNT(*)
								FROM 			public.sales_by_sku
								GROUP BY		productSKU, total_ordered
									HAVING 		COUNT(*) > 1) AS duplicate_records
					) AS duplicates
FROM			sales_by_sku;


SELECT		
				COUNT(*) AS total_record,
				(SELECT
								COUNT(*)
				FROM			(SELECT 
												COUNT(*)
								FROM 			public.sales_report
								GROUP BY		name, stocklevel, restockingleadtime, sentimentscore,
								 				sentimentmagnitude, ratio, productSKU, total_ordered
									HAVING 		COUNT(*) > 1) AS duplicate_records
					) AS duplicates
FROM			sales_report;
```

3. I grouped the number of visitors by the channel used to bring them to the website.

```sql
SELECT
				channelgrouping,
				COUNT(DISTINCT fullVisitorID) AS count_unique_visitors,
				COUNT(fullVisitorID) AS count_total_visitors
FROM			analytics
GROUP BY		channelgrouping;
```

4. I also carried out data profiling by checking for the total records in each table.

```sql
SELECT 
				'all_sessions' AS table_name, 
				COUNT(*) AS record_count 
FROM			all_sessions
UNION ALL
SELECT 
				'analytics' AS table_name, 
				COUNT(*) AS record_count 
FROM			analytics
UNION ALL
SELECT 
				'products' AS table_name, 
				COUNT(*) AS record_count 
FROM			products
UNION ALL
SELECT 
				'sales_by_sku' AS table_name, 
				COUNT(*) AS record_count 
FROM			sales_by_sku
UNION ALL
SELECT 
				'sales_report' AS table_name, 
				COUNT(*) AS record_count 
FROM			sales_report;
```

5. I checked for the data types of all the columns present to ensure they're in the right format.

```sql
SELECT 
			table_name, column_name, 
			data_type, is_nullable
FROM 		information_schema.columns
WHERE 		table_schema = 'public'
ORDER BY	table_name;
```

6. For data profiling, I observed that the totaltransactionrevenue column (under the all_sessions table), units_sold and revenue (under the analytics table) had a great number of missing values. I kept them only because they are vital to the business analytics of the data.

```sql
SELECT
				COUNT(*) AS total_counts,
				'total counts' AS comparison
FROM			all_sessions
UNION
SELECT
				COUNT(*),
				'revenue'
FROM			all_sessions
WHERE			totaltransactionrevenue IS NULL;

SELECT
				ROUND((COUNT(*) FILTER (WHERE totaltransactionrevenue IS NULL)) * 100.0 / 
					  COUNT(*)::INTEGER, 2) || '%' AS percentage_null
FROM			all_sessions;



SELECT
				COUNT(*) AS counts,
				'total count' AS comparison
FROM			analytics
UNION
SELECT
				COUNT(*),
				'units_sold'
FROM			analytics
WHERE			units_sold IS NULL
UNION
SELECT
				COUNT(*),
				'revenue'
FROM			analytics
WHERE			revenue IS NULL;


SELECT
				ROUND((COUNT(*) FILTER (WHERE revenue IS NULL)) * 100.0 / 
					  COUNT(*)::INTEGER, 2) || '%' AS percentage_null,
				'revenue' AS category
FROM			analytics
UNION
SELECT
				ROUND((COUNT(*) FILTER (WHERE units_sold IS NULL)) * 100.0 / 
					  COUNT(*)::INTEGER, 2) || '%' AS percentage_null,
				'units_sold' AS category
FROM			analytics;

```
The calculated percentages shows that 97% of the total counts are null values for all of these three important metrics, business wise.



7. I have done some part of the quality assurance process under the cleaning_data.md file. In particular, I will be borrowing a data quality check from the cleaning_data file because it seems more relevant here than there. There are only 206 unique visitors compared to a total of almost 4 million recorded visits. This is a lurking data quality issue. It also means that the query results from the analytics table would need to be carefully scrutinized.

```sql
SELECT
				COUNT(*) AS total_visits,
				(SELECT
								COUNT(DISTINCT visitnumber)
				FROM			analytics) AS unique_visits
FROM			analytics
```
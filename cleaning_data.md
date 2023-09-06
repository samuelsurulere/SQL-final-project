What issues will you address by cleaning the data?

For the ALL_SESSIONS table, the following issues were cleaned:

1. I checked for columns with all null values and observed that since the non-NULL values for these columns are less than 10% of the total count, I can ignore all of them. Their values are irrelevant to our analysis. However, I left the totaltransactionrevenue because it is important for financial analysis.

SQL Queries:

```sql
SELECT 
				COUNT(*) AS total_count,
				CASE
					WHEN COUNT(totaltransactionrevenue) = 0 THEN NULL 
					ELSE COUNT(totaltransactionrevenue)
				END AS totaltransactionrevenue,
				CASE
					WHEN COUNT(transactions) = 0 THEN NULL 
					ELSE COUNT(transactions)
				END AS transactions,
				CASE 
					WHEN COUNT(sessionqualitydim) = 0 THEN NULL
					ELSE COUNT(sessionqualitydim)
				END AS sessionqualitydim,
				CASE 
					WHEN COUNT(productrefundamount) = 0 THEN NULL
					ELSE COUNT(productrefundamount)
				END AS productrefundamount,
				CASE 
					WHEN COUNT(productquantity) = 0 THEN NULL
					ELSE COUNT(productquantity)
				END AS productquantity,
				CASE 
					WHEN COUNT(productrevenue) = 0 THEN NULL
					ELSE COUNT(productrevenue)
				END AS productrevenue,
				CASE 
					WHEN COUNT(itemquantity) = 0 THEN NULL
					ELSE COUNT(itemquantity)
				END AS itemquantity,
				CASE 
					WHEN COUNT(itemrevenue) = 0 THEN NULL
					ELSE COUNT(itemrevenue)
				END AS itemrevenue,
				CASE 
					WHEN COUNT(transactionrevenue) = 0 THEN NULL
					ELSE COUNT(transactionrevenue)
				END AS transactionrevenue,
				CASE 
					WHEN COUNT(transactionid) = 0 THEN NULL
					ELSE COUNT(transactionid)
				END AS transactionid,
				CASE 
					WHEN COUNT(searchkeyword) = 0 THEN NULL
					ELSE COUNT(searchkeyword)
				END AS searchkeyword,
				CASE 
					WHEN COUNT(ecommerceaction_option) = 0 THEN NULL
					ELSE COUNT(ecommerceaction_option)
				END AS ecommerceaction_option
FROM			all_sessions;


ALTER TABLE 		all_sessions
  DROP COLUMN 		transactions, 
  DROP COLUMN 		sessionqualitydim, 
  DROP COLUMN 		productrefundamount,
  DROP COLUMN 		productquantity,
  DROP COLUMN 		productrevenue,
  DROP COLUMN 		itemquantity,
  DROP COLUMN 		itemrevenue,
  DROP COLUMN 		transactionrevenue,
  DROP COLUMN 		transactionid,
  DROP COLUMN 		searchkeyword,
  DROP COLUMN 		ecommerceaction_option;

```

2. The data type of timeonsite was converted from integer to the proper time format measured in hours, minutes and seconds. It is not clear what the time column represents. I can only assume that it represents the time between when a visitor visits a site and thereafter makes a purchase after considering all possible options and constraints. It can also mean that a visitor opened a site and kept the page open for that long, until a final decision was reached. There is no background information about what kind of time interval it is describing. I could also deem it to be redundant information for the purpose of this task. But I kept it anyways, just in case it might later come off as helpful.

SQL Queries:

```sql
ALTER TABLE			all_sessions 
    ALTER COLUMN	timeonsite TYPE TIME
    	USING		TO_CHAR((timeonsite || ' second')::interval, 'HH24:MI:SS')::TIME;

		
ALTER TABLE			all_sessions 
    ALTER COLUMN	time TYPE TIME
    	USING		TO_CHAR(TO_TIMESTAMP(time), 'HH24:MI:SS')::TIME;

SELECT
				MIN(time),
				MAX(time)
FROM			all_sessions;
```


3. Instances where city IN ('not set', 'not available in the demo dataset', 'not available in demo dataset') were updated to country.

SQL Queries:

```sql
SELECT
				(SELECT
								COUNT(city)
				FROM			all_sessions
				WHERE			city = '(not set)'),
				(SELECT
								COUNT(city)
				FROM			all_sessions
				WHERE			city = 'not available in demo dataset'),
				COUNT(*)
FROM			all_sessions;


UPDATE			all_sessions 
    SET			city = CASE
							WHEN city = '(not set)' THEN country
							WHEN city = 'not available in demo dataset' THEN country
							ELSE city
						END;
```


4. All the rows where the currencycode was not 'USD' or was NULL was dropped because the level of uncertainties surrounding filling them up with 'USD' will likely influence the analysis negatively.

SQL Queries:

```sql
SELECT
				*
FROM			all_sessions
WHERE			currencycode IS NULL OR currencycode <> 'USD';

DELETE FROM		all_sessions
WHERE			currencycode IS NULL OR currencycode <> 'USD';
```

5. The productprice and totaltransactionrevenue columns were normalized by dividing it with 1000000

SQL Queries:

```sql
UPDATE			all_sessions 
    SET			productprice = productprice / 1000000;
	
UPDATE			all_sessions 
    SET			totaltransactionrevenue = totaltransactionrevenue::REAL / 1000000;
```

6. The type column was dropped because it is redundant.

SQL Queries:

```sql
SELECT
				COUNT(*)
FROM			all_sessions
WHERE			type <> 'PAGE';

ALTER TABLE 		all_sessions
  DROP COLUMN 		type; 
```

7. It seems likely that the visitid is more of a unique key compared to the fullvisitorid. This means the fullvisitorid is simply more of a repitition and we can drop it from the all_sessions table. Especially as we have the fullvisitorid also in the analytics table.

SQL Queries:

```sql
SELECT 
				COUNT(visitid) AS visit_id,
				COUNT(DISTINCT visitid) AS distinct_id,
				COUNT(fullvisitorid) AS visitor_id,
				COUNT(DISTINCT fullvisitorid) AS distinct_fullvis,
				COUNT(DISTINCT visitid) - COUNT(DISTINCT fullvisitorid) AS distinct_diff
FROM 			all_sessions;

ALTER TABLE 		all_sessions
  DROP COLUMN 		fullvisitorid; 
```

8. I also deleted the empty timeonsite rows because it does provide any related information to the purpose of the database. We were asked to examine data related to visitors browsing through a website before placing an order. When timeonsite is NULL, this means the visitor did not visit the site at all.

SQL Queries:

```sql
  
DELETE FROM		all_sessions
WHERE			timeonsite IS NULL;

```

9. I converted the pageviews from CHARACTER VARYING datatype to INTEGER datatype. This will enable me perform more operations and overcome the limitations of numerical analysis being done on VARCHAR data type.

SQL Queries:

```sql
ALTER TABLE			all_sessions 
    ALTER COLUMN	pageviews TYPE INT
		USING		pageviews::INTEGER;
```

10. The ecommerceaction_type and ecommerceaction_step has no meaning because there is no background information regarding their values. But I left them nonetheless.

SQL Queries:

```sql
SELECT			
				COUNT(*),
				COUNT(ecommerceaction_type),
				COUNT(ecommerceaction_step)
FROM			all_sessions
GROUP BY		ecommerceaction_type, ecommerceaction_step;
```

For the ANALYTICS table, the following issues were cleaned:

1. The visitstarttime column was dropped because the values are almost the same as visitid. And the percentage of the difference between them compared to the entire dataset is less than 1%. Which means their difference is highly negligible. I preferred the visitid to the visitstarttime because I can use the visitstarttime as a non-unique key to join the all_sessions and analytics tables.

SQL Queries:

```sql
SELECT 
				COUNT(visitid) AS visit_id,
				COUNT(DISTINCT visitid) AS distinct_id,
				COUNT(fullvisitorid) AS visitor_id,
				COUNT(DISTINCT fullvisitorid) AS distinct_fullvis,
				COUNT(visitstarttime) AS start_time,
				COUNT(DISTINCT visitstarttime) AS distinct_st,
				COUNT(DISTINCT visitstarttime) - COUNT(DISTINCT visitid) AS diff_between_id
FROM 			analytics;


SELECT
				((SELECT 
								COUNT(*)
				FROM 			analytics
				WHERE			visitid <> visitstarttime)::REAL / COUNT(*)::REAL) * 100||'%'		
FROM			analytics;


ALTER TABLE 		analytics
  DROP COLUMN 		visitstarttime; 
```

2. The userid column was dropped because all its values were NULL.

SQL Queries:

```sql
SELECT
				(SELECT 
								COUNT(*)
				FROM 			analytics
				WHERE			userid IS NULL),
				COUNT(*)
FROM			analytics;


ALTER TABLE 		analytics
  DROP COLUMN 		userid; 
```

3. The unit_price and revenue column were normalized by dividing it by 1000000.

SQL Queries:

```sql
UPDATE			analytics 
    SET			unit_price = unit_price / 1000000;

UPDATE			analytics 
    SET			revenue = revenue / 1000000;
```

4. I was checking for the need to keep or drop the units_sold and revenue column. Although, the non_NULL values are quite small compared to the entire dataset, I cannot deem them negligible because they are vital for business analytics.

SQL Queries:

```sql
SELECT
				(SELECT 
								COUNT(*)
				FROM			analytics
				WHERE			units_sold IS NULL) AS count_sales,
				(SELECT 
								COUNT(*)
				FROM			analytics
				WHERE			revenue IS NULL
				) AS income,
				COUNT(*) AS total_count
FROM			analytics;


SELECT 
				*
FROM			analytics
WHERE			units_sold IS NOT NULL;

SELECT 
				*
FROM			analytics
WHERE			revenue IS NOT NULL;
```

5. As was done under the all_sessions table, I deleted all the timeonsite rows that were NULL because they that implies the information related to those rows did not visit the website. Defeats the purpose of this analysis. All the products are on a website, and a user can only make a purchase by visiting the website. If there is no recorded timeonsite, then there is simply no revenue that will be generated.

SQL Queries:

```sql
SELECT
				(SELECT 
								COUNT(*) 
				FROM			analytics
				WHERE			timeonsite IS NULL),
				COUNT(*) AS total_count
FROM			analytics;
  
DELETE FROM		analytics
WHERE			timeonsite IS NULL;
```

6. The data type of timeonsite was converted from integer to the proper time format measured in hours, minutes and seconds.

SQL Queries:

```sql
ALTER TABLE			analytics 
    ALTER COLUMN	timeonsite TYPE TIME
    	USING		TO_CHAR((timeonsite || ' second')::interval, 'HH24:MI:SS')::TIME;
```

7. The values of bounces are all NULL. Hence, I also dropped it.

SQL Queries:

```sql
SELECT
				(SELECT 
								COUNT(*)
				FROM			analytics
				WHERE			bounces IS NULL),
				COUNT(*) AS total_count
FROM			analytics;

ALTER TABLE 		analytics
  DROP COLUMN 		bounces;
```

8. The column socialengagementtype is redundant because it contains the same string text all through the data set. And it has no connection to the channelgrouping because on the channelgrouping. The column was dropped from the database.

SQL Queries:

```sql
SELECT
				(SELECT
								COUNT(*)
				FROM			analytics
				WHERE			socialengagementtype = 'Not Socially Engaged'
				),
				(SELECT
								COUNT(*)
				FROM			analytics
				WHERE			socialengagementtype <> 'Not Socially Engaged'
				),
				COUNT(*)
FROM			analytics;


ALTER TABLE 		analytics
  DROP COLUMN 		socialengagementtype;
```

9. Here I was checking for the total number of unique visitors to website. Which is 206 unique visits compared to a total number of almost 4 million visits.

SQL Queries:

```sql
SELECT
				COUNT(*) AS total_visits,
				(SELECT
								COUNT(DISTINCT visitnumber)
				FROM			analytics) AS unique_visits
FROM			analytics
```


10. Here, I was checking to see that the total count respectively of the pageviews and visitnumber does not contain any NULL values.

SQL Queries:

```sql
SELECT
				COUNT(*)
FROM			analytics
GROUP BY		visitnumber
	HAVING		COUNT(*) IS NULL;


SELECT
				COUNT(*)
FROM			analytics
GROUP BY		pageviews
	HAVING		COUNT(*) IS NULL;
```

For the PRODUCTS, SALES_BY_SKU and SALES_REPORT tables, the following issues were cleaned:

1. I was checking for variances in the SKU and the possibility of assigning it as the primary key.

SQL Queries:

```sql
SELECT 	
				COUNT(*),
				COUNT(DISTINCT SKU) AS unique_count
FROM 			products;
```

2. I was checking for variances in the productSKU and the possibility of assigning it as the primary key.

SQL Queries:

```sql
SELECT 	
				COUNT(*),
				COUNT(DISTINCT productSKU) AS unique_count
FROM 			sales_by_sku;

```


3. I was checking for variances in the productSKU and the possibility of assigning it as the primary key. It was clear that there are different values for productSKU (synonymous with SKU) which was confusing for me initially. But I came to the conclusion that the products table is more like an inventory of all products the company has. Hence the reason why it has the highest number of counts. Then the productSKU in sales_by_sku contains the product IDs that were sold to customers while the productSKU in sales_report were those products that made it to the final report to be submitted to a superior or for record purposes. The discrepancy between the last two productSKU was probably due to human errors while collating the data from the internet or maybe a technical bug in the software that was automated for that purpose or some other reason that's unaccounted for.

SQL Queries:

```sql
SELECT 	
				COUNT(*),
				COUNT(DISTINCT productSKU) AS unique_count
FROM 			sales_report;

	
	
SELECT 	
				COUNT(DISTINCT sr.productSKU),
				COUNT(DISTINCT ss.productSKU),
				COUNT(DISTINCT p.SKU)
FROM 			sales_report sr
FULL JOIN		sales_by_sku ss
	USING		(productSKU)
FULL JOIN		products p
	ON			p.SKU = ss.productSKU;
```
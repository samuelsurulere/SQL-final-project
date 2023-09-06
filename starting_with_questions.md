Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:

```sql
SELECT 
				MAX(totaltransactionrevenue) AS total_revenue,
				city,
				country
FROM 			all_sessions
WHERE			totaltransactionrevenue IS NOT NULL
GROUP BY		city, country
ORDER BY 		total_revenue DESC;
```

Answer:
The city that has the highest level of transaction revenue in the United States was missing (NULL value), so I replaced it with the country during the data cleaning process. The highest level of transaction in the United States is 1015.48 USD.

Tel Aviv-Yafo in Isreal had the highest level of transaction revenue at 602 USD. In Australia, Sydney had the highest level of transaction revenue at 358 USD. In Canada, Toronto was the city with the highest level of transaction at 82.16 USD. Zurich was the only city that had any transaction in Switzerland. The highest level of transaction is 16.99 USD.



**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:

```sql
SELECT 
				ROUND(AVG(sr.total_ordered), 2) AS avg_products_ordered,
				als.city,
				als.country
FROM 			sales_report sr
LEFT JOIN		all_sessions als
	ON			als.productSKU = sr.productSKU
LEFT JOIN		analytics al
	ON			al.visitid = als.visitid
WHERE			al.units_sold IS NOT NULL AND sr.total_ordered > 0
GROUP BY		als.city, als.country
ORDER BY 		avg_products_ordered DESC;
```

Answer:

The answer is provided in the above code. There are 33 countries and cities with calculated averages. The highest average number of products ordered was 146, placed by a visitor from Houston, United States. The least average number of products ordered was 1 placed by a visitor from Japan.



**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:

```sql
(SELECT 
				als.productname,
				als.productcategory,
				MAX(sr.total_ordered) AS order_volume,
				als.city,
				als.country,
				als.productprice
FROM 			sales_report sr
LEFT JOIN		all_sessions als
	ON			als.productSKU = sr.productSKU
LEFT JOIN		analytics al
	ON			al.visitid = als.visitid
WHERE			al.units_sold IS NOT NULL AND sr.total_ordered > 0
GROUP BY		als.productcategory, als.city, als.country, als.productname, als.productprice
ORDER BY 		order_volume DESC
LIMIT			10)

UNION

(SELECT 
				als.productname,
				als.productcategory,
				MIN(sr.total_ordered) AS order_volume,
				als.city,
				als.country,
				als.productprice
FROM 			sales_report sr
LEFT JOIN		all_sessions als
	ON			als.productSKU = sr.productSKU
LEFT JOIN		analytics al
	ON			al.visitid = als.visitid
WHERE			al.units_sold IS NOT NULL AND sr.total_ordered > 0
GROUP BY		als.productcategory, als.city, als.country, als.productname, als.productprice
ORDER BY 		order_volume
LIMIT			7)

ORDER BY		order_volume DESC;



SELECT 
				MAX(sr.total_ordered) AS order_volume,
				als.productname,
				als.city,
				als.country,
				als.productprice
FROM 			sales_report sr
LEFT JOIN		all_sessions als
	ON			als.productSKU = sr.productSKU
LEFT JOIN		analytics al
	ON			al.visitid = als.visitid
WHERE			al.units_sold IS NOT NULL AND sr.total_ordered > 0
GROUP BY		als.city, als.country, als.productname, als.productprice
ORDER BY 		order_volume DESC
LIMIT			20;

SELECT
				productname,
				ROUND(AVG(productprice)::INTEGER, 2)
FROM			all_sessions
WHERE			productname = 'Nest® Cam Outdoor Security Camera - USA'
GROUP BY		productname;
```

Answer:
Yes, there seems to be a high number of the "Nest® Cam Outdoor Security Camera - USA" as a reasonable amount of visitors ordered for this product in the United States. This was the case when I filtered the query results to also include the number of units sold (i.e. WHERE units_sold from all_sessions table, IS NOT NULL). On removing this condition, there seems to be an enormous interest in the "Ballpoint LED Light Pen". Top visitors that bought this product were from India, United States, Spain, Japan, Germany, Netherlands, Malaysia, South Korea and Croatia. Looking further at the data set (through the productprice), the prices of the products is another reaosn for the huge difference between the volume of orders between the products. The ballpoint pen costs 2.50 USD while the security camera costs 174 USD on average.

The least ordered products category items are Andriod hats and Google vests (for men) and the visitors placing this order are from United States, India, Canada, Maldives and Japan. When I removed the WHERE filter (for units_sold NOT NULL), the least ordered items are same and the previous ones and also include the female versions (vests) and also vintage badges.




**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:

```sql
WITH RankedProducts AS (
    SELECT
					als1.city,
					als1.country,
					als1.productname,
					sr.total_ordered,
					ROW_NUMBER() OVER(PARTITION BY als1.city, als1.country ORDER BY sr.total_ordered DESC) AS rank
    FROM			all_sessions als1
    JOIN			sales_report sr 
		ON			als1.productSKU = sr.productSKU
)

SELECT
				rp.city,
				rp.country,
				rp.productname
FROM			RankedProducts rp
WHERE			rp.rank = 1;
```

Answer:

The only pattern I could observe was that the Ballpoint pen was the top selling product among the top selling products per country. This pattern seems to agree with previous query results.



**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:

```sql

SELECT 
				als.city,
				als.country,
				SUM(al.revenue) AS total_revenue
FROM			analytics al
JOIN			all_sessions als
	ON			al.visitid = als.visitid
WHERE			al.revenue IS NOT NULL
GROUP BY		als.city, als.country
ORDER BY		total_revenue DESC;

SELECT 
				als.city,
				als.country,
				SUM(al.revenue) OVER(PARTITION BY als.country ORDER BY als.city) AS total_revenue
FROM			analytics al
JOIN			all_sessions als
	ON			al.visitid = als.visitid
WHERE			al.revenue IS NOT NULL;
```

Answer:
There is a huge market turnover from the United States alone as the highest revenue comes from the country. The city with the highest market generated revenue is unknown. This will significantly influence the financial analysis. Sunnyvale is the second most profitable market in the United States. From this analysis, there should be more focus on the United States and more market budget can be allocated to the country. The highest proportions should go to Sunnyvale, then Seattle, San Francsico, Pato Alto, New York, Mountain View and last Chicago. Unfortunately, the state(s) bringing in the highest turnover from the United States is unknown. There is potential for growth in Swizerland, particularly, Zurich.







**Question 1: Since the market revenue from Sunnyvale is the second highest in the United States (the first being an unknown city), what are the top 5 products ordered by order volume?**

SQL Queries:

```sql
SELECT
				als.productname,
				SUM(sr.total_ordered) AS order_volume
FROM			all_sessions als
JOIN			sales_report sr 
	ON			als.productSKU = sr.productSKU
JOIN			analytics al 
	ON			al.visitid = als.visitid
WHERE			als.city = 'Sunnyvale'
GROUP BY		als.productname
ORDER BY		order_volume DESC
LIMIT 			5;

SELECT
				als.productname,
				SUM(sr.total_ordered) AS order_volume
FROM			all_sessions als
JOIN			sales_report sr 
	ON			als.productSKU = sr.productSKU
JOIN			analytics al 
	ON			al.visitid = als.visitid
WHERE			als.city = 'United States'
GROUP BY		als.productname
ORDER BY		order_volume DESC
LIMIT 			5;
```
Answer: 

"SPF-15 Slim & Slender Lip Balm" was the most ordered product (having an order volume of 15300). The ballpoint pen and security cameras also managed to take a place on the list of top five products ordered from visitors in the highest revenue generating city in the United States. This pattern agrees with previous query results and insights. Other items that made the list keyboard stickers and sports bottles. I also checked for the top five products ordered from the highest revenue generating city in the United States (which is unknown). There was a huge order volume for bottle coolers and bottle infusers. The ballpoint pen and security cameras also managed to make the top five products ordered.

Since the United States has the largest order volume among countries, it would make sense that the top products ordered by volume globally will also be the top products ordered in the United States.


**Question 2: What is the average order value for each city and country?**

SQL Queries:

```sql
SELECT
				city, country,
				ROUND(AVG(revenue_value), 2) AS average_revenue_value
FROM			(
				SELECT
							als.city, als.country, als.visitid,
							SUM(revenue) AS revenue_value
				FROM		analytics al
				JOIN		all_sessions als ON al.visitid = als.visitid
				WHERE		revenue IS NOT NULL
				GROUP BY	als.city, als.country, als.visitid
				) AS subquery
GROUP BY		city, country
ORDER BY		average_revenue_value DESC;
```
Answer:
The results of this query shows that New York in the United States has the highest average revenue with an average of 379 USD. Seattle follows closesly behind with 357 USD and then Sunnyvale comes in third with average revenue of 333 USD. In order to try to investigate this phenomenon, I calculated the entire sum of order volume from those three cities as follows

SQL Queries:

```sql
WITH order_volume_NY AS (
	SELECT
					als.productname,
					SUM(sr.total_ordered) AS order_volume
	FROM			all_sessions als
	JOIN			sales_report sr 
		ON			als.productSKU = sr.productSKU
	JOIN			analytics al 
		ON			al.visitid = als.visitid
	WHERE			als.city = 'New York'
	GROUP BY		als.productname
	ORDER BY		order_volume
),
			
order_volume_SV AS (
	SELECT
					als.productname,
					SUM(sr.total_ordered) AS order_volume
	FROM			all_sessions als
	JOIN			sales_report sr 
		ON			als.productSKU = sr.productSKU
	JOIN			analytics al 
		ON			al.visitid = als.visitid
	WHERE			als.city = 'Sunnyvale'
	GROUP BY		als.productname
	ORDER BY		order_volume
),

order_volume_ST AS (
	SELECT
					als.productname,
					SUM(sr.total_ordered) AS order_volume
	FROM			all_sessions als
	JOIN			sales_report sr 
		ON			als.productSKU = sr.productSKU
	JOIN			analytics al 
		ON			al.visitid = als.visitid
	WHERE			als.city = 'Seattle'
	GROUP BY		als.productname
	ORDER BY		order_volume
)

SELECT
				SUM(ny.order_volume) AS NY_order_vol,
				SUM(sv.order_volume) AS SV_order_vol,
				SUM(st.order_volume) AS ST_order_vol
FROM			order_volume_NY ny
FULL JOIN		order_volume_SV sv
	ON			ny.productname = sv.productname
FULL JOIN		order_volume_ST st
	ON			sv.productname = st.productname;

```
Interestingly, Sunnyvale has the highest order volume but it doesn't directly influence the revenue generated. Most likely that the most expensive products were being bought from New York city, then Seattle and lastly Sunnyvale. This also agrees with common observation, because the standard of living in New York is quite high. So visitors to the site from New York would be high income earners.

**Question 3: What is the total volume of orders per month in the country with the highest revenue? Analyse the results.**

SQL Queries:

```sql

SELECT 
                MIN(date), MAX(date) 
FROM            analytics;


SELECT
				order_month,
				productname,
				MAX(order_volume) OVER (PARTITION BY order_month) AS max_volume
FROM			(SELECT
								TO_CHAR(al.date, 'Month') AS order_month,
								als.productname,
								SUM(sr.total_ordered) AS order_volume
				FROM			all_sessions als
				JOIN			sales_report sr 
					ON			als.productSKU = sr.productSKU
				JOIN			analytics al 
					ON			al.visitid = als.visitid
				WHERE			als.country = 'United States'
				GROUP BY		als.productname, TO_CHAR(al.date, 'Month')
				ORDER BY		order_volume) AS subquery
WHERE			order_volume > 0
GROUP BY		order_month, productname, order_volume
ORDER BY		max_volume DESC;
```

Answer: The database was obtained over a period of four months (May, June, July and August) in 2017. The highest total orders volume was recorded in July (47094 orders), followed by June (40748 orders), next was May (25300) and the least volume of orders was recorded in August (1700 orders). The volume of orders was probably seasonal as well. July was the middle of summer and June was the beginning of summer. Hence, the reason why there were a lot of sales related to sport gears and outdoor wears and fashion items. The security camera was ordered in the month of June because since winter is over, everyone can freely go out. Meaning the rate of break-ins would be on the rise. August is the end of summer, hence the reason why there was a huge drop in orders during this month.



**Question 4: Find the visitors has spent the most and the products they spent on? Analyzse the results.**

SQL Queries:

```sql
SELECT
				al.visitid, als.city, als.country, als.productname,
				SUM(sr.total_ordered) AS total_quantity,
				ROUND(CAST(SUM(sr.total_ordered * al.unit_price) AS NUMERIC), 2) AS total_spending
FROM			analytics al
JOIN			all_sessions als
	ON			als.visitid = al.visitid
JOIN			sales_report sr
	ON			sr.productSKU = als.productsku
WHERE			revenue IS NOT NULL
GROUP BY		al.visitid, als.city, als.country, als.productname
ORDER BY		total_spending DESC
LIMIT 			10;
```

Answer: The visitors that spent the most were limited to the top ten total spenders. The top visitor by total spending spent 24276 USD, followed by another visitor that spent 21432 USD. The next visitor spent a total of 19557 USD. The quantity ordered by the three top spenders are 204, 188, and 1080 respectively. This means it is probably some small-scale companies or small-scale businesses are placing these huge orders. Going through the entire list, the least number of orders is 7, which means the top spenders are not individuals. And the product with the highest spending is the security camera. This also agrees with previous query results and tells us that security is probably a huge concern.



**Question 5: Which channel generated the highest revenue? Analyze the results**

SQL Queries:

```sql
WITH revenue_generated AS (
	SELECT
					channelgrouping,
					COUNT(DISTINCT fullVisitorID) AS count_unique_visitors,
					COUNT(fullVisitorID) AS count_total_visitors,
					SUM(revenue) AS total_revenue
	FROM			analytics
	GROUP BY		channelgrouping
	ORDER BY		total_revenue DESC
)

SELECT
				*
FROM			revenue_generated
WHERE			total_revenue IS NOT NULL;
```
Answer: Referrals generated the highest market revenue which is over 800 000 USD. Visitors directly locating the website based on their personal needs came in second with a high of over 180 000 USD. The sum of the revenues generated by the remaining channels don't sum up to the Direct channel. This analysis means that there should be more focus on the Referrals channels.

Answer the following questions and provide the SQL queries used to find the answer.

    
### Question 1
**Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:
```sql
SELECT
	country,
	city,
	SUM(totaltransactionrevenue) AS total_revenue
FROM all_sessions_clean
GROUP BY country, city
HAVING SUM(totaltransactionrevenue) IS NOT NULL AND city IS NOT NULL
ORDER BY total_revenue DESC
;
```
The above output can be shortened by uncommenting the ```AND SUM()``` statement in the ```HAVING``` clause, but is not necessary to answer this particular question.  

Answer: Top 5 Cities and countries:

1. San Francisco, United States 
2. Sunnyvale, United States
3. Atlanta, United States
4. Palo Alto, United States
5. Tel Aviv-Yafo, Israel

The (Country, City) pair with the highest total revenue was the United States but with the city listed as "not available in demo dataset" and was therefore excluded, as it may be spread across many different cities. 


### Question 2
**What is the average number of products ordered from visitors in each city and country?**


SQL Queries:
```sql
WITH average_by_id AS (SELECT 
	country,
	city,
	fullvisitorid,
	total_ordered,
	AVG(total_ordered) OVER (PARTITION BY city) AS avg_num_orders
FROM all_sessions_clean AS s
FULL OUTER JOIN sales_by_sku_raw AS sk
	ON s.product_sku = sk.productsku
WHERE city IS NOT NULL
	AND total_ordered > 0
GROUP BY country, city, fullvisitorid, total_ordered
ORDER BY country, city, fullvisitorid
)
SELECT
	country,
	city,
	avg_num_orders::INT
FROM average_by_id
GROUP BY country, city, avg_num_orders
ORDER BY country, city
```

Query to identify Brno Orders
```sql
SELECT
	country,
	city,
	product_name,
	total_ordered
FROM all_sessions_clean AS s
JOIN sales_by_sku_raw AS sk
	ON sk.productsku = s.product_sku
WHERE city = 'Brno';
```

Answer:

See table below. The top ten cities by average were included here as a sample. Note the relatively high average for the number of products ordered in Brno, Czechia. This is likely due a large order of a small, or cheap item, or a limited number of orders meaning that the average is sensitive to extreme values. In the case of Brno, it appears a single order was placed for 319 Journals, mean the average was simply divided by 1.  

| Country         | City             | Average Number of Orders |
|-----------------|------------------|---------------------------|
| Czechia         | Brno             | 319                       |
| Saudi Arabia    | Riyadh           | 319                       |
| United States   | Rexburg          | 251                       |
| Portugal        | Lisbon           | 189                       |
| United States   | Sacramento       | 189                       |
| Canada          | Sherbrooke       | 146                       |
| Russia          | Saint Petersburg | 135                       |
| Italy           | Rome             | 130                       |
| United Arab Emirates | Dubai       | 112                       |
| United States   | Kalamazoo        | 105                       |





### Question 3
**Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:
```sql
SELECT 
	country,
	city,
	product_category,
	total_ordered
FROM (
    SELECT 
        country,
        city,
        product_category,
        SUM(total_ordered) AS total_ordered
    FROM sales_by_sku_raw AS sbs
    JOIN all_sessions_clean AS s
        ON sbs.productsku = s.product_sku
    WHERE city IS NOT NULL 
		AND country IS NOT NULL
		AND product_category IS NOT NULL
		AND product_category NOT IN ('YouTube', 'Android', 'Google', 'Waze')
		AND total_ordered > 0
    GROUP BY country, city, product_category
) AS subquery
ORDER BY country, city, total_ordered DESC;
```
Answer: 

In the above query, only rows that have some quantity of orders are include. Some trends that can be seen pertain to the larger cities. Popular categories in these cities are often Office or Drinkware. One can imagine that a business hub like a large city may have high demand for office supplies. The drinkware may be explained by the larger concentration of restuarants in the city. As smaller cities have smaller total order counts, it is difficult to discern any patterns. 

### Question 4
**What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:
```sql
WITH orders_ranked AS (
		SELECT
			s.country,
			s.city,
			s.product_name,
			sk.total_ordered,
			RANK () OVER (PARTITION BY city ORDER BY sk.total_ordered DESC) AS quantity_rank
		FROM all_sessions_clean AS s
		JOIN sales_by_sku_raw AS sk
			ON s.product_sku = sk.productsku
		WHERE city IS NOT NULL AND country IS NOT NULL
		GROUP BY s.country, s.city, s.product_name, sk.total_ordered
		ORDER BY s.country, s.city, quantity_rank
	)
	
SELECT
	country,
	city,
	product_name,
	total_ordered,
	quantity_rank
FROM orders_ranked
WHERE quantity_rank = 1
ORDER BY country,city,quantity_rank;
```


Answer:





### Question 5
**Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:



Answer:








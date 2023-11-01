Answer the following questions and provide the SQL queries used to find the answer.

## Table of Contents

- [Question 1](#question-1)
- [Question 2](#question-2)
- [Question 3](#question-3)
- [Question 4](#question-4)
- [Question 5](#question-5)
    
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

Answer: Top 5 Cities and countries:

1. San Francisco, United States 
2. Sunnyvale, United States
3. Atlanta, United States
4. Palo Alto, United States
5. Tel Aviv-Yafo, Israel

NULL values were excluded in this calculation, however, it is important to note that the sum of transaction revenues with City listed as NULL was greater than any city that did have a value in the the city column. 


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
WITH units_sold AS (
SELECT DISTINCT fullvisitorid, SUM(units_sold) OVER (PARTITION BY fullvisitorid) AS sum_of_units
FROM analytics_clean
WHERE units_sold > 0
ORDER BY fullvisitorid
)
SELECT
	DISTINCT product_category,
	country,
	city,
	sum_of_units
FROM all_sessions_clean
JOIN units_sold
	USING (fullvisitorid)
WHERE 
	city IS NOT NULL
	AND product_category IS NOT NULL
	AND product_category NOT IN ('YouTube', 'Android', 'Google', 'Waze')
GROUP BY country, city, product_category, sum_of_units
ORDER BY country, city, sum_of_units DESC
```
Answer: 

In the above query, rows where there is a visitor ID tied to an order are included. Cities and Product Categories that are `NULL` are excluded. Also product categories that match a tech company name are excluded as they encompass a wide range of products. A pattern that can be seen is that the most popular item for most cities is often far more popular than any other item. This could imply singular large orders were executed, which would skew the averages. 

### Question 4
**What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:
```sql
WITH units_sold AS (
SELECT DISTINCT fullvisitorid, SUM(units_sold) OVER (PARTITION BY fullvisitorid) AS sum_of_units
FROM analytics_clean
WHERE units_sold > 0
ORDER BY fullvisitorid
)

SELECT * FROM (
SELECT
	country,
	city,
	product_name,
	sum_of_units,
	RANK() OVER (PARTITION BY city ORDER BY sum_of_units DESC) AS item_rank
FROM all_sessions_clean
JOIN units_sold
	USING (fullvisitorid)
WHERE 
	city IS NOT NULL
	AND product_category IS NOT NULL
	AND product_category NOT IN ('YouTube', 'Android', 'Google', 'Waze')
GROUP BY country, city, product_name, sum_of_units
ORDER BY country, city, sum_of_units DESC
) AS ranked_date
WHERE item_rank = 1
;
```


Answer:

Just by visually inspecting the 25 ranked cities, there are no apparent patterns. The products and their categories seem to cover a variety of items and categories respectively. There Are some items which may warrant a further investigation as their total order count appears to be an extreme outlier. A more detailed analysis may yield different results. 



### Question 5
**Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:
```sql
WITH city_rev AS (
SELECT
	country,
	city,
	SUM(revenue) AS city_revenue
FROM all_sessions_clean
JOIN analytics_clean
	USING (fullvisitorid)
WHERE 
	revenue > 0
	AND city IS NOT NULL
GROUP BY country, city
ORDER BY city_revenue DESC, country, city
)

SELECT
	country,
	city,
	ROUND((city_revenue/(SELECT SUM(city_revenue) FROM city_rev))*100, 2) AS prcnt_of_total_rev
FROM city_rev;
```
Calculating average contribution
```sql
WITH city_rev AS (
SELECT
	country,
	city,
	SUM(revenue) AS city_revenue
FROM all_sessions_clean
JOIN analytics_clean
	USING (fullvisitorid)
WHERE 
	revenue > 0
	AND city IS NOT NULL
GROUP BY country, city
ORDER BY city_revenue DESC, country, city
)

SELECT
	AVG(ROUND((city_revenue/(SELECT SUM(city_revenue) FROM city_rev))*100, 2))
FROM city_rev
```

Answer:

The 1st query lists the percent of the total revenue that each city is responsible for. The average contribution is 4% of the total revenue (see second query). From the first query, we see the maximum contribution is 31.65% and the minimum contribution is 0.06% of the total revenue. This set of percentages has a standard deviation of 6.7 and a variance of 45.5.

The table below shoes the top 5 of 25 cities based on their percent share of the total revenue. 
| Country         | City           | Revenue Impact |
|-----------------|----------------|----------------|
| United States   | Mountain View  | 31.65          |
| United States   | San Bruno      | 13.95          |
| United States   | New York       | 10.40          |
| United States   | Sunnyvale      | 9.98           |
| United States   | Kirkland       | 3.67           |

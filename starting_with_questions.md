Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


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


**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:
```sql
SELECT
	country,
	city,
	ROUND(AVG(totaltransactionrevenue/productprice), 0) AS average_quant
FROM all_sessions_clean 
WHERE totaltransactionrevenue IS NOT NULL AND productprice IS NOT NULL AND city IS NOT NULL
GROUP BY country, city
ORDER BY average_quant DESC
;
```
Answer:

See table below. Note the relatively high average for the number of products ordered in Sunnyvale, CA. This is likely due to how the "Number of Products Ordered" was calculated. There is a transaction record of $649.24, for a $1.40 per unit Lip Balm. This would imply that approximately 465 lip balms were ordered. This is reflected in the average which is sensitive to extreme values. 
| Country          | City           | Count |
|------------------|----------------|-------|
| United States    | Sunnyvale      | 117   |
| United States    | Atlanta        | 36    |
| United States    | Chicago        | 17    |
| Israel           | Tel Aviv-Yafo  | 8     |
| United States    | Austin         | 6     |
| United States    | Los Angeles    | 5     |
| United States    | San Bruno      | 5     |
| United States    | San Francisco  | 4     |
| United States    | New York       | 4     |
| Canada           | Toronto        | 4     |
| Australia        | Sydney         | 3     |
| United States    | Seattle        | 3     |
| United States    | Houston        | 2     |
| United States    | Palo Alto      | 2     |
| United States    | San Jose       | 2     |
| Switzerland      | Zurich         | 1     |
| United States    | Mountain View  | 1     |
| United States    | Nashville      | 1     |
| United States    | Columbus       | 1     |





**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:
```sql
SELECT
	country,
	city,
	product_category,
	COUNT(product_category) AS cat_count
FROM all_sessions_clean
GROUP BY country, city, product_category
HAVING 
	city IS NOT NULL 
	AND product_category IS NOT NULL 
	AND COUNT(product_category) > 1 
	AND product_category NOT IN ('YouTube', 'Android', 'Google', 'Waze')
ORDER BY cat_count DESC;
```
In the output of the above query, it is clear that Men's clothing, specifically Men's T-Shirts is a highly popular category. It is the most frequently ordered category in a single City.  The median for the number of different categories grouped by City and Country is 1. With that considered there are 136 (out of 744) cities with a Men's category ordered more than once. See query below:
```sql
SELECT
	country,
	city,
	product_category,
	COUNT(product_category) AS cat_count
FROM all_sessions_clean
GROUP BY country, city, product_category
HAVING 
	city IS NOT NULL 
	AND product_category IS NOT NULL
	AND COUNT(product_category) > 1
	AND product_category NOT IN ('YouTube', 'Android', 'Google', 'Waze')
	AND product_category LIKE '%Men%'
ORDER BY cat_count DESC;
```
There are 86 instances of cities having product orders for Men's goods in the 4th quartile of all orders. That is 86 out of 372 different numbers-of-categories ordered, or 23% of the 4th quartile is for orders regarding Men's goods. See query below. 
```spl
WITH quartiles_cte AS (
  SELECT
    country,
    city,
    product_category,
    COUNT(product_category) AS cat_count,
    NTILE(4) OVER (ORDER BY COUNT(product_category)) AS quartile
  FROM all_sessions_clean
  WHERE
    city IS NOT NULL 
    AND product_category IS NOT NULL
    AND product_category NOT IN ('YouTube', 'Android', 'Google', 'Waze')
  GROUP BY country, city, product_category
)
SELECT
  country,
  city,
  product_category,
  quartile
FROM quartiles_cte
WHERE product_category LIKE '%Men%' AND quartile = 4
ORDER BY quartile DESC;
```
Answer: 
In conclusion, there appears to be a pattern of high levels of orders for Men's goods/clothing amongst a wide range of cities. 





**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:



Answer:





**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:



Answer:








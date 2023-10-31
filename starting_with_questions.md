Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:
```sql
SELECT
	country,
	city,
	ROUND(SUM(totaltransactionrevenue)::NUMERIC/1000000, 2) AS total_revenue
FROM all_sessions_raw
GROUP BY country, city
HAVING SUM(totaltransactionrevenue) IS NOT NULL --AND SUM(totaltransactionrevenue) > 0
ORDER BY total_revenue DESC
```
The above output can be shortened by uncommenting the "AND SUM()" statement in the ```SQL HAVING``` clause, but is not necessary to answer this particular question.  

Answer: Top 5 Cities and countries:

1. San Francisco, United States 
2. Sunnyvale, United States
3. Atlanta, United States
4. Palo Alto, United States
5. Tel Aviv-Yafo, Israel

The (Country, City) pair with the highest total revenue was the United States but with the city listed as "not available in demo dataset" and was therefore excluded, as it may be spread across many different cities. 


**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:



Answer:





**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:



Answer:





**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:



Answer:





**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:



Answer:








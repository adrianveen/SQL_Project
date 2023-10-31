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
- United States"	"Sunnyvale"	117
- United States"	"Atlanta"	36
- United States"	"Chicago"	17
- Israel"		"Tel Aviv-Yafo"	8
- United States"	"Austin"	6
- United States"	"Los Angeles"	5
- United States"	"San Bruno"	5
- United States"	"San Francisco"	4
- United States"	"New York"	4
- Canada"		"Toronto"	4
- Australia"		"Sydney"	3
- United States"	"Seattle"	3
- United States"	"Houston"	2
- United States"	"Palo Alto"	2
- United States"	"San Jose"	2
- Switzerland"		"Zurich"	1
- United States"	"Mountain View"	1
- United States"	"Nashville"	1
- United States"	"Columbus"	1




**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:



Answer:





**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:



Answer:





**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:



Answer:








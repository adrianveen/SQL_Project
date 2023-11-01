## Question 1: What is the percentage of revenue that originated from each different channelgrouping?

SQL Queries:
```sql
WITH cg_rev AS (
	SELECT 
		channelgrouping,
		SUM(revenue) AS chan_sum
	FROM analytics_clean
	WHERE revenue > 0
	GROUP BY channelgrouping
)

SELECT
	channelgrouping,
	ROUND(chan_sum/(SELECT SUM(chan_sum) FROM cg_rev)*100,3) AS chan_impact
FROM cg_rev
GROUP BY channelgrouping, chan_sum
ORDER BY chan_sum DESC;
```

Answer: 
Channel grouping and their percent of the total revenue:
| Channel Grouping | Channel Impact(%) |
|------------------|----------------|
| Referral         | 72.023         |
| Direct           | 15.089         |
| Organic Search   | 9.646          |
| Paid Search      | 1.668          |
| Display          | 1.299          |
| Social           | 0.233          |
| Affiliates       | 0.042          |



## Question 2: Which product categories generate the most revenue?

SQL Queries:
```sql
SELECT
	product_category,
	SUM(revenue) AS cat_revenue
FROM all_sessions_clean
JOIN analytics_clean
	USING (fullvisitorid)
WHERE 
	revenue > 0
	AND product_category IS NOT NULL
GROUP BY product_category
ORDER BY cat_revenue DESC;
```

Answer:
Top 5 categories by total revenue:
| Product Category | Category Revenue |
|------------------|-------------------|
| Pet              | 17857.80         |
| Nest-USA         | 12044.58         |
| Women's          | 9645.57          |
| Fun              | 9600.93          |
| Headgear         | 9498.58          |



## Question 3: Which hours of the day result in the highest number of products sold by units sold? 

SQL Queries: 
Top 5 hours of the day by units sold
```sql
SELECT
	EXTRACT(hour FROM visitstarttime) AS hour_of_day,
	SUM(units_sold) AS units_per_hour
FROM analytics_clean
WHERE units_sold > 0
GROUP BY EXTRACT(hour FROM visitstarttime)
ORDER BY units_per_hour DESC
LIMIT 5
```
Answer:
| Hour of the Day | Total Units Sold |
|-----------------|-------------------|
| 15              | 19810            |
| 16              | 19210            |
| 13              | 17545            |
| 14              | 15462            |
| 18              | 13507            |







What are your risk areas? Identify and describe them.
Due to lack of time, this will cover the QA process in theory. The SQL queries are a sample of what is included in cleaning_data.md.

QA concers with the ecommerce data set:
- duplicate rows, particularly with values that are used as a unique identifier
  - product SKU
  - full visitor id
  - visitid
  - transaction id
- using the time of day in the visit id or the date
- the visit id contains the date of the visit, however, there is no guarantee that two visits will never happen at the same time.
- entries that should be NULL, have text values
  - this can have an adverse impact on various calculations  
- data types entering the database are inflate
  - eg. price columns needing to be divided by zero
  - if an uncorrected value is missed, it can greatly affect the analysis
- product sku's have varying length
  - inconsistency can make it more difficult to spot check
- large id lengths instead of using alphanumeric (ie. fullvisitorid)
  - can encounter data type issues if values continue to grow
- Product and category names entering data base with inconsistencies
  - can result in inaccurate analysis based on poor grouping/filtering     
## QA Process:
**Describe your QA process and include the SQL queries used to execute it.**

After applying permanent changes to the clean versions of the tables, a variety of relatively straightforward select queries were used to check that the updates were applied correctly. Some manual correction was applied if the initial change did not catch some issues. In particular, this happened with product names. 

Typically a count, or other central tendency value may be included to provide additional insight on during the check. 

Validating changes to cities
```sql
-- Validation
SELECT 
	city,
	COUNT(*)
FROM all_sessions_clean
GROUP BY city
;
```

Validating prices for 1e6 multipliers
```sql
SELECT productprice, totaltransactionrevenue
FROM all_sessions_clean
WHERE totaltransactionrevenue > 0;
```
```sql
-- Validation for all product name updates above
SELECT v2productname, COUNT(*) FROM all_sessions_clean GROUP BY v2productname ORDER BY v2productname;
```
Checking for negative values in units_sold
```sql
SELECT units_sold FROM analytics_clean GROUP BY units_sold ORDER BY units_sold;
```
Validating changes made to product names and SKUs
```sql
SELECT product_name, product_sku FROM products_clean GROUP BY product_name, product_sku ORDER BY product_name;
```

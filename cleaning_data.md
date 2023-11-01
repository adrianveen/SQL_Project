What issues will you address by cleaning the data?

See comments in SQL code blocks for more details. In general, the following items were addressed:
- extracting dates and times from integer or text values
- correctly formatting dollar amounts
- removing duplicate product or sku value, or unused sku values
- cleaning up the product name and category name to remove additional characters or prefixes/additional words
- dropping unused columns (ie. all NULL) or columns with insignificant values

Queries:

## Table of Contents
1. [Cleaning the Products table](#header-1)
2. [Cleaning the All Sessions Table](#header-2)
3. [Cleaning the Analytics table](#header-3)
4. [Cleaning the Sales by SKU table](#header-4)
5. [Cleaning the Sales Report table](#header-5)


## Cleaning the Products table
```sql
-- Products Table Clean up
SELECT * FROM products_clean;

-- Table setup
INSERT INTO products_clean
SELECT * FROM products_raw
;
-- Columns renamed in parent table
-- Update Sku Format
-- Drop sku's of different format
DELETE FROM products_clean
WHERE product_sku NOT LIKE 'GG%'
-- Validation
SELECT * FROM products_clean WHERE product_sku NOT LIKE 'GG%'

-- Update Product Names
UPDATE products_clean
SET
	product_name = TRIM(BOTH ' ' FROM REGEXP_REPLACE(product_name, '\s*(Google|Android|YouTube|You Tube|Waze)\s*','', 'g'));

-- Manual product_name consolidation or fixes
UPDATE products_clean
SET
	product_name = '16 oz. Hot and Cold Tumbler'
WHERE product_name = '16 oz. Hot/Cold Tumbler';

UPDATE products_clean
SET
	product_name = 'Baby Essentials Set'
WHERE product_name = 'Baby Essentials Baby Set';

UPDATE products_clean
SET
	product_name = 'Engraved Ceramic Mug'
WHERE product_name = 'Engraved CeramicMug';

UPDATE products_clean
SET
	product_name = 'Micro Wireless Earbuds'
WHERE product_name = 'Micro Wireless Earbud';

UPDATE products_clean
SET
	product_name = 'Dog Frisbee'
WHERE product_name LIKE '7%Dog Frisbee';

-- Validation of all product_name changes
SELECT product_name, product_sku FROM products_clean GROUP BY product_name, product_sku ORDER BY product_name;


-- Removing duplicates based on cleaned all_sessions table Skus
WITH extra_products_cte AS (
SELECT 
	DISTINCT p.product_sku,
	p.product_name
FROM products_clean AS p
FULL OUTER JOIN all_sessions_clean AS ses
	ON p.product_sku = ses.product_sku
WHERE ses.product_sku IS NULL
	AND ses.product_name IS NULL
ORDER BY p.product_name
)

DELETE FROM products_clean
WHERE product_sku IN (SELECT product_sku FROM extra_products_cte)

-- Validation - Only correct SKU's - down to 310 values
SELECT COUNT(DISTINCT product_sku) FROM products_clean
```

## Cleaning the All Sessions Table
```sql
-- All Sessions Clean-up
SELECT * FROM all_sessions_clean
-- Dropping empty columns or columns without enough values
ALTER TABLE all_sessions_clean
DROP COLUMN productrefundamount,
DROP COLUMN productrevenue,
DROP COLUMN itemquantity,
DROP COLUMN itemrevenue,
DROP COLUMN transactionrevenue,
DROP COLUMN searchkeyword
;

-- Validation
SELECT * FROM all_sessions_clean WHERE country LIKE '%(%)%'
;

-- Set Countries or City's as NULL if no value filled in

UPDATE all_sessions_clean
SET 
	city = NULL
WHERE 
	city = '(not set)'
	OR city = 'not available in demo dataset'
;

-- Validation
SELECT 
	city,
	COUNT(*)
FROM all_sessions_clean
GROUP BY city
;

-- Update countries to NULL it not filled in
UPDATE all_sessions_clean
SET
	country = NULL
WHERE country = '(not set)'
;

-- Validation
SELECT 
	country,
	COUNT(*)
FROM all_sessions_clean
GROUP BY country
;

-- Product Variant to NULL if (not set)
SELECT * FROM all_sessions_clean;

UPDATE all_sessions_clean
SET
	productvariant = NULL
WHERE productvariant = '(not set)'
;

-- Validation
SELECT
	productvariant,
	COUNT(*)
FROM all_sessions_clean
GROUP BY productvariant
;

SELECT * FROM all_sessions_clean

-- Formatting dollar values
-- Product Price
SELECT
	ROUND(productprice/1000000, 2)
FROM all_sessions_clean;

UPDATE all_sessions_clean
SET
	productprice = ROUND(productprice/1000000, 2)
;

-- Total Transaction Revenue
SELECT ROUND(totaltransactionrevenue/1000000,2)
FROM all_sessions_clean
WHERE totaltransactionrevenue > 0;

UPDATE all_sessions_clean
SET	
	totaltransactionrevenue = ROUND(totaltransactionrevenue/1000000,2)
;

-- Validation for both dollar amounts
SELECT productprice, totaltransactionrevenue
FROM all_sessions_clean
WHERE totaltransactionrevenue > 0;

-- Formatting Date
SELECT TO_DATE(date, 'YYYYMMDD') AS formatted_date
FROM all_sessions_clean
WHERE date_part('year', TO_DATE(date, 'YYYYMMDD')) > '2017'

UPDATE all_sessions_clean
SET
	date = TO_DATE(date, 'YYYYMMDD');
	
-- Validation for Date
SELECT
	date,
	COUNT(*)
FROM all_sessions_clean
GROUP BY date

-- Update Product Names
UPDATE all_sessions_clean
SET
	v2productname = REGEXP_REPLACE(v2productname, '(\s*(Google|Android|YouTube|You Tube|Waze)\s*)','', 'g');

-- Validation
SELECT v2productname, COUNT(*) FROM all_sessions_clean GROUP BY v2productname ORDER BY v2productname;

-- Manual product name consolidation or fixes for items that bypassed the above query
UPDATE all_sessions_clean
SET
	v2productname = '16 oz. Hot and Cold Tumbler'
WHERE v2productname = '16 oz. Hot/Cold Tumbler';

UPDATE all_sessions_clean
SET
	v2productname = 'Baby Essentials Set'
WHERE v2productname = 'Baby Essentials Baby Set';

UPDATE all_sessions_clean
SET
	v2productname = 'Engraved Ceramic Mug'
WHERE v2productname = 'Engraved CeramicMug';

UPDATE all_sessions_clean
SET
	v2productname = 'Micro Wireless Earbuds'
WHERE v2productname = 'Micro Wireless Earbud';

UPDATE all_sessions_clean
SET
	v2productname = 'Dog Frisbee'
WHERE v2productname LIKE '7%Dog Frisbee';

UPDATE all_sessions_clean
SET
	country = 'United States'
WHERE country = 'Canada' AND city = 'New York'

-- Validation for all product name updates above
SELECT v2productname, COUNT(*) FROM all_sessions_clean GROUP BY v2productname ORDER BY v2productname;

-- Drop rows with 'bad' product skus
DELETE FROM all_sessions_clean
WHERE productsku NOT LIKE 'GG%'

-- Validation - returns no rows
SELECT * 
FROM all_sessions_clean
WHERE productsku NOT LIKE 'GG%'

-- Cleaning Category Names
select v2productcategory, count(*) FROM all_sessions_clean GROUP BY v2productcategory ORDER BY v2productcategory
SELECT 
		REGEXP_REPLACE(v2productcategory, '^.*/([^/]+)/$', E'\\1') AS category
FROM all_sessions_clean

UPDATE all_sessions_clean
SET
	v2productcategory = REGEXP_REPLACE(v2productcategory, '^.*/([^/]+)/$', E'\\1')
	
UPDATE all_sessions_clean
SET
	v2productcategory = NULL
WHERE v2productcategory IN ('(not set)', '${escCatTitle}')

-- Manual Cleanup of Categories that were not caught above
UPDATE all_sessions_clean
SET
	v2productcategory = 'Drinkware'
WHERE v2productcategory LIKE 'Bottles%'

UPDATE all_sessions_clean
SET
	v2productcategory = 'Kid''s'
WHERE v2productcategory LIKE 'Kids'

UPDATE all_sessions_clean
SET
	v2productcategory = 'Men''s-T-Shirts'
WHERE v2productcategory LIKE 'Men''s T-Shirts'

UPDATE all_sessions_clean
SET
	v2productcategory = 'Lifestyle'
WHERE v2productcategory LIKE 'Lifestyle/'

-- Renaming Product Category and Product Name
ALTER TABLE all_sessions_clean
RENAME COLUMN v2productcategory TO product_category;

ALTER TABLE all_sessions_clean
RENAME COLUMN v2productname TO product_name;

ALTER TABLE all_sessions_clean
RENAME COLUMN productsku TO product_sku;
```

## Cleaning the Analytics table
```sql
-- Analytics Table Cleanup
SELECT * FROM analytics_raw;
SELECT * FROM analytics_clean;

-- Clean table creation
CREATE TABLE analytics_clean(
	visitnumber INTEGER,
	visitid INTEGER,
	visitstarttime INTEGER,
	date TEXT,
	fullvisitorid NUMERIC,
	channelgrouping VARCHAR(50),
	socialengagementtype VARCHAR(50),
	units_sold INTEGER,
	pageviews INTEGER,
	timeonsite INTEGER,
	bounces INTEGER,
	revenue NUMERIC,
	unit_price NUMERIC
);

INSERT INTO analytics_clean
SELECT *
FROM analytics_raw;

-- Compressed version of analytics_raw with unit price removed
SELECT * FROM analytics_clean
GROUP BY visitnumber,
	visitid,
	visitstarttime,
	date,
	fullvisitorid,
	channelgrouping,
	socialengagementtype,
	units_sold,
	pageviews,
	timeonsite,
	bounces,
	revenue
;

-- Start of cleaning process
-- Dropping unit price to further compress the table
ALTER TABLE analytics_clean
DROP COLUMN unit_price
;

SELECT * FROM analytics_clean;

CREATE TEMP TABLE analytics_temp (
	visitnumber INTEGER,
	visitid INTEGER,
	visitstarttime INTEGER,
	date TEXT,
	fullvisitorid NUMERIC,
	channelgrouping VARCHAR(50),
	socialengagementtype VARCHAR(50),
	units_sold INTEGER,
	pageviews INTEGER,
	timeonsite INTEGER,
	bounces INTEGER,
	revenue NUMERIC
);

-- Validate temp table
SELECT * FROM analytics_temp;

INSERT INTO analytics_temp
SELECT * FROM analytics_clean
GROUP BY visitnumber,
	visitid,
	visitstarttime,
	date,
	fullvisitorid,
	channelgrouping,
	socialengagementtype,
	units_sold,
	pageviews,
	timeonsite,
	bounces,
	revenue
;

-- Clear values from clean table then insert temp table as compressed table
-- DELETE FROM analytics_clean; (commented out to avoid deleting values from clean)

INSERT INTO analytics_clean
SELECT * FROM analytics_temp;

-- Validation
SELECT * FROM analytics_clean;

-- Update Visit starttime to time format
-- checking query on clean table
SELECT
	visitstarttime
FROM analytics_clean
ORDER BY visitstarttime DESC;

-- Updating to time zone data type and converting the values at the same time, then updating to the "time" data type by casting the time stamp as time
ALTER TABLE analytics_clean
ALTER COLUMN visitstarttime SET DATA TYPE TIMESTAMP WITH TIME ZONE
USING TO_TIMESTAMP(visitstarttime);

ALTER TABLE analytics_clean
ALTER COLUMN visitstarttime SET DATA TYPE TIME
USING visitstarttime::TIME;

-- Updating Date to be a date format
ALTER TABLE analytics_clean
ALTER COLUMN date SET DATA TYPE date
USING TO_DATE(date, 'YYYYMMDD');

-- Set -ve values in units sold to positive
UPDATE analytics_clean
SET
	units_sold = ABS(units_sold)
WHERE units_sold < 0;

-- Validation
SELECT units_sold FROM analytics_clean GROUP BY units_sold ORDER BY units_sold;

-- Change time on site to an interval in HH:MM:SS - no negative values to remove
ALTER TABLE analytics_clean
ALTER COLUMN timeonsite SET DATA TYPE interval
USING MAKE_INTERVAL(secs => timeonsite);

SELECT
	timeonsite
FROM analytics_clean
ORDER BY timeonsite;

-- Correct/Reformat Revenue values
UPDATE analytics_clean
SET
	revenue = ROUND(revenue/1000000, 2);
	
-- Validation
SELECT revenue FROM analytics_clean WHERE revenue IS NOT NULL LIMIT 100;

-- Can be used as primary key in visits table
SELECT DISTINCT visitid FROM analytics_clean

SELECT
	COUNT(*)
FROM analytics_clean
GROUP BY visitid
```

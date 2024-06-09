# Google Analytics Session data querying: Ecommerce Dataset
Execute SQL queries on a Google Analytics session dataset using BigQuery to fulfill analysis requests.
## Dataset
The dataset provides 12 months (August 2016 to August 2017) of obfuscated Google Analytics 360 data from the Google Merchandise Store , a real ecommerce store that sells Google-branded merchandise, in BigQuery.

Table ID: bigquery-public-data.google_analytics_sample.ga_sessions_20170801

Table Schema: https://support.google.com/analytics/answer/3437719?hl=en
## Case Study Questions:
### 1. Calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month)
Query #1
```c
SELECT
  FORMAT_DATE("%Y%m", PARSE_DATE("%Y%m%d", DATE)) AS month,
  SUM(totals.visits) AS visits,
  SUM(totals.pageviews) AS pageviews,
  SUM(totals.transactions) AS transactions,
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
WHERE _table_suffix BETWEEN '0101' AND '0331'
GROUP BY month
ORDER BY month;
```
*Result*:
|month|visits|pageviews|transactions|
|:----|:----:|:-------:|-----------:|
|201701|64694|257708|713|
|201702|62192|233373|733|
|201703|69931|259522|993|
---
### 2. Bounce rate per traffic source in July 2017 (Bounce_rate = num_bounce/total_visit) (order by total_visit DESC)
Query #2
```c
SELECT 
  trafficSource.source AS source,
  SUM(totals.visits) AS total_visits,
  SUM(totals.bounces) AS total_no_of_bounces,
  100.0*(SUM(totals.bounces)/SUM(totals.visits)) AS bouce_rate
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
GROUP BY trafficSource.source
ORDER BY total_visits DESC;
```
*Result*:
|source|total_visits|total_no_of_bounces|bouce_rate|
|:-----|:----------:|:-----------------:|---------:|
|google|38400|19798|51.557291666666671|
|(direct)|19891|8606|43.265798602382986|
|youtube.com|6351|4238|66.729648874193032|
|analytics.google.com|1972|1064|53.9553752535497|
|...|
***
### 3. Revenue by traffic source by week, by month in June 2017
Query #3
```c
WITH month AS
  (SELECT
      'Month' AS time_type,
      FORMAT_DATE("%Y%m", PARSE_DATE("%Y%m%d", DATE)) AS time,
      trafficSource.source AS source,
      SUM(p.productRevenue)/POWER(10,6) AS revenue
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
    UNNEST(hits) AS hits, 
    UNNEST(product) AS p
  GROUP BY time, source),
week AS
  (SELECT
    'Week' AS time_type,
    FORMAT_DATE("%Y%V", PARSE_DATE("%Y%m%d", DATE)) AS time,
    trafficSource.source AS source,
    SUM(p.productRevenue)/POWER(10,6) AS revenue
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
    UNNEST(hits) AS hits,
    UNNEST(product) AS p
  GROUP BY time, source)
SELECT *
FROM month
UNION ALL
SELECT *
FROM week
ORDER BY revenue DESC;
```
*Result*:
|time_type|time|source|revenue|
|:--------|:--:|:----:|------:|
|Month|201706|(direct)|97333.619695|
|Week|201724|(direct)|30908.909927|
|Week|201725|(direct)|27295.319924|
|Month|201706|google|18757.17992|
|...|
***
### 4. Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017.
Query #4
```c
WITH purchaser_data AS(
  SELECT
      format_date("%Y%m",parse_date("%Y%m%d",date)) AS month,
      (sum(totals.pageviews)/count(distinct fullvisitorid)) AS avg_pageviews_purchase,
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
    UNNEST(hits) hits,
    UNNEST(product) product
  WHERE _table_suffix BETWEEN '0601' AND '0731'
  AND totals.transactions>=1
  AND product.productRevenue IS NOT NULL
  GROUP BY month
),

non_purchaser_data AS(
  SELECT
      format_date("%Y%m",parse_date("%Y%m%d",date)) AS month,
      sum(totals.pageviews)/count(distinct fullvisitorid) AS avg_pageviews_non_purchase,
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
    UNNEST(hits) hits,
    UNNEST(product) product
  WHERE _table_suffix BETWEEN '0601' AND '0731'
  AND totals.transactions IS NULL
  AND product.productRevenue IS NULL
  GROUP BY month
)

SELECT
    pd.*,
    avg_pageviews_non_purchase
FROM purchaser_data pd
FULL JOIN non_purchaser_data using(month)
ORDER BY pd.month;
```
*Result*:
|month|avg_pageviews_purchase|avg_pageviews_non_purchase|
|:----|:--------------------:|-------------------------:|
|201706|94.02050113895217|316.86558846341671|
|201707|124.23755186721992|334.05655979568053|
***
### 5. Average number of transactions per user that made a purchase in July 2017
Query #5
```c
SELECT
  FORMAT_DATE("%Y%m", PARSE_DATE("%Y%m%d", DATE)) AS month,
  SUM(totals.transactions)/COUNT(DISTINCT(CASE WHEN totals.transactions >=1 THEN fullVisitorId ELSE NULL END)) Avg_total_transactions_per_user
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
UNNEST (hits) AS hits,
UNNEST (hits.product) AS product
WHERE productRevenue IS NOT NULL
GROUP BY month;
```
*Result*:
|month|Avg_total_transactions_per_user|
|:----|------------------------------:|
|201707|4.16390041493776|
***
### 6. Average amount of money spent per session. Only include purchaser data in July 2017
Query #6
```c
SELECT
    FORMAT_DATE("%Y%m",parse_date("%Y%m%d",date)) AS month,
    ((SUM(product.productRevenue)/SUM(totals.visits))/POWER(10,6)) AS avg_revenue_by_user_per_visit
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
  UNNEST(hits) AS hits,
  UNNEST(product) AS product
WHERE product.productRevenue IS NOT NULL
  AND product.productRevenue IS NOT NULL
GROUP BY month;
```
*Result*:
|month|avg_revenue_by_user_per_visit|
|:----|----------------------------:|
|201707|43.856598348051243|
***
### 7. Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017. Output should show product name and the quantity was ordered.
Query #7
```c
WITH id AS
  (SELECT
    DISTINCT fullVisitorId
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
    UNNEST (hits) AS hits,
    UNNEST (hits.product) AS product
  WHERE productRevenue IS NOT NULL
    AND v2ProductName IN("YouTube Men's Vintage Henley"))
SELECT
  v2ProductName AS other_purchased_products,
  SUM(productQuantity) AS quantity
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
  UNNEST (hits) AS hits,
  UNNEST (hits.product) AS product
INNER JOIN id
USING (fullVisitorId)
WHERE productRevenue IS NOT NULL
  AND v2ProductName NOT IN("YouTube Men's Vintage Henley")
GROUP BY v2ProductName
ORDER BY quantity DESC;
```
*Result*:
|other_purchased_products|quantity|
|:-----------------------|-------:|
|Google Sunglasses|20|
|Google Women's Vintage Hero Tee Black|7|
|SPF-15 Slim & Slender Lip Balm|6|
|Google Women's Short Sleeve Hero Tee Red Heather|4|
|...|
***
### 8. Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017. For example, 100% product view then 40% add_to_cart and 10% purchase.
Add_to_cart_rate = number product  add to cart/number product view. 
Purchase_rate = number product purchase/number product view. 

Note: hits.eCommerceAction.action_type = '2' is view product page; hits.eCommerceAction.action_type = '3' is add to cart; hits.eCommerceAction.action_type = '6' is purchase.
Query #8
```c
SELECT *,
  ROUND(100.0*num_addtocart/num_product_view,2) AS add_to_cart_rate,
  ROUND(100.0*num_purchase/num_product_view,2) AS purchase_rate
FROM
(SELECT
  FORMAT_DATE("%Y%m", PARSE_DATE("%Y%m%d", DATE)) AS month,
  COUNT( CASE 
    WHEN hits.eCommerceAction.action_type='2' THEN 1
    ELSE NULL END) AS num_product_view,
  COUNT( CASE 
    WHEN hits.eCommerceAction.action_type='3' THEN 1
    ELSE NULL END) AS num_addtocart,
  COUNT( CASE 
    WHEN hits.eCommerceAction.action_type='6'
      AND productRevenue IS NOT NULL THEN 1
    ELSE NULL END) AS num_purchase
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
  UNNEST (hits) AS hits,
  UNNEST (hits.product) AS product
WHERE _table_suffix BETWEEN '0101' AND '0331'
  AND eCommerceAction.action_type IN ('2','3','6')   
GROUP BY month) AS number
ORDER BY month;
```
*Result*:
|month|num_product_view|num_addtocart|num_purchase|add_to_cart_rate|purchase_rate|
|:----|:--------------:|:-----------:|:----------:|:--------------:|------------:|
|201701|25787|7342|2143|28.47|8.31|
|201702|21489|7360|2060|34.25|9.59|
|201703|23549|8782|2977|37.29|12.64|
***

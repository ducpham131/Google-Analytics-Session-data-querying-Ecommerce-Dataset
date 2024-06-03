# Google-Analytics-Session-data-querying-Ecommerce-Dataset
Execute SQL queries on a Google Analytics session dataset using BigQuery to fulfill analysis requests.
## Dataset
Table Schema: https://support.google.com/analytics/answer/3437719?hl=en
## Case Study Questions:
### 1: Calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month)
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
*Result*
|source|total_visits|total_no_of_bounces|bouce_rate|
|:-----|:----------:|:-----------------:|---------:|
|google|38400|19798|51.557291666666671|
|(direct)|19891|8606|43.265798602382986|
|youtube.com|6351|4238|66.729648874193032|
|analytics.google.com|1972|1064|53.9553752535497|
|...|

***

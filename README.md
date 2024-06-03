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
### 2: Bounce rate per traffic source in July 2017 (Bounce_rate = num_bounce/total_visit) (order by total_visit DESC)
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
### 3: Revenue by traffic source by week, by month in June 2017
Query #3
```c
WITH month AS
  (SELECT
      'Month' AS time_type,
      FORMAT_DATE("%Y%m", PARSE_DATE("%Y%m%d", DATE)) AS time,
      trafficSource.source AS source,
      SUM(p.productRevenue)/1000000 AS revenue
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
  unnest(hits) hits, 
  unnest(product) p
  GROUP BY time, source),
week AS
  (SELECT
    'Week' AS time_type,
    FORMAT_DATE("%Y%V", PARSE_DATE("%Y%m%d", DATE)) AS time,
    trafficSource.source AS source,
    SUM(p.productRevenue)/1000000 AS revenue
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
  unnest(hits) hits,
  unnest(product) p
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



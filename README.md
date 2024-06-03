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
|month|visits|pageviews|transactions|
|:----|:----:|:-------:|-----------:|
|201701|64694|257708|713|
|201702|62192|233373|733|
|201703|69931|259522|993|

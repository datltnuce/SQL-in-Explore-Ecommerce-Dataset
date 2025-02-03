# SQL Explore Ecommerce Dataset

## [SQL] Explore Ecommerce Dataset

## I. Introduction
This project uses BigQuery to analyze Ecommerce data, focusing on user behavior and sales patterns. It examines visits, transactions, and revenue by time and traffic sources, evaluates bounce rates and pageviews by customer types, and calculates average spending per session. Additionally, it identifies related product purchases and maps the customer journey from product views to purchases, offering insights to improve business strategies.

## II. Recommendations:
- This dataset provides crucial insights into user behavior, engagement, and revenue trends over the studied period. Optimizing landing pages and enhancing the user experience can help reduce high bounce rates from sources like Google and direct traffic. Additionally, leveraging cross-selling opportunities can help increase the average revenue per customer.
- Furthermore, analyzing traffic sources with high bounce rates, such as YouTube or less popular search engines, helps identify the quality of traffic and improve customer acquisition strategies. To gain a deeper understanding of customer behavior, businesses should also integrate historical data and demographic information, which will help refine marketing strategies and optimize advertising campaigns.
- In summary, combining user behavior analysis with appropriate marketing strategies will improve sales effectiveness and drive revenue growth.

## III. Focus Data
1. Analyze total visits, transactions, and revenue by month, week, and traffic source for specific periods.
2. Evaluate bounce rates, pageviews, and average transactions based on customer types or traffic sources.
3. Calculate average spending per session and identify related products purchased by customers who bought a specific product.
4. Build a cohort map tracking product views to cart additions and purchases.

## IV. Data Acess
The Ecommerce Dataset is available in the public Google BigQuery dataset named **"ga_session"**. To access it, follow these steps:
1. Log in to your Google Cloud Platform account and create a new project.
2. Open the BigQuery console and choose the project you just created.
3. In the navigation panel, click on **"Add Data"**, then select **"Search a project"**.
4. Enter the project ID **"bigquery-public-data.google_analytics_sample.ga_sessions"** 
5. Click on the **"ga_sessions_"** table to open it.

Table Schema: https://support.google.com/analytics/answer/3437719?hl=en

## V. Exploring the Dataset
Query 01: Calculate total visit, pageview, transaction for Jan, Feb and March 2017
```sql
SELECT  
  FORMAT_DATE('%Y-%m', DATE(PARSE_DATE('%Y%m%d', date))) AS month
  ,SUM(totals.visits) AS visit
  ,SUM(totals.pageviews) AS pageviews
  ,SUM(totals.transactions) AS transactions
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*` 
WHERE _table_suffix BETWEEN '0101' AND '0331'
GROUP BY month
ORDER BY month;
```
| month  | visits | pageviews | transactions |
|--------|--------|-----------|--------------|
| 2017-01 | 64694  | 257708    | 713          |
| 2017-02 | 62192  | 233373    | 733          |
| 2017-03 | 69931  | 259522    | 993          |

Analysis: In the first three months of 2017, visits fluctuated, decreasing in February but increasing sharply in March. Pageviews dropped in February but recovered and exceeded January's level in March. Transactions grew steadily, especially in March, with a significant increase compared to the previous months.
### Query 2: Bounce rate per traffic source in July 2017
```sql
SELECT  
  trafficSource.source,
  SUM(totals.visits) AS total_visit,
  SUM(totals.bounces) AS total_no_of_bounces,
  ROUND(100.00 * SUM(totals.bounces) / NULLIF(SUM(totals.visits), 0), 3) AS bounce_rate
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*` 
WHERE _TABLE_SUFFIX BETWEEN '0701' AND '0731'
  AND FORMAT_DATE('%Y-%m', DATE(PARSE_DATE('%Y%m%d', date))) = '2017-07'
GROUP BY trafficSource.source
ORDER BY total_visit DESC;
```
| Row | source                | total_visits | total_no_of_bounces | bounce_rate |
|-----|-----------------------|--------------|---------------------|-------------|
| 1   | google                | 38400        | 19798               | 51.557      |
| 2   | (direct)              | 19891        | 8606                | 43.266      |
| 3   | youtube.com           | 6351         | 4238                | 66.73      |
| 4   | analytics.google.com  | 1972         | 1064                | 53.955      |
| 5   | Partners              | 1788         | 936                 | 52.349      |
| 6   | m.facebook.com        | 669          | 430                 | 64.275      |
| 7   | google.com            | 368          | 183                 | 49.728      |
| 8   | dfa                   | 302          | 124                 | 41.06      |
| 9   | sites.google.com      | 230          | 97                  | 42.174      |
| 10  | facebook.com          | 191          | 102                 | 53.403      |
.........

Analysis: 

- **Google** and **Direct** are the major traffic sources with high bounce rates, so it's important to consider optimizing the content to reduce bounce rates from these sources.
- **YouTube** and other sources like **m.facebook.com** and **reddit.com** have high bounce rates, indicating a need to improve landing pages to keep users engaged longer.
- **Less frequent traffic sources** like **search.mysearch.com** and **duckduckgo.com** have extremely high bounce rates, suggesting that these might be low-quality visits, which require further investigation to improve user acquisition strategies.
### Query 3: Revenue by traffic source by week, by month in June 2017
```sql
SELECT 'Month' as time_type
  ,FORMAT_DATE('%Y-%m', DATE(PARSE_DATE('%Y%m%d', date))) AS time
  ,trafficSource.source AS source
  ,ROUND(SUM(product.productRevenue) / 1000000, 4) AS revenue
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*` ,
 UNNEST(hits) AS hits
,UNNEST(hits.product) AS product
WHERE _TABLE_SUFFIX BETWEEN '0601' AND '0630'
  AND FORMAT_DATE('%Y-%m', DATE(PARSE_DATE('%Y%m%d', date))) = '2017-06'
  AND product.productRevenue IS NOT NULL
GROUP BY time_type, time, source

UNION ALL 

SELECT 'Week' as time_type
  ,FORMAT_DATE('%Y-W%V', DATE(PARSE_DATE('%Y%m%d', date))) AS time
  ,trafficSource.source AS source
  ,ROUND(SUM(product.productRevenue) / 1000000, 4) AS revenue
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*` ,
 UNNEST(hits) AS hits
,UNNEST(hits.product) AS product
WHERE _TABLE_SUFFIX BETWEEN '0601' AND '0630'
  AND FORMAT_DATE('%Y-%m', DATE(PARSE_DATE('%Y%m%d', date))) = '2017-06'
  AND product.productRevenue IS NOT NULL
GROUP BY time_type, time, source
ORDER BY revenue DESC;
```
| Row | time_type | time   | source   | revenue    |
|-----|-----------|--------|----------|------------|
| 1   | Month     | 2017-06 | (direct) | 97333.6197 |
| 2   | Week      | 2017-W24 | (direct) | 30908.9099 |
| 3   | Week      | 2017-W25 | (direct) | 27295.3199 |
| 4   | Month     | 2017-06 | google   | 18757.1799 |
| 5   | Week      | 2017-W23 | (direct) | 17325.6799 |
| 6   | Week      | 2017-W26 | (direct) | 14914.81 |
| 7   | Week      | 2017-W24 | google   | 9217.17  |
| 8   | Month     | 2017-06 | dfa      | 8862.23  |
| 9   | Week      | 2017-W22 | (direct) | 6888.9  |
| 10  | Week      | 2017-W26 | google   | 5330.57  |
.........

Analysis: The table contains revenue data categorized by time type, time period, source, and revenue amount. Each row represents a specific time frame (week or month) and details the revenue generated from various sources. The author highlights key insights from the data, such as revenue from direct website visits, organic search, and referrals from sources like Google, DFA, mail.google.com, and search engines like myway.com. The revenue is tracked weekly and monthly, with data spanning several months.
### Query 4: Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017
```sql
WITH purchase AS
              (SELECT FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d', date)) AS month,
                            SUM(totals.pageviews)/COUNT(DISTINCT fullVisitorId) AS avg_pageviews_purchase
              FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`, 
              UNNEST (hits) AS hits, 
              UNNEST (hits.product) AS product 
              WHERE _table_suffix BETWEEN '0601' AND '0731' 
                      AND totals.transactions >=1 
                      AND product.productRevenue IS NOT NULL
              GROUP BY month),
    nonpurchase AS
              (SELECT FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d', date)) AS month,
                            SUM(totals.pageviews)/COUNT(DISTINCT fullVisitorId) AS avg_pageviews_non_purchase
              FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`, 
              UNNEST (hits) AS hits, 
              UNNEST (hits.product) AS product 
              WHERE _table_suffix BETWEEN '0601' AND '0731' 
                    AND totals.transactions IS NULL
                    AND product.productRevenue IS NULL
              GROUP BY month)

SELECT purchase.month,
        avg_pageviews_purchase,
        avg_pageviews_non_purchase
FROM purchase
FULL JOIN nonpurchase
USING (month)
ORDER BY purchase.month;
```
| Row | month  | avg_pageviews_purchase  | avg_pageviews_non_purchase  |
|-----|--------|-------------------------|-----------------------------|
| 1   | 2017-06 | 94.02050113895217       | 316.86558846341671           |
| 2   | 2017-07 | 124.23755186721992       | 334.05655979568053           |

Analysis: The increase in pageviews indicates that users tended to engage more in July compared to June, especially the group of buyers. However, there is still a noticeable gap between buyers and non-buyers, with non-buyers maintaining a high number of pageviews without converting into transactions.
### Query 5: Average number of transactions per user that made a purchase in July 2017
```sql
SELECT
    FORMAT_DATE("%Y%m",PARSE_DATE("%Y%m%d",date)) AS month,
    SUM(totals.transactions)/COUNT(DISTINCT fullvisitorid) as Avg_total_transactions_per_user
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
    ,UNNEST(hits) AS hits
    ,UNNEST(product) AS product
WHERE totals.transactions >= 1
AND product.productRevenue IS NOT NULL
GROUP BY month;
```

| Month  | Avg_total_transactions_per_user |
|--------|---------------------------------|
| 201707 | 4.16390041493776                |

Analysis: The table presents the average total transactions per user for July 2017. According to the data, the average user made approximately 4.16 transactions during that month. This information can help analyze user behavior, assess engagement levels on your platform, and evaluate the impact of marketing campaigns or promotions conducted in July.
### Query 6: Average amount of money spent per session. Only include purchaser data in July 2017
```sql
SELECT
    FORMAT_DATE("%Y%m",PARSE_DATE("%Y%m%d",date)) AS month,
    ((SUM(product.productRevenue)/SUM(totals.visits))/power(10,6)) AS avg_revenue_by_user_per_visit
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
  ,UNNEST(hits) AS hits
  ,UNNEST(product) AS product
WHERE product.productRevenue IS NOT NULL
AND totals.transactions >= 1
GROUP BY month;
```

| Month  | avg_revenue_by_user_per_visit |
|--------|-------------------------------|
| 201707 | 43.856598348051243            |

Analysis: The average total transactions per user in July 2017 is 43.86, indicating that each user made around 44 transactions on average during that month. This metric is valuable for businesses to assess user engagement and activity levels.
### Query 7: Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017
```sql
WITH buyer_list AS(
    SELECT DISTINCT fullVisitorId  
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
      ,UNNEST(hits) AS hits
      ,UNNEST(hits.product) as product
    WHERE product.v2ProductName = "YouTube Men's Vintage Henley"
    AND totals.transactions>=1
    AND product.productRevenue IS NOT NULL
)

SELECT
  product.v2ProductName AS other_purchased_products,
  SUM(product.productQuantity) AS quantity
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
  ,UNNEST(hits) AS hits
  ,UNNEST(hits.product) AS product
JOIN buyer_list using(fullVisitorId)
WHERE product.v2ProductName != "YouTube Men's Vintage Henley"
AND product.productRevenue IS NOT NULL
GROUP BY other_purchased_products
ORDER BY quantity DESC;
```

| Row | other_purchased_products                | quantity |
|-----|-----------------------------------------|----------|
| 1   | Google Sunglasses                       | 20       |
| 2   | Google Women's Vintage Hero ...         | 7        |
| 3   | SPF-15 Slim & Slender Lip Balm          | 6        |
| 4   | Google Women's Short Sleeve ...         | 4        |
| 5   | YouTube Men's Fleece Hoodie ...         | 3        |
| 6   | Google Men's Short Sleeve Bad...        | 3        |
| 7   | Recycled Mouse Pad                      | 2        |
| 8   | Red Shine 15 oz Mug                     | 2        |
| 9   | Google Doodle Decal                     | 2        |
| 10  | YouTube Twill Cap                       | 2        |
.......

Analysis: In summary, the data offers valuable insights into customer preferences, product popularity, and potential opportunities for marketing and merchandising strategies. A deeper analysis of historical data, along with integration of customer demographics, could offer a more thorough understanding of these trends.
### Query 8: Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017
```sql
WITH viewnum AS        
WITH product_data AS(
SELECT
    FORMAT_DATE("%Y%m",PARSE_DATE("%Y%m%d",date)) AS month,
    COUNT(CASE WHEN eCommerceAction.action_type = '2' THEN product.v2ProductName END) AS num_product_view,
    COUNT(CASE WHEN eCommerceAction.action_type = '3' THEN product.v2ProductName END) AS num_add_to_cart,
    COUNT(CASE WHEN eCommerceAction.action_type = '6' AND product.productRevenue IS NOT NULL THEN product.v2ProductName END) AS num_purchase
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
  ,UNNEST(hits) AS hits
  ,UNNEST(hits.product) AS product
WHERE _table_suffix BETWEEN '20170101' AND '20170331'
AND eCommerceAction.action_type IN ('2','3','6')
GROUP BY month
ORDER BY month
)

SELECT *,
    ROUND(num_add_to_cart/num_product_view * 100, 2) AS add_to_cart_rate,
    ROUND(num_purchase/num_product_view * 100, 2) AS purchase_rate
FROM product_data;
```

| Row | month  | num_product_view | num_addtocart | num_purchase | add_to_cart_rate | purchase_rate |
|-----|--------|------------------|---------------|--------------|------------------|---------------|
| 1   | 201701 | 25787            | 7342          | 2143         | 28.47            | 8.31          |
| 2   | 201702 | 21489            | 7360          | 2060         | 34.25            | 9.59          |
| 3   | 201703 | 23549            | 8782          | 2977         | 37.29            | 12.64         |

Analysis: 

- The table displays five different metrics and rates related to user behavior from January to March 2017.
- In general, there was a steady increase in product views from January to March 2017. Along with this, both the add-to-cart and purchase rates also saw an upward trend, reflecting better user engagement and conversion rates.
- Notably, the add-to-cart and purchase rates were significantly higher in March 2017, which could indicate improvements in the website's user experience or the effectiveness of marketing strategies during that period.

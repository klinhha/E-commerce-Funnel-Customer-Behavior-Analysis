# E-commerce Analytics Project (Google Analytics - BigQuery)
## I. Overview
 This project analyzes an e-commerce dataset using Google Analytics sample data on BigQuery. 
 
 The goal is to understand user behavior, conversion performance, and revenue patterns across different traffic sources and devices.

## II. Objectives
- Analyze website traffic (visits, pageviews, transactions)
- Evaluate marketing performance by traffic source
- Measure conversion rate and user behavior
- Understand purchase patterns and product relationships
- Build funnel and cohort analysis

## III. Tools & Technologies
- SQL (Google BigQuery)
- Google Analytics Sample Dataset

## IV. Dataset
 Source: ga_sessions_@BigQueryDateShardedTable

## V. Key Analyses & Results
### 1. Calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month)
``` sql
SELECT format_date('%Y%m', parse_date('%Y%m%d', date)) month
  , sum(totals.visits) visits
  , sum(totals.pageviews) pageviews
  , sum(totals.transactions) transactions
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*` 
where _table_suffix between '0101' and '0331'
group by month
order by month;
```
#### Result: 
<img width="636" height="107" alt="Image" src="https://github.com/user-attachments/assets/fd068a9a-e940-4a96-9edb-e79d8ab86bd5" />

#### Insights:
March 2017 delivered the strongest performance, with 69,931 visits, 259,522 pageviews, and 993 transactions. Although traffic declined in February, transactions continued to increase, suggesting an improvement in transaction efficiency. Overall, transactions grew by 39.4% from January to March, significantly outpacing the 8.1% growth in visits.

### 2. Bounce Rate by Traffic Source (Bounce_rate = num_bounce/total_visit) (order by total_visit DESC)
``` sql
SELECT trafficSource.source source
  , sum(totals.visits) total_visits
  , sum(totals.bounces) total_no_of_bounces
  , round(sum(totals.bounces) / sum(totals.visits) * 100, 3) bounce_rate
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
where _table_suffix between '0701' and '0731'
group by source
order by total_visits desc;
```
#### Result: 
<img width="525" height="677" alt="Image" src="https://github.com/user-attachments/assets/714e7d43-bf7e-4427-8108-9ae9bd2958ce" />

#### Insights:
- Google was the largest traffic source in July 2017, generating 38,400 visits, but its bounce rate was relatively high at 51.56%.
- YouTube had the highest bounce rate among the listed sources at 66.73%, while Reddit recorded the lowest at 28.57%, albeit with only 189 visits.
- Direct traffic showed a lower bounce rate than Google, suggesting stronger on-site engagement.

### 3. Revenue by traffic source by week, by month in June 2017
``` sql
     -- Revenue by month
SELECT 'Month' time_type
  , format_date('%Y%m', parse_date('%Y%m%d', date)) time
  , trafficSource.source source
  , round(sum(product.productRevenue) / 1000000, 4) revenue
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
UNNEST(hits) hits,
UNNEST(hits.product) product
where product.productRevenue is not NULL
group by time_type, time, source

UNION ALL

     -- Revenue by week
SELECT 'Week' time_type
  , format_date('%Y%W', parse_date('%Y%m%d', date)) time
  , trafficSource.source source
  , round(sum(product.productRevenue) / 1000000, 4) revenue
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
UNNEST(hits) hits,
UNNEST(hits.product) product
where product.productRevenue is not NULL
group by time_type, time, source
order by revenue desc;
```
#### Result: 
<img width="781" height="677" alt="Image" src="https://github.com/user-attachments/assets/ab76e8bc-191a-4b31-a4fb-c62497e9bfad" />

#### Insights:
- Direct traffic generated the highest reported monthly revenue in June 2017, reaching 97,333.62, compared with 18,757.18 from Google.
- Direct revenue fluctuated across the reported weeks, peaking at 30,908.91 in week 24 and remaining relatively high in week 25.
- These results highlight the importance of Direct traffic as a revenue source and suggest that weekly performance should be monitored to understand revenue fluctuations across channels.

### 4. Conversion rate by traffic source in 2017. (order by conversion_rate desc)
``` sql
SELECT trafficSource.source source
  , sum(totals.visits) visits
  , sum(totals.transactions) transactions
  , round(sum(totals.transactions) / sum(totals.visits) * 100) conversion_rate
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
where _table_suffix between '0101' and '1231'
group by source
having transactions >= 50
order by conversion_rate desc;
```
#### Result:
<img width="628" height="108" alt="Image" src="https://github.com/user-attachments/assets/fcbe51d9-6509-4d11-8896-9352a30fb3f7" />

#### Insights:
- DFA achieved the highest conversion rate at 3.0%, outperforming Direct at 2.0% and Google at 1.0%, despite having significantly less traffic.
- Direct generated the highest number of transactions (4,697), highlighting the importance of traffic volume alongside conversion efficiency.
- These findings suggest that evaluating traffic sources by both conversion rate and transaction volume provides a more complete view of performance.

### 5. Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017.
``` sql
     -- Avg by purchasers
with purchasers as(
  SELECT format_date('%Y%m', parse_date('%Y%m%d', date)) month
    , round(sum(totals.pageviews) / count(distinct fullVisitorId), 7) avg_pageviews_purchase
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
  UNNEST(hits) hits,
  UNNEST(hits.product) product
  where _table_suffix between '0601' and '0731'
    and totals.transactions >= 1
    and product.productRevenue is not NULL
  group by month)

     -- Avg by non-purchasers
, non_purchasers as(
  SELECT format_date('%Y%m', parse_date('%Y%m%d', date)) month
    , round(sum(totals.pageviews) / count(distinct fullVisitorId), 7) avg_pageviews_non_purchase
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
  UNNEST(hits) hits,
  UNNEST(hits.product) product
  where _table_suffix between '0601' and '0731'
    and totals.transactions is NULL
    and product.productRevenue is NULL
  group by month)

    -- Avg by all type
select *
from purchasers
left join non_purchasers
using(month)
order by month;
```
#### Result:  
<img width="603" height="80" alt="Image" src="https://github.com/user-attachments/assets/d7291696-702f-4c09-a836-c4feaa1553f7" />

#### Insights:
- The data highlights a substantial difference in average pageviews between purchasers and non-purchasers. Users who did not complete a purchase generally viewed more pages than those who made a purchase. For example, in June 2017, non-purchasers averaged 316.87 pageviews, compared with just 94.02 for purchasers.
- Despite this high level of browsing activity, many users did not convert into customers, suggesting potential obstacles in the purchasing journey. To improve conversion rates, the business could focus on simplifying website navigation, optimizing product pages, and streamlining the checkout process to make purchasing easier and more convenient.

### 6. Average number of transactions per user that made a purchase in July 2017
``` sql
SELECT format_date('%Y%m', parse_date('%Y%m%d', date)) Month
  , round(sum(totals.transactions) / count(distinct fullVisitorId), 9) Avg_total_transactions_per_user
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
UNNEST(hits) hits,
UNNEST(hits.product) product
where totals.transactions >= 1
  and product.productRevenue is not NULL
group by Month;
```

#### Result: 
<img width="343" height="53" alt="Image" src="https://github.com/user-attachments/assets/bd78da18-d26f-4d3f-b549-d6b55a10b25e" />

#### Insights:
The table shows that purchasing users averaged approximately 4.16 transactions per user in July 2017. This metric provides insights into purchasing behavior and can help businesses monitor customer activity and assess changes in purchasing patterns over time. It may also support the evaluation of marketing campaigns and promotional activities, although further analysis would be needed to determine their actual impact.

### 7. Revenue contribution by device (desktop,mobile...) (order by ratio desc)
``` sql
with device_revenue as ( 
 SELECT device.deviceCategory device
    , round(sum(product.productRevenue) / 1000000, 2) revenue_by_device
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`,
  UNNEST(hits) hits,
  UNNEST(hits.product) product
  where totals.transactions is not NULL 
    and product.productRevenue is not NULL
  group by device
)

, total_rev as (
  select round(sum(revenue_by_device), 2) total_revenue
  from device_revenue
)

select d.device
  , d.revenue_by_device
  , t.total_revenue
  , round((d.revenue_by_device / t.total_revenue) * 100, 2) ratio
from device_revenue d
cross join total_rev t
order by ratio desc;
```

#### Result: 
<img width="525" height="108" alt="Image" src="https://github.com/user-attachments/assets/7e30b0b0-bd51-421a-952c-6c40ef715888" />

#### Insights:
- Desktop devices drive the vast majority of the revenue, accounting for 96.14% ($1,674,745.74 out of $1,742,046.97).
- Mobile & Tablet Share: Mobile contributes 3.25% ($56,553.10), while tablets make up a minimal 0.62% ($10,748.13) of total revenue.
- While traffic might be distributed differently, desktop users hold an overwhelming share of high-value conversions. Optimization efforts should focus on ensuring the desktop checkout experience remains frictionless, while investigating why mobile conversion/revenue contribution is disproportionately low.

### 8. Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017. 
``` sql
-- List users bought product "YouTube Men's Vintage Henley"
with buyers as (
  SELECT distinct fullVisitorId purchasers
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
  UNNEST(hits) hits,
  UNNEST(hits.product) product
  where product.productRevenue is not NULL
    and totals.transactions >= 1
    and product.v2ProductName = "YouTube Men's Vintage Henley"
)

SELECT product.v2ProductName other_purchased_products
  , sum(product.productQuantity) quantity
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*` a,
UNNEST(hits) hits,
UNNEST(hits.product) product
inner join buyers b
  on a.fullVisitorId = b.purchasers
where product.productRevenue is not NULL
  and totals.transactions >= 1
  and product.v2ProductName != "YouTube Men's Vintage Henley"
group by other_purchased_products
order by quantity desc;
```

#### Result:
<img width="485" height="568" alt="Image" src="https://github.com/user-attachments/assets/9f6e5a69-950f-4d6c-96a6-7d5403379fc7" />

#### Insights:
- Google Sunglasses were the most popular complementary product, with 20 units sold, significantly more than any other item.
- Other popular products included Google Women's Vintage Hero Tee Black (7 units) and SPF-15 Slim & Slender Lip Balm (6 units).
- These findings suggest opportunities for cross-selling and product bundling, especially with Google-branded accessories and apparel.

### 9. Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017. For example, 100% product view then 40% add_to_cart and 10% purchase.
- Option 1:
``` sql
with view_page as (
  SELECT format_date('%Y%m', parse_date('%Y%m%d', date)) month
    , count(hits.eCommerceAction.action_type) num_product_view
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
  UNNEST(hits) AS hits
  where _table_suffix between '0101' and '0331'
    and hits.eCommerceAction.action_type = '2'
  group by month
)
, add_to_cart as (
  SELECT format_date('%Y%m', parse_date('%Y%m%d', date)) month
    , count(hits.eCommerceAction.action_type) num_addtocart
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
  UNNEST(hits) AS hits
  where _table_suffix between '0101' and '0331'
    and hits.eCommerceAction.action_type = '3'
  group by month
)
, purchase as (
  SELECT format_date('%Y%m', parse_date('%Y%m%d', date)) month
    , count(hits.eCommerceAction.action_type) num_purchase
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
  UNNEST(hits) AS hits,
  UNNEST(hits.product) AS product
  where _table_suffix between '0101' and '0331'
    and hits.eCommerceAction.action_type = '6'
    and product.productRevenue is not NULL
  group by month
)

select *
  , round((num_addtocart / num_product_view) * 100, 2) add_to_cart_rate
  , round((num_purchase / num_product_view) * 100, 2) purchase_rate
FROM view_page 
left join add_to_cart 
using (month)
left join purchase 
using (month)
order by month;
```

- Option 2:
``` sql
with raw_data as (
  SELECT format_date('%Y%m', parse_date('%Y%m%d', date)) month
    , sum(case when hits.eCommerceAction.action_type = '2' then 1 else 0 end) num_product_view
    , sum(case when hits.eCommerceAction.action_type = '3' then 1 else 0 end) num_addtocart
    , sum(case when hits.eCommerceAction.action_type = '6'
            and product.productRevenue is not NULL 
              then 1 else 0 end) num_purchase
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
  UNNEST(hits) hits,
  UNNEST(hits.product) product
  where _table_suffix between '0101' and '0331'
  group by month
)

select *
  , round((num_addtocart / num_product_view) * 100, 2) add_to_cart_rate
  , round((num_purchase / num_product_view) * 100, 2) purchase_rate
FROM raw_data
order by month;
```

#### Result:
<img width="784" height="109" alt="Image" src="https://github.com/user-attachments/assets/79574d43-f5de-4355-ae64-38ad63b01a35" />

#### Insights:
- The results show a consistent improvement in the conversion funnel from January to March 2017. The add-to-cart rate increased from 28.47% in January to 37.29% in March, while the purchase rate rose from 8.31% to 12.64%.
- Although product views fluctuated, both add-to-cart and purchase counts increased, indicating improved customer engagement and purchasing performance.
- The business could investigate which marketing campaigns or website changes contributed to this growth and apply successful strategies to further improve conversions.

### 10. Calculate revenue by week from May to July 2017 and culmulative revenue.
``` sql
with week_revenue as(
  SELECT format_date('%Y%W', parse_date('%Y%m%d', date)) Week
    , round(sum(product.productRevenue) / 1000000, 2) weekly_revenue
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
  UNNEST(hits) hits,
  UNNEST(hits.product) product
  where _table_suffix between '0501' and '0731'
    and product.productRevenue is not null
  group by Week
  order by Week
)

-- caculate cumulative revenue
select *
  , sum(weekly_revenue) over(order by Week) cumulative_revenue
from week_revenue 
order by Week;
```

#### Result:
<img width="417" height="406" alt="Image" src="https://github.com/user-attachments/assets/216a9a7d-3411-43ae-9d19-854be30f482f" />

#### Insights:
- Weekly revenue fluctuated considerably, suggesting that sales may be influenced by promotions, customer demand, or seasonal patterns.
- Week 201729 generated the highest revenue in the displayed results (approximately 57.1 million), making it worth investigating which products or campaigns drove this performance.
- Total revenue reached approximately 425.3 million by week 201731, showing sustained revenue generation despite weekly fluctuations.

## Conclusions & Recommendations:
### 1. Conclusion
This project used SQL and Google BigQuery to analyze eCommerce performance, revealing insights into customer behavior, marketing channels, conversion rates, and revenue trends. The results highlighted opportunities to improve conversion, strengthen marketing effectiveness, and increase sales through data-driven decisions.

### 2. Recommendations
- Improve conversion rates: Simplify website navigation and checkout to reduce drop-offs in the purchasing funnel.
- Optimize marketing channels: Evaluate traffic sources based on both conversion rates and revenue, not just visit volume.
- Enhance mobile performance: Investigate the low mobile revenue contribution and improve the mobile shopping experience.
- Increase sales opportunities: Use complementary product recommendations and monitor weekly revenue trends to identify opportunities for cross-selling and targeted promotions.

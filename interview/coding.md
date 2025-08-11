### BQ1
## What product categories bring the highest increase in revenue when discounted?
I would like to identify which categoryies respond best to discounting allowing targeted investment by comparing the category average revenue/product difference between discounted and non-discounted products by using clustered column chart. I extracted the top 10 and bottom 10 categories for analysis.

```sql
WITH ChainedData AS(
SELECT category_name,
    discounted_product,
    SUM(total_revenue),2 AS total_revenue,
    SUM(total_unique_product) AS total_unique_product
FROM `casestudy-foodpanda.Data.salesperformancedata`
WHERE chain_name = 'A'
GROUP BY category_name, discounted_product
),

AggregatedData AS(
  SELECT
    category_name, discounted_product,
    SUM(CASE WHEN discounted_product = TRUE THEN total_revenue ELSE 0 END) AS discounted_total_revenue,
    SUM(CASE WHEN discounted_product = TRUE THEN total_unique_product ELSE 0 END) AS discounted_total_unique_product,
    SUM(CASE WHEN discounted_product = FALSE THEN total_revenue ELSE 0 END) AS non_discounted_total_revenue,
    SUM(CASE WHEN discounted_product = FALSE THEN total_unique_product ELSE 0 END) AS non_discounted_total_unique_product
    FROM ChainedData
    GROUP BY category_name
)

SELECT category_name, 
ROUND((non_discounted_total_revenue / non_discounted_total_unique_product),2) AS non_discounted_avg_rev_per_unique_product,
ROUND((discounted_total_revenue / discounted_total_unique_product),2) AS discounted_avg_rev_per_unique_product,
ROUND((discounted_total_revenue / discounted_total_unique_product) - (non_discounted_total_revenue / non_discounted_total_unique_product),2) AS increase_in_avg_rev_per_unique_product
FROM
  AggregatedData
WHERE
  (discounted_total_unique_product IS NOT NULL AND discounted_total_unique_product != 0) AND
  (non_discounted_total_unique_product IS NOT NULL AND non_discounted_total_unique_product != 0)
ORDER BY
--- finding Top 10 Category Average Sales per Product
  increase_in_avg_rev_per_unique_product DESC
--- finding Bottom 10 Category Average Sales per Product
  increase_in_avg_rev_per_unique_product ASC
LIMIT 10;
```

### BQ2
## What categories need more or less products on discount?
I would like to explore the relationship of # of discounted category product vs revenue performance to check if current discount are distributed optimally and explore opportunity to enhance the discount effectiveness. Scatter Plot will be used for answering this business question.

```sql
SELECT
    category_name,
    SUM(total_discounted_product) AS total_discounted_products_sold,
    ROUND(SUM(CASE WHEN total_discounted_product > 0 THEN total_revenue ELSE 0 END),2) AS total_revenue
FROM `casestudy-foodpanda.Data.salesperformancedata`
GROUP BY category_name;
```

### BQ3 
## Are there months or seasons where discounts generate more incremental revenue?
Timing is also another essential factor to determine the effectiveness of discount promotion. Therefore, I am trying to understand the sales pattern by time to see if any specific season/month influences the revenue. 

Finding out which month has the highest orders driven by the discounted orders comparing with the non-discounted. Clustered column chart is used.
```sql
SELECT
    order_date_local_month,
    ROUND(AVG(CASE WHEN `total_revenue_from_discounted_products_` > 0 THEN `total_revenue_from_discounted_products_` / `total_no_order` ELSE NULL END),2) AS `avg_revenue_per_discounted_order`,
    ROUND(AVG(CASE WHEN `total_revenue_from_discounted_products_` = 0 THEN `total_revenue` / `total_no_order` ELSE NULL END),2) AS `avg_revenue_per_non_discounted_order`
FROM
    `casestudy-foodpanda.Data.salesperformancedata`
GROUP BY order_date_local_month
ORDER BY order_date_local_month;
```

Finding out the trend of monthly average revenue per order for both discounted and non-disocunted orders.
```sql
SELECT order_date_local_month,
  SUM(CASE WHEN discounted_product IS True THEN total_no_order ELSE 0 END) AS DiscountedOrder,
  SUM(CASE WHEN discounted_product IS FALSE THEN total_no_order ELSE 0 END) AS NonDiscountedOrder,
FROM `casestudy-foodpanda.Data.salesperformancedata`
GROUP BY order_date_local_month
ORDER BY order_date_local_month;
```

### Appendix

## Product Distribution by month
```sql
select order_date_local_month,
SUM(CASE WHEN `total_revenue_from_discounted_products_` > 0 THEN total_discounted_product ELSE NULL END) AS discounted_product,
SUM(CASE WHEN `total_revenue_from_discounted_products_` = 0 THEN total_unique_product ELSE NULL END) AS nondiscounted_product
FROM
    `casestudy-foodpanda.Data.salesperformancedata`
GROUP BY order_date_local_month
ORDER BY order_date_local_month;
```

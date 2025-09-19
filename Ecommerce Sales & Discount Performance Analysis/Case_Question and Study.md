# BQ1 What product categories bring the highest increase in revenue when discounted?

<img width="1092" height="506" alt="Image" src="https://github.com/user-attachments/assets/45f23e37-79c7-4051-a090-256e3c672002" />

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

# BQ2 What categories need more or less products on discount?
I would like to explore the relationship of # of discounted category product vs revenue performance to check if current discount are distributed optimally and explore opportunity to enhance the discount effectiveness. Scatter Plot will be used for answering this business question.

<img width="1097" height="465" alt="Image" src="https://github.com/user-attachments/assets/3caa30b0-698c-41c5-a9d0-3c9f87b6c0f4" />

```sql
SELECT
    category_name,
    SUM(total_discounted_product) AS total_discounted_products_sold,
    ROUND(SUM(CASE WHEN total_discounted_product > 0 THEN total_revenue ELSE 0 END),2) AS total_revenue
FROM `casestudy-foodpanda.Data.salesperformancedata`
GROUP BY category_name;
```

# BQ3 Are there months or seasons where discounts generate more incremental revenue?
Timing is also another essential factor to determine the effectiveness of discount promotion. Therefore, I am trying to understand the sales pattern by time to see if any specific season/month influences the revenue. 

<img width="1087" height="469" alt="Image" src="https://github.com/user-attachments/assets/cae00df2-9868-4333-8631-97c708534d81" />

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

# Recommendation
Based on my above analysis, 3 actions can be done by Chain A for improving the effectiveness of applying discount to products while striving for higher ROI.

<img width="1101" height="476" alt="Image" src="https://github.com/user-attachments/assets/388386cd-8f90-47a1-b716-62062020f4d1" />

# Appendix

### Product Distribution by month
<img width="426" height="264" alt="Image" src="https://github.com/user-attachments/assets/3ba8c0c0-d5bc-4019-9321-d08d84dffe9e" />

### Total Sales Quantity of Discounted Products by Month
<img width="466" height="433" alt="Image" src="https://github.com/user-attachments/assets/b85f9ed2-b172-495e-8a13-2c2f844a4aa7" />

### Categories Discounted Product Distribution (Top & Bottom 10)
<img width="1072" height="438" alt="Image" src="https://github.com/user-attachments/assets/0e6a53d3-c20d-4580-8d2c-a522c24bf160" />

### Discounted Categories and Orders in Apr 2025
Support information for BQ3 to explain why Apr 2025 had a higher monthly average revenue per discounted order instead of the non-discounted.

<img width="579" height="343" alt="Image" src="https://github.com/user-attachments/assets/d37d129b-db05-48da-898c-8dd3afe71216" />



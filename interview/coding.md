### BQ1

```sql
WITH ChainedData AS(
SELECT category_name,
    discounted_product,
    SUM(CAST(REPLACE(REPLACE(CAST(total_revenue AS string),',',''),' ','') AS NUMERIC))AS total_revenue,
    SUM(total_unique_product) AS total_unique_product

FROM `casestudy-foodpanda.Data.salesperformancedata`
WHERE chain_name = 'A'
GROUP BY category_name, discounted_product
),

AggregatedData AS(
  SELECT
    category_name,
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
  increase_in_avg_rev_per_unique_product DESC
LIMIT 10;
```

### BQ1 Which product should order more or less?
To optimze the supply and prevent out-of-stock for best-selling products, related product stock pressure and sales performance will be examinated. Product out-of-stock means high value in stock pressure (quantity of product ordered divided by quantity of product in stock). After selecting the top items with high stock pressure, review performance to check the values can be broguht from the product, and then prioritize those products for restocking.

```sql
WITH 
low_stock_table AS(
	SELECT p.productCode,
		ROUND(SUM(o.quantityOrdered)*1.0/p.quantityInStock , 2) AS stockpressure_ratio
	FROM products AS p
	LEFT JOIN orderdetails AS o
		ON p.productcode = o.productCode
	GROUP BY p.productCode
	ORDER BY stockpressure_ratio DESC
	LIMIT 10
),
ProductPerformance AS(
	SELECT productCode,
		SUM(uantityOrdered*priceEach) AS product_perf
	FROM orderdetails AS o
	WHERE productCode IN (SELECT productCode FROM low_stock_table)
	GROUP BY productCode
	ORDER BY product_perf DESC
	LIMIT 10
)

SELECT productName, productLine, low_stock_table.stockpressure_ratio
FROM product
INNER JOIN ProductPerformance
ON products.productCode = ProductPerformance.productCode
INNER JOIN low_stock_table
ON products.productCode = low_stock_table.productCode;
```

<img width="890" height="463" alt="Image" src="https://github.com/user-attachments/assets/0436b9d8-7701-4b7d-b082-baebbe8def4c" />

The above products have a comparatively high stock pressure ratio（>1) , which implies that the demand is larger than current supply of the company, while 1968 Ford Mustang, 1911 Ford Town Car and 1928 Mercedes-Benz SSK have the highest ratio. Vintage cars and motocycles are top priority to restock. They sell frequently with great sales performances.

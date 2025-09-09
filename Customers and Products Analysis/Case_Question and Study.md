# BQ1 Which product should order more or less?
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

The above products have a comparatively high stock pressure ratio（>1) , which implies that current product demand is larger than its supply, while 1968 Ford Mustang, 1911 Ford Town Car and 1928 Mercedes-Benz SSK have the highest ratio. Vintage cars and motocycles are top priority to restock. They sell frequently with great sales performances.


# BQ2 How should we tailor marketing and communication strategies to customer behaviors? 
By analysing customer purchase influence, we can understand the customer loyalty and profit bring to the company. In this practice, I am going to analyse the profit performance by country and categorize customers (VIP and less-enegaged) for specializing the marketing and communication campaigns to retain, as well as attract new customers.

```sql
WITH customerinfo AS(
  SELECT customerNumber,contactLastName,contactFirstName,city,country
  FROM customers
)

SELECT
  contactLastName, contactFirstName, country,
  COUNT(DISTINCT o.orderNumber) AS numberOfOrders,
  ROUND(SUM(od.quantityOrdered * (od.priceEach - p.buyPrice)),2) AS profit
FROM products AS p
INNER JOIN
  orderdetails AS od
  ON od.productCode = p.productCode
INNER JOIN 
  orders AS o
  ON o.orderNumber = od.orderNumber
INNER JOIN
  customerinfo AS ci
  ON o.customerNumber = ci.customerNumber
GROUP BY
  ci.contactLastName, 
  ci.contactFirstName,
  ci.country
ORDER BY 
  profit DESC;
```
<img width="869" height="537" alt="Image" src="https://github.com/user-attachments/assets/5341a665-234c-4190-8443-24f01661e9ca" />

From above table, majority of the profit is driven by the local US customers, followed by Spain and France. Oceania countries also performed well and contributed relatively high profits compared with some European and Asian countries. This can be explained by the origin of the company (local reputation and market influence) and the customer interests and preference on the classic and vintage motors. Besides, shipping costs were much higher for oversea shipping leading to the decrease in profit. 

### Top VIP Customers
<img width="551" height="97" alt="Image" src="https://github.com/user-attachments/assets/9c4709c1-7b58-4eb3-ae49-3f5a0cf8c205" />

### Less-engaged Customers
<img width="551" height="97" alt="Image" src="https://github.com/user-attachments/assets/f883df92-58b6-401b-a1fa-4bcfae3b0d1a" />

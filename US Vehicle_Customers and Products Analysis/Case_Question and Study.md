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

From above table, majority of the profit is driven by the local US customers, followed by Spain and France. Oceania countries also performed well and contributed relatively high profits compared with some European and Asian countries. This is likely due to the origin of the company, which influence its the local reputation and market presence. The US customer interests in classic and vintage motors might also play a role. 

For those high-contributing profit countries, the company can leverage company's local reputation with campaigns and highlight local heritage, like holding showcase or even vintage-style motor brand pop-up to expose in public and offer unique experience to attendees in order to stimulate potential sales. For Spain and France, tailor advertisements presenting the elegance and artistry of the products.

Although countries from Oceania contributed quite a number of protfit, shipping costs are inescapable and challenging for overseas orders and can reduce the profit. Company can collaborate with local distributors which cater to luxury and classic products, ensuring secure delivery experience. Offering shipping discounts with minimum purchase amount/quantity encourage larger purchases.

### Top VIP Customers
<img width="551" height="97" alt="Image" src="https://github.com/user-attachments/assets/9c4709c1-7b58-4eb3-ae49-3f5a0cf8c205" />

VIP system can be set up for premium customers, by offering invitations to exclusive events like private showing of newly acquired vintage cars and curated tour to car collection. The personalized approach can make customers feel valued and exclusive.

### Less-engaged Customers
<img width="551" height="97" alt="Image" src="https://github.com/user-attachments/assets/f883df92-58b6-401b-a1fa-4bcfae3b0d1a" />

In luxury market, less-engaged customers are not the necessarily lost cause. Instead of providing price promotion, communication should be tailored by personal interests, like sharing recent development of specfic model/brand, or gathering with other customers who also interested on the items to reignite their passion.


# BQ3 How much can we spend on acquiring new customers?
Extend to previous question, we have reviewed existing customers. Meanwhile maintaining customer relationship with current customers, obtaining new customers can stimulate purchase sales. Before planning new strategy for new customer acquisition, we have to check if it is worth spending on new customer acquisition and examine whether how much can be spent for better costing control on marketing.

```sql
ITH
  payment_with_year_month_table AS (
    SELECT
      *,
      EXTRACT(YEAR FROM paymentDate) * 100 + EXTRACT(MONTH FROM paymentDate) AS year_month
    FROM
     	payments AS p
  ),
  customers_by_month_table AS (
    SELECT
      p1.year_month,
      COUNT(DISTINCT p1.customerNumber) AS number_of_customers,
      SUM(p1.amount) AS total
    FROM
      payment_with_year_month_table AS p1
    GROUP BY p1.year_month
  ),
  new_customers_by_month_table AS (
    SELECT
      p1.year_month,
      COUNT(DISTINCT p1.customerNumber) AS number_of_new_customers,
      SUM(p1.amount) AS new_customer_total,
      (
        SELECT
          number_of_customers
        FROM
          customers_by_month_table AS c
        WHERE
          c.year_month = p1.year_month
      ) AS number_of_customers,
      (
        SELECT
          total
        FROM
          customers_by_month_table AS c
        WHERE
          c.year_month = p1.year_month
      ) AS total
    FROM
      payment_with_year_month_table AS p1
    WHERE
      p1.customerNumber NOT IN (
        SELECT
          customerNumber
        FROM
          payment_with_year_month_table AS p2
        WHERE
          p2.year_month < p1.year_month
      )
    GROUP BY p1.year_month
  )
SELECT
  year_month,
  ROUND(number_of_new_customers * 100 / number_of_customers, 1) AS number_of_new_customers_props,
  ROUND(new_customer_total * 100 / total, 1) AS new_customers_total_props
FROM
  new_customers_by_month_table
ORDER BY year_month;

```
<img width="489" height="353" alt="Image" src="https://github.com/user-attachments/assets/6ead34fd-6122-48ed-b833-adf7d6453e1f" />

<img width="3015" height="1870" alt="Image" src="https://github.com/user-attachments/assets/3783a933-74b2-44d2-8281-023beda3706b" />

From above data, the number of clients keep decling since 2003, and had the lowest new customers and sales generated in 2004. In year 2005, data is present in dataset but yet no record found, which implies that no new customers since September 2004. It is essential for the company to spend money acquiring new customers.

To Calculate how much money can be spent for new customer marketing campaigns, we can compute customer lifetime value to find average amount of money customer generates and predict future profits. 



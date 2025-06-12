
# E-commerce Sales Data Analysis

This project performs an in-depth analysis of an e-commerce sales database using SQL, Python (pandas, matplotlib, seaborn), and MySQL. It answers key business questions and visualizes the data for better understanding.

## 📦 Dataset

The analysis uses an e-commerce dataset consisting of tables like `customers`, `orders`, `order_items`, `payments`, `products`, and `sellers` stored in a MySQL database.

## 📊 Analysis & Insights

### 1. Unique Cities of Customers
```sql
SELECT DISTINCT customer_city FROM customers;
```
Displays all unique customer cities.

### 2. Orders Placed in 2017
```sql
SELECT COUNT(order_id) FROM orders WHERE YEAR(order_purchase_timestamp) = 2017;
```
**Total Orders**: 45,101

### 3. Total Sales per Category
```sql
SELECT UPPER(products.product_category), ROUND(SUM(payments.payment_value),2)
FROM products
JOIN order_items ON products.product_id = order_items.product_id
JOIN payments ON payments.order_id = order_items.order_id
GROUP BY product_category;
```

### 4. Percentage of Installment Payments
```sql
SELECT ((SUM(CASE WHEN payment_installments >= 1 THEN 1 ELSE 0 END)) / COUNT(*)) * 100 FROM payments;
```
**Result**: 99.9981%

### 5. Customer Count by State (with Bar Chart)
```sql
SELECT customer_state, COUNT(customer_id) FROM customers GROUP BY customer_state;
```

### 6. Monthly Order Count in 2018 (with Bar Chart)
```sql
SELECT MONTHNAME(order_purchase_timestamp), COUNT(order_id)
FROM orders
WHERE YEAR(order_purchase_timestamp) = 2018
GROUP BY MONTHNAME(order_purchase_timestamp);
```

### 7. Average Products per Order by City
```sql
WITH count_per_order AS (
  SELECT orders.order_id, orders.customer_id, COUNT(order_items.order_id) AS oc
  FROM orders JOIN order_items ON orders.order_id = order_items.order_id
  GROUP BY orders.order_id, orders.customer_id
)
SELECT customers.customer_city, ROUND(AVG(count_per_order.oc),2)
FROM customers
JOIN count_per_order ON customers.customer_id = count_per_order.customer_id
GROUP BY customers.customer_city
ORDER BY AVG(count_per_order.oc) DESC;
```

### 8. Sales Percentage by Category
```sql
SELECT UPPER(products.product_category),
ROUND((SUM(payments.payment_value) / (SELECT SUM(payment_value) FROM payments)) * 100, 2)
FROM products
JOIN order_items ON products.product_id = order_items.product_id
JOIN payments ON payments.order_id = order_items.order_id
GROUP BY product_category
ORDER BY 2 DESC;
```

### 9. Correlation Between Product Price and Purchases
```sql
SELECT product_category, COUNT(order_items.product_id), ROUND(AVG(order_items.price),2)
FROM products JOIN order_items ON products.product_id = order_items.product_id
GROUP BY product_category;
```
**Correlation**: -0.1063

### 10. Seller Revenue and Ranking (Top 5)
```sql
SELECT seller_id, SUM(payments.payment_value), DENSE_RANK() OVER (ORDER BY SUM(payments.payment_value) DESC)
FROM order_items JOIN payments ON order_items.order_id = payments.order_id
GROUP BY seller_id;
```

### 11. Moving Average of Order Value per Customer
```sql
SELECT customer_id, order_purchase_timestamp, payment,
AVG(payment) OVER (PARTITION BY customer_id ORDER BY order_purchase_timestamp ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)
FROM (
  SELECT orders.customer_id, orders.order_purchase_timestamp, payments.payment_value AS payment
  FROM payments JOIN orders ON payments.order_id = orders.order_id
) AS a;
```

### 12. Monthly Cumulative Sales
```sql
SELECT year, month, payment,
SUM(payment) OVER (ORDER BY year, month) AS cumulative_sales
FROM (
  SELECT YEAR(order_purchase_timestamp) AS year, MONTH(order_purchase_timestamp) AS month,
  ROUND(SUM(payment_value), 2) AS payment
  FROM orders JOIN payments ON orders.order_id = payments.order_id
  GROUP BY year, month
) AS a;
```

### 13. Year-over-Year Sales Growth
```sql
WITH yearly AS (
  SELECT YEAR(order_purchase_timestamp) AS year, SUM(payment_value) AS payment
  FROM orders JOIN payments ON orders.order_id = payments.order_id
  GROUP BY year
)
SELECT year, ((payment - LAG(payment, 1) OVER(ORDER BY year)) / LAG(payment, 1) OVER(ORDER BY year)) * 100
FROM yearly;
```

### 14. Customer Retention within 6 Months
```sql
WITH first_order AS (
  SELECT customer_id, MIN(order_purchase_timestamp) AS first_order
  FROM orders GROUP BY customer_id
),
reorders AS (
  SELECT o.customer_id, COUNT(DISTINCT o.order_purchase_timestamp)
  FROM orders o
  JOIN first_order f ON o.customer_id = f.customer_id
  WHERE o.order_purchase_timestamp > f.first_order AND o.order_purchase_timestamp < DATE_ADD(f.first_order, INTERVAL 6 MONTH)
  GROUP BY o.customer_id
)
SELECT 100 * COUNT(DISTINCT reorders.customer_id) / COUNT(DISTINCT first_order.customer_id)
FROM first_order LEFT JOIN reorders ON first_order.customer_id = reorders.customer_id;
```

### 15. Top 3 Customers by Yearly Spending (with Bar Chart)
```sql
SELECT year, customer_id, payment, d_rank
FROM (
  SELECT YEAR(order_purchase_timestamp) AS year, customer_id, SUM(payment_value) AS payment,
  DENSE_RANK() OVER (PARTITION BY YEAR(order_purchase_timestamp) ORDER BY SUM(payment_value) DESC) AS d_rank
  FROM orders JOIN payments ON orders.order_id = payments.order_id
  GROUP BY year, customer_id
) AS ranked
WHERE d_rank <= 3;
```

---

## 📌 Technologies Used
- MySQL
- Python (pandas, seaborn, matplotlib)
- SQL Joins, Aggregations, CTEs, Window Functions

## 📈 Key Takeaways
- Sales distribution, city-based analysis, customer retention, and sales trends.
- Strong use of SQL for complex queries and insights.
- Data visualization for easier interpretation.

---

## 📁 Author

Vyomesh Katariya

---

Feel free to explore and modify the queries to generate your own insights!

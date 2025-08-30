# 🛒 Olist E-Commerce Data Analysis  

### 📌 Introduction  
This project explores the **Olist Brazilian E-Commerce dataset**, a comprehensive collection of real transaction records from an online marketplace. The dataset captures every step of the order lifecycle — from purchase, payment, and shipping to customer feedback.  

It contains information across multiple dimensions:  
- **Orders** 📝 – purchase timestamps, delivery performance  
- **Products** 📦 – categories, descriptions, attributes  
- **Sellers** 🏬 – marketplace vendor profiles  
- **Customers** 👥 – demographics and locations  
- **Payments & Reviews** 💳⭐ – payment methods, installment patterns, and customer satisfaction  

By analyzing these tables through SQL, we uncover insights into **sales trends, logistics efficiency, customer behavior, and market growth opportunities**.  

---

## 📑 Table of Contents  
- [📈 Order Volume by Year](#-order-volume-by-year)  
- [💳 Sequential Payments](#-sequential-payments)  
- [💵 Sales by Year](#-sales-by-year)  
- [📦 Sales by Product Category](#-sales-by-product-category)  
- [🚚 Delivery Efficiency](#-delivery-efficiency)  
- [⏱️ Average Delivery Time](#️-average-delivery-time)  
- [🏬 Key Suppliers](#-key-suppliers)  
- [🌍 Top Markets](#-top-markets)  
- [⭐ Customer Reviews Metrics](#-customer-reviews-metrics)  

---

## 📈 Order Volume by Year  

```sql
SELECT COUNT(*) AS Total_orders
FROM orders;

SELECT YEAR(order_purchase_timestamp) AS year, COUNT(order_id) AS orders
FROM orders
GROUP BY year
ORDER BY year ASC;
```

## 💳 Sequential Payments  

```sql
WITH recurring_payment_orders AS (
    SELECT order_id
    FROM order_payments
    GROUP BY order_id
    HAVING SUM(payment_sequential) > COUNT(order_id) 
)
SELECT op.order_id, SUM(op.payment_value) AS Total_Payment
FROM order_payments op
JOIN recurring_payment_orders rpo 
    ON op.order_id = rpo.order_id
GROUP BY op.order_id
ORDER BY op.order_id;
```

## 💵 Sales by Year  

```sql
SELECT YEAR(ord.order_purchase_timestamp) AS Year, SUM(orp.payment_value) AS Sales
FROM orders ord
LEFT JOIN order_payments orp
    ON ord.order_id = orp.order_id
GROUP BY Year
ORDER BY Year;
```

## 📦 Sales by Product Category  

```sql
WITH product_detail AS (
    SELECT ord.order_id, ordi.product_id, prdt.product_category_name_english
    FROM orders ord
    LEFT JOIN order_items ordi
        ON ord.order_id = ordi.order_id
    LEFT JOIN products prd
        ON ordi.product_id = prd.product_id
    LEFT JOIN product_category_name_translation prdt
        ON prd.product_category_name = prdt.product_category_name
), 
payment_detail AS (
    SELECT order_id, SUM(payment_value) AS Payment
    FROM order_payments
    GROUP BY order_id
)
SELECT prod_detail.product_category_name_english AS Product_Category, 
       SUM(pay_detail.payment) AS Sales_Product_Category,
       ROUND((SUM(pay_detail.payment) / (SELECT SUM(payment) FROM payment_detail)) * 100, 1) AS '%_of_sales'
FROM product_detail prod_detail
LEFT JOIN payment_detail pay_detail
    ON prod_detail.order_id = pay_detail.order_id
GROUP BY product_category_name_english
HAVING product_category IS NOT NULL
ORDER BY Sales_Product_Category DESC;


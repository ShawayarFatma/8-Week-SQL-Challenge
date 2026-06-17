# Case Study #1 - Danny's Diner
## 8-Week SQL Challenge

---

## 📋 Overview
This document contains comprehensive SQL solutions for Danny's Diner case study, addressing business intelligence questions ranging from customer spending analysis to loyalty program metrics.

---

## 🎯 Main Questions

### 1. Total Amount Each Customer Spent at the Restaurant

**Objective:** Calculate total spending per customer

```sql
SELECT s.customer_id, 
       SUM(m.price) AS total_amt_spent
FROM sales s 
JOIN menu m ON s.product_id = m.product_id
GROUP BY 1
ORDER BY 1;
```

---

### 2. Number of Days Each Customer Visited the Restaurant

**Objective:** Count distinct visit days per customer

```sql
SELECT customer_id, 
       COUNT(DISTINCT order_date) AS days
FROM sales
GROUP BY 1
ORDER BY 1;
```

---

### 3. First Item Purchased by Each Customer

**Objective:** Identify the first menu item purchased by each customer

```sql
WITH cte AS 
(
    SELECT customer_id, 
           order_date, 
           product_id,
           RANK() OVER(PARTITION BY customer_id ORDER BY order_date) AS rn
    FROM sales
)
SELECT c.customer_id, 
       m.product_name AS first_purchase
FROM menu m 
JOIN cte c ON m.product_id = c.product_id
WHERE rn = 1
GROUP BY 1, 2
ORDER BY customer_id;
```

---

### 4. Most Purchased Item on the Menu

**Objective:** Find the most popular item and purchase count

```sql
SELECT m.product_name, 
       COUNT(s.product_id) AS num_purchases
FROM menu m 
JOIN sales s ON s.product_id = m.product_id
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1;
```

---

### 5. Most Popular Item for Each Customer

**Objective:** Identify top purchased item per customer

```sql
WITH cte AS 
(
    SELECT customer_id, 
           m.product_name, 
           COUNT(s.product_id) AS num_purc,
           DENSE_RANK() OVER(PARTITION BY customer_id 
                             ORDER BY COUNT(s.product_id) DESC) AS rn
    FROM sales s 
    JOIN menu m ON s.product_id = m.product_id
    GROUP BY 1, 2
)
SELECT customer_id, 
       product_name, 
       num_purc
FROM cte 
WHERE rn = 1;
```

---

### 6. First Purchase After Membership Joining

**Objective:** Find the first item purchased after becoming a member

```sql
WITH cte AS
(
    SELECT s.customer_id, 
           m.product_name,
           ROW_NUMBER() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) AS rn
    FROM sales s 
    JOIN menu m ON s.product_id = m.product_id
    JOIN members mem ON s.customer_id = mem.customer_id 
                    AND s.order_date > mem.join_date
)
SELECT customer_id, 
       product_name 
FROM cte 
WHERE rn = 1;
```

---

### 7. Last Purchase Before Membership Joining

**Objective:** Identify the item purchased right before becoming a member

```sql
WITH cte AS 
(
    SELECT s.customer_id, 
           m.product_name,
           ROW_NUMBER() OVER(PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS rn
    FROM sales s 
    JOIN menu m ON s.product_id = m.product_id
    JOIN members mem ON s.customer_id = mem.customer_id
    WHERE s.order_date < mem.join_date
)
SELECT customer_id, 
       product_name 
FROM cte 
WHERE rn = 1;
```

---

### 8. Total Items and Amount Spent Before Membership

**Objective:** Calculate pre-membership purchases and spending

```sql
SELECT s.customer_id, 
       COUNT(s.product_id) AS cnt,
       SUM(m.price) AS total_spend
FROM sales s 
JOIN menu m ON s.product_id = m.product_id
JOIN members mem ON s.customer_id = mem.customer_id 
                AND s.order_date < mem.join_date
GROUP BY 1
ORDER BY 1;
```

---

### 9. Loyalty Points Calculation with Sushi Multiplier

**Objective:** Calculate points earned ($1 = 10 points, Sushi = 2x multiplier)

```sql
WITH cte AS 
(
    SELECT product_id,
           CASE WHEN product_id = 1 THEN price * 20
                ELSE price * 10 
           END AS points
    FROM menu
)
SELECT customer_id, 
       SUM(points) AS total_points
FROM sales s 
JOIN cte c ON s.product_id = c.product_id
GROUP BY 1
ORDER BY 1;
```

---

### 10. Points Calculation with First Week Bonus (End of January)

**Objective:** Calculate points with 2x multiplier for first week after joining (through January 31st)

```sql
WITH dates_cte AS 
(
    SELECT customer_id, 
           join_date, 
           join_date + 6 AS valid_date, 
           DATE_TRUNC('month', '2021-01-31'::DATE) + INTERVAL '1 month' 
           - INTERVAL '1 day' AS last_date
    FROM members
)
SELECT s.customer_id, 
       SUM(CASE WHEN m.product_name = 'sushi' THEN 20 * m.price
                WHEN s.order_date BETWEEN dc.join_date AND dc.valid_date 
                THEN 20 * m.price 
                ELSE 10 * m.price 
           END) AS points
FROM sales s 
JOIN dates_cte dc ON s.customer_id = dc.customer_id 
                 AND s.order_date <= dc.last_date
JOIN menu m ON s.product_id = m.product_id
GROUP BY 1;
```

---

## 🎁 Bonus Questions

### Bonus 1: Join All The Things

**Objective:** Recreate comprehensive customer activity table with membership status

```sql
SELECT s.customer_id, 
       s.order_date, 
       m.product_name, 
       m.price,
       CASE WHEN s.order_date >= mem.join_date THEN 'Y'
            ELSE 'N' 
       END AS member
FROM sales s 
JOIN menu m ON s.product_id = m.product_id
LEFT JOIN members mem ON s.customer_id = mem.customer_id
ORDER BY 1, 2;
```

---

### Bonus 2: Rank All The Things

**Objective:** Add ranking for member purchases only (NULL for non-members)

```sql
WITH cte AS 
(
    SELECT s.customer_id, 
           s.order_date, 
           m.product_name, 
           m.price,
           CASE WHEN s.order_date >= mem.join_date THEN 'Y'
                ELSE 'N' 
           END AS member
    FROM sales s 
    JOIN menu m ON s.product_id = m.product_id
    LEFT JOIN members mem ON s.customer_id = mem.customer_id
)
SELECT *,
       CASE WHEN member = 'Y' THEN 
            DENSE_RANK() OVER(PARTITION BY customer_id, member 
                              ORDER BY order_date)
       END AS ranking
FROM cte
ORDER BY 1, 2;
```

---

## 📊 Key Insights & Techniques Used

| Technique | Usage |
|-----------|-------|
| **Window Functions** | RANK(), DENSE_RANK(), ROW_NUMBER() for ranking and sequencing |
| **CTEs (Common Table Expressions)** | Complex multi-step logic with readable intermediate results |
| **Aggregations** | SUM(), COUNT() for business metrics |
| **Joins** | INNER and LEFT JOINs for data relationships |
| **CASE Statements** | Conditional logic for business rules |
| **Date Functions** | DATE_TRUNC(), date arithmetic for temporal analysis |

---

## 🔍 Summary

This case study demonstrates essential SQL skills including:
- ✅ Customer behavior analysis
- ✅ Loyalty program management
- ✅ Advanced window functions
- ✅ Date/temporal analysis
- ✅ Business metrics calculation

---

**Challenge:** 8-Week SQL Challenge by Data With Danny  
**Date:** 2026-06-17  
**Difficulty:** Intermediate to Advanced

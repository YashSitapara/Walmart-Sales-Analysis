## Business Questions and SQL Queries

---

### **Q1: Find different payment methods, number of transactions, and quantity sold by payment method**

```sql
SELECT 
    payment_method,
    COUNT(*) AS no_payments,
    SUM(quantity) AS no_qty_sold
FROM walmart
GROUP BY payment_method;
```

---

### **Q2: Identify the highest-rated category in each branch, display the branch, category, and avg rating**

```sql
WITH avg_ratings AS (
    SELECT 
        branch,
        category,
        AVG(rating) AS avg_rating
    FROM walmart
    GROUP BY branch, category
),
max_ratings AS (
    SELECT 
        branch,
        MAX(avg_rating) AS max_rating
    FROM avg_ratings
    GROUP BY branch
)
SELECT 
    a.branch,
    a.category,
    a.avg_rating
FROM avg_ratings a
JOIN max_ratings m 
  ON a.branch = m.branch AND a.avg_rating = m.max_rating;
```

---

### **Q3: Identify the busiest day for each branch based on the number of transactions**

```sql
WITH transactions AS (
    SELECT 
        branch,
        DAYNAME(STR_TO_DATE(date, '%d/%m/%Y')) AS day_name,
        COUNT(*) AS no_transactions
    FROM walmart
    GROUP BY branch, day_name
),
max_tx AS (
    SELECT 
        branch,
        MAX(no_transactions) AS max_tx
    FROM transactions
    GROUP BY branch
)
SELECT 
    t.branch,
    t.day_name,
    t.no_transactions
FROM transactions t
JOIN max_tx m ON t.branch = m.branch AND t.no_transactions = m.max_tx;
```

---

### **Q4: Calculate the total quantity of items sold per payment method**

```sql
SELECT 
    payment_method,
    SUM(quantity) AS no_qty_sold
FROM walmart
GROUP BY payment_method;
```

---

### **Q5: Determine the average, minimum, and maximum rating of categories for each city**

```sql
SELECT 
    city,
    category,
    MIN(rating) AS min_rating,
    MAX(rating) AS max_rating,
    AVG(rating) AS avg_rating
FROM walmart
GROUP BY city, category;
```

---

### **Q6: Calculate the total profit for each category**

```sql
SELECT 
    category,
    SUM(unit_price * quantity * profit_margin) AS total_profit
FROM walmart
GROUP BY category
ORDER BY total_profit DESC;
```

---

### **Q7: Determine the most common payment method for each branch**

```sql
WITH payment_counts AS (
    SELECT 
        branch,
        payment_method,
        COUNT(*) AS total_trans
    FROM walmart
    GROUP BY branch, payment_method
),
max_counts AS (
    SELECT 
        branch,
        MAX(total_trans) AS max_trans
    FROM payment_counts
    GROUP BY branch
)
SELECT 
    p.branch,
    p.payment_method AS preferred_payment_method
FROM payment_counts p
JOIN max_counts m 
  ON p.branch = m.branch AND p.total_trans = m.max_trans;
```

---

### **Q8: Categorize sales into Morning, Afternoon, and Evening shifts**

```sql
SELECT
    branch,
    CASE 
        WHEN HOUR(TIME(time)) < 12 THEN 'Morning'
        WHEN HOUR(TIME(time)) BETWEEN 12 AND 17 THEN 'Afternoon'
        ELSE 'Evening'
    END AS shift,
    COUNT(*) AS num_invoices
FROM walmart
GROUP BY branch, shift
ORDER BY branch, num_invoices DESC;
```

---

### **Q9: Identify the 5 branches with the highest revenue decrease ratio from last year to current year (e.g., 2022 to 2023)**

```sql
WITH yearly_revenue AS (
    SELECT 
        branch,
        YEAR(STR_TO_DATE(date, '%d/%m/%Y')) AS year,
        SUM(total) AS total_revenue
    FROM walmart
    WHERE YEAR(STR_TO_DATE(date, '%d/%m/%Y')) IN (2022, 2023)
    GROUP BY branch, year
),
revenue_diff AS (
    SELECT 
        r2023.branch,
        r2023.total_revenue AS revenue_current_year,
        r2022.total_revenue AS revenue_last_year,
        (r2022.total_revenue - r2023.total_revenue) / r2022.total_revenue AS revenue_decrease_ratio
    FROM yearly_revenue r2023
    JOIN yearly_revenue r2022 ON r2023.branch = r2022.branch AND r2023.year = 2023 AND r2022.year = 2022
)
SELECT 
    branch,
    revenue_decrease_ratio
FROM revenue_diff
ORDER BY revenue_decrease_ratio DESC
LIMIT 5;
```

---

### **Q10: Identify the top 3 most profitable products per branch**

```sql
WITH product_profit AS (
    SELECT 
        branch,
        category,
        unit_price * quantity * profit_margin AS profit
    FROM walmart
)
SELECT 
    branch,
    category,
    SUM(profit) AS total_profit
FROM product_profit
GROUP BY branch, category
ORDER BY total_profit DESC
LIMIT 3;
```

---

### **Q11: What time of day generates the highest revenue?**

```sql
WITH time_shift AS (
    SELECT 
        total,
        CASE
            WHEN HOUR(STR_TO_DATE(time, '%H:%i')) BETWEEN 6 AND 11 THEN 'Morning'
            WHEN HOUR(STR_TO_DATE(time, '%H:%i')) BETWEEN 12 AND 17 THEN 'Afternoon'
            WHEN HOUR(STR_TO_DATE(time, '%H:%i')) BETWEEN 18 AND 23 THEN 'Evening'
            ELSE 'Night'
        END AS time_of_day
    FROM walmart
)
SELECT 
    time_of_day,
    SUM(total) AS total_revenue
FROM time_shift
GROUP BY time_of_day
ORDER BY total_revenue DESC
LIMIT 1;
```

---

### **Q12: Which city has the highest average profit per transaction?**

```sql
SELECT 
    city,
    AVG(profit_margin * unit_price * quantity) AS avg_profit_per_transaction
FROM walmart
GROUP BY city
ORDER BY avg_profit_per_transaction DESC
LIMIT 1;
```

---

### **Q13: Determine the least performing category in terms of quantity sold**

```sql
SELECT 
    category,
    SUM(quantity) AS total_quantity_sold
FROM walmart
GROUP BY category
ORDER BY total_quantity_sold ASC
LIMIT 1;
```

---

### **Q14: Which branch has the highest average spending per customer?**

```sql
SELECT 
    branch,
    AVG(total) AS avg_spending_per_customer
FROM walmart
GROUP BY branch
ORDER BY avg_spending_per_customer DESC
LIMIT 1;
```

---

### **Q17: What are the top 3 categories with the highest profit margins overall?**

```sql
SELECT 
    category,
    AVG(profit_margin) AS avg_profit_margin
FROM walmart
GROUP BY category
ORDER BY avg_profit_margin DESC
LIMIT 3;
```

---

### **Q18: Calculate the average basket size (quantity per invoice) per branch**

```sql
WITH basket_size AS (
    SELECT 
        invoice_id,
        branch,
        SUM(quantity) AS total_quantity
    FROM walmart
    GROUP BY invoice_id, branch
)
SELECT 
    branch,
    AVG(total_quantity) AS avg_basket_size
FROM basket_size
GROUP BY branch;
```

---

### **Q22: What is the relationship between quantity sold and rating for each category?**

```sql
SELECT 
    category,
    AVG(rating) AS avg_rating,
    SUM(quantity) AS total_quantity_sold
FROM walmart
GROUP BY category
ORDER BY total_quantity_sold DESC;
```

---

### **Q23: Identify the top 5 best-selling products in terms of quantity and total revenue**

```sql
SELECT 
    category,
    SUM(quantity) AS total_quantity_sold,
    SUM(quantity * unit_price) AS total_revenue
FROM walmart
GROUP BY category
ORDER BY total_quantity_sold DESC
LIMIT 5;
```

---

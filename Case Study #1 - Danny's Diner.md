# SQL Challenge Case Study 1: Danny's Diner
![image](https://github.com/oanhphuong16/8weeksqlchallenge/assets/102425313/b9c56ec2-9529-4d43-9d6a-ad33da72457a)

## Introduction

This document presents my solutions for the SQL Challenge Case Study 1 [8 Week SQL Challenge](https://8weeksqlchallenge.com/case-study-1/), focusing on analyzing customer transactions at Danny's Diner. The goal is to derive insights into customer behavior and item popularity using SQL queries.

## Data Model
The data model for the challenge consists of three tables: `sales`, `menu`, and `members`. 
The data model for Danny's Diner is as follows:
![image](https://github.com/oanhphuong16/8weeksqlchallenge/assets/102425313/94dadd25-119b-4dfa-a176-5929f9ec42fc)

## Questions and Solutions

### Q1: What is the total amount each customer spent at the restaurant?

```sql
SELECT s.customer_id, SUM(m.price) AS total_spent
FROM dannys_diner.sales s
JOIN dannys_diner.menu m ON s.product_id = m.product_id
GROUP BY s.customer_id;
```
| customer_id | total_spent |
| ----------- | ----------- |
| B           | 74          |
| C           | 36          |
| A           | 76          |

### Q2: How many days has each customer visited the restaurant?

```sql
SELECT customer_id, COUNT(DISTINCT order_date) AS days_visited
FROM dannys_diner.sales
GROUP BY customer_id;
```
| customer_id | days_visited |
| ----------- | ------------ |
| A           | 4            |
| B           | 6            |
| C           | 2            |

### Q3: What was the first item from the menu purchased by each customer?

```sql
SELECT s.customer_id, m.product_name
FROM dannys_diner.sales s
JOIN dannys_diner.menu m ON s.product_id = m.product_id
WHERE (s.customer_id, s.order_date) IN (
  SELECT customer_id, MIN(order_date)
  FROM dannys_diner.sales
  GROUP BY customer_id
)
GROUP BY s.customer_id, m.product_name;
```
| customer_id | product_name |
| ----------- | ------------ |
| C           | ramen        |
| B           | curry        |
| A           | sushi        |
| A           | curry        |

### Q4: What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
SELECT m.product_name, COUNT(s.product_id) AS times_purchased
FROM dannys_diner.sales s
JOIN dannys_diner.menu m ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY times_purchased DESC
LIMIT 1;
```
| product_name | times_purchased |
| ------------ | --------------- |
| ramen        | 8               |

### Q5: Which item was the most popular for each customer?

```sql
SELECT s.customer_id, m.product_name, COUNT(s.product_id) AS times_purchased
FROM dannys_diner.sales s
JOIN dannys_diner.menu m ON s.product_id = m.product_id
GROUP BY s.customer_id, m.product_name
ORDER BY s.customer_id, times_purchased DESC;
```
| customer_id | product_name | times_purchased |
| ----------- | ------------ | --------------- |
| A           | ramen        | 3               |
| A           | curry        | 2               |
| A           | sushi        | 1               |
| B           | ramen        | 2               |
| B           | curry        | 2               |
| B           | sushi        | 2               |
| C           | ramen        | 3               |

### Q6: Which item was purchased first by the customer after they became a member?

```sql
SELECT s.customer_id, MIN(s.order_date) AS first_purchase_after_join, m.product_name
FROM dannys_diner.sales s
JOIN dannys_diner.members me ON s.customer_id = me.customer_id
JOIN dannys_diner.menu m ON s.product_id = m.product_id
WHERE s.order_date >= me.join_date
GROUP BY s.customer_id, m.product_name;
```
| customer_id | first_purchase_after_join | product_name |
| ----------- | ------------------------- | ------------ |
| B           | 2021-01-11T00:00:00.000Z  | sushi        |
| A           | 2021-01-07T00:00:00.000Z  | curry        |
| A           | 2021-01-10T00:00:00.000Z  | ramen        |
| B           | 2021-01-16T00:00:00.000Z  | ramen        |

### Q7: Which item was purchased just before the customer became a member?

```sql
SELECT s.customer_id, MAX(s.order_date) AS last_purchase_before_join, m.product_name
FROM dannys_diner.sales s
JOIN dannys_diner.members me ON s.customer_id = me.customer_id
JOIN dannys_diner.menu m ON s.product_id = m.product_id
WHERE s.order_date < me.join_date
GROUP BY s.customer_id, m.product_name;
```
| customer_id | last_purchase_before_join | product_name |
| ----------- | ------------------------- | ------------ |
| B           | 2021-01-04T00:00:00.000Z  | sushi        |
| A           | 2021-01-01T00:00:00.000Z  | curry        |
| B           | 2021-01-02T00:00:00.000Z  | curry        |
| A           | 2021-01-01T00:00:00.000Z  | sushi        |

### Q8: What is the total items and amount spent for each member before they became a member?

```sql
SELECT s.customer_id, COUNT(s.product_id) AS total_items, SUM(m.price) AS total_spent
FROM dannys_diner.sales s
JOIN dannys_diner.members me ON s.customer_id = me.customer_id
JOIN dannys_diner.menu m ON s.product_id = m.product_id
WHERE s.order_date < me.join_date
GROUP BY s.customer_id;
```
| customer_id | total_items | total_spent |
| ----------- | ----------- | ----------- |
| B           | 3           | 40          |
| A           | 2           | 25          |

### Q9: If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```sql
SELECT s.customer_id, 
       SUM(CASE WHEN m.product_name = 'sushi' THEN m.price * 20 ELSE m.price * 10 END) AS points
FROM dannys_diner.sales s
JOIN dannys_diner.menu m ON s.product_id = m.product_id
GROUP BY s.customer_id;
```
| customer_id | points |
| ----------- | ------ |
| B           | 940    |
| C           | 360    |
| A           | 860    |

### Q10: In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```sql
SELECT s.customer_id, 
       SUM(CASE 
             WHEN m.product_name = 'sushi' OR s.order_date BETWEEN me.join_date AND me.join_date + interval '6 days' THEN m.price * 20 
             ELSE m.price * 10 
           END) AS points
FROM dannys_diner.sales s
JOIN dannys_diner.menu m ON s.product_id = m.product_id
JOIN dannys_diner.members me ON s.customer_id = me.customer_id
WHERE s.order_date <= '2021-01-31'
GROUP BY s.customer_id;
```
| customer_id | points |
| ----------- | ------ |
| A           | 1370   |
| B           | 820    |

---
### bonus
```sql
    SELECT
      s.customer_id,
      s.order_date,
      m.product_name,
      m.price,
      CASE
        WHEN s.order_date >= me.join_date THEN 'Yes'
        ELSE 'No'
      END AS member,
      CASE
        WHEN s.order_date >= me.join_date THEN
          RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date, m.price DESC)
      END AS ranking
    FROM dannys_diner.sales s
    JOIN dannys_diner.menu m ON s.product_id = m.product_id
    LEFT JOIN dannys_diner.members me ON s.customer_id = me.customer_id
    ORDER BY s.customer_id, s.order_date;
```

| customer_id | order_date               | product_name | price | member | ranking |
| ----------- | ------------------------ | ------------ | ----- | ------ | ------- |
| A           | 2021-01-01T00:00:00.000Z | curry        | 15    | No     |         |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 10    | No     |         |
| A           | 2021-01-07T00:00:00.000Z | curry        | 15    | Yes    | 3       |
| A           | 2021-01-10T00:00:00.000Z | ramen        | 12    | Yes    | 4       |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Yes    | 5       |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Yes    | 5       |
| B           | 2021-01-01T00:00:00.000Z | curry        | 15    | No     |         |
| B           | 2021-01-02T00:00:00.000Z | curry        | 15    | No     |         |
| B           | 2021-01-04T00:00:00.000Z | sushi        | 10    | No     |         |
| B           | 2021-01-11T00:00:00.000Z | sushi        | 10    | Yes    | 4       |
| B           | 2021-01-16T00:00:00.000Z | ramen        | 12    | Yes    | 5       |
| B           | 2021-02-01T00:00:00.000Z | ramen        | 12    | Yes    | 6       |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | No     |         |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | No     |         |
| C           | 2021-01-07T00:00:00.000Z | ramen        | 12    | No     |         |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)

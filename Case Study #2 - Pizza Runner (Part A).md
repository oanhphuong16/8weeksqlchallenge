# SQL Challenge Case Study 2: Pizza Runner
![image](https://github.com/oanhphuong16/8weeksqlchallenge/assets/102425313/27ece36a-a568-43ab-937b-8e8bb75fcb37)

## Introduction

This document presents my solutions for the SQL Challenge Case Study 2: Pizza Runner.
https://8weeksqlchallenge.com/case-study-2/

The data model for the challenge:
![image](https://github.com/oanhphuong16/8weeksqlchallenge/assets/102425313/a47d3464-4c7e-48ab-b8d1-16c448b79d68)

## Questions and Solutions

### A. Pizza Metrics

**Question 1:** How many pizzas were ordered?

```sql
SELECT COUNT(*) AS pizzas_commandees
FROM pizza_runner.customer_orders;
```

| pizzas_commandees |
| ----------------- |
| 14                |

---

**Question 2:** How many unique customer orders were made?

```sql
SELECT COUNT(DISTINCT order_id) AS commandes_uniques
FROM pizza_runner.customer_orders;
```

| commandes_uniques |
| ----------------- |
| 10                |

---

**Question 3:** How many successful orders were delivered by each runner?

```sql
SELECT runner_id, COUNT(*) AS commandes_reussies
FROM pizza_runner.runner_orders
WHERE cancellation IS NULL OR cancellation = ''
GROUP BY runner_id;
```

| runner_id | commandes_reussies |
| --------- | ------------------ |
| 1         | 3                  |
| 2         | 1                  |
| 3         | 1                  |

---

**Question 4:** How many of each type of pizza was delivered?

```sql
SELECT co.pizza_id, pn.pizza_name, COUNT(*) AS nombre_livre
FROM pizza_runner.customer_orders co
JOIN pizza_runner.runner_orders ro ON co.order_id = ro.order_id
JOIN pizza_runner.pizza_names pn ON co.pizza_id = pn.pizza_id
WHERE ro.cancellation IS NULL OR ro.cancellation = ''
GROUP BY co.pizza_id, pn.pizza_name;
```

| pizza_id | pizza_name | nombre_livre |
| -------- | ---------- | ------------ |
| 1        | Meatlovers | 6            |
| 2        | Vegetarian | 2            |

---

**Question 5:** How many Vegetarian and Meatlovers were ordered by each customer?

```sql
SELECT co.customer_id, pn.pizza_name, COUNT(*) AS nombre_commande
FROM pizza_runner.customer_orders co
JOIN pizza_runner.pizza_names pn ON co.pizza_id = pn.pizza_id
WHERE pn.pizza_name IN ('Vegetarian', 'Meatlovers')
GROUP BY co.customer_id, pn.pizza_name;
```

| customer_id | pizza_name | nombre_commande |
| ----------- | ---------- | --------------- |
| 104         | Meatlovers | 3               |
| 101         | Meatlovers | 2               |
| 105         | Vegetarian | 1               |
| 103         | Meatlovers | 3               |
| 103         | Vegetarian | 1               |
| 102         | Meatlovers | 2               |
| 102         | Vegetarian | 1               |
| 101         | Vegetarian | 1               |
Continuing from where we left off, here's the completion of the Markdown document with the remaining questions and their results:

---
**Question 6:** What was the maximum number of pizzas delivered in a single order?

```sql
SELECT order_id, COUNT(*) AS nombre_pizzas
FROM pizza_runner.customer_orders
GROUP BY order_id
ORDER BY nombre_pizzas DESC
LIMIT 1;
```

| order_id | nombre_pizzas |
| -------- | ------------- |
| 4        | 3             |

---

**Question 7:** For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

```sql
SELECT 
  co.customer_id,
  SUM(CASE WHEN co.exclusions != '' OR co.extras != '' THEN 1 ELSE 0 END) AS pizzas_avec_changements,
  SUM(CASE WHEN co.exclusions = '' AND co.extras = '' THEN 1 ELSE 0 END) AS pizzas_sans_changements
FROM pizza_runner.customer_orders co
JOIN pizza_runner.runner_orders ro ON co.order_id = ro.order_id
WHERE ro.cancellation IS NULL OR ro.cancellation = ''
GROUP BY co.customer_id;
```

| customer_id | pizzas_avec_changements | pizzas_sans_changements |
| ----------- | ----------------------- | ----------------------- |
| 101         | 0                       | 2                       |
| 102         | 0                       | 1                       |
| 104         | 1                       | 0                       |
| 103         | 3                       | 0                       |

---

**Question 8:** How many pizzas were delivered that had both exclusions and extras?

```sql
SELECT COUNT(*) AS pizzas_avec_exclusions_et_extras
FROM pizza_runner.customer_orders co
JOIN pizza_runner.runner_orders ro ON co.order_id = ro.order_id
WHERE (co.exclusions != '' AND co.exclusions IS NOT NULL) 
  AND (co.extras != '' AND co.extras IS NOT NULL)
  AND (ro.cancellation IS NULL OR ro.cancellation = '');
```

| pizzas_avec_exclusions_et_extras |
| -------------------------------- |
| 1                                |

---

**Question 9:** What was the total volume of pizzas ordered for each hour of the day?

```sql
SELECT EXTRACT(HOUR FROM order_time) AS heure, COUNT(*) AS volume_commandes
FROM pizza_runner.customer_orders
GROUP BY heure
ORDER BY heure;
```

| heure | volume_commandes |
| ----- | ---------------- |
| 11    | 1                |
| 13    | 3                |
| 18    | 3                |
| 19    | 1                |
| 21    | 3                |
| 23    | 3                |

---

**Question 10:** What was the volume of orders for each day of the week?

```sql
SELECT TO_CHAR(order_time, 'Day') AS jour_semaine, COUNT(*) AS volume_commandes
FROM pizza_runner.customer_orders
GROUP BY jour_semaine
ORDER BY jour_semaine;
```

| jour_semaine | volume_commandes |
| ------------ | ---------------- |
| Friday       | 1                |
| Saturday     | 5                |
| Thursday     | 3                |
| Wednesday    | 5                |

## B. Runner and Customer Experience

![image](https://github.com/oanhphuong16/8weeksqlchallenge/assets/102425313/ae70e475-a6b2-4b6e-a8ed-aedcaca91ed8)

### Question #1: How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

```sql
SELECT
  DATE_TRUNC('week', registration_date::date) AS week_start,
  COUNT(runner_id) AS number_of_runners
FROM pizza_runner.runners
GROUP BY week_start
ORDER BY week_start;
```

| week_start               | number_of_runners |
| ------------------------ | ----------------- |
| 2020-12-28T00:00:00.000Z | 2                 |
| 2021-01-04T00:00:00.000Z | 1                 |
| 2021-01-11T00:00:00.000Z | 1                 |

---

### Question #2: What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pick up the order?

```sql
SELECT
  runner_id,
  AVG(EXTRACT(EPOCH FROM (CAST(pickup_time AS TIMESTAMP) - order_time))/60) AS avg_time_to_pickup
FROM pizza_runner.runner_orders
JOIN pizza_runner.customer_orders USING (order_id)
WHERE pickup_time != 'null'
GROUP BY runner_id;
```

| runner_id | avg_time_to_pickup |
| --------- | ------------------ |
| 3         | 10.47              |
| 2         | 23.72              |
| 1         | 15.68              |

---

### Question #3: Is there any relationship between the number of pizzas and how long the order takes to prepare?

```sql
WITH preparation_times AS (
    SELECT
        co.order_id,
        COUNT(co.order_id) AS number_of_pizzas,
        MIN(EXTRACT(EPOCH FROM (CAST(ro.pickup_time AS TIMESTAMP) - co.order_time))/60) AS prep_time_minutes 
    FROM pizza_runner.customer_orders co
    INNER JOIN pizza_runner.runner_orders ro ON co.order_id = ro.order_id
    WHERE ro.pickup_time != 'null' 
      AND ro.cancellation IS NULL OR ro.cancellation = '' 
    GROUP BY co.order_id
)
SELECT
    number_of_pizzas,
    AVG(prep_time_minutes) AS avg_prep_time_minutes
FROM preparation_times
GROUP BY number_of_pizzas
ORDER BY number_of_pizzas;
```

| number_of_pizzas | avg_prep_time_minutes |
| ---------------- | --------------------- |
| 1                | 10.34                 |
| 2                | 21.23                 |
| 3                | 29.28                 |

---

### Question #4: What was the average distance travelled for each customer?

```sql
SELECT
  customer_id,
  AVG(CAST(SPLIT_PART(distance, 'km', 1) AS NUMERIC)) AS avg_distance_km
FROM pizza_runner.runner_orders
JOIN pizza_runner.customer_orders USING (order_id)
WHERE distance != 'null' AND distance IS NOT NULL
GROUP BY customer_id;
```

| customer_id | avg_distance_km |
| ----------- | --------------- |
| 101         | 20.00           |
| 103         | 23.40           |
| 104         | 10.00           |
| 105         | 25.00           |
| 102         | 16.73           |

---

### Question #5: What was the difference between the longest and shortest delivery times for all orders?

```sql
WITH delivery_times AS (
  SELECT
    order_id,
    (EXTRACT(EPOCH FROM (CAST(pickup_time AS TIMESTAMP) - order_time))/60) AS delivery_time
  FROM pizza_runner.runner_orders
  JOIN pizza_runner.customer_orders USING (order_id)
  WHERE pickup_time != 'null'
)
SELECT MAX(delivery_time) - MIN(delivery_time) AS time_difference
FROM delivery_times;
```

| time_difference |
| --------------- |
| 19.25           |

---

### Question #6: What was the average speed for each runner for each delivery and do you notice any trend for these values?

```sql
SELECT
  ro.runner_id,
  ro.order_id,
  ROUND(
    (CAST(REGEXP_REPLACE(ro.distance, '[^\d.]+', '', 'g') AS NUMERIC) / 
    (CAST(REGEXP_REPLACE(ro.duration, '[^\d.]+', '', 'g') AS NUMERIC) / 60.0)),
    2
  ) AS avg_speed_km_per_hour
FROM
  pizza_runner.runner_orders ro
WHERE
  ro.pickup_time != 'null' AND
  ro.distance NOT LIKE '%null%' AND
  ro.duration NOT LIKE '%null%'
ORDER BY
  ro.runner_id, ro

.order_id;
```

| runner_id | order_id | avg_speed_km_per_hour |
| --------- | -------- | --------------------- |
| 1         | 1        | 37.50                 |
| 1         | 2        | 44.44                 |
| 1         | 3        | 40.20                 |
| 1         | 10       | 60.00                 |
| 2         | 4        | 35.10                 |
| 2         | 7        | 60.00                 |
| 2         | 8        | 93.60                 |
| 3         | 5        | 40.00                 |

---

### Question #7: What is the successful delivery percentage for each runner?

```sql
SELECT
  runner_id,
  COUNT(*) AS total_deliveries,
  COUNT(NULLIF(cancellation, '')) AS successful_deliveries,
  (COUNT(NULLIF(cancellation, ''))::FLOAT / COUNT(*)) * 100 AS success_percentage
FROM pizza_runner.runner_orders
GROUP BY runner_id;
```

| runner_id | total_deliveries | successful_deliveries | success_percentage |
| --------- | ---------------- | --------------------- | ------------------ |
| 3         | 2                | 1                     | 50.00              |
| 2         | 4                | 3                     | 75.00              |
| 1         | 4                | 4                     | 100.00             |


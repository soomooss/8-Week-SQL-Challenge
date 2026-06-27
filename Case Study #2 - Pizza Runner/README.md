# 🍕 Case Study #2 - Pizza Runner 
URL https://8weeksqlchallenge.com/case-study-2/

Este proyecto expande el uso de SQL abordando la limpieza de datos y métricas operativas de entrega para una pizzería.

## 🗂️ Índice de Secciones
* [🧼 Fase de Limpieza de Datos (Data Cleansing)](#-fase-de-limpieza-de-datos-data-cleansing)
* [📊 Sección A: Pizza Metrics](#-sección-a-pizza-metrics)
* [🏃‍♂️ Sección B: Runner and Customer Experience](#-sección-b-runner-and-customer-experience)
* [🍪 Sección C: Ingredient Optimisation](#-sección-c-ingredient-optimisation)
* [💰 Sección D: Pricing and Ratings](#-sección-d-pricing-and-ratings)
* [🚀 Sección E: Bonus Challenge](#-sección-e-bonus-challenge)

---

## 🧼 Fase de Limpieza de Datos (Data Cleansing)
```sq
CREATE TEMP TABLE clean_customer_orders AS
	SELECT
		order_id,
		customer_id,
		pizza_id,
		CASE 
			WHEN TRIM(exclusions) = '' OR LOWER(TRIM(exclusions)) = 'null' THEN NULL 
			ELSE exclusions 
		END AS exclusions,
		CASE 
			WHEN TRIM(extras) = '' OR LOWER(TRIM(extras)) = 'null' THEN NULL 
			ELSE extras 
		END AS extras,
		order_time
	FROM pizza_runner.customer_orders;
```
**Resultado:**
| order_id | customer_id | pizza_id | exclusions | extras | order_time          |
|----------|-------------|----------|------------|--------|---------------------|
| 1        | 101         | 1        |            |        | 2020-01-01 18:05:02 |
| 2        | 101         | 1        |            |        | 2020-01-01 19:00:52 |
| 3        | 102         | 1        |            |        | 2020-01-02 23:51:23 |
| 3        | 102         | 2        |            |        | 2020-01-02 23:51:23 |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4        | 103         | 2        | 4          |        | 2020-01-04 13:23:46 |
| 5        | 104         | 1        |            | 1      | 2020-01-08 21:00:29 |
| 6        | 101         | 2        |            |        | 2020-01-08 21:03:13 |
| 7        | 105         | 2        |            | 1      | 2020-01-08 21:20:29 |
| 8        | 102         | 1        |            |        | 2020-01-09 23:54:33 |
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10 11:22:59 |
| 10       | 104         | 1        |            |        | 2020-01-11 18:34:49 |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11 18:34:49 |


```sql
CREATE TEMP TABLE clean_runner_orders AS
	SELECT
		order_id,
		runner_id,
		CASE 
			WHEN LOWER(TRIM(pickup_time)) = 'null' OR TRIM(pickup_time) = '' THEN NULL 
			ELSE pickup_time 
		END::TIMESTAMP AS pickup_time,
		CASE 
			WHEN LOWER(TRIM(distance)) = 'null' OR TRIM(distance) = '' THEN NULL 
			ELSE TRIM(REPLACE(LOWER(distance),'km','')) 
		END::NUMERIC AS distance,
		CASE 
			WHEN LOWER(TRIM(duration)) = 'null' OR TRIM(duration) = '' THEN NULL 
			ELSE REGEXP_REPLACE(duration, '[^0-9]', '', 'g') 
		END::INTEGER AS duration,
		CASE 
			WHEN LOWER(TRIM(cancellation)) = 'null' OR TRIM(cancellation) =  '' THEN NULL 
			ELSE  cancellation 
		END AS cancellation
	FROM pizza_runner.runner_orders;
```
**Resultado:**
| order_id | runner_id | pickup_time         | distance | duration | cancellation            |
|----------|-----------|---------------------|----------|----------|-------------------------|
| 1        | 1         | 2020-01-01 18:15:34 | 20       | 32       |                         |
| 2        | 1         | 2020-01-01 19:10:54 | 20       | 27       |                         |
| 3        | 1         | 2020-01-03 00:12:37 | 13.4     | 20       |                         |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4     | 40       |                         |
| 5        | 3         | 2020-01-08 21:10:57 | 10       | 15       |                         |
| 6        | 3         |                     |          |          | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25       | 25       |                         |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4     | 15       |                         |
| 9        | 2         |                     |          |          | Customer Cancellation   |
| 10       | 1         | 2020-01-11 18:50:20 | 10       | 10       |                         |

---

## 📊 Sección A: Pizza Metrics

### 1. How many pizzas were ordered?
```sql
SELECT 
	COUNT(order_id) AS total_pizzas_ordered
FROM clean_customer_orders;
```
**Resultado:**

| total_pizzas_ordered |
|---|
| 14 |

### 2. How many unique customer orders were made?
```sql
SELECT 
	COUNT(DISTINCT order_id) AS customer_orders
FROM clean_customer_orders;
```
**Resultado:**
| customer_orders |
|-----------------|
| 10              |


### 3. How many successful orders were delivered by each runner?
```sql
SELECT 
	runner_id,
	COUNT(order_id) AS deliveries
FROM clean_runner_orders
WHERE cancellation IS NULL
GROUP BY runner_id;
```
**Resultado:**
| runner_id | deliveries |
|-----------|------------|
| 1         | 4          |
| 2         | 3          |
| 3         | 1          |


### 4. How many of each type of pizza was delivered?
```sql
SELECT 
	pn.pizza_name,
	COUNT(cco.pizza_id) AS number_of_pizzas
FROM clean_customer_orders AS cco
INNER JOIN pizza_runner.pizza_names AS pn
	ON cco.pizza_id = pn.pizza_id
INNER JOIN clean_runner_orders AS cro
	ON cco.order_id = cro.order_id
WHERE cro.cancellation IS NULL
GROUP BY pn.pizza_name;
```
**Resultado:**
| pizza_name | number_of_pizzas |
|------------|------------------|
| Vegetarian | 3                |
| Meatlovers | 9                |


### 5. How many Vegetarian and Meatlovers were ordered by each customer?
```sql
SELECT 
	cco.customer_id,
	pn.pizza_name,
	COUNT(cco.pizza_id) AS number_of_pizzas
FROM clean_customer_orders AS cco
INNER JOIN pizza_runner.pizza_names AS pn
	ON cco.pizza_id = pn.pizza_id
GROUP BY cco.customer_id, pn.pizza_name
ORDER BY cco.customer_id;
```
**Resultado:**
| customer_id | pizza_name | number_of_pizzas |
|-------------|------------|------------------|
| 101         | Meatlovers | 2                |
| 101         | Vegetarian | 1                |
| 102         | Meatlovers | 2                |
| 102         | Vegetarian | 1                |
| 103         | Meatlovers | 3                |
| 103         | Vegetarian | 1                |
| 104         | Meatlovers | 3                |
| 105         | Vegetarian | 1                |


### 6. What was the maximum number of pizzas delivered in a single order?
```sql
SELECT 
	cco.order_id,
	COUNT(cco.order_id) AS number_of_pizzas
FROM clean_customer_orders AS cco
INNER JOIN clean_runner_orders AS cro
	ON cco.order_id = cro.order_id
WHERE cro.cancellation IS NULL
GROUP BY cco.order_id
ORDER BY number_of_pizzas DESC
LIMIT 1;
```
**Resultado:**
| order_id | number_of_pizzas |
|----------|------------------|
| 4        | 3                |

### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```sql
SELECT 
	cco.customer_id,
	SUM( CASE WHEN cco.exclusions IS NULL AND cco.extras IS NULL THEN 1 ELSE 0 END) AS sin_cambios,
	SUM( CASE WHEN cco.exclusions IS NOT NULL OR cco.extras IS NOT NULL THEN 1 ELSE 0 END) AS con_cambios
FROM clean_customer_orders AS cco
INNER JOIN clean_runner_orders AS cro
	ON cco.order_id = cro.order_id
WHERE cro.cancellation IS NULL
GROUP BY cco.customer_id;
```
**Resultado:**
| customer_id | sin_cambios | con_cambios |
|-------------|-------------|-------------|
| 101         | 2           | 0           |
| 102         | 3           | 0           |
| 103         | 0           | 3           |
| 104         | 1           | 2           |
| 105         | 0           | 1           |



### 8. How many pizzas were delivered that had both exclusions and extras?
```sql
SELECT 
	COUNT(cco.pizza_id) AS pizzas_with_exclusions_and_extras
FROM clean_customer_orders AS cco
INNER JOIN pizza_runner.pizza_names AS pn
	ON cco.pizza_id = pn.pizza_id
INNER JOIN clean_runner_orders AS cro
	ON cco.order_id = cro.order_id
WHERE cro.cancellation IS NULL AND cco.exclusions IS NOT NULL AND cco.extras IS NOT NULL;
```
**Resultado:**
| pizzas_with_exclusions_and_extras |
|-----------------------------------|
| 1                                 |


### 9. What was the total volume of pizzas ordered for each hour of the day?
```sql
SELECT 
	order_time::DATE AS day,
	EXTRACT(HOUR FROM order_time) AS hour_of_the_day,
	COUNT(pizza_id) AS number_of_pizzas
FROM clean_customer_orders
GROUP BY order_time::DATE, EXTRACT(HOUR FROM order_time)
ORDER BY order_time::DATE,EXTRACT(HOUR FROM order_time);
```
**Resultado:**
| day        | hour_of_the_day | number_of_pizzas |
|------------|-----------------|------------------|
| 2020-01-01 | 18              | 1                |
| 2020-01-01 | 19              | 1                |
| 2020-01-02 | 23              | 2                |
| 2020-01-04 | 13              | 3                |
| 2020-01-08 | 21              | 3                |
| 2020-01-09 | 23              | 1                |
| 2020-01-10 | 11              | 1                |
| 2020-01-11 | 18              | 2                |

### 10. What was the volume of orders for each day of the week?
```sql
SELECT 
	TO_CHAR(order_time, 'Day') AS day_of_the_week,
	COUNT(pizza_id) AS total_pizzas_ordered
FROM clean_customer_orders
GROUP BY TO_CHAR(order_time, 'Day'),EXTRACT(DOW FROM order_time)
ORDER BY EXTRACT(DOW FROM order_time);
```
**Resultado:**
| day_of_the_week | total_pizzas_ordered |
|-----------------|----------------------|
| Wednesday       | 5                    |
| Thursday        | 3                    |
| Friday          | 1                    |
| Saturday        | 5                    |

---

## 🏃‍♂️ Sección B: Runner and Customer Experience
###
```sql
```
**Resultado:**
### 1. How many runners signed up for each 1 week period?
*(Aquí va tu consulta avanzada de la semana con el DATE_TRUNC)*

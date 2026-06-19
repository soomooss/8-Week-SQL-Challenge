# 🍜 Case Study #1 - Danny's Diner

Este proyecto forma parte del **8 Week SQL Challenge** de Danny Ma. Consiste en analizar el comportamiento de compra de los clientes de un restaurante japonés utilizando consultas avanzadas de SQL para ayudar al dueño a tomar decisiones de negocio.
URL: https://8weeksqlchallenge.com/case-study-1/

## 🛠️ Tecnologías y Habilidades
- **Base de Datos:** PostgreSQL
- **Conceptos Aplicados:** Window Functions (`DENSE_RANK`), Common Table Expressions (CTEs), Agregaciones condicionales (`CASE WHEN`), Joins complejos, Filtrado de fechas.

---

## 🚀 Preguntas y Soluciones

### 1. What is the total amount each customer spent at the restaurant?
```sql
SELECT 
	s.customer_id as client, 
	SUM(m.price) as amount_spent
FROM dannys_diner.sales AS s
LEFT JOIN dannys_diner.menu  AS m
	ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;
```
**Resultado:**
|  client | amount_spent   |
|:---:|:---:|
| A | 76 |
| B | 74 |
| C | 36 |

---

### 2. How many days has each customer visited the restaurant?
```sql
SELECT 
	customer_id,
	COUNT(DISTINCT order_date) AS days_visited
FROM dannys_diner.sales
GROUP BY customer_id;
```
**Resultado:**
|  customer_id | days_visited  |
|:---:|:---:|
| A | 4 |
| B | 6 |
| C | 2 |

---

### 3. What was the first item from the menu purchased by each customer?
```sql
WITH ctee AS (SELECT 
		customer_id,
		order_date,
		product_id,
		DENSE_RANK () OVER(
			PARTITION BY customer_id
			ORDER BY order_date
		) AS rank_order
FROM dannys_diner.sales)

SELECT 
	c.customer_id,
	c.order_date,
	m.product_name
FROM ctee AS c
LEFT JOIN dannys_diner.menu AS m
	ON c.product_id = m.product_id
WHERE c.rank_order = 1
GROUP BY c.customer_id,c.order_date,m.product_name;
```
**Resultado:**
|  customer_id |   oder_date         |  product_name     |
|:---:|:----------:|:-----:|
| A | 2021-01-01 | curry |
| A | 2021-01-01 | sushi |
| B | 2021-01-01 | curry |
| C | 2021-01-01 | ramen |
---

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
SELECT 
	s.product_id,
	m.product_name, 
	COUNT(s.product_id) AS number_of_times_sale
FROM dannys_diner.sales AS s
INNER JOIN dannys_diner.menu AS m
	ON s.product_id = m.product_id
GROUP BY s.product_id, m.product_name
ORDER BY number_of_times_sale DESC
LIMIT 1;
```
**Resultado:**
| product_id  | product_name      | number_of_times_sale  |
|:---:|:-----:|:---:|
| 3 | ramen | 8 |
---

### 5. Which item was the most popular for each customer?
```sql
WITH ctee AS (SELECT 
	customer_id,
	product_id,
	COUNT(product_id) AS number_sales,
	DENSE_RANK() OVER(
		PARTITION BY customer_id
		ORDER BY COUNT(product_id) DESC
	) AS product_rank
FROM dannys_diner.sales AS s
GROUP BY customer_id, product_id)

SELECT 
	c.customer_id,
	c.product_rank,
	c.product_id, 
	m.product_name
FROM ctee AS c
INNER JOIN dannys_diner.menu AS m
	ON c.product_id = m.product_id
WHERE c.product_rank = 1
ORDER BY c.customer_id;
```
**Resultado:**
| customer_id  | product_rank  | product_id  | product_name|
|:---:|:---:|:---:|:-----:|
| A | 1 | 3 | ramen |
| B | 1 | 1 | sushi |
| B | 1 | 2 | curry |
| B | 1 | 3 | ramen |
| C | 1 | 3 | ramen |
---

### 6. Which item was purchased first by the customer after they became a member?
```sql
WITH ctee AS (SELECT 
	mem.customer_id,
	mem.join_date,
	s.order_date,
	s.product_id,
	DENSE_RANK () OVER(
		PARTITION BY mem.customer_id
		ORDER BY order_date) 
	AS rank_num
FROM dannys_diner.members AS mem
LEFT JOIN dannys_diner.sales AS s
	ON mem.customer_id = s.customer_id
WHERE mem.join_date <= s.order_date)

SELECT 
	c.customer_id,
	c.join_date,
	c.order_date,
	c.product_id,
	m.product_name
FROM ctee AS c
INNER JOIN dannys_diner.menu AS m
	ON c.product_id = m.product_id
WHERE rank_num = 1;
```
**Resultado:**
| customer_id  |     join_date       | order_date | product_id  | product_name      |
|:---:|:----------:|:----------:|:---:|:-----:|
| B | 2021-01-09 | 2021-01-11 | 1 | sushi |
| A | 2021-01-07 | 2021-01-07 | 2 | curry |

---

### 7. Which item was purchased just before the customer became a member?
```sql
WITH ctee AS (SELECT 
	mem.customer_id,
	mem.join_date,
	s.order_date,
	s.product_id,
	DENSE_RANK () OVER(
		PARTITION BY mem.customer_id
		ORDER BY order_date DESC) 
	AS rank_num
FROM dannys_diner.members AS mem
LEFT JOIN dannys_diner.sales AS s
	ON mem.customer_id = s.customer_id
WHERE mem.join_date > s.order_date)

SELECT 
	c.customer_id,
	c.join_date,
	c.order_date,
	c.product_id,
	m.product_name
FROM ctee AS c
INNER JOIN dannys_diner.menu AS m
	ON c.product_id = m.product_id
WHERE rank_num = 1
ORDER BY c.customer_id;
```
**Resultado:**
| customer_id | join_date  | order_date | product_id | product_name |
|:-----------:|:----------:|:----------:|:----------:|:------------:|
| A           | 2021-01-07 | 2021-01-01 | 1          | sushi        |
| A           | 2021-01-07 | 2021-01-01 | 2          | curry        |
| B           | 2021-01-09 | 2021-01-04 | 1          | sushi        |


---

### 8. What is the total items and amount spent for each member before they became a member?
```sql
SELECT 
	mem.customer_id,
	COUNT(s.product_id) AS total_items,
	SUM(m.price) AS total_spent
FROM dannys_diner.members AS mem
LEFT JOIN dannys_diner.sales AS s
	ON mem.customer_id = s.customer_id
INNER JOIN dannys_diner.menu AS m
	ON s.product_id = m.product_id
WHERE mem.join_date > s.order_date
GROUP BY mem.customer_id;
```
**Resultado:**
| customer_id | total_items | total_spent |
|:-----------:|:-----------:|:-----------:|
| B           | 3           | 40          |
| A           | 2           | 25          |


---

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```sql
SELECT 
	s.customer_id,
	SUM(CASE WHEN m.product_name = 'sushi' THEN m.price * 20 ELSE m.price * 10 END) AS points
FROM dannys_diner.sales AS s
INNER JOIN dannys_diner.menu AS m
	ON s.product_id = m.product_id
GROUP BY 1
ORDER BY 1;
```
**Resultado:**
| customer_id | points |
|:-----------:|:------:|
| A           | 860    |
| B           | 940    |
| C           | 360    |
---

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```sql
SELECT 
	mem.customer_id,
	SUM(
		CASE WHEN s.order_date >= mem.join_date AND s.order_date <= mem.join_date + 6 THEN m.price * 20 
		WHEN m.product_name = 'sushi' THEN m.price * 20 
		ELSE m.price * 10 END) AS january_points
FROM dannys_diner.members AS mem
LEFT JOIN dannys_diner.sales AS s
	ON mem.customer_id = s.customer_id
INNER JOIN dannys_diner.menu AS m
	ON s.product_id = m.product_id
WHERE order_date <= '2021-01-31'
GROUP BY mem.customer_id
ORDER BY 1;
```
**Resultado:**
| customer_id | january_points |
|:-----------:|:--------------:|
| A           | 1370           |
| B           | 820            |


---

## 💎Bonus Challenge: 

### Join All The Things
```sql
SELECT 
	s.customer_id,
	s.order_date,
	m.product_name,
	m.price,
	CASE 
		WHEN s.order_date >= mem.join_date AND mem.join_date IS NOT NULL THEN 'Y' ELSE 'N' END AS member
FROM dannys_diner.sales AS s
INNER JOIN dannys_diner.menu AS m
	ON s.product_id = m.product_id
LEFT JOIN dannys_diner.members AS mem
	ON s.customer_id = mem.customer_id;
```
**Resultado:**
| customer_id | order_date | product_name | price | member |
|:-----------:|:----------:|:------------:|:-----:|:------:|
| A           | 2021-01-07 | curry        | 15    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| A           | 2021-01-10 | ramen        | 12    | Y      |
| A           | 2021-01-01 | sushi        | 10    | N      |
| A           | 2021-01-01 | curry        | 15    | N      |
| B           | 2021-01-04 | sushi        | 10    | N      |
| B           | 2021-01-11 | sushi        | 10    | Y      |
| B           | 2021-01-01 | curry        | 15    | N      |
| B           | 2021-01-02 | curry        | 15    | N      |
| B           | 2021-01-16 | ramen        | 12    | Y      |
| B           | 2021-02-01 | ramen        | 12    | Y      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-07 | ramen        | 12    | N      |
---
### Rank All The Things
```sql
WITH ctee AS (SELECT 
	s.customer_id,
	s.order_date,
	m.product_name,
	m.price,
	CASE 
		WHEN s.order_date >= mem.join_date AND mem.join_date IS NOT NULL THEN 'Y' ELSE 'N' END AS member
FROM dannys_diner.sales AS s
INNER JOIN dannys_diner.menu AS m
	ON s.product_id = m.product_id
LEFT JOIN dannys_diner.members AS mem
	ON s.customer_id = mem.customer_id)

SELECT 
*,
	CASE WHEN member = 'Y' 
		THEN 
			DENSE_RANK() OVER(
				PARTITION BY customer_id,member
				ORDER BY order_date
			)
		ELSE NULL
		END AS ranking
FROM ctee;
```
**Resultado:**

| customer_id | order_date | product_name | price | member | ranking |
|:-----------:|:----------:|:------------:|:-----:|:------:|:-------:|
| A           | 2021-01-01 | sushi        | 10    | N      |  NULL   |
| A           | 2021-01-01 | curry        | 15    | N      |  NULL   |
| A           | 2021-01-07 | curry        | 15    | Y      | 1       |
| A           | 2021-01-10 | ramen        | 12    | Y      | 2       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| B           | 2021-01-01 | curry        | 15    | N      |  NULL   |
| B           | 2021-01-02 | curry        | 15    | N      |  NULL   |
| B           | 2021-01-04 | sushi        | 10    | N      |  NULL   |
| B           | 2021-01-11 | sushi        | 10    | Y      | 1       |
| B           | 2021-01-16 | ramen        | 12    | Y      | 2       |
| B           | 2021-02-01 | ramen        | 12    | Y      | 3       |
| C           | 2021-01-01 | ramen        | 12    | N      |  NULL   |
| C           | 2021-01-01 | ramen        | 12    | N      |  NULL   |
| C           | 2021-01-07 | ramen        | 12    | N      |  NULL   |

# 🍕 Case Study #2 - Pizza Runner

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
*(Aquí pegas tus dos mega-consultas donde creaste las tablas temporales limpias)*

---

## 📊 Sección A: Pizza Metrics

### 1. How many pizzas were ordered?
```sql
SELECT COUNT(*) AS total_pizzas_ordered FROM clean_customer_orders;
```
**Resultado:**

| total_pizzas_ordered |
|:---:|
| 14 |

---

## 🏃‍♂️ Sección B: Runner and Customer Experience

### 1. How many runners signed up for each 1 week period?
*(Aquí va tu consulta avanzada de la semana con el DATE_TRUNC)*

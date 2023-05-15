# Case Study #1: Danny's Diner
Feel free to test these queries out [here.](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)

## Table of Contents
- [1. What is the total amount each customer spent at the restaurant?](#1-what-is-the-total-amount-each-customer-spent-at-the-restaurant)
- [2. How many days has each customer visited the restaurant?](#2-how-many-days-has-each-customer-visited-the-restaurant)
- [3. What was the first item from the menu purchased by each customer?](#3-what-was-the-first-item-from-the-menu-purchased-by-each-customer)
- [4. What is the most purchased item on the menu and how many times was it purchased by all customers?](#4-what-is-the-most-purchased-item-on-the-menu-and-how-many-times-was-it-purchased-by-all-customers)
- [5. Which item was the most popular for each customer?](#5-which-item-was-the-most-popular-for-each-customer)
- [6. Which item was purchased first by the customer after they became a member?](#6-which-item-was-purchased-first-by-the-customer-after-they-became-a-member)
- [7. Which item was purchased just before the customer became a member?](#7-which-item-was-purchased-just-before-the-customer-became-a-member)
- [8. What is the total items and amount spent for each member before they became a member?](#8-what-is-the-total-items-and-amount-spent-for-each-member-before-they-became-a-member)
- [9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?](#9-if-each-1-spent-equates-to-10-points-and-sushi-has-a-2x-points-multiplier---how-many-points-would-each-customer-have)
- [10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?](#10-in-the-first-week-after-a-customer-joins-the-program-including-their-join-date-they-earn-2x-points-on-all-items-not-just-sushi---how-many-points-do-customer-a-and-b-have-at-the-end-of-january)
- [Bonus Questions](#bonus-questions)

## 1. What is the total amount each customer spent at the restaurant?
### SQL Code
```sql
SELECT
    sales.customer_id customer_id,
    SUM(menu.price) total_sales
FROM dannys_diner.sales
JOIN dannys_diner.menu
ON sales.product_id=menu.product_id
GROUP BY 1
ORDER BY 1 ;
```

### Results

| customer_id | total_sales |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

We can see that customer A spent $76 which was the most out these customers and customer C spent $36 which was the least.


## 2. How many days has each customer visited the restaurant?

### SQL Code
```sql
SELECT
    sales.customer_id customer_id,
    COUNT (DISTINCT sales.order_date) days_visited
FROM dannys_diner.sales
GROUP BY 1
ORDER BY 1 ;
```

### Results
| customer_id | days_visited |
| ----------- | ------------ |
| A           | 4            |
| B           | 6            |
| C           | 2            |

We can see that customer B visited the most days. It would be interesting to see who visited the most times altogether as some customers might visit multiple times a day.

## 3. What was the first item from the menu purchased by each customer?

### SQL Code
```sql
WITH cte_sales AS (
  SELECT 
    ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date) AS row_number,
    *
  FROM dannys_diner.sales
)
SELECT 
	cte_sales.customer_id,
	menu.product_name
FROM cte_sales
JOIN dannys_diner.menu
ON cte_sales.product_id=menu.product_id
WHERE cte_sales.row_number = 1;
```

### Results
| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

We can see that each customer bought a different meal firt. Unfortunately we don't have a time stamp on the date the customers bought these meals so we have to assume that the the rows of data were recorded chronologically.

## 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

### SQL Code
```sql
SELECT 
	menu.product_name,
	COUNT(sales.product_id)	
FROM dannys_diner.sales
JOIN dannys_diner.menu
ON sales.product_id=menu.product_id
GROUP BY 1
ORDER BY 2 desc
LIMIT 1;
```

### Results

| product_name | count |
| ------------ | ----- |
| ramen        | 8     |

Ramen is the most ordered product, being ordered a total of 8 times.


## 5. Which item was the most popular for each customer?

### SQL Code
```sql
WITH cte_sales AS (
SELECT 
	sales.customer_id,
  	sales.product_id,
	COUNT(sales.product_id) AS count,
    DENSE_RANK() OVER (PARTITION BY sales.customer_id ORDER BY COUNT(sales.product_id) DESC) AS count_rank
FROM dannys_diner.sales
GROUP BY sales.customer_id, sales.product_id)
  
SELECT 
	cte_sales.customer_id,
  	menu.product_name,
  	cte_sales.count
FROM cte_sales
JOIN dannys_diner.menu
ON cte_sales.product_id=menu.product_id
WHERE cte_sales.Count_rank = 1
ORDER BY 1
```

### Results

| customer_id | product_name | count |
| ----------- | ------------ | ----- |
| A           | ramen        | 3     |
| B           | curry        | 2     |
| B           | ramen        | 2     |
| B           | sushi        | 2     |
| C           | ramen        | 3     |

Customers A and C both ordered ramen the most whereas customer B ordered all three options twice.


## 6. Which item was purchased first by the customer after they became a member?

### SQL Code 
```sql
WITH cte_ranking AS (
  SELECT 
    DENSE_RANK() OVER (PARTITION BY members.customer_id ORDER BY sales.order_date) AS row_number,
    members.customer_id,
    menu.product_name
  FROM dannys_diner.menu
  JOIN dannys_diner.sales 
    ON sales.product_id = menu.product_id
  JOIN dannys_diner.members 
    ON members.customer_id = sales.customer_id
  WHERE sales.order_date > members.join_date
)
SELECT 
  cte_ranking.customer_id,
  cte_ranking.product_name
FROM cte_ranking
WHERE cte_ranking.row_number = 1;
```

### Results

| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | sushi        |

We can see that customer A bought ramen first after being a member and customer B bought sushi. Customer C doesnt yet appear on the list because they aren't a member yet.


## 7. Which item was purchased just before the customer became a member?

### SQL Code
```sql
WITH cte_ranking AS (
  SELECT 
    DENSE_RANK() OVER (PARTITION BY members.customer_id ORDER BY sales.order_date desc) AS row_number,
    members.customer_id,
    menu.product_name
  FROM dannys_diner.menu
  JOIN dannys_diner.sales 
    ON sales.product_id = menu.product_id
  JOIN dannys_diner.members 
    ON members.customer_id = sales.customer_id
  WHERE sales.order_date < members.join_date
)
SELECT 
  cte_ranking.customer_id,
  cte_ranking.product_name
FROM cte_ranking
WHERE cte_ranking.row_number = 1;
```
### Results

| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| A           | curry        |
| B           | sushi        |

Before becoming a member both customers bought sushi. Customer A also bought a curry on the last day he bought something before becoming a member. We cant be sure if customer A bought sushi and curry at the same time or different times the last they bought something before becoming a member.

## 8. What is the total items and amount spent for each member before they became a member?

### SQL Code
```sql
SELECT 
  members.customer_id,
  COUNT(sales.product_id) total_items,
  SUM(menu.price) amount_spent
FROM dannys_diner.menu
JOIN dannys_diner.sales 
  ON sales.product_id = menu.product_id
JOIN dannys_diner.members 
  ON members.customer_id = sales.customer_id
WHERE sales.order_date < members.join_date
GROUP BY 1
ORDER BY 1;
```

### Results

| customer_id | total_items | amount_spent |
| ----------- | ----------- | ------------ |
| A           | 2           | 25           |
| B           | 3           | 40           |

Customer A bought 2 items totaling $25 and customer B bought 3 items totaling $40. Customer C will be added to the table once them become a member.

## 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

### SQL Code
```sql
WITH cte_points AS (
SELECT 
    *,
    CASE
        WHEN 
            sales.product_id = 1 
         THEN menu.price * 20
         ELSE menu.price * 10
    END AS points
FROM dannys_diner.menu
JOIN dannys_diner.sales 
    ON sales.product_id = menu.product_id)

SELECT 
  cte_points.customer_id,
  SUM(cte_points.points) points
FROM cte_points
GROUP BY 1
Order by 1
```

### Results

| customer_id | points |
| ----------- | ------ |
| A           | 860    |
| B           | 940    |
| C           | 360    |

If these customers were all in the scheme from the start customer A would have 860 points, B would have 940 points and C would have 360 points.


## 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

### SQL Code

```sql
WITH cte_points AS (
SELECT 
    members.customer_id,
    CASE
        WHEN 
            sales.product_id = 1 
            OR sales.order_date < members.join_date + INTERVAL '6 days'  
         THEN menu.price * 20
         ELSE menu.price * 10
    END AS points
FROM dannys_diner.menu
JOIN dannys_diner.sales 
    ON sales.product_id = menu.product_id
JOIN dannys_diner.members 
    ON members.customer_id = sales.customer_id
WHERE sales.order_date >= members.join_date)

SELECT 
  cte_points.customer_id,
  SUM(cte_points.points) points
FROM cte_points
GROUP BY 1
ORDER BY 1;
```

### Results

| customer_id | points |
| ----------- | ------ |
| A           | 1020   |
| B           | 440    |

We can see that customer A has 1020 points and customer B had 440 points.

## Bonus Questions

### SQL Code

```sql
SELECT 
    sales.customer_id,
    TO_CHAR(sales.order_date, 'YYYY-MM-DD') order_date,
    menu.product_name,
    menu.price,
    CASE
      WHEN 
        sales.order_date >= members.join_date
      THEN 'Y'
      ELSE 'N'
    END AS member
FROM dannys_diner.menu
FULL OUTER JOIN dannys_diner.sales 
    ON sales.product_id = menu.product_id
FULL OUTER JOIN dannys_diner.members 
    ON members.customer_id = sales.customer_id
ORDER BY 1, 2
```
### Result

| customer_id | order_date | product_name | price | member |
| ----------- | ---------- | ------------ | ----- | ------ |
| A           | 2021-01-01 | sushi        | 10    | N      |
| A           | 2021-01-01 | curry        | 15    | N      |
| A           | 2021-01-07 | curry        | 15    | Y      |
| A           | 2021-01-10 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| B           | 2021-01-01 | curry        | 15    | N      |
| B           | 2021-01-02 | curry        | 15    | N      |
| B           | 2021-01-04 | sushi        | 10    | N      |
| B           | 2021-01-11 | sushi        | 10    | Y      |
| B           | 2021-01-16 | ramen        | 12    | Y      |
| B           | 2021-02-01 | ramen        | 12    | Y      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-07 | ramen        | 12    | N      |

### SQL Code
```sql
WITH cte_ranking AS (
SELECT 
    sales.customer_id,
    TO_CHAR(sales.order_date, 'YYYY-MM-DD') AS order_date,
    menu.product_name,
    menu.price,
    CASE
        WHEN sales.order_date >= members.join_date THEN 'Y'
        ELSE 'N'
    END AS member  
FROM dannys_diner.menu
LEFT JOIN dannys_diner.sales 
    ON sales.product_id = menu.product_id
LEFT JOIN dannys_diner.members 
    ON members.customer_id = sales.customer_id
ORDER BY 1, 2)

SELECT
	*,
	CASE
    WHEN member = 'N' 
   		THEN null 
    	ELSE RANK() OVER (PARTITION BY customer_id, member ORDER BY order_date)
    END AS ranking
FROM cte_ranking;

```

### Result

| customer_id | order_date | product_name | price | member | ranking |
| ----------- | ---------- | ------------ | ----- | ------ | ------- |
| A           | 2021-01-01 | sushi        | 10    | N      |         |
| A           | 2021-01-01 | curry        | 15    | N      |         |
| A           | 2021-01-07 | curry        | 15    | Y      | 1       |
| A           | 2021-01-10 | ramen        | 12    | Y      | 2       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| B           | 2021-01-01 | curry        | 15    | N      |         |
| B           | 2021-01-02 | curry        | 15    | N      |         |
| B           | 2021-01-04 | sushi        | 10    | N      |         |
| B           | 2021-01-11 | sushi        | 10    | Y      | 1       |
| B           | 2021-01-16 | ramen        | 12    | Y      | 2       |
| B           | 2021-02-01 | ramen        | 12    | Y      | 3       |
| C           | 2021-01-01 | ramen        | 12    | N      |         |
| C           | 2021-01-01 | ramen        | 12    | N      |         |
| C           | 2021-01-07 | ramen        | 12    | N      |         |





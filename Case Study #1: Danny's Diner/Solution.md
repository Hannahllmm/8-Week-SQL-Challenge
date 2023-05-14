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

### Result

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

## 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

## 5. Which item was the most popular for each customer?

## 6. Which item was purchased first by the customer after they became a member?

## 7. Which item was purchased just before the customer became a member?

## 8. What is the total items and amount spent for each member before they became a member?

## 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

## 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

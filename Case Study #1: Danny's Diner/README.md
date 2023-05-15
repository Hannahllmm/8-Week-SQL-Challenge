# Case Study #1: Danny's Diner


## Table of Contents

- [Introduction](#introduction)
- [Problem Statement](#problem-statement)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Example Datasets](#example-datasets)
- [Case Study Questions](#case-study-questions)
- [Bonus Questions](#bonus-questions)
- [Solutions](Solution.md)


## Introduction
Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

<img src="https://github.com/Hannahllmm/8weeksqlchallenge/assets/39679731/d846f7d0-2b9e-43ec-992b-18be79f43fb9" alt="Image" width="600" height="620">

## Problem Statement
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers

## Entity Relationship Diagram
![image](https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/0e252464-7c73-46c1-823e-e0332dc77f0d)

## Example Datasets
All datasets exist within the dannys_diner database schema - be sure to include this reference within your SQL scripts as you start exploring the data and answering the case study questions.

### Table 1: sales
The sales table captures all customer_id level purchases with an corresponding order_date and product_id information for when and what menu items were ordered.

![image](https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/6f7bfb1e-cf4d-4d57-96f0-0dba138ebbac)

### Table 2: menu
The menu table maps the product_id to the actual product_name and price of each menu item.
![image](https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/044a46c7-ed73-4a6b-83c5-58768088ac26)

### Table 3: members
The final members table captures the join_date when a customer_id joined the beta version of the Danny’s Diner loyalty program.
![image](https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/44f48452-b6ec-4ab0-91c0-545f01406ba4)

## Case Study Questions
1. What is the total amount each customer spent at the restaurant?
2. How many days has each customer visited the restaurant?
3. What was the first item from the menu purchased by each customer?
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
5. Which item was the most popular for each customer?
6. Which item was purchased first by the customer after they became a member?
7. Which item was purchased just before the customer became a member?
8. What is the total items and amount spent for each member before they became a member?
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

## Bonus Questions

Recreate the following table output using the available data:

![image](https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/e8d25c36-1d44-4c76-b542-057f4af4e8cb)

Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program. Recreate this table:

![image](https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/5aa30e31-4c74-4ea1-85e5-04409c76318d)


## Solutions

My solutions can be found [here](Solution.md).

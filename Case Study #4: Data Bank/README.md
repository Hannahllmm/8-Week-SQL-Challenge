# Case Study #4: Data Bank


## Table of Contents

- [Introduction](#introduction)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Data](#data)
- [Case Study Questions](#case-study-questions)
- [Solutions](Solution.md)


## Introduction
There is a new innovation in the financial industry called Neo-Banks: new aged digital only banks without physical branches.

Danny thought that there should be some sort of intersection between these new age banks, cryptocurrency and the data world… so he decides to launch a new initiative - Data Bank!

Data Bank runs just like any other digital bank - but it isn’t only for banking activities, they also have the world’s most secure distributed data storage platform!

Customers are allocated cloud data storage limits which are directly linked to how much money they have in their accounts. 

This case study is all about calculating metrics, growth and helping the business analyse their data in a smart way to better forecast and plan for their future developments!

<img src="https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/fcd0a331-00aa-4500-9649-88af6770ff4a" alt="Image" width="600" height="620">

## Entity Relationship Diagram
![image](https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/7501d966-7271-4ef7-929b-27fd109284d6)

## Data
### Table 1: Regions
Just like popular cryptocurrency platforms - Data Bank is also run off a network of nodes where both money and data is stored across the globe. In a traditional banking sense - you can think of these nodes as bank branches or stores that exist around the world.

<img src="https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/ca8fdfd2-bb7c-418b-9b4d-6e06ff9893c4" alt="Image" width="200">

### Table 2: Customer Nodes

Customers are randomly distributed across the nodes according to their region - this also specifies exactly which node contains both their cash and data.

This random distribution changes frequently to reduce the risk of hackers getting into Data Bank’s system and stealing customer’s money and data!

Below is a sample of the top 10 rows of the data_bank.customer_nodes

<img src="https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/01554664-17ef-4b2b-b1f0-07de3ba7b498" alt="Image" width="400">

### Table 3: Customer Transactions

This table stores all customer deposits, withdrawals and purchases made using their Data Bank debit card.

<img src="https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/b59c4be3-6375-46ba-80dc-a4c6b733f265" alt="Image" width="400">

## Case Study Questions
### A. Customer Nodes Exploration
1. How many unique nodes are there on the Data Bank system?
2. What is the number of nodes per region?
3. How many customers are allocated to each region?
4. How many days on average are customers reallocated to a different node?
5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?

### B. Customer Transactions
1. What is the unique count and total amount for each transaction type?
2. What is the average total historical deposit counts and amounts for all customers?
3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
4. What is the closing balance for each customer at the end of the month?
5. Comparing the closing balance of a customer’s first month and the closing balance from their second nth, what percentage of customers:
* Have a negative first month balance?
* Have a positive first month balance?
* Increase their opening month’s positive closing balance by more than 5% in the following month?
* Reduce their opening month’s positive closing balance by more than 5% in the following month?
* Move from a positive balance in the first month to a negative balance in the second month?

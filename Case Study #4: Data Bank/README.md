# Case Study #3: Foodie-Fi


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
5. What is the percentage of customers who increase their closing balance by more than 5%?

### C. Data Allocation Challenge
To test out a few different hypotheses - the Data Bank team wants to run an experiment where different groups of customers would be allocated data using 3 different options:

* Option 1: data is allocated based off the amount of money at the end of the previous month
* Option 2: data is allocated on the average amount of money kept in the account in the previous 30 days
* Option 3: data is updated real-time

For this multi-part challenge question - you have been requested to generate the following data elements to help the Data Bank team estimate how much data will need to be provisioned for each option:

* running customer balance column that includes the impact each transaction
* customer balance at the end of each month
* minimum, average and maximum values of the running balance for each customer

Using all of the data available - how much data would have been required for each option on a monthly basis?

### D. Extra Challenge
Data Bank wants to try another option which is a bit more difficult to implement - they want to calculate data growth using an interest calculation, just like in a traditional savings account you might have with a bank.

If the annual interest rate is set at 6% and the Data Bank team wants to reward its customers by increasing their data allocation based off the interest calculated on a daily basis at the end of each day, how much data would be required for this option on a monthly basis?

### Extension

The Data Bank team wants you to use the outputs generated from the above sections to create a quick Powerpoint presentation which will be used as marketing materials for both external investors who might want to buy Data Bank shares and new prospective customers who might want to bank with Data Bank.

1. Using the outputs generated from the customer node questions, generate a few headline insights which Data Bank might use to market it’s world-leading security features to potential investors and customers.
2. With the transaction analysis - prepare a 1 page presentation slide which contains all the relevant information about the various options for the data provisioning so the Data Bank management team can make an informed decision.

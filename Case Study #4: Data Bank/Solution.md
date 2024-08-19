# Case Study #5: Data Bank
Feel free to test these queries out [here.](https://www.db-fiddle.com/f/2GtQz4wZtuNNu7zXH5HtV4/3)

## Table of Contents
- [A. Customer Nodes Exploration](#a-customer-nodes-exploration)
- [B. Customer Transactions](#b-customer-transactions)
- [C. Data Allocation Challenge](#c-data-allocation-challenge)
- [D. Extra Challenge](#d-extra-challenge)
- [Extension](#extension)

## A. Customer Nodes Exploration
### How many unique nodes are there on the Data Bank system?
This isn't as simple as doing a distinct count on the node_id because there are different nodes in different regions. To answer this question we need to count the distinct nodes and regions recorded.
#### Solution
```sql
WITH combinations AS (
    SELECT DISTINCT node_id, region_id
    FROM data_bank.customer_nodes
)
SELECT COUNT(*)
FROM combinations;
```
![image](https://github.com/user-attachments/assets/f1fa7069-df46-474a-b58b-efff70d01b17)

There are 25 unique nodes.

### What is the number of nodes per region?
#### Solution
```sql
SELECT 
	region_id,
	COUNT(DISTINCT node_id) node_count
FROM data_bank.customer_nodes
GROUP BY region_id;
```
![image](https://github.com/user-attachments/assets/ba88d0c1-13da-44e7-8bfe-dbaffbec1c7b)

There are 5 nodes in each region.

### How many customers are allocated to each region?
#### Solution
```sql
SELECT 
	r.region_name,
	COUNT(DISTINCT cn.customer_id) node_count
FROM data_bank.customer_nodes cn
INNER JOIN data_bank.regions r
	ON cn.region_id = r.region_id
GROUP BY r.region_name;
```
![image](https://github.com/user-attachments/assets/0bbf10de-e149-40e1-bfd0-6b12f7b039c7)


### How many days on average are customers reallocated to a different node?


### What is the median, 80th and 95th percentile for this same reallocation days metric for each region?


### B. Customer Transactions
### C. Data Allocation Challenge
### D. Extra Challenge
### Extension


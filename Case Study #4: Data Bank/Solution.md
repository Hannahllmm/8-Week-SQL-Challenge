# Case Study #4: Data Bank
Feel free to test these queries out [here.](https://www.db-fiddle.com/f/2GtQz4wZtuNNu7zXH5HtV4/3)

## Table of Contents
- [A. Customer Nodes Exploration](#a-customer-nodes-exploration)
- [B. Customer Transactions](#b-customer-transactions)

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
This isn't as easy as it first seems. Below looks at the journey of customer 1 as an example.
```sql
    SELECT 
    	*
    FROM data_bank.customer_nodes
    WHERE customer_id = 1;
```

| customer_id | region_id | node_id | start_date               | end_date                 |
|-------------|-----------|---------|--------------------------|--------------------------|
| 1           | 3         | 4       | 2020-01-02T00:00:00.000Z | 2020-01-03T00:00:00.000Z |
| 1           | 3         | 4       | 2020-01-04T00:00:00.000Z | 2020-01-14T00:00:00.000Z |
| 1           | 3         | 2       | 2020-01-15T00:00:00.000Z | 2020-01-16T00:00:00.000Z |
| 1           | 3         | 5       | 2020-01-17T00:00:00.000Z | 2020-01-28T00:00:00.000Z |
| 1           | 3         | 3       | 2020-01-29T00:00:00.000Z | 2020-02-18T00:00:00.000Z |
| 1           | 3         | 2       | 2020-02-19T00:00:00.000Z | 2020-03-16T00:00:00.000Z |
| 1           | 3         | 2       | 2020-03-17T00:00:00.000Z | 9999-12-31T00:00:00.000Z |

Here we can see that on 4/01/20 the same node was generated. So we need a way to account for this as it took 12 days to assign a new node. 

### What is the median, 80th and 95th percentile for this same reallocation days metric for each region?


### B. Customer Transactions



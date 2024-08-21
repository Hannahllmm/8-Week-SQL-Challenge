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

To start answering this questions first we'll create a tempory table that calculates the duration between the start and end date and we will rank each row partitioned by the customer_id and ordered by the start_date to use later to handle the above scenario. 

```sql
DROP TABLE IF EXISTS ranked_customer_nodes;
CREATE TEMP TABLE ranked_customer_nodes AS
(SELECT
  customer_id,
  node_id,
  region_id,
  DATE_PART('day', AGE(end_date, start_date))::INTEGER AS duration,
  ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date) AS rn
FROM data_bank.customer_nodes
ORDER BY customer_id, start_date);
```
This is an extract of this tempory table:
![image](https://github.com/user-attachments/assets/2ea07214-c2cf-4fe2-a22f-b583180b5724)

We're then going to use a recursive CTE to generate our ouput. Our anchor member which is our starting point will be all the records where the rn is 1, this is the first record for each customer. Which looks like this:

```sql
DROP TABLE IF EXISTS ranked_customer_nodes;
CREATE TEMP TABLE ranked_customer_nodes AS
(SELECT
  customer_id,
  node_id,
  region_id,
  DATE_PART('day', AGE(end_date, start_date))::INTEGER AS duration,
  ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date) AS rn
FROM data_bank.customer_nodes
ORDER BY customer_id, start_date);

SELECT
  customer_id,
  node_id,
  duration,
  rn,
  1 AS run_id
FROM ranked_customer_nodes
WHERE rn = 1
```

This is an extract of this starting point:
![image](https://github.com/user-attachments/assets/4586caa5-1713-4583-99ff-f9c98e013565)

Next we need to union this table with the output of the recursive table.

```sql
DROP TABLE IF EXISTS ranked_customer_nodes;
CREATE TEMP TABLE ranked_customer_nodes AS
(SELECT
  customer_id,
  node_id,
  region_id,
  DATE_PART('day', AGE(end_date, start_date))::INTEGER AS duration,
  ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date) AS rn
FROM data_bank.customer_nodes
ORDER BY customer_id, start_date);

WITH RECURSIVE output_table AS (
  SELECT
    customer_id,
    node_id,
    duration,
    rn,
    1 AS run_id
  FROM ranked_customer_nodes
  WHERE rn = 1

  UNION ALL

  SELECT
    t2.customer_id,
    t2.node_id,
    t2.duration,
    t2.rn,
    CASE
      WHEN t1.node_id != t2.node_id THEN t1.run_id + 1
      ELSE t1.run_id
    END AS run_id
  FROM output_table t1
  INNER JOIN ranked_customer_nodes t2
    ON t1.rn + 1 = t2.rn
    AND t1.customer_id = t2.customer_id
)

SELECT * FROM output_table
ORDER BY customer_id, rn;
```
Sample output:
![image](https://github.com/user-attachments/assets/571a83b7-df5b-4152-a2f2-3972972c0b20)

Our aim here is to have a column run_id that will increase only when the node has changed for the customer. We increase t1.rn by 1 to look at the next record for each customer. We check to see if the node_id is the same, if it's not then we increase the run_id by one, but if it is then keep the run_id as the same. We carry on increasing the rn by 1 until there are none left. 

Next we can take an average of the duration by the run_id and the customer_id
```sql
DROP TABLE IF EXISTS ranked_customer_nodes;
CREATE TEMP TABLE ranked_customer_nodes AS
(SELECT
  customer_id,
  node_id,
  region_id,
  DATE_PART('day', AGE(end_date, start_date))::INTEGER AS duration,
  ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date) AS rn
FROM data_bank.customer_nodes
ORDER BY customer_id, start_date);

WITH RECURSIVE output_table AS (
  SELECT
    customer_id,
    node_id,
    duration,
    rn,
    1 AS run_id
  FROM ranked_customer_nodes
  WHERE rn = 1

  UNION ALL

  SELECT
    t2.customer_id,
    t2.node_id,
    t2.duration,
    t2.rn,
    CASE
      WHEN t1.node_id != t2.node_id THEN t1.run_id + 1
      ELSE t1.run_id
    END AS run_id
  FROM output_table t1
  INNER JOIN ranked_customer_nodes t2
    ON t1.rn + 1 = t2.rn
    AND t1.customer_id = t2.customer_id
),
cte_customer_nodes AS (
  SELECT
    customer_id,
    run_id,
    SUM(duration) AS node_duration
  FROM output_table
  GROUP BY
    customer_id,
    run_id
)
SELECT
  ROUND(AVG(node_duration)) AS average_node_duration
FROM cte_customer_nodes;
```
![image](https://github.com/user-attachments/assets/062e529a-55c5-4693-94c7-aaadc903969d)

The average duration is 17 days.

### What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
To answer this question we can reuse the query for the previous questions. We just need to add in the region and use PERCENTILE_CONT rather than the average.

```sql
DROP TABLE IF EXISTS ranked_customer_nodes;
CREATE TEMP TABLE ranked_customer_nodes AS
(SELECT
  customer_id,
  node_id,
  region_id,
  DATE_PART('day', AGE(end_date, start_date))::INTEGER AS duration,
  ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date) AS rn
FROM data_bank.customer_nodes
ORDER BY customer_id, start_date);

WITH RECURSIVE output_table AS (
  SELECT
    customer_id,
    region_id,
    node_id,
    duration,
    rn,
    1 AS run_id
  FROM ranked_customer_nodes
  WHERE rn = 1

  UNION ALL

  SELECT
    t2.customer_id,
    t2.region_id,
    t2.node_id,
    t2.duration,
    t2.rn,
    CASE
      WHEN t1.node_id != t2.node_id THEN t1.run_id + 1
      ELSE t1.run_id
    END AS run_id
  FROM output_table t1
  INNER JOIN ranked_customer_nodes t2
    ON t1.rn + 1 = t2.rn
    AND t1.customer_id = t2.customer_id
),
cte_customer_nodes AS (
  SELECT
    customer_id,
    region_id,
    run_id,
    SUM(duration) AS node_duration
  FROM output_table
  GROUP BY
    customer_id,
    run_id,
    region_id
)
SELECT
  regions.region_name,
  ROUND(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY node_duration)) AS median_node_duration,
  ROUND(PERCENTILE_CONT(0.8) WITHIN GROUP (ORDER BY node_duration)) AS pct80_node_duration,
  ROUND(PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY node_duration)) AS pct95_node_duration
FROM cte_customer_nodes
INNER JOIN data_bank.regions
  ON cte_customer_nodes.region_id = regions.region_id
GROUP BY regions.region_name, regions.region_id
ORDER BY regions.region_id;
```
![image](https://github.com/user-attachments/assets/11d91f7c-208e-4626-b654-5614066898ed)


## B. Customer Transactions
### What is the unique count and total amount for each transaction type?
```sql
SELECT 
	txn_type transaction_type,
    COUNT(txn_type) transaction_type_count,
    SUM(txn_amount) total_amount
FROM data_bank.customer_transactions
GROUP BY txn_type;
```
![image](https://github.com/user-attachments/assets/344c22b2-3d1e-419a-b9de-3e88c9438429)

### What is the average historical deposit counts and amounts for all customers?
```sql
WITH customer_deposits AS (
 SELECT 
	customer_id,
    COUNT(*) deposit_count,
    SUM(txn_amount) total_amount
FROM data_bank.customer_transactions
WHERE txn_type = 'deposit'
GROUP BY customer_id)

SELECT 
ROUND(AVG(deposit_count)) avg_deposit_count,
ROUND(AVG(total_amount)) avg_total_amount
FROM customer_deposits
```
![image](https://github.com/user-attachments/assets/d48863d0-40fe-4cb9-9358-72d95eed02b1)

### For each month - how many Data Bank customers make more than 1 deposit and at least either 1 purchase or 1 withdrawal in a single month?
```sql
WITH customer_transactions AS (
 SELECT 
	customer_id,
  	TO_CHAR(txn_date, 'Month') AS transaction_month,
  	SUM(CASE WHEN txn_type = 'deposit' THEN 1 ELSE 0 END) AS deposit_count,
    SUM(CASE WHEN txn_type = 'purchase' THEN 1 ELSE 0 END) AS purchase_count,
    SUM(CASE WHEN txn_type = 'withdrawal' THEN 1 ELSE 0 END) AS withdrawal_count
FROM data_bank.customer_transactions
GROUP BY customer_id, transaction_month
ORDER BY customer_id)

SELECT 
transaction_month,
COUNT(DISTINCT customer_id) customer_count
FROM customer_transactions
WHERE deposit_count > 1 AND (purchase_count >= 1 OR withdrawal_count >= 1)
GROUP BY transaction_month
```



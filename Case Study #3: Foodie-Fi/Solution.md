# Case Study #3: Foodie-Fi
Feel free to test these queries out [here.](https://www.db-fiddle.com/f/rHJhRrXy5hbVBNJ6F6b9gJ/16)

## Table of Contents
- [A. Customer Journey](#a-customer-journey)
- [B. Data Analysis Questions](#b-data-analysis-questions)
- [C. Challenge Payment Questions](#c-challenge-payment-questions)
- [D. Outside The Box Questions](#d-outside-the-box-questions)

## A. Customer Journey
```sql
    SELECT 
    	s.customer_id,
        s.start_date,
        p.plan_name,
        p.price
    FROM foodie_fi.plans AS p
    LEFT JOIN foodie_fi.subscriptions AS s
    ON p.plan_id = s.plan_id 
    WHERE s.customer_id IN (1, 2, 11, 13, 15, 16, 18, 19)
    ORDER BY 
     s.customer_id,
     s.start_date;
```

| customer_id | start_date               | plan_name     | price  |
| ----------- | ------------------------ | ------------- | ------ |
| 1           | 2020-08-01T00:00:00.000Z | trial         | 0.00   |
| 1           | 2020-08-08T00:00:00.000Z | basic monthly | 9.90   |
| 2           | 2020-09-20T00:00:00.000Z | trial         | 0.00   |
| 2           | 2020-09-27T00:00:00.000Z | pro annual    | 199.00 |
| 11          | 2020-11-19T00:00:00.000Z | trial         | 0.00   |
| 11          | 2020-11-26T00:00:00.000Z | churn         |        |
| 13          | 2020-12-15T00:00:00.000Z | trial         | 0.00   |
| 13          | 2020-12-22T00:00:00.000Z | basic monthly | 9.90   |
| 13          | 2021-03-29T00:00:00.000Z | pro monthly   | 19.90  |
| 15          | 2020-03-17T00:00:00.000Z | trial         | 0.00   |
| 15          | 2020-03-24T00:00:00.000Z | pro monthly   | 19.90  |
| 15          | 2020-04-29T00:00:00.000Z | churn         |        |
| 16          | 2020-05-31T00:00:00.000Z | trial         | 0.00   |
| 16          | 2020-06-07T00:00:00.000Z | basic monthly | 9.90   |
| 16          | 2020-10-21T00:00:00.000Z | pro annual    | 199.00 |
| 18          | 2020-07-06T00:00:00.000Z | trial         | 0.00   |
| 18          | 2020-07-13T00:00:00.000Z | pro monthly   | 19.90  |
| 19          | 2020-06-22T00:00:00.000Z | trial         | 0.00   |
| 19          | 2020-06-29T00:00:00.000Z | pro monthly   | 19.90  |
| 19          | 2020-08-29T00:00:00.000Z | pro annual    | 199.00 |


#### The journey of these 8 customers:

1. This customer started with a free trial on the 1st August then downgraded to the basic plan.
2. This customer starter with a free trial on the 20th September then upgraded to a pro annual subscription.
11. Again this customer started with a free trial on the 19th November. But then they cancelled the plan.
13. This customer started with a free trial then after a week downgraded to a basic plan, then after a few months upgraded to the monthly pro.
15. Started with a free trial and continued onto the pro monthly subscription then cancelled their subscription.
16. Started with the free trial then downgraded to the monthly basic account, then after a few months upgraded to an annual pro licence.
18. Started with a free trial then continued onto the pro monthly subscription.
19. Started with the free trial then continued to the pro monthly before upgrading to the pro annual. 

## B. Data Analysis Questions
### 1. How many customers has Foodie-Fi ever had?
``` sql
    SELECT 
    	COUNT(DISTINCT s.customer_id) as total_customers
    FROM foodie_fi.subscriptions AS s;
```
| total_customers |
| --------------- |
| 1000            |

### 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
```sql
    SELECT 
        TO_CHAR(DATE_TRUNC('month',s.start_date),'month') AS "month",
    	COUNT(DISTINCT s.customer_id) AS trial_plans
    FROM foodie_fi.subscriptions AS s
    WHERE s.plan_id = 0
    GROUP BY DATE_TRUNC('month',s.start_date)
    ORDER BY DATE_TRUNC('month',s.start_date);
```

| month     | trial_plans |
| --------- | ----------- |
| january   | 88          |
| february  | 68          |
| march     | 94          |
| april     | 81          |
| may       | 88          |
| june      | 79          |
| july      | 89          |
| august    | 88          |
| september | 87          |
| october   | 79          |
| november  | 75          |
| december  | 84          |

The monthly distribution is fairly even with March having the most trials and February the least.

### 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name
```sql
    SELECT 
         p.plan_name,
         COUNT(*)  AS events 
    FROM foodie_fi.plans AS p
    LEFT JOIN foodie_fi.subscriptions AS s
    ON p.plan_id = s.plan_id 
    WHERE start_date >= '01-01-2021'
    GROUP BY plan_name, s.plan_id
    ORDER BY s.plan_id;
```

| plan_name     | events |
| ------------- | ----- |
| basic monthly | 8     |
| pro monthly   | 60    |
| pro annual    | 63    |
| churn         | 71    |

Here we filtered to only include the plans that happened on or after the 1st Jan 2021 and grouped by the plan and plan_id to count the events that happened.

### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
```sql
    SELECT
        COUNT(DISTINCT customer_id) as total_count,
        SUM(CASE WHEN plan_id = 4 THEN 1 ELSE 0 END) AS churned_customers,
        (100* (SUM(CASE WHEN plan_id = 4 THEN 1 ELSE 0 END)::float /
            COUNT(DISTINCT customer_id)::float))::text || '%'
            AS percentage_churned
    FROM foodie_fi.subscriptions;
```

| total_count | churned_customers | percentage_churned |
| ----------- | ----------------- | ------------------ |
| 1000        | 307               | 30.7%              |

Here we used SUM, CASE and WHEN to do conditional summations of the customers churned. In order to get the correct percentage we multiplied the ratio by 100 and both the denominator and numerator were converted to floats so that the percentage also came out as a float, we then converted this to text and added a percentage sign. 30.7% of the customers churned. 


### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
```sql
WITH cte_churned_after_trial AS(
SELECT s2.customer_id 
FROM 
    foodie_fi.subscriptions AS s1
JOIN 
    foodie_fi.subscriptions AS s2 ON s1.customer_id = s2.customer_id
WHERE 
    s1.plan_id = 0 
    AND s2.plan_id = 4
    AND s2.start_date - s1.start_date <= 7)
   
SELECT 
	COUNT(DISTINCT cte.customer_id) AS churn_count,
	(100* COUNT(DISTINCT cte.customer_id)::float /
            COUNT(DISTINCT s.customer_id)::float)::text || '%'
            AS percentage_churned
FROM 
	foodie_fi.subscriptions AS s
LEFT JOIN 
	cte_churned_after_trial AS cte
    ON cte.customer_id = s.customer_id;
```
    
| churn_count | percentage_churned |
| ----------- | ------------------ |
| 92          | 9.2%               |

Here we created a cte that uses a join and filtering to list out the customers who churned straight after a trial. This table is then joined to the original subscriptions table and we use the same method as before to calculate the percentage. Another way we could have done this that may have been more effective would be to use PARTITION BY to rank the plans of each customer by start_date and look at the second plan that was subscribed to. This would remove the need to join the subscription table to itself. 


### 6. What is the number and percentage of customer plans after their initial free trial?
```sql
    WITH ranked_plans AS (
      SELECT
        customer_id,
        plan_id,
        ROW_NUMBER() OVER (
          PARTITION BY customer_id
          ORDER BY start_date ASC
        ) AS plan_rank
      FROM foodie_fi.subscriptions
    )
    SELECT
    	plans.plan_id,
        plans.plan_name,
      COUNT(*) AS customers,
      (100 * COUNT(*)::float / SUM(COUNT(*)::float) OVER ())::text || '%' AS percentage
    FROM ranked_plans
    JOIN foodie_fi.plans
    	ON ranked_plans.plan_id = plans.plan_id
    WHERE plan_rank = 2
    GROUP BY plans.plan_id, plans.plan_name
    ORDER BY plans.plan_id;
```

| plan_id | plan_name     | customers | percentage |
| ------- | ------------- | --------- | ---------- |
| 1       | basic monthly | 546       | 54.6%      |
| 2       | pro monthly   | 325       | 32.5%      |
| 3       | pro annual    | 37        | 3.7%       |
| 4       | churn         | 92        | 9.2%       |

Here we used the PARTITION BY method to rank the plans of each customer by start_date to look at what plan they moved onto after their initial free trial.

### 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

### 8. How many customers have upgraded to an annual plan in 2020?

### 9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?

### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)

### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?


## C. Challenge Payment Questions
The Foodie-Fi team wants you to create a new payments table for the year 2020 that includes amounts paid by each customer in the subscriptions table with the following requirements:

monthly payments always occur on the same day of month as the original start_date of any monthly paid plan
upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
once a customer churns they will no longer make payments


## D. Outside The Box Questions
### 1. How would you calculate the rate of growth for Foodie-Fi?

### 2. What key metrics would you recommend Foodie-Fi management to track over time to assess performance of their overall business?

### 3. What are some key customer journeys or experiences that you would analyse further to improve customer retention?

### 4. If the Foodie-Fi team were to create an exit survey shown to customers who wish to cancel their subscription, what questions would you include in the survey?

### 5. What business levers could the Foodie-Fi team use to reduce the customer churn rate? How would you validate the effectiveness of your ideas?

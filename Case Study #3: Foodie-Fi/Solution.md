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
```sql
    WITH ranked_plans AS (
          SELECT
            customer_id,
            plan_id,
            ROW_NUMBER() OVER (
              PARTITION BY customer_id
              ORDER BY start_date DESC
            ) AS plan_rank
          FROM foodie_fi.subscriptions
          WHERE start_date <= '12-31-2020'
        )
        SELECT
        	plans.plan_id,
            plans.plan_name,
          COUNT(*) AS customers,
          (100 * COUNT(*)::float / SUM(COUNT(*)::float) OVER ())::text || '%' AS percentage
        FROM ranked_plans
        JOIN foodie_fi.plans
        	ON ranked_plans.plan_id = plans.plan_id
        WHERE plan_rank = 1
        GROUP BY plans.plan_id, plans.plan_name
        ORDER BY plans.plan_id;
```

| plan_id | plan_name     | customers | percentage |
| ------- | ------------- | --------- | ---------- |
| 0       | trial         | 19        | 1.9%       |
| 1       | basic monthly | 224       | 22.4%      |
| 2       | pro monthly   | 326       | 32.6%      |
| 3       | pro annual    | 195       | 19.5%      |
| 4       | churn         | 236       | 23.6%      |

We'ce used the same logic as the previous question but added a filter to the cte to only look at subscriptions from before 31/12/2020. We've also ordered the partition in descending order so that plans with rank 1 can be filtered, these are the current plans active at 31/12/202.

### 8. How many customers have upgraded to an annual plan in 2020?
```sql
    SELECT
    	COUNT(DISTINCT customer_id) AS annual_customers
    FROM foodie_fi.subscriptions
    WHERE EXTRACT(YEAR FROM start_date) = 2020 AND plan_id = 3;
```

| annual_customers |
| ---------------- |
| 195              |

All we needed to do here was add some filters to the subscriptions table and count the results. There were 195 customers that upgraded to annual subscriptions in 2020.

### 9. How many days on average does it take for a customer to upgrade to an annual plan from the day they join Foodie-Fi?
```sql
    SELECT 
    	AVG(pro_annual.start_date - trial.start_date)::int AS average_days
    FROM foodie_fi.subscriptions AS trial
    JOIN foodie_fi.subscriptions AS pro_annual 
    	ON trial.customer_id = pro_annual.customer_id
    WHERE trial.plan_id = 0 
    AND pro_annual.plan_id = 3;
```

| average_days |
| ------------ |
| 105          |

Here we joined the subscription table to itself to work out the average days between trial and pro annual subscription. On average it takes 105 for a customer to conver to an annual subscription.

### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
```sql
    WITH 
    annual_days AS (
        SELECT 
            pro_annual.customer_id,
            (pro_annual.start_date - trial.start_date)::int AS days
        FROM foodie_fi.subscriptions AS trial
        JOIN foodie_fi.subscriptions AS pro_annual 
            ON trial.customer_id = pro_annual.customer_id
        WHERE trial.plan_id = 0 
            AND pro_annual.plan_id = 3),
    
    breakdown_periods AS (
        SELECT
    		(annual_days.days / 30) * 30 + 1 || ' - ' || ((annual_days.days / 30) + 1) * 30 || ' days' 
    			AS breakdown_period,
            annual_days.customer_id
        FROM annual_days)
    
    SELECT
        breakdown_period,
        COUNT(customer_id) AS customers
    FROM breakdown_periods
    GROUP BY breakdown_period
    ORDER BY MIN(split_part(breakdown_period, ' - ', 1)::int);
```

| breakdown_period | customers |
| ---------------- | --------- |
| 1 - 30 days      | 48        |
| 31 - 60 days     | 25        |
| 61 - 90 days     | 33        |
| 91 - 120 days    | 35        |
| 121 - 150 days   | 43        |
| 151 - 180 days   | 35        |
| 181 - 210 days   | 27        |
| 211 - 240 days   | 4         |
| 241 - 270 days   | 5         |
| 271 - 300 days   | 1         |
| 301 - 330 days   | 1         |
| 331 - 360 days   | 1         |

We used the same query as the previous question to calculate the days it took to convert from a trial customer to an annual customer. We then used this to create a second cte breakdown_period where we converted each of the values from the previous query into a period. The days are slightly different to what the question asks for as 1-30 then 31-60 is less ambiguous. Finally this was used to create a table that counted the instances in each breakdown_period.

### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
```sql
    SELECT 
        	COUNT(DISTINCT pro_monthly.customer_id) AS downgrade_count
        FROM foodie_fi.subscriptions AS pro_monthly
        JOIN foodie_fi.subscriptions AS basic_monthly
        	ON pro_monthly.customer_id = basic_monthly.customer_id
        WHERE
		pro_monthly.plan_id = 2
        	AND basic_monthly.plan_id = 1
            	AND pro_monthly.start_date < basic_monthly.start_date
        	AND EXTRACT(YEAR FROM basic_monthly.start_date) = 2020;
```
| downgrade_count |
| --------------- |
| 0               |

We used a similar query to Question 9 but changed some of the conditions. There were no instances where people downgraded in 2020.

## C. Challenge Payment Questions
>The Foodie-Fi team wants you to create a new payments table for the year 2020 that includes amounts paid by each customer in the subscriptions table with the following requirements:
>* monthly payments always occur on the same day of month as the start_date of any monthly paid plan
>* upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
>* upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
>* once a customer churns they will no longer make payments

This is a complex question so we'll break it down. We’ll start by analysing the data to understand what we need to do. Then to the final query will involve a few ctes which we’ll explain as we go along. Creating temporary tables would be preferable but because our dataset is so small it’s not needed.

Firstly we use the LEAD windows function to find the next plan_id and start_date for each customer_id using the start_date as an order. We’ve filtered out where the plan_id is 0 because every customer starts with a trial plan. We’ve looked at an example set of customers to analyse their journey.
```sql
SELECT
  customer_id,
  plan_id,
  start_date,
  LEAD(plan_id) OVER (
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS lead_plan_id,
  LEAD(start_date) OVER (
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS lead_start_date
FROM foodie_fi.subscriptions
WHERE DATE_PART('year', start_date) = 2020
AND plan_id <> 0  -- Why do you think we need to remove these records?
-- example customers
AND customer_id IN (1, 2, 7, 11, 13, 15, 16, 18, 19, 25, 39);
```

| customer_id | plan_id | start_date               | lead_plan_id | lead_start_date          |
| ----------- | ------- | ------------------------ | ------------ | ------------------------ |
| 1           | 1       | 2020-08-08T00:00:00.000Z |              |                          |
| 2           | 3       | 2020-09-27T00:00:00.000Z |              |                          |
| 7           | 1       | 2020-02-12T00:00:00.000Z | 2            | 2020-05-22T00:00:00.000Z |
| 7           | 2       | 2020-05-22T00:00:00.000Z |              |                          |
| 11          | 4       | 2020-11-26T00:00:00.000Z |              |                          |
| 13          | 1       | 2020-12-22T00:00:00.000Z |              |                          |
| 15          | 2       | 2020-03-24T00:00:00.000Z | 4            | 2020-04-29T00:00:00.000Z |
| 15          | 4       | 2020-04-29T00:00:00.000Z |              |                          |
| 16          | 1       | 2020-06-07T00:00:00.000Z | 3            | 2020-10-21T00:00:00.000Z |
| 16          | 3       | 2020-10-21T00:00:00.000Z |              |                          |
| 18          | 2       | 2020-07-13T00:00:00.000Z |              |                          |
| 19          | 2       | 2020-06-29T00:00:00.000Z | 3            | 2020-08-29T00:00:00.000Z |
| 19          | 3       | 2020-08-29T00:00:00.000Z |              |                          |
| 25          | 1       | 2020-05-17T00:00:00.000Z | 2            | 2020-06-16T00:00:00.000Z |
| 25          | 2       | 2020-06-16T00:00:00.000Z |              |                          |
| 39          | 1       | 2020-06-04T00:00:00.000Z | 2            | 2020-08-25T00:00:00.000Z |
| 39          | 2       | 2020-08-25T00:00:00.000Z | 4            | 2020-09-10T00:00:00.000Z |
| 39          | 4       | 2020-09-10T00:00:00.000Z |              |                          |

Next we want to understand over the whole dataset what path users take. For this we will remove the filters from the table above and aggregate by the plan_id and lead_id.
```sql
WITH lead_plans AS (
SELECT
  customer_id,
  plan_id,
  start_date,
  LEAD(plan_id) OVER (
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS lead_plan_id,
  LEAD(start_date) OVER (
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS lead_start_date
FROM foodie_fi.subscriptions
WHERE DATE_PART('year', start_date) = 2020
AND plan_id <> 0)

SELECT
	ROW_NUMBER() OVER () AS row_number,
    plan_id,
    lead_plan_id,
    COUNT(*) AS transition_count
FROM lead_plans
GROUP BY plan_id, lead_plan_id
ORDER BY plan_id, lead_plan_id;
```
| row_number | plan_id | lead_plan_id | transition_count |
| ---------- | ------- | ------------ | ---------------- |
| 1          | 1       | 2            | 163              |
| 2          | 1       | 3            | 88               |
| 3          | 1       | 4            | 63               |
| 4          | 1       |              | 224              |
| 5          | 2       | 3            | 70               |
| 6          | 2       | 4            | 83               |
| 7          | 2       |              | 326              |
| 8          | 3       |              | 195              |
| 9          | 4       |              | 236              |

We can now see the paths users take that we need to consider. From a basic monthly plan users can then take any path except trial. From a pro monthly plan they only upgrade to pro annual or churn. Once users either churn or start a pro annual plan they don’t move on to anything else. Where the lead_plan_id is null this is the current plan the user is on. So the cases we need to consider when we create our final query are:

* Case 1: Customers who are on either the pro or basic monthly subscription (row_number 4, 7 and 9)
* Case 2: Customers who churn (row_number 3, 6 and 9)
* Case 3: Customers moving from a basic monthly plan to either pro plan (row_number 1 and 2)
* Case 4: Pro monthly customers upgrading to pro annual plans (row_number 5)
* Case 5: Annual pro payments (row_number 8)

### Case 1: Case 1: Customers who are on either the pro or basic monthly subscription

Lets start by breaking down case 1. The user will have paid on the date that they started their monthly plan and will have carried on paying until the end of 2020. So to start er will calculate how many more payment they’ll make after the start_date we have recorded. 

```sql
WITH lead_plans AS (
SELECT
  customer_id,
  plan_id,
  start_date,
  LEAD(plan_id) OVER (
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS lead_plan_id,
  LEAD(start_date) OVER (
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS lead_start_date
FROM foodie_fi.subscriptions
WHERE DATE_PART('year', start_date) = 2020
AND plan_id <> 0),

-- case 1: non churn monthly customers
case_1 AS (
SELECT
  customer_id,
  plan_id,
  start_date,
  DATE_PART('mon', AGE('2020-12-31'::DATE, start_date))::INTEGER AS month_diff
FROM lead_plans
WHERE lead_plan_id IS null
AND plan_id IN (1, 2)
)
SELECT * FROM case_1
  WHERE customer_id IN (1, 2, 7, 11, 13, 15, 16, 18, 19, 25, 39);
```

| customer_id | plan_id | start_date               | month_diff |
| ----------- | ------- | ------------------------ | ---------- |
| 1           | 1       | 2020-08-08T00:00:00.000Z | 4          |
| 7           | 2       | 2020-05-22T00:00:00.000Z | 7          |
| 13          | 1       | 2020-12-22T00:00:00.000Z | 0          |
| 18          | 2       | 2020-07-13T00:00:00.000Z | 5          |
| 25          | 2       | 2020-06-16T00:00:00.000Z | 6          |

We've restricted the data for demonstration purposes and will continute to do this throughout. Here we used DATE_PART and AGE to work out the number of months between the start date and the end of 2020 and filter for only the monthy subscribers that never churned.
Next we need to use this information to create a table with the customer_id, plan_id and payment_date where we have all the payment dates the user paid for their monthly subscription. 

```sql
WITH lead_plans AS (
SELECT
  customer_id,
  plan_id,
  start_date,
  LEAD(plan_id) OVER (
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS lead_plan_id,
  LEAD(start_date) OVER (
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS lead_start_date
FROM foodie_fi.subscriptions
WHERE DATE_PART('year', start_date) = 2020
AND plan_id <> 0),

-- case 1: non churn monthly customers
case_1 AS (
SELECT
  customer_id,
  plan_id,
  start_date,
  DATE_PART('mon', AGE('2020-12-31'::DATE, start_date))::INTEGER AS month_diff
FROM lead_plans
WHERE lead_plan_id IS null
AND plan_id IN (1, 2)
),
-- generate a series to add the months to each start_date
case_1_payments AS (
  SELECT
    customer_id,
    plan_id,
    (start_date + GENERATE_SERIES(0, month_diff) * INTERVAL '1 month')::DATE AS payment_date
  FROM case_1
)
SELECT * FROM case_1_payments
WHERE customer_id IN (1, 2, 7, 11, 13, 15, 16, 18, 19, 25, 39);
```
| customer_id | plan_id | payment_date             |
| ----------- | ------- | ------------------------ |
| 1           | 1       | 2020-08-08T00:00:00.000Z |
| 1           | 1       | 2020-09-08T00:00:00.000Z |
| 1           | 1       | 2020-10-08T00:00:00.000Z |
| 1           | 1       | 2020-11-08T00:00:00.000Z |
| 1           | 1       | 2020-12-08T00:00:00.000Z |
| 7           | 2       | 2020-05-22T00:00:00.000Z |
| 7           | 2       | 2020-06-22T00:00:00.000Z |
| 7           | 2       | 2020-07-22T00:00:00.000Z |
| 7           | 2       | 2020-08-22T00:00:00.000Z |
| 7           | 2       | 2020-09-22T00:00:00.000Z |
| 7           | 2       | 2020-10-22T00:00:00.000Z |
| 7           | 2       | 2020-11-22T00:00:00.000Z |
| 7           | 2       | 2020-12-22T00:00:00.000Z |
| 13          | 1       | 2020-12-22T00:00:00.000Z |
| 18          | 2       | 2020-07-13T00:00:00.000Z |
| 18          | 2       | 2020-08-13T00:00:00.000Z |
| 18          | 2       | 2020-09-13T00:00:00.000Z |
| 18          | 2       | 2020-10-13T00:00:00.000Z |
| 18          | 2       | 2020-11-13T00:00:00.000Z |
| 18          | 2       | 2020-12-13T00:00:00.000Z |
| 25          | 2       | 2020-06-16T00:00:00.000Z |
| 25          | 2       | 2020-07-16T00:00:00.000Z |
| 25          | 2       | 2020-08-16T00:00:00.000Z |
| 25          | 2       | 2020-09-16T00:00:00.000Z |
| 25          | 2       | 2020-10-16T00:00:00.000Z |
| 25          | 2       | 2020-11-16T00:00:00.000Z |
| 25          | 2       | 2020-12-16T00:00:00.000Z |

Here, payment_date is calculated by using GENERATE_SERIES to create a series of numbers from 0 up to the month_diff, each of these is multipled by an interval of 1 month. This is then added to start_date so for each customer_id and plan_id we end up with a series of dates that monthly payments were made.

### Case 2: Customers who churn
We'll use a similar method. It's worth noting that there are no annual customers that then churn so we only need to consider monthly payments here. We want to know how many monthly payments are made before the customer churns. Again we've limited the data for demonstration purposes.

```sql
    WITH lead_plans AS (
    SELECT
      customer_id,
      plan_id,
      start_date,
      LEAD(plan_id) OVER (
          PARTITION BY customer_id
          ORDER BY start_date
        ) AS lead_plan_id,
      LEAD(start_date) OVER (
          PARTITION BY customer_id
          ORDER BY start_date
        ) AS lead_start_date
    FROM foodie_fi.subscriptions
    WHERE DATE_PART('year', start_date) = 2020
    AND plan_id <> 0),
    
    
    case_2 AS (
      SELECT
        customer_id,
        plan_id,
        start_date,
        DATE_PART('mon', AGE(lead_start_date, start_date))::INTEGER AS month_diff
      FROM lead_plans
      
      WHERE lead_plan_id = 4
    )
    SELECT * FROM case_2
    WHERE customer_id IN (113);
    ```

| customer_id | plan_id | start_date               | month_diff |
| ----------- | ------- | ------------------------ | ---------- |
| 113         | 2       | 2020-09-13T00:00:00.000Z | 1          |

Again we've used DATE_PART and AGE to work out how many months the customer was paying for this membership before they churned. Next we need to use GENERATE_SERIES to create rows for each payment.
```sql
    WITH lead_plans AS (
    SELECT
      customer_id,
      plan_id,
      start_date,
      LEAD(plan_id) OVER (
          PARTITION BY customer_id
          ORDER BY start_date
        ) AS lead_plan_id,
      LEAD(start_date) OVER (
          PARTITION BY customer_id
          ORDER BY start_date
        ) AS lead_start_date
    FROM foodie_fi.subscriptions
    WHERE DATE_PART('year', start_date) = 2020
    AND plan_id <> 0),
    
    
    case_2 AS (
      SELECT
        customer_id,
        plan_id,
        start_date,
        DATE_PART('mon', AGE(lead_start_date, start_date))::INTEGER AS month_diff
      FROM lead_plans
      
      WHERE lead_plan_id = 4
    ),
    case_2_payments AS (
      SELECT
        customer_id,
        plan_id,
        (start_date + GENERATE_SERIES(0, month_diff) * INTERVAL '1 month')::DATE AS start_date
      from case_2
    )
    SELECT * FROM case_2_payments
    WHERE customer_id IN (113);
```

| customer_id | plan_id | start_date               |
| ----------- | ------- | ------------------------ |
| 113         | 2       | 2020-09-13T00:00:00.000Z |
| 113         | 2       | 2020-10-13T00:00:00.000Z |


### Case 3: Customers moving from a basic monthly plan to either pro plan
Here we need to output the dates that customers are paying the basic monthly membership. We'll calculate any deductions at a later stage. For now, we want the dates from the date that they started the basic monthly membersip up until the month before they changed to another subscription. 

```sql
WITH lead_plans AS (
SELECT
  customer_id,
  plan_id,
  start_date,
  LEAD(plan_id) OVER (
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS lead_plan_id,
  LEAD(start_date) OVER (
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS lead_start_date
FROM foodie_fi.subscriptions
WHERE DATE_PART('year', start_date) = 2020
AND plan_id <> 0),

-- case 3: customers who move from basic to pro plans
case_3 AS (
  SELECT
    customer_id,
    plan_id,
    start_date,
    DATE_PART('mon', AGE(lead_start_date , start_date))::INTEGER AS month_diff
  FROM lead_plans
  WHERE plan_id = 1 AND lead_plan_id IN (2, 3)
)
SELECT * FROM case_3
WHERE customer_id IN (7,16)
```

| customer_id | plan_id | start_date               | month_diff |
| ----------- | ------- | ------------------------ | ---------- |
| 7           | 1       | 2020-02-12T00:00:00.000Z | 3          |
| 16          | 1       | 2020-06-07T00:00:00.000Z | 4          |

Here we can see a couple examples of where customers have upgraded to a pro plan. We used the same method as we did for case1 but this time we wanted to know the number of months between the start_date when they started the basic plan and the lead_start_date when they upgraded. Next we want to generate the dates:

```sql
WITH lead_plans AS (
SELECT
  customer_id,
  plan_id,
  start_date,
  LEAD(plan_id) OVER (
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS lead_plan_id,
  LEAD(start_date) OVER (
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS lead_start_date
FROM foodie_fi.subscriptions
WHERE DATE_PART('year', start_date) = 2020
AND plan_id <> 0),

-- case 3: customers who move from basic to pro plans
case_3 AS (
  SELECT
    customer_id,
    plan_id,
    start_date,
    DATE_PART('mon', AGE(lead_start_date , start_date))::INTEGER AS month_diff
  FROM lead_plans
  WHERE plan_id = 1 AND lead_plan_id IN (2, 3)
),
case_3_payments AS (
  SELECT
    customer_id,
    plan_id,
    (start_date + GENERATE_SERIES(0, month_diff) * INTERVAL '1 month')::DATE AS payment_date
  from case_3
)

SELECT * FROM case_3_payments
WHERE customer_id IN (7,16)
```
| customer_id | plan_id | payment_date             |
| ----------- | ------- | ------------------------ |
| 7           | 1       | 2020-02-12T00:00:00.000Z |
| 7           | 1       | 2020-03-12T00:00:00.000Z |
| 7           | 1       | 2020-04-12T00:00:00.000Z |
| 7           | 1       | 2020-05-12T00:00:00.000Z |
| 16          | 1       | 2020-06-07T00:00:00.000Z |
| 16          | 1       | 2020-07-07T00:00:00.000Z |
| 16          | 1       | 2020-08-07T00:00:00.000Z |
| 16          | 1       | 2020-09-07T00:00:00.000Z |
| 16          | 1       | 2020-10-07T00:00:00.000Z |

Here we again used a similar method to case 1, we used GENERATE_SERIES  and INTERVAL to calculate the dates of payment. We minused 1 from the month_diff because we don't need the month that the upgrade happened as payments are calculated differently and will be covered later. 

### Case 4: Pro monthly customers upgrading to pro annual plans
Again we use the same sort of logic. We want to generate the dates that the user paid for a pro monthly subscription before they upgraded to an annual plan. 
```sql
WITH lead_plans AS (
SELECT
  customer_id,
  plan_id,
  start_date,
  LEAD(plan_id) OVER (
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS lead_plan_id,
  LEAD(start_date) OVER (
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS lead_start_date
FROM foodie_fi.subscriptions
WHERE DATE_PART('year', start_date) = 2020
AND plan_id <> 0),

-- case 4: pro monthly customers who move up to annual plans
case_4 AS (
  SELECT
    customer_id,
    plan_id,
    start_date,
    DATE_PART('mon', AGE(lead_start_date, start_date))::INTEGER AS month_diff
  FROM lead_plans
  WHERE plan_id = 2 AND lead_plan_id = 3
),
case_4_payments AS (
  SELECT
    customer_id,
    plan_id,
    (start_date + GENERATE_SERIES(0, month_diff-1) * INTERVAL '1 month')::DATE AS payment_date
  from case_4
)

SELECT * FROM case_4_payments
WHERE customer_id = 31
```

| customer_id | plan_id | payment_date               |
| ----------- | ------- | ------------------------ |
| 31          | 2       | 2020-06-29T00:00:00.000Z |
| 31          | 2       | 2020-07-29T00:00:00.000Z |
| 31          | 2       | 2020-08-29T00:00:00.000Z |
| 31          | 2       | 2020-09-29T00:00:00.000Z |
| 31          | 2       | 2020-10-29T00:00:00.000Z |

Here we've generated the payment dates from the date the user started the pro monthly plan until the month before they upgraded to an annual plan.

### Case 5: Annual pro payments
This is a simpler query because we dont have to generate multiple payments dates, because we're only looking at 2020 whatever date is the start_date when the users changes to an annual plan is the only payment_date we need.

```sql
WITH lead_plans AS (
SELECT
  customer_id,
  plan_id,
  start_date,
  LEAD(plan_id) OVER (
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS lead_plan_id,
  LEAD(start_date) OVER (
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS lead_start_date
FROM foodie_fi.subscriptions
WHERE DATE_PART('year', start_date) = 2020
AND plan_id <> 0),

-- case 5: annual pro payments
case_5_payments AS (
  SELECT
    customer_id,
    plan_id,
    start_date AS payment_date
  FROM lead_plans
  WHERE plan_id = 3
)

SELECT * FROM case_5_payments
WHERE customer_id = 2
```
| customer_id | plan_id | payment_date             |
| ----------- | ------- | ------------------------ |
| 2           | 3       | 2020-09-27T00:00:00.000Z |

### Putting this together

Finally we need to union these tables together and record the correct payment details. This is a long query but breaking it down like we have above helps to understand what is going on. 

```sql
    WITH lead_plans AS (
    SELECT
      customer_id,
      plan_id,
      start_date,
      LEAD(plan_id) OVER (
          PARTITION BY customer_id
          ORDER BY start_date
        ) AS lead_plan_id,
      LEAD(start_date) OVER (
          PARTITION BY customer_id
          ORDER BY start_date
        ) AS lead_start_date
    FROM foodie_fi.subscriptions
    WHERE DATE_PART('year', start_date) = 2020
    AND plan_id <> 0),
    
    
    case_1 AS (
    SELECT
      customer_id,
      plan_id,
      start_date,
      DATE_PART('mon', AGE('2020-12-31'::DATE, start_date))::INTEGER AS month_diff
    FROM lead_plans
    WHERE lead_plan_id IS null
    AND plan_id IN (1, 2)
    ),    
    case_1_payments AS (
      SELECT
        customer_id,
        plan_id,
        (start_date + GENERATE_SERIES(0, month_diff) * INTERVAL '1 month')::DATE AS payment_date
      FROM case_1
    ),
    
    
    case_2 AS (
      SELECT
        customer_id,
        plan_id,
        start_date,
        DATE_PART('mon', AGE(lead_start_date, start_date))::INTEGER AS month_diff
      FROM lead_plans      
      WHERE lead_plan_id = 4
    ),
    case_2_payments AS (
      SELECT
        customer_id,
        plan_id,
        (start_date + GENERATE_SERIES(0, month_diff) * INTERVAL '1 month')::DATE AS start_date
      from case_2
    ),
    
    
    case_3 AS (
      SELECT
        customer_id,
        plan_id,
        start_date,
        DATE_PART('mon', AGE(lead_start_date , start_date))::INTEGER AS month_diff
      FROM lead_plans
      WHERE plan_id = 1 AND lead_plan_id IN (2, 3)
    ),
    case_3_payments AS (
      SELECT
        customer_id,
        plan_id,
        (start_date + GENERATE_SERIES(0, month_diff) * INTERVAL '1 month')::DATE AS payment_date
      from case_3
    ),
    
    
    case_4 AS (
      SELECT
        customer_id,
        plan_id,
        start_date,
        DATE_PART('mon', AGE(lead_start_date, start_date))::INTEGER AS month_diff
      FROM lead_plans
      WHERE plan_id = 2 AND lead_plan_id = 3
    ),
    case_4_payments AS (
      SELECT
        customer_id,
        plan_id,
        (start_date + GENERATE_SERIES(0, month_diff-1) * INTERVAL '1 month')::DATE AS payment_date
      from case_4
    ),
    
    
    case_5_payments AS (
      SELECT
        customer_id,
        plan_id,
        start_date AS payment_date
      FROM lead_plans
      WHERE plan_id = 3
    ),
    
    
    union_output AS (
      SELECT * FROM case_1_payments
      UNION
      SELECT * FROM case_2_payments
      UNION
      SELECT * FROM case_3_payments
      UNION
      SELECT * FROM case_4_payments
      UNION
      SELECT * FROM case_5_payments
    )
    
    SELECT
      customer_id,
      plans.plan_id,
      plans.plan_name,
      payment_date,      
      CASE
        WHEN union_output.plan_id IN (2, 3) AND
          LAG(union_output.plan_id) OVER w = 1
        THEN plans.price - 9.90
        ELSE plans.price
        END AS amount,
      RANK() OVER w AS payment_order
    FROM union_output
    INNER JOIN foodie_fi.plans
      ON union_output.plan_id = plans.plan_id
    WHERE customer_id IN (1, 2, 7, 11, 16, 31, 39, 113)
    WINDOW w AS (
      PARTITION BY union_output.customer_id
      ORDER BY customer_id, payment_date
    );
```

| customer_id | plan_id | plan_name     | payment_date             | amount | payment_order |
| ----------- | ------- | ------------- | ------------------------ | ------ | ------------- |
| 1           | 1       | basic monthly | 2020-08-08T00:00:00.000Z | 9.90   | 1             |
| 1           | 1       | basic monthly | 2020-09-08T00:00:00.000Z | 9.90   | 2             |
| 1           | 1       | basic monthly | 2020-10-08T00:00:00.000Z | 9.90   | 3             |
| 1           | 1       | basic monthly | 2020-11-08T00:00:00.000Z | 9.90   | 4             |
| 1           | 1       | basic monthly | 2020-12-08T00:00:00.000Z | 9.90   | 5             |
| 2           | 3       | pro annual    | 2020-09-27T00:00:00.000Z | 199.00 | 1             |
| 7           | 1       | basic monthly | 2020-02-12T00:00:00.000Z | 9.90   | 1             |
| 7           | 1       | basic monthly | 2020-03-12T00:00:00.000Z | 9.90   | 2             |
| 7           | 1       | basic monthly | 2020-04-12T00:00:00.000Z | 9.90   | 3             |
| 7           | 1       | basic monthly | 2020-05-12T00:00:00.000Z | 9.90   | 4             |
| 7           | 2       | pro monthly   | 2020-05-22T00:00:00.000Z | 10.00  | 5             |
| 7           | 2       | pro monthly   | 2020-06-22T00:00:00.000Z | 19.90  | 6             |
| 7           | 2       | pro monthly   | 2020-07-22T00:00:00.000Z | 19.90  | 7             |
| 7           | 2       | pro monthly   | 2020-08-22T00:00:00.000Z | 19.90  | 8             |
| 7           | 2       | pro monthly   | 2020-09-22T00:00:00.000Z | 19.90  | 9             |
| 7           | 2       | pro monthly   | 2020-10-22T00:00:00.000Z | 19.90  | 10            |
| 7           | 2       | pro monthly   | 2020-11-22T00:00:00.000Z | 19.90  | 11            |
| 7           | 2       | pro monthly   | 2020-12-22T00:00:00.000Z | 19.90  | 12            |
| 16          | 1       | basic monthly | 2020-06-07T00:00:00.000Z | 9.90   | 1             |
| 16          | 1       | basic monthly | 2020-07-07T00:00:00.000Z | 9.90   | 2             |
| 16          | 1       | basic monthly | 2020-08-07T00:00:00.000Z | 9.90   | 3             |
| 16          | 1       | basic monthly | 2020-09-07T00:00:00.000Z | 9.90   | 4             |
| 16          | 1       | basic monthly | 2020-10-07T00:00:00.000Z | 9.90   | 5             |
| 16          | 3       | pro annual    | 2020-10-21T00:00:00.000Z | 189.10 | 6             |
| 31          | 2       | pro monthly   | 2020-06-29T00:00:00.000Z | 19.90  | 1             |
| 31          | 2       | pro monthly   | 2020-07-29T00:00:00.000Z | 19.90  | 2             |
| 31          | 2       | pro monthly   | 2020-08-29T00:00:00.000Z | 19.90  | 3             |
| 31          | 2       | pro monthly   | 2020-09-29T00:00:00.000Z | 19.90  | 4             |
| 31          | 2       | pro monthly   | 2020-10-29T00:00:00.000Z | 19.90  | 5             |
| 31          | 3       | pro annual    | 2020-11-29T00:00:00.000Z | 199.00 | 6             |
| 39          | 1       | basic monthly | 2020-06-04T00:00:00.000Z | 9.90   | 1             |
| 39          | 1       | basic monthly | 2020-07-04T00:00:00.000Z | 9.90   | 2             |
| 39          | 1       | basic monthly | 2020-08-04T00:00:00.000Z | 9.90   | 3             |
| 39          | 2       | pro monthly   | 2020-08-25T00:00:00.000Z | 10.00  | 4             |
| 113         | 1       | basic monthly | 2020-04-17T00:00:00.000Z | 9.90   | 1             |
| 113         | 1       | basic monthly | 2020-05-17T00:00:00.000Z | 9.90   | 2             |
| 113         | 1       | basic monthly | 2020-06-17T00:00:00.000Z | 9.90   | 3             |
| 113         | 1       | basic monthly | 2020-07-17T00:00:00.000Z | 9.90   | 4             |
| 113         | 1       | basic monthly | 2020-08-17T00:00:00.000Z | 9.90   | 5             |
| 113         | 2       | pro monthly   | 2020-09-13T00:00:00.000Z | 10.00  | 6             |
| 113         | 2       | pro monthly   | 2020-10-13T00:00:00.000Z | 19.90  | 7             |

In this final part of our query we have used UNION to union all the tables we created cases for. We then need to format this into a table where we could read off the payment details. Firstly we joined the union_output and the foodie_fi.plans tables on the plan_id to output the prices for each of the plans. But we need to consider where there are price reductions where a user has upgraded from a basic plan (which costs 9.90) to a pro plan. We define a window specification named w where we partition the data bt the customer_id, this is then used in the amount payment_order column. The amount is calculated by creating a CASE. Most of the time whatever the plan_id is we can take that plan price. Where there is an exception is when a user has gone from a basic plan to either a pro annual or pro monthly plan. We've used the LAG funtion in the context of WINDOWS w to see that the plan_id is in the previous row. If the current plan_id is 2 or 3 and the previous plan_id was 1 we know this is a time where the amount is the price of the current plan minus 9.90 (price of the basic monthly plan), any other time the amount is just the price of the current plan. payment_order is calculated by using RANK to rank each row in the context of WINDOW w, so ranking each row partitioned by the customer_id and ordered by the payment_date.

## D. Outside The Box Questions
Each of teh following questions has multiple potential answers. We'll investigate just one potential answer for each.
### 1. How would you calculate the rate of growth for Foodie-Fi?



### 2. What key metrics would you recommend Foodie-Fi management to track over time to assess performance of their overall business?

### 3. What are some key customer journeys or experiences that you would analyse further to improve customer retention?

### 4. If the Foodie-Fi team were to create an exit survey shown to customers who wish to cancel their subscription, what questions would you include in the survey?

### 5. What business levers could the Foodie-Fi team use to reduce the customer churn rate? How would you validate the effectiveness of your ideas?

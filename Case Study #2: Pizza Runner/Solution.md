# Case Study #2: Pizza Runner
Feel free to test these queries out [here.](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)

## Table of Contents
- [A. Pizza Metrics](#a-pizza-metrics)
- [B. Runner and Customer Experience](#b-runner-and-customer-experience)
- [C. Ingredient Optimisation](#c-ingredient-optimisation)
- [D. Pricing and Ratings](#d-pricing-and-ratings)
- [E. Bonus Questions](#e-bonus-questions)

## A. Pizza Metrics
### How many pizzas were ordered?
This is a simple question with a simple answer. We can just take a count of all the rows in the pizza_orders table. We're assuming we want a count of all pizzas ordered regardless of whether they were cancelled.

```sql
SELECT 
COUNT(*)
FROM cleaned_customer_orders;
```
![image](https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/2e064f16-e1a6-42a7-ac0f-e3930bac4a8e)

There were 14 pizzas ordered.

### How many unique customers are there?
We can count the distinct values in customer_id in the customer_orders table.

```sql
SELECT 
COUNT(DISTINCT customer_id)
FROM cleaned_customer_orders;
```

![image](https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/6dc46c24-0870-4fed-ab47-d678eff8389c)

There are 5 customers.

### How many successful orders were delivered by each runner?
We can take the runner_id and a count of the order_id where the cancellation field is null.

```SQL
SELECT
  runner_id,
  COUNT(order_id) AS successful_orders
FROM cleaned_runner_orders
WHERE cancellation IS NULL
GROUP BY runner_id
ORDER BY runner_id;
```
![image](https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/8c2bc60a-55b2-4a45-9b56-9f1aca82d942)

We can see that runner 1 delivered 4 succesful orders, runner 2 delivered 3 and runner 3 delivered 1.

### How many of each type of pizza was delivered?
We can join the cleaned customer and runners orders table with the pizza_names table where the order wasn't cancelled to count the number of each type of pizzas delivered.

```sql
SELECT
  t2.pizza_name,
  COUNT(t1.*) AS delivered_pizza_count
FROM cleaned_customer_orders AS t1
LEFT JOIN pizza_runner.pizza_names AS t2
  ON t1.pizza_id = t2.pizza_id
LEFT JOIN cleaned_runner_orders AS t3
  ON t1.order_id = t3.order_id
WHERE t3.cancellation IS NULL
GROUP BY t2.pizza_name
ORDER BY t2.pizza_name;
```
![image](https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/11f3c001-e19c-4d05-a773-f4d25ba59bb2)

There were 9 meat and 3 veggie pizzas delivered.


### How many Vegetarian and Meatlovers were ordered by each customer?
We could just add in the customer_id to the query above to get the answer. However we want this to be as easy as possible for others to interpret the data. So instead we'll create two count columns for each type of pizza. 

```sql
SELECT
  customer_id,
  SUM(CASE WHEN pizza_id = 1 THEN 1 ELSE 0 END) AS meatlovers,
  SUM(CASE WHEN pizza_id = 2 THEN 1 ELSE 0 END) AS vegetarian
FROM cleaned_customer_orders
GROUP BY customer_id
ORDER BY customer_id;
```
![image](https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/c6446745-1e15-4167-82f6-8f848af686e1)

### What was the maximum number of pizzas in a single order?
We can take a count of the order_id in the customer orders, then by ordering by the count and setting the limit to 1 we can see that the maximum number of pizzas delivered in a single order is 3.

```sql
SELECT 
COUNT(order_id) as number_of_pizzas
FROM cleaned_customer_orders
GROUP BY order_id
ORDER BY number_of_pizzas DESC
LIMIT 1;
```
![image](https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/d9aaba0b-8dca-4435-93c8-5da2ffe9b888)


### For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
This is where our clean up of the customer orders comes in handy. We can assign a value os 1 to the instances where changes were made and 0 to the rest then sum this to calcualte the number of pizzas ordered with at least one change. The same process can be done to calculate the pizzas with no changes. 

```sql
SELECT
  customer_id,
  SUM(
    CASE
      WHEN exclusions IS NOT NULL OR extras IS NOT NULL THEN 1
      ELSE 0
    END
  ) AS at_least_1_change,
  SUM(
    CASE
      WHEN exclusions IS NULL AND extras IS NULL THEN 1
      ELSE 0
    END
  ) AS no_changes
FROM cleaned_customer_orders AS t1
GROUP BY customer_id
ORDER BY customer_id;
```

![image](https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/6e333093-259a-4fea-a85a-6196b27ddc17)

### How many pizzas were had both exclusions and extras?
Again, with the cleaned version of the customers orders table this is a very simple question to answer. We can just count the orders where both the exclusions and extras are not null.

```sql
SELECT
  COUNT(*)
FROM cte_cleaned_customer_orders
WHERE exclusions IS NULL OR extras IS NOT NULL;
```

![image](https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/3b40c780-7e78-4490-8990-b4bd374520e5)

There were 2 pizzas that had both exclusions and extras.

### What was the total volume of pizzas ordered for each hour of the day?
We can answer this by taking the hour from the order_time after casting it to a timestamp and taking a count of the orders for each hour. 

```sql
SELECT
DATE_PART('hour', order_time::TIMESTAMP) AS hour_of_day,
	COUNT(order_id) AS pizza_count	
FROM cleaned_customer_orders
GROUP BY hour_of_day
ORDER BY hour_of_day;
```
![image](https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/884bc185-fe79-4bc8-a3a4-f4f1131e522d)

### What was the volume of orders for each day of the week?
To answer this we can first use the TO_CHAR function to format the order_time as just the day of teh week. We can then use the DATE_PART to extract the day of the week as a number to order the table correctly.

```sql
SELECT
  TO_CHAR(order_time, 'day') AS day_of_week,
  COUNT(order_id) AS pizza_count
FROM cleaned_customer_orders
GROUP BY day_of_week, DATE_PART('dow', order_time)
ORDER BY DATE_PART('dow', order_time);
```

![image](https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/62f2cd94-3f97-4f53-9abf-bc564f6fb3b8)

## B. Runner and Customer Experience
### How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
We can use the DATE_TRUNC function to truncate the registration_date to the nearest week. We can also use the To_CHAR funtion to remove the timestamp for an easier to read format.

```sql
SELECT
  TO_CHAR(DATE_TRUNC('week', registration_date), 'YYYY-MM-DD') AS registration_week,
  COUNT(*) AS signups
FROM pizza_runner.runners
GROUP BY registration_week
ORDER BY registration_week;
```

![image](https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/a7621b4d-3d29-4668-b5ff-76025369376b)


### What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
We can use the AGE function to calculate the time between the pick up and the order time. We can then use the DATE_PART function to calculate how many minutes that is. It's also important to use the DISTINCT function as there are duplicates when more than one pizza is ordered in one order.

```sql
WITH cte_pickup_minutes AS (
  SELECT DISTINCT
    t1.order_id,
    t1.runner_id,
    DATE_PART('minutes', AGE(t1.pickup_time::TIMESTAMP, t2.order_time))::INTEGER AS pickup_minutes
  FROM pizza_runner.runner_orders AS t1
  INNER JOIN pizza_runner.customer_orders AS t2
    ON t1.order_id = t2.order_id
  WHERE t1.pickup_time != 'null'
)
SELECT
  runner_id,
  ROUND(AVG(pickup_minutes), 3) AS avg_pickup_minutes
FROM cte_pickup_minutes
GROUP BY runner_id
```
![image](https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/4b812e30-c611-4a1e-b037-b4785d5f377c)


### Is there any relationship between the number of pizzas and how long the order takes to prepare?
We only have a small dataset to lok at however it does look like the more pizzas are ordered the longer it takes. We can edit our previous query to answer this question. We can remove the DISTINCT function and add in a count of the pizzas ordered. 

```sql
WITH cte_pickup_minutes AS (
  SELECT
    t1.order_id,
    t1.runner_id,
    DATE_PART('minutes', AGE(t1.pickup_time::TIMESTAMP, t2.order_time))::INTEGER AS pickup_minutes
  FROM pizza_runner.runner_orders AS t1
  INNER JOIN pizza_runner.customer_orders AS t2
    ON t1.order_id = t2.order_id
  WHERE t1.pickup_time != 'null'
)
SELECT
  order_id,
  COUNT(order_id) pizzas,
  ROUND(AVG(pickup_minutes),0) AS avg_pickup_minutes
FROM cte_pickup_minutes
GROUP BY order_id
ORDER BY pizzas;
```
![image](https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/eccd877f-dc07-4e45-8b65-9a1a8876094f)

### What was the average distance travelled for each customer?
We need to be careful when handling the duplicated here.
```sql
WITH cte_customer_order_distances AS (
SELECT DISTINCT
  t1.customer_id,
  t2.distance_km
FROM cleaned_customer_orders AS t1
JOIN cleaned_runner_orders AS t2
  ON t1.order_id = t2.order_id)
SELECT
  customer_id,
  AVG(distance_km) AS avg_distance
FROM cte_customer_order_distances
GROUP BY customer_id
ORDER BY customer_id;
```
![image](https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/eb1cc25e-7905-4b49-95a9-b4ea4d83b603)

### What was the difference between the longest and shortest delivery times for all orders?
This is easy to answer after we cleaned the duration field in the runenr_orders table. We can just calculate the difference between the min and max. 

```sql
SELECT
  MAX(t1.duration_mins)-MIN(t1.duration_mins)as max_difference
FROM cleaned_runner_orders AS t1;
```
![image](https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/e28b6954-e206-4498-b847-23c420f144b5)


### What was the average speed for each runner for each delivery?
```sql
SELECT DISTINCT
  order_id,
  runner_id,
  distance_km,
  duration_mins,
  ROUND((distance_km / duration_mins) * 60) AS speed
FROM cleaned_runner_orders
WHERE pickup_time != 'null'
ORDER BY runner_id;
```

![image](https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/9d5a983a-a9e6-462b-9171-7c06e4033a5b)

It's possible that runner 2 recorded the time wrong because their speed differers quite dramatically. Danny might want to investigate this.

### What is the successful delivery percentage for each runner?



## C. Ingredient Optimisation
### What are the standard ingredients for each pizza?
### What was the most commonly added extra?
### What was the most common exclusion?
### Generate an order item for each record in the customers_orders table in the format of  'Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers'
### Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients. For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"
### What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

## D. Pricing and Ratings
### If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
### What if there was an additional $1 charge for any pizza extras?
### The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
### Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
- customer_id
- order_id
- runner_id
- rating
- order_time
- pickup_time
- Time between order and pickup
- Delivery duration
- Average speed
- Total number of pizzas
### If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?


## E. Bonus Questions
### If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?

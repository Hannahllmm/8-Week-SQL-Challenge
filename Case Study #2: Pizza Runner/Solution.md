# Case Study #2: Pizza Runner
Feel free to test these queries out [here.](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)

## Table of Contents
- [A. Pizza Metrics](#a-pizza-metrics)
- [B. Runner and Customer Experience](b-runner-and-customer-experience)
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

### How many unique customer orders were made?
We can count the distinct values in customer_id in the customer_orders table.

```sql
SELECT 
COUNT(DISTINCT customer_id)
FROM cleaned_customer_orders;
```

![image](https://github.com/Hannahllmm/8-Week-SQL-Challenge/assets/39679731/6dc46c24-0870-4fed-ab47-d678eff8389c)

There are 5 customers.


### How many successful orders were delivered by each runner?
### How many of each type of pizza was delivered?
### How many Vegetarian and Meatlovers were ordered by each customer?
### What was the maximum number of pizzas delivered in a single order?
### For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
### How many pizzas were delivered that had both exclusions and extras?
### What was the total volume of pizzas ordered for each hour of the day?
### What was the volume of orders for each day of the week?


## B. Runner and Customer Experience
### How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
### What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
### Is there any relationship between the number of pizzas and how long the order takes to prepare?
### What was the average distance travelled for each customer?
### What was the difference between the longest and shortest delivery times for all orders?
### What was the average speed for each runner for each delivery and do you notice any trend for these values?
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

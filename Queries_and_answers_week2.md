WEEK 2, 8WeekSQlChallenge, Pizza Runner 

Source: https://8weeksqlchallenge.com/case-study-2/. Kindly refer to the source for background details

Note: The challenge stated the questions for this case study could be answered with a single statement so I strived to achieve this .

Analyst: Ajike Taiwo 

Tools: PostGreSQL v13 via DB fiddle and Sublime Text

INVESTIGATING AND CLEANING DATA 

CLEANING LOG

A. TABLE NAME : customer_orders

1. Edited the 'exclusions' column to ensure that data was consistent for orders that had no exclusions

```SQL
UPDATE pizza_runner.customer_orders

SET exclusions = NULL
WHERE exclusions = '' OR exclusions = 'null';

```

2. Edited the 'extras' column to ensure that data was consistent for orders that had no extras

```SQL
UPDATE pizza_runner.customer_orders

SET extras = NULL
WHERE extras = '' OR extras = 'null' ;

```

B. TABLE NAME : runners_orders

1. Edited the 'pickup_time' column to exclude null (inorder to be able to change to TIMESTAMP)

```SQL 
UPDATE pizza_runner.runner_orders

SET pickup_time = NULL
WHERE pickup_time = 'null';

```

2. Changed the  data type 'pickup_time' column from VARCHAR(19) to TIMESTAMP

```SQL
ALTER TABLE pizza_runner.runner_orders
ALTER COLUMN pickup_time 
TYPE TIMESTAMP 
USING pickup_time::TIMESTAMP

```

3. Removing kilometer and minutes inconsistencies in  the 'duration' and 'distance' columns

```SQL
UPDATE pizza_runner.runner_orders
SET distance = trim('km' from distance);

UPDATE pizza_runner.runner_orders
SET duration = trim('minutes' from duration);

UPDATE pizza_runner.runner_orders
SET duration = trim('mins' from duration);
```

4. Ensurring that null values are  qualified consistently across the 'duration', "cancellation" and 'distance' columns

```SQL
UPDATE pizza_runner.runner_orders

SET distance = NULL
WHERE distance = 'null';

UPDATE pizza_runner.runner_orders

SET duration = NULL
WHERE duration = 'null';

UPDATE pizza_runner.runner_orders

SET cancellation = NULL
WHERE cancellation = '' OR cancellation = 'null';
```
There were no changes to other tables.



SQL QUERIES AND ANSWERS 

A. PIZZA METRICS

1. How many pizzas were ordered? (i made the assumption here to include canellations as the question said ordered not delivered)

```SQL
SELECT  
	COUNT(pizza_id) AS number_of_orders

FROM 
	pizza_runner.customer_orders

```

ANSWER:14 pizzas were ordered


2. How many unique customer orders were made?

```SQL
SELECT 
	COUNT(DISTINCT(order_id)) AS number_of_unique_orderes


FROM 
	pizza_runner.customer_orders

```

ANSWER: 10 unique orderes were made 


3. How many successful orders were delivered by each runner?

```SQL
SELECT 
	runner_id, COUNT (order_id) AS number_of_successful_deliveries

FROM 
	pizza_runner.runner_orders
WHERE 
	pickup_time IS NOT NULL
GROUP BY 
	runner_orders.runner_id

```

ANSWER: RIDER 1: 4 deliveries, RIDER 2 : 3 deliveries, RIDER 3 : 1 deliery


4. How many of each type of pizza was delivered?

```SQL
SELECT 
	t.pizza_name, COUNT(t.pizza_name) AS number_of_pizza_delivered

FROM 
	(SELECT runner_orders.cancellation, customer_orders.order_id, pizza_names.pizza_name
	FROM pizza_runner.runner_orders
	JOIN pizza_runner.customer_orders
	ON runner_orders.order_id=customer_orders.order_id
	JOIN pizza_runner.pizza_names
      ON customer_orders.pizza_id = pizza_names.pizza_id
     ) AS t

WHERE 
	t.cancellation IS NULL
GROUP BY 
	t.pizza_name;
```

ANSWER: MeatLovers: 9 , Vegetarian : 3 


5. How many Vegetarian and Meatlovers were ordered by each customer (I INCLUDED CANCELLED ORDERS HERE)

```SQL
SELECT 
COUNT(case when t.customer_id = '101' AND t.pizza_name ='Meatlovers' THEN 1 ELSE NULL END) AS meatlover_101,
COUNT (case when t.customer_id = '102' AND t.pizza_name ='Meatlovers' THEN 1 ELSE NULL END) AS meatlover_102,
	COUNT (case when t.customer_id = '103' AND t.pizza_name ='Meatlovers' THEN 1 ELSE NULL END) AS meatlover_103,
	COUNT (case when t.customer_id = '104' AND t.pizza_name ='Meatlovers' THEN 1 ELSE NULL END) AS meatlover_104,
	COUNT (case when t.customer_id = '105' AND t.pizza_name ='Meatlovers' THEN 1 ELSE NULL END) AS meatlover_105,
	COUNT (case when t.customer_id = '101' AND t.pizza_name ='Vegetarian' THEN 1 ELSE NULL END) AS veg_101,
	COUNT (case when t.customer_id = '102' AND t.pizza_name ='Vegetarian' THEN 1 ELSE NULL END) AS veg_102,
	COUNT (case when t.customer_id = '103' AND t.pizza_name ='Vegetarian' THEN 1 ELSE NULL END) AS veg_103,
	COUNT (case when t.customer_id = '104' AND t.pizza_name ='Vegetarian' THEN 1 ELSE NULL END) AS veg_104,
	COUNT (case when t.customer_id = '105' AND t.pizza_name ='Vegetarian' THEN 1 ELSE NULL END) AS veg_105

FROM
	(SELECT runner_orders.cancellation, customer_orders.order_id, customer_orders.customer_id, pizza_names.pizza_name
	FROM pizza_runner.runner_orders
	JOIN pizza_runner.customer_orders
	ON runner_orders.order_id=customer_orders.order_id
	JOIN pizza_runner.pizza_names
      ON customer_orders.pizza_id = pizza_names.pizza_id
     ) AS t


```

ANSWER: CUSTOMER 101 ORDERED 2 Meatlovers and 1 Vegetarian 
		CUSTOMER 102 ORDERED 2 Meatlovers and 1 Vegetarian 	
		CUSTOMER 103 ORDERED 3 Meatlovers and 1 Vegetarian 
		CUSTOMER 104 ORDERED 3 Meatlovers and 0 Vegetarian
		CUSTOMER 105 ORDERED 0 Meatlovers and 1 Vegetarian  


6. What was the maximum number of pizzas delivered in a single order?

```SQL
SELECT
	 t.order_id, count(t.order_id) AS number_of_pizzas
FROM
	(SELECT runner_orders.cancellation, customer_orders.order_id, customer_orders.customer_id, pizza_names.pizza_name
	FROM pizza_runner.runner_orders
	JOIN pizza_runner.customer_orders
	ON runner_orders.order_id=customer_orders.order_id
	JOIN pizza_runner.pizza_names
      ON customer_orders.pizza_id = pizza_names.pizza_id
     ) AS t

WHERE
	 t.cancellation IS NULL
GROUP BY 
	t.order_id
ORDER BY 
	number_of_pizzas DESC

```

ANSWER: 3


7. How many pizzas were delivered that had both exclusions and extras?

```SQL 

SELECT 
COUNT(t.customer_id) AS number_of_orders


FROM
(SELECT runner_orders.cancellation, customer_orders.exclusions,customer_orders.extras, customer_orders.customer_id
	FROM pizza_runner.runner_orders
	JOIN pizza_runner.customer_orders
	ON runner_orders.order_id=customer_orders.order_id
	JOIN pizza_runner.pizza_names
      ON customer_orders.pizza_id = pizza_names.pizza_id
  WHERE
    runner_orders.cancellation IS NULL ) AS t

WHERE
	t.exclusions IS NOT  NULL  AND t.extras IS NOT NULL ;

     
```

ANSWER: 1


8. What was the total volume of pizzas ordered for each hour of the day?

```SQL 
SELECT
	DATE_TRUNC('hour', order_time) AS hours, count(order_time)AS volume_pizzas

FROM
	pizza_runner.customer_orders

GROUP BY 
	order_time


```

	ANSWER :
2020-01-02T23:00:00.000Z	2
2020-01-08T21:00:00.000Z	1
2020-01-09T23:00:00.000Z	1
2020-01-08T21:00:00.000Z	1
2020-01-01T19:00:00.000Z	1
2020-01-10T11:00:00.000Z	1
2020-01-01T18:00:00.000Z	1
2020-01-08T21:00:00.000Z	1
2020-01-04T13:00:00.000Z	3
2020-01-11T18:00:00.000Z	2


9. What was the volume of orders for each day of the week?


```SQL
SELECT
	To_char(order_time, 'DAY') AS day_of_the_week, count(order_time)AS volume_pizzas

FROM
	pizza_runner.customer_orders

GROUP BY 
	day_of_the_week


```

ANSWER:
WEDNESDAY	5
THURSDAY	3
FRIDAY	1
SATURDAY	5




A. RUNNER AND CUSTOMER EXPERIENCE 


1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

```SQL 
SELECT
	To_char(registration_date, 'w') AS week, count(runner_id)AS runners

FROM
	pizza_runner.runners

GROUP BY 
	week

```

ANSWERS
Week 1: 2 runners signed up, Week 2: 1 runner signed up , Week 3: 1 runner signed up
SQL QUERIES AND ANSWERS - WEEK 1, 8WeekSQlChallenge, Danny's Diner
SOURCE: https://8weeksqlchallenge.com/case-study-1/ 
Kindly refer to the source for background details

1. What is the total amount each customer spent at the restaurant?

```SQL
SELECT
	customer_id,
	SUM(price) AS total_amount_spent
FROM 
	(SELECT
  	sales.customer_id,
    sales.product_id,
    menu.product_name,
    menu.price
	FROM dannys_diner.sales
	INNER JOIN dannys_diner.menu ON sales.product_id = menu.product_id) AS newtable
GROUP BY 
	newtable.customer_id;

```

ANSWER:A spent $76 , B spent $74 and C spent $36


2. How many days has each customer visited the restaurant?

```SQL
SELECT
  	customer_id,
    COUNT (order_date) AS number_of_vists
FROM 
	dannys_diner.sales
GROUP BY 
	customer_id;

```

ANSWER: A - 6 , B -  6 and C - 3

3. What was the first item from the menu purchased by each customer?

```SQL
SELECT
  	sales.customer_id,
    sales.order_date,
    sales.product_id,
    menu.product_name
FROM 
	dannys_diner.sales
INNER JOIN 
	dannys_diner.menu ON sales.product_id = menu.product_id
ORDER BY 
	sales.order_date;

```

ANSWER: A - Curry/Sushi  , B -  Curry  and C - Ramen 

4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```SQL
SELECT
    menu.product_name,
    count(menu.product_name)AS number_of_orders
FROM 
	dannys_diner.sales
INNER JOIN 
	dannys_diner.menu ON sales.product_id = menu.product_id
GROUP BY 
	menu.product_name;


```
ANSWER: Ramen, purchased 8 times  

5. Which item was the most popular for each customer?

```SQL
SELECT 
	COUNT (case when customer_id = 'A' AND product_id ='1' THEN 1 ELSE NULL END) AS a_sushi,
	COUNT (case when customer_id = 'B' AND product_id ='1' THEN 1 ELSE NULL END) AS b_sushi,
	COUNT (case when customer_id = 'C' AND product_id ='1' THEN 1 ELSE NULL END) AS c_sushi,
	COUNT (case when customer_id = 'A' AND product_id ='2' THEN 1 ELSE NULL END) AS a_curry,
	COUNT (case when customer_id = 'B' AND product_id ='2' THEN 1 ELSE NULL END) AS b_curry,
	COUNT (case when customer_id = 'C' AND product_id ='2' THEN 1 ELSE NULL END) AS c_curry,
	COUNT (case when customer_id = 'A' AND product_id ='3' THEN 1 ELSE NULL END) AS a_ramen,
	COUNT (case when customer_id = 'B' AND product_id ='3' THEN 1 ELSE NULL END) AS b_ramen,
	COUNT (case when customer_id = 'C' AND product_id ='3' THEN 1 ELSE NULL END) AS c_ramen

FROM 
	(SELECT
	    sales.customer_id,
	    sales.product_id,
	    menu.product_name
	FROM dannys_diner.sales
	INNER JOIN dannys_diner.menu ON sales.product_id = menu.product_id) as newtable

```
ANSWER A - Ramen  , B -  Sushi/Curry/Ramen (no clear favourite)  and C - Ramen 

6. Which item was purchased first by the customer after they became a member (ONLY CUSTOMER A AND B are members)

```SQL
SELECT
    sales.customer_id,
    sales.product_id,
    sales.order_date,
    members.join_date
FROM 
	dannys_diner.sales
INNER JOIN 
	dannys_diner.members ON sales.customer_id = members.customer_id
WHERE 
	sales.order_date > members.join_date
ORDER BY 
	sales.order_date;
```

ANSWER A - Ramen  , B -  Sushi

7. Which item was purchased just before the customer became a member?

```SQL
SELECT
    sales.customer_id,
    sales.product_id,
    sales.order_date,
    members.join_date
FROM 
	dannys_diner.sales
INNER JOIN 
	dannys_diner.members ON sales.customer_id = members.customer_id
WHERE 
	sales.order_date <  members.join_date
ORDER BY 
	sales.order_date;

```

ANSWER A - Sushi and Curry  , B -  Sushi

8. What is the total items and amount spent for each member before they became a member?

```SQL
SELECT
SUM(price) AS amount_spent,
count(product_name) AS number_of_items,
customer_id

FROM
	(SELECT
	    menu.price,
	    menu.product_name,
	    sales.customer_id,
	    sales.order_date,
	    members.join_date
	FROM 
		dannys_diner.menu
	INNER JOIN 
		dannys_diner.sales ON menu.product_id = sales.product_id
	INNER JOIN    
	    dannys_diner.members ON sales.customer_id = members.customer_id
	WHERE 
		sales.order_date <  members.join_date) AS htable

GROUP BY customer_id;

```

ANSWER A - Bought 2 items and spent $25 , B -  bought 3 items and spent $40

9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```SQL 
SELECT 

	SUM(points_earned) AS total_points_earned,
	customer_id

FROM
	(SELECT
	    menu.price,
	    menu.product_name,
	    sales.customer_id,
	   CASE when menu.product_name = 'sushi' then menu.price * 20 	ELSE menu.price * 10  END AS points_earned
	   
	FROM 
		dannys_diner.menu
	INNER JOIN 
		dannys_diner.sales ON menu.product_id = sales.product_id) AS ptable

GROUP BY
	customer_id;

```

ANSWER A - 860  , B -  940  and C - 360 


10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```SQL 
SELECT 
SUM(points_earned) AS total_points,
customer_id

FROM

(SELECT
    sales.customer_id,
    sales.product_id,
    menu.price,
    sales.order_date,
    menu.price * 20 AS points_earned,
    members.join_date
FROM 
	dannys_diner.menu
INNER JOIN 
	dannys_diner.sales ON menu.product_id = sales.product_id
INNER JOIN    
dannys_diner.members ON sales.customer_id = members.customer_id
WHERE 
	sales.order_date >=  members.join_date AND sales.order_date < '2021-01-31T00:00:00.000Z') AS jtable
    
 GROUP BY
 customer_id;

```
ANSWER A - 1,020  , B -  440   




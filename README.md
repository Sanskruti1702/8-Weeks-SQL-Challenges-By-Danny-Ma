# Case Study 1 - Danny's Diner
![image](https://github.com/Sanskruti1702/Case-study-1/assets/91273577/70f30682-0152-41d5-a636-b0c315fb4704)
Please refer [here](https://8weeksqlchallenge.com/case-study-1/) to view the complete problem statement.
**Schema (PostgreSQL v13)**

    CREATE SCHEMA dannys_diner;
    SET search_path = dannys_diner;
    
    CREATE TABLE sales (
      "customer_id" VARCHAR(1),
      "order_date" DATE,
      "product_id" INTEGER
    );
    
    INSERT INTO sales
      ("customer_id", "order_date", "product_id")
    VALUES
      ('A', '2021-01-01', '1'),
      ('A', '2021-01-01', '2'),
      ('A', '2021-01-07', '2'),
      ('A', '2021-01-10', '3'),
      ('A', '2021-01-11', '3'),
      ('A', '2021-01-11', '3'),
      ('B', '2021-01-01', '2'),
      ('B', '2021-01-02', '2'),
      ('B', '2021-01-04', '1'),
      ('B', '2021-01-11', '1'),
      ('B', '2021-01-16', '3'),
      ('B', '2021-02-01', '3'),
      ('C', '2021-01-01', '3'),
      ('C', '2021-01-01', '3'),
      ('C', '2021-01-07', '3');
     
    
    CREATE TABLE menu (
      "product_id" INTEGER,
      "product_name" VARCHAR(5),
      "price" INTEGER
    );
    
    INSERT INTO menu
      ("product_id", "product_name", "price")
    VALUES
      ('1', 'sushi', '10'),
      ('2', 'curry', '15'),
      ('3', 'ramen', '12');
      
    
    CREATE TABLE members (
      "customer_id" VARCHAR(1),
      "join_date" DATE
    );
    
    INSERT INTO members
      ("customer_id", "join_date")
    VALUES
      ('A', '2021-01-07'),
      ('B', '2021-01-09');

---

**Query #1** What is the total amount each customer spent at the restaurant?

    SELECT s.customer_id, Sum(m.price) AS total_price
        FROM dannys_diner.sales AS s 
        JOIN dannys_diner.menu AS m ON s.product_id=m.product_id
        GROUP BY s.customer_id
        ORDER BY s.customer_id ASC;

| customer_id | total_price |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

---
**Query #2** How many days has each customer visited the restaurant?

    SELECT customer_id, count (distinct order_date)
    FROM dannys_diner.sales  
    GROUP BY customer_id;

| customer_id | count |
| ----------- | ----- |
| A           | 4     |
| B           | 6     |
| C           | 2     |

---
**Query #3** What was the first item from the menu purchased by each customer?

    WITH ordered_sales AS (
      SELECT 
        sales.customer_id, 
        sales.order_date, 
        menu.product_name,
        DENSE_RANK() OVER (
          PARTITION BY sales.customer_id 
          ORDER BY sales.order_date) AS rank
      FROM dannys_diner.sales
      INNER JOIN dannys_diner.menu
        ON sales.product_id = menu.product_id
    )
    
    SELECT 
      customer_id, 
      product_name
    FROM ordered_sales
    WHERE rank = 1
    GROUP BY customer_id, product_name;

| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

---
**Query #4** What is the most purchased item on the menu and how many times was it purchased by all customers?

    SELECT menu.product_name, COUNT(sales.product_id) AS most_purchased_item
    FROM dannys_diner.sales 
    JOIN dannys_diner.menu  ON sales.product_id=menu.product_id
    GROUP BY menu.product_name
    ORDER BY most_purchased_item DESC
    LIMIT 1;

| product_name | most_purchased_item |
| ------------ | ------------------- |
| ramen        | 8                   |

---
**Query #5** Which item was the most popular for each customer?

    WITH most_popular_sales AS (
      SELECT 
        sales.customer_id, menu.product_name,
        COUNT (menu.product_id) AS order_count,
        DENSE_RANK() OVER (
          PARTITION BY sales.customer_id 
          ORDER BY COUNT(sales.customer_id) DESC) AS rank
      FROM dannys_diner.sales
      INNER JOIN dannys_diner.menu
        ON sales.product_id = menu.product_id
      GROUP BY sales.customer_id,menu.product_name
    )
    
    SELECT 
      customer_id,product_name,order_count  
    FROM most_popular_sales
    WHERE Rank=1;

| customer_id | product_name | order_count |
| ----------- | ------------ | ----------- |
| A           | ramen        | 3           |
| B           | ramen        | 2           |
| B           | curry        | 2           |
| B           | sushi        | 2           |
| C           | ramen        | 3           |

---
**Query #6** Which item was purchased just before the customer became a member?

    WITH orders_by_members AS(
      SELECT sales.product_id,members.customer_id,
      ROW_NUMBER() OVER (
        PARTITION BY members.customer_id
        ORDER BY sales.order_date) AS row_num
      FROM dannys_diner.sales
      JOIN dannys_diner.members ON sales.customer_id=members.customer_id
          AND sales.order_date>members.join_date)
    SELECT customer_id,product_name
    FROM orders_by_members
    JOIN dannys_diner.menu ON orders_by_members.product_id=menu.product_id
    WHERE row_num=1
    ORDER BY customer_id ASC ;

| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | sushi        |

---
**Query #7** What is the total items and amount spent for each member before they became a member?

    WITH orders_by_members AS(
      SELECT sales.product_id,members.customer_id,
      ROW_NUMBER() OVER (
        PARTITION BY members.customer_id
        ORDER BY sales.order_date DESC) AS row_num
      FROM dannys_diner.sales
      JOIN dannys_diner.members ON sales.customer_id=members.customer_id
          AND sales.order_date<members.join_date)
    SELECT customer_id,product_name
    FROM orders_by_members
    JOIN dannys_diner.menu ON orders_by_members.product_id=menu.product_id
    WHERE row_num=1
    ORDER BY customer_id ASC ;

| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| B           | sushi        |

---
**Query #8**  What is the total items and amount spent for each member before they became a member?

    SELECT sales.customer_id,
           COUNT(sales.product_id)AS total_items,
           SUM(menu.price) AS total_sales 
    FROM dannys_diner.sales 
    INNER JOIN dannys_diner.members 
      ON sales.customer_id=members.customer_id
      AND sales.order_date < members.join_date
    INNER JOIN dannys_diner.menu 
      ON sales.product_id=menu.product_id
    GROUP BY sales.customer_id
    ORDER BY sales.customer_id;

| customer_id | total_items | total_sales |
| ----------- | ----------- | ----------- |
| A           | 2           | 25          |
| B           | 3           | 40          |

---
**Query #9** If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

    WITH total_pints_cte AS(
      SELECT menu.product_id,
      CASE WHEN product_id=1 THEN price*20
      ELSE price*10 END AS total_points
      FROM dannys_diner.menu )
    SELECT sales.customer_id,SUM (total_pints_cte.total_points) as points
    FROM dannys_diner.sales
    JOIN total_pints_cte ON sales.product_id=total_pints_cte.product_id
    GROUP BY sales.customer_id
    ORDER BY sales.customer_id;

| customer_id | points |
| ----------- | ------ |
| A           | 860    |
| B           | 940    |
| C           | 360    |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/4778)

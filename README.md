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

**Query #1**

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
**Query #2**

    SELECT customer_id, count (distinct order_date)
    FROM dannys_diner.sales  
    GROUP BY customer_id;

| customer_id | count |
| ----------- | ----- |
| A           | 4     |
| B           | 6     |
| C           | 2     |

---
**Query #3**

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
**Query #4**

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
**Query #5**

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

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/4778)

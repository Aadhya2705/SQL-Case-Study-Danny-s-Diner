# SQL Case Study- Danny's Diner 
Danny wants to use data to answer a few questions about his customers, especially about their visiting patterns and also which menu item is their favourite. Answering these questions will enable Danny to understand his customer base better and provide a more personalised experience to them! 


   Case Study Questions
  

 1. What is the total amount each customer spent at the restaurant?
 2. How many days has each customer visited the restaurant?
 3. What was the first item from the menu purchased by each customer?
 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
 5. Which item was the most popular for each customer?
 6. Which item was purchased first by the customer after they became a member?
 7. Which item was purchased just before the customer became a member?
 8.  8. What is the total items and amount spent for each member before they became a member?
 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?


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
      
1. What is the total amount each customer spent at the restaurant?
    
**Query #1**

    SELECT sales.customer_id, SUM(price) AS total_amount_spent
    FROM dannys_diner.sales 
    JOIN dannys_diner.menu 
    ON sales.product_id = menu.product_id
    GROUP BY sales.customer_id;

| customer_id | total_amount_spent |
| ----------- | ------------------ |
| B           | 74                 |
| C           | 36                 |
| A           | 76                 |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)



2. How many days has each customer visited the restaurant?

**Query #2**
   
    SELECT sales.customer_id, COUNT(distinct order_date) as days 
    FROM dannys_diner.sales
    GROUP BY sales.customer_id;

| customer_id | days |
| ----------- | ---- |
| A           | 4    |
| B           | 6    |
| C           | 2    |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)

 3. What was the first item from the menu purchased by each customer?

**Query #3**

    SELECT 
    p.customer_id,p.product_name AS first_product
    FROM
    (SELECT s.*,m.product_name,ROW_NUMBER() OVER (partition by customer_id order by order_date) AS rnk
    FROM dannys_diner.sales as s
    JOIN dannys_diner.menu as m
    ON s.product_id = m.product_id) AS p
    WHERE p.rnk=1;

| customer_id | first_product |
| ----------- | ------------- |
| A           | curry         |
| B           | curry         |
| C           | ramen         |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)


---

4. What is the most purchased item on the menu and how many times was it purchased by all customers?

**Query #4**

    SELECT 
    sales.product_id,
    menu.product_name,
    Count(sales.product_id) as number_pur
    FROM dannys_diner.sales 
    JOIN dannys_diner.menu 
    ON sales.product_id= menu.product_id
    GROUP BY menu.product_name, sales.product_id;

| product_id | product_name | number_pur |
| ---------- | ------------ | ---------- |
| 2          | curry        | 4          |
| 3          | ramen        | 8          |
| 1          | sushi        | 3          |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)



---

 5. Which item was the most popular for each customer?

**Query #5**

    SELECT 
    sales.customer_id ,
    menu.product_name,
    count(sales.product_id) as order_count
    FROM dannys_diner.sales 
    JOIN dannys_diner.menu 
    ON sales.product_id=menu.product_id
    GROUP BY sales.customer_id,menu.product_name
    ORDER BY sales.customer_id;

| customer_id | product_name | order_count |
| ----------- | ------------ | ----------- |
| A           | curry        | 2           |
| A           | ramen        | 3           |
| A           | sushi        | 1           |
| B           | curry        | 2           |
| B           | ramen        | 2           |
| B           | sushi        | 2           |
| C           | ramen        | 3           |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)


---

 6. Which item was purchased first by the customer after they became a member?

**Query #6**

    SELECT 
    X.product_name, X.customer_id,X.rnk
    FROM 
    (SELECT 
    s.*,m.product_name,Rank() over (partition by s.customer_id order by order_date) as rnk
    FROM dannys_diner.sales s
    JOIN dannys_diner.menu m
    ON s.product_id=m.product_id 
    JOIN dannys_diner.members mem
    ON s.customer_id=mem.customer_id
    WHERE order_date>=join_date) as X 
    WHERE X.rnk=1;

| product_name | customer_id | rnk |
| ------------ | ----------- | --- |
| curry        | A           | 1   |
| sushi        | B           | 1   |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)

---

 7. Which item was purchased just before the customer became a member?
 
**Query #7**

    SELECT 
    X.product_name, X.customer_id,X.rnk
    FROM
    (SELECT 
    s.*,m.product_name,Rank() over (partition by s.customer_id order by order_date) as rnk
    FROM dannys_diner.sales s
    JOIN dannys_diner.menu m
    ON s.product_id=m.product_id 
    JOIN dannys_diner.members mem
    ON s.customer_id=mem.customer_id
    WHERE order_date<join_date) as X 
    WHERE X.rnk=1;

| product_name | customer_id | rnk |
| ------------ | ----------- | --- |
| sushi        | A           | 1   |
| curry        | A           | 1   |
| curry        | B           | 1   |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)



---

 8.  8. What is the total items and amount spent for each member before they became a member?

**Query #8**

    SELECT 
    s.customer_id,Count(m.product_name) as total_items,SUM(m.price) as total_amount
    FROM dannys_diner.sales s
    JOIN dannys_diner.menu m
    ON s.product_id=m.product_id
    JOIN dannys_diner.members mem
    ON s.customer_id=mem.customer_id
    WHERE order_date<join_date 
    GROUP BY s.customer_id
    ORDER BY s.customer_id;

| customer_id | total_items | total_amount |
| ----------- | ----------- | ------------ |
| A           | 2           | 25           |
| B           | 3           | 40           |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)

---

 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

**Query #9**

    SELECT 
    X.customer_id, SUM(X.points)
    FROM 
    (SELECT s.customer_id, 
    CASE WHEN m.product_name = 'sushi' THEN  2*m.price*10
    ELSE m.price*10
    END AS points
    FROM dannys_diner.sales s
    JOIN dannys_diner.menu m
    ON s.product_id = m.product_id) as X 
    GROUP BY X.customer_id
    ORDER BY X.customer_id;

| customer_id | sum |
| ----------- | --- |
| A           | 860 |
| B           | 940 |
| C           | 360 |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)


---

**JOIN EVERYTHING**

    SELECT 
    	s.customer_id,s.order_date,m.product_name,m.price,
     	CASE WHEN (s.order_date >= me.join_date) AND (s.customer_id = me.customer_id) THEN 'Y'
     	ELSE 'N'
     	END AS member
    FROM dannys_diner.sales s
    JOIN dannys_diner.menu m
    ON s.product_id = m.product_id
    LEFT JOIN dannys_diner.members me
    ON s.customer_id = me.customer_id 
    GROUP BY s.customer_id,s.order_date,m.product_name,m.price,me.join_date,me.customer_id;

| customer_id | order_date               | product_name | price | member |
| ----------- | ------------------------ | ------------ | ----- | ------ |
| A           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 10    | N      |
| A           | 2021-01-07T00:00:00.000Z | curry        | 15    | Y      |
| A           | 2021-01-10T00:00:00.000Z | ramen        | 12    | Y      |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      |
| B           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-02T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-04T00:00:00.000Z | sushi        | 10    | N      |
| B           | 2021-01-11T00:00:00.000Z | sushi        | 10    | Y      |
| B           | 2021-01-16T00:00:00.000Z | ramen        | 12    | Y      |
| B           | 2021-02-01T00:00:00.000Z | ramen        | 12    | Y      |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |
| C           | 2021-01-07T00:00:00.000Z | ramen        | 12    | N      |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)

<!-- query 8,  -->
<!-- rank everything.  -->

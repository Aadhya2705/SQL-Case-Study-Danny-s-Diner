# SQL Case Study- Danny's Diner 
Danny wants to use data to answer a few questions about his customers, especially about their visiting patterns and also which menu item is their favourite. Answering these questions will enable Danny to understand his customer base better and provide a more personalised experience to them! 

1. What is the total amount each customer spent at the restaurant?

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

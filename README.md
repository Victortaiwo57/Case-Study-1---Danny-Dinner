# Case-Study-1---Danny-Dinner
This study reveals insights into customer visits, spending habits, and favorite items. These findings can help personalize experiences for loyal customers and support effective customer retention strategies.


#  Case Study #1: Danny's Diner 


## 📚 Table of Contents
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Question and Solution](#question-and-solution)

Data source: [here](https://8weeksqlchallenge.com/case-study-1/). 

***

## Business Task
This study provides valuable insights into customer behavior — including their visit patterns, spending habits, and favorite menu items. By understanding these key areas, the business can deliver a more personalized and rewarding experience to its loyal customers. These insights can also be used to design targeted customer retention strategies and improve overall customer satisfaction.
***

## Entity Relationship Diagram

![image](https://github.com/Victortaiwo57/Case-Study-1---Danny-Dinner/blob/main/danny_dinner.png))

***

## Question and Solution

I will be answering the following questions using MySQL Workbench, which has slight differences compared to other SQL databases. [DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138). 

In case of any question or feedback, reach out to me on [LinkedIn](https://www.linkedin.com/in/victor-taiwo-076094167/).

Please note: All monetary values in this query are represented in US dollars ($).

**1. What is the total amount each customer spent at the restaurant?**

````sql
SELECT
	s.customer_id,
	SUM(m.price) AS total_spent
    FROM dannys_diner.sales s
	JOIN menu m 
	ON m.product_id = s.product_id
    GROUP BY s.customer_id
	ORDER BY Total_spent DESC;
````

#### Actions:
- The query joins the `sales` and `menu` tables on `product_id` to retrieve item prices.
- It calculates the total amount spent by each customer using `SUM(menu.price)` and groups by `customer_id`.
- The result is ordered by total spending in descending order to show the highest spenders first.

#### Answer:
| customer_id | total_spent |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

***

**2. How many days has each customer visited the restaurant?**

````sql
SELECT
	customer_id, 
	COUNT(DISTINCT order_date) AS visit_count
    FROM dannys_diner.sales
    GROUP BY customer_id;
````

#### Actions:
- It uses **COUNT(DISTINCT `s.order_date`)** to count how many unique days each customer visited the restaurant..
- Results are grouped by `customer_id` and sorted in descending order of visit frequency.

#### Answer:
| customer_id | visit_count |
| ----------- | ----------- |
| B           | 6          |
| A           | 4          |
| C           | 2          |

***

**3. What was the first item from the menu purchased by each customer?**

````sql
-- Step 1: Define a CTE named `first_item`
WITH  first_item AS (
SELECT
	s.customer_id AS Customer,
	s.product_id,
	s.order_date,
	m.product_name AS Product,
	
	-- Step 2: Create a new column `rank` using ROW_NUMBER() or rank()
	-- Partition the data by customer_id and order rows by order_date
	ROW_NUMBER() OVER(PARTITION BY s.customer_id --
	ORDER BY s.order_date) AS rank_
    FROM dannys_diner.sales s
       	JOIN menu m ON m.product_id = s.product_id
       	)

-- Step 3: Select only the first-ranked purchase for each customer
SELECT
	Customer, 
	Product
    FROM first_item
     	WHERE rank_ =1;

````

#### Steps:
- There can be two interpretations based on assumptions:
    1. If `order_date` reflects a proper timestamp (i.e., chronological order),
   then the use of **ROW_NUMBER()** is appropriate to determine the first purchase per customer.
    2. if multiple products were purchased on the same day, we shall consider all of them as       potential "first" purchases. Therefore, we retrieve all items bought on the customer's
    earliest order date. The **RANK()** window function is used instead of **ROW_NUMBER()**, 
   as it
    assigns the same rank to items with identical order dates.
-  The Common Table Expression (CTE) `first_item` ranks all orders made by each customer by
  order_date.
-  **ROW_NUMBER()** is used to assign a unique rank to each purchase, even if multiple 
   purchases occurred on the same date — only one row will have rank = 1.
-  In the final `SELECT`, we retrieve the first item each customer purchased based on their
   earliest order_date.

#### Answer (First condition):
| customer | product | 
| ----------- | ----------- | 
| A           | sushi        | 
| B           | curry        | 
| C           | ramen        |

#### Answer (second condition):
| customer | product | 
| ----------- | ----------- |
| A           | curry        | 
| A           | sushi        | 
| B           | curry        | 
| C           | ramen        |


***

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

````sql
SELECT
      	m.product_name AS Product, 
	COUNT(s.order_date) AS Product_count
    FROM dannys_diner.sales s
	JOIN menu m 
	ON m.product_id = s.product_id
    GROUP BY Product
	ORDER BY Product_count DESC
	LIMIT 1;

````

#### Actions:
- **COUNT(`s.order_date`)** counts how many times each product was ordered.
- **GROUP BY** ensures the count is calculated per product.
- **ORDER BY ... DES**C sorts the products from most to least purchased.
= LIMIT 1 restricts the result to just the most purchased item.

#### Answer:
| Product | Product_count | 
| ----------- | ----------- |
| ramen       | 8           |

***

**5. Which item was the most popular for each customer?**

````sql
WITH popular_item AS (
SELECT
	s.customer_id AS Customer,
	m.product_name AS Product, 
	COUNT(s.order_date) AS Product_count,
	RANK() over(PARTITION BY s.customer_id
	ORDER BY count(s.order_date) DESC) as rank_
    FROM dannys_diner.sales s
	JOIN menu m 
	ON m.product_id = s.product_id
    GROUP BY s.customer_id, Product
      )
SELECT
	Customer, 
	Product,
      	Product_count
    FROM popular_item
	WHERE rank_ = 1;

````

*Each user may have more than 1 favourite item.*

#### Actions:
- The CTE `popular_item` groups the data by each customer and product.
- **COUNT(`s.order_date`)** counts how often each product was purchased by the customer.
- RANK() is used to rank the products in descending order of frequency for each customer.
- PARTITION BY separates the rankings for each customer.

#### Answer:
| customer_id | product_name | order_count |
| ----------- | ---------- |------------  |
| A           | ramen        |  3   |
| B           | sushi        |  2   |
| B           | curry        |  2   |
| B           | ramen        |  2   |
| C           | ramen        |  3   |

***

**6. Which item was purchased first by the customer after they became a member?**

```sql
WITH purchase_after AS (
SELECT
      	s.customer_id AS Customer,
	m.product_name AS Product, 
	s.order_date,
	RANK() OVER(PARTITION BY s.customer_id
	ORDER BY s.order_date ASC) as rank_
    FROM dannys_diner.sales s
	JOIN menu m 
	ON m.product_id = s.product_id
	JOIN members me
	ON me.customer_id =s.customer_id
	WHERE order_date >= join_date -- Items Purchased on the Day the Customer Joined
	-- WHERE order_date >= join_date -- For Items Purchased After the Customer Joined
)
SELECT
	Customer, 
	Product
    FROM purchase_after
	WHERE rank_ = 1;

```

#### Actions:

- Filtered records where `order_date > join_date`.
- Applied the **RANK()** window function to rank post-join purchases by date.
- Selected the first post-join purchase per customer.

#### Answer (Items Purchased on the Day the Customer Joined):
| customer_id | product_name |
| ----------- | ---------- |
| A           | curry        |
| B           | sushi        |
  
#### Answer (Items Purchased After the Customer Joined):
| customer_id | product_name |
| ----------- | ---------- |
| A           | ramen        |
| B           | sushi        |


***

**7. Which item was purchased just before the customer became a member?**

````sql
WITH purchase_after AS (
SELECT
	s.customer_id AS Customer,
  	m.product_name AS Product, 
  	s.order_date,
  	RANK() OVER(PARTITION BY s.customer_id
      	ORDER BY s.order_date DESC) as rank_
    FROM dannys_diner.sales s
  	JOIN menu m 
  	ON m.product_id = s.product_id
  	JOIN members me
      	ON me.customer_id =s.customer_id
	WHERE order_date < join_date
 )
SELECT
      	Customer, 
	Product
    FROM purchase_after
      	WHERE rank_ = 1;

````

#### Actions:
- Join Tables: Combined sales, menu, and members tables using `customer_id`.
- Filter: Restricted results to orders where order_date < join_date (i.e., made before becoming 
  a member).
- Rank Purchases: Used the **RANK()** window function with **PARTITION BY `customer_id`** and ORDER BY 
  `order_date` DESC to assign a rank.
- This ensures the most recent purchase before joining gets rank = 1.
- Final Output: Selected the top-ranked item for each customer to identify their last pre- 
  membership purchase.

#### Answer:
| customer_id | product_name |
| ----------- | ----------   |
| A           | sushi        |
| A           | curry        |
| B           | sushi        |


***

**8. What is the total items and amount spent for each member before they became a member?**

```sql
SELECT
      	s.customer_id AS Customer,
	COUNT(s.order_date) AS Total_item,
      	SUM(m.price) AS Total_amount
    FROM dannys_diner.sales s
  	JOIN menu m 
  	ON m.product_id = s.product_id
  	JOIN members me
	ON me.customer_id =s.customer_id
	WHERE s.order_date < me.join_date -- you can change "where' clause to
      	-- "And" to get the same result
    GROUP BY Customer
	ORDER BY Total_amount;

```

#### Actions:
- Join Tables: Joined sales, menu, and members using `customer_id` to bring in product prices 
  and membership dates.
- Filter: Applied a WHERE clause: `s.order_date < me.join_date` to focus on only purchases made 
  before joining.
- Aggregate:
  - **COUNT(`s.order_date`)** to count total purchase transactions (items).
  - **SUM(`m.price`)** to compute total amount spent on those items.
- Group and Order:
  - Grouped results by customer.
  - Sorted the results by total amount spent (ascending).

#### Answer:
| customer_id | total_items | total_sales |
| ----------- | ---------- |----------  |
| B           | 3          |  40       |
| A           | 2          |  25       |

***

**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?**

```sql
WITH point_rank AS (
SELECT
	m.product_id AS Product,
	CASE
		WHEN m.product_ID = 1 THEN 20 * m.price
		ELSE 10 * m.price END points
    FROM dannys_diner.menu m
	)
SELECT
	s.customer_id AS customer,
	SUM(points) as total_point
    FROM point_rank
	JOIN sales s 
	ON s.product_id = point_rank.Product
    GROUP BY customer;

```

#### Actions:
CTE (`point_rank`):   Assigns point value per product:
                    If `product_id` = 1 (sushi), then points = 20 * price
                    Else, points = 10 * price
                    But it doesn't yet associate this with actual sales (customers or     
                    quantities purchased).

Main Query: Joins the CTE with the sales table to associate each purchase with the calculated               point value.
            Sums points per customer using **SUM(points)**.

#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 860 |
| B           | 940 |
| C           | 360 |

***

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?**

```sql
WITH dates_cte AS (
SELECT 
	customer_id, 
	join_date, 
	DATE_ADD(join_date, INTERVAL 6 DAY) AS valid_date, 
	LAST_DAY('2021-01-31') AS last_date
    FROM dannys_diner.members
)
SELECT 
	sales.customer_id, 
	SUM(CASE
		WHEN menu.product_name = 'sushi' THEN 2 * 10 * menu.price
		WHEN sales.order_date BETWEEN dates.join_date AND dates.valid_date THEN 2 * 10 *
		menu.price ELSE 10 * menu.price END) AS points
    FROM dannys_diner.sales
	JOIN dates_cte AS dates
	ON sales.customer_id = dates.customer_id
	AND sales.order_date BETWEEN dates.join_date AND dates.last_date
	JOIN dannys_diner.menu
	ON sales.product_id = menu.product_id
    GROUP BY sales.customer_id;

```

#### Assumptions:
- From the day before membership (Day -X) up to the membership start date (Day 1), members earn 
  10 points per $1 spent, except for sushi, which earns 20 points per $1.
- During the first 7 days of membership (Day 1 to Day 7), all purchases earn 20 points per $1, 
  regardless of the item.
- From Day 8 to the end of January 2021, members earn 10 points per $1 on all items, except 
  sushi, which continues to earn double points (20 points per $1).

#### Actions:
- A Common Table Expression (CTE) named `dates_cte` is created to:
  - Select each customer's `join_date`.
  - Calculate valid_date as 7 days after joining.
  - Define last_date as January 31, 2021.
- The main query:
  - Joins the sales, menu, and dates_cte tables using `customer_id`.
  - Filters sales records to include only those between the customer's `join_date` and         
    last_date 
    (January 31).
  - Uses a CASE statement to assign loyalty points:
    - 2 × 10 × price for sushi
    - 2 × 10 × price for any purchase within 7 days of joining
    - 10 × price for all other purchases 
  - Groups the results by `customer_id`.
  - Returns the total points earned by each customer within the specified period.

#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 1020 |
| B           | 320 |



***


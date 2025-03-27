**Case-Study-4 - Data-Bank**

**Introduction**

There is a new innovation in the financial industry called Neo-Banks: new aged digital only banks without physical branches.

Danny thought that there should be some sort of intersection between these new age banks, cryptocurrency and the data world…so he decides to launch a new initiative - Data Bank!

Data Bank runs just like any other digital bank - but it isn’t only for banking activities, they also have the world’s most secure distributed data storage platform!

Customers are allocated cloud data storage limits which are directly linked to how much money they have in their accounts. There are a few interesting caveats that go with this business model, and this is where the Data Bank team need your help!

The management team at Data Bank want to increase their total customer base - but also need some help tracking just how much data storage their customers will need.

This case study is all about calculating metrics, growth and helping the business analyse their data in a smart way to better forecast and plan for their future developments!

**Available Data**

The Data Bank team have prepared a data model for this case study as well as a few example rows from the complete dataset below to get you familiar with their tables.

**Table 1: Regions**

Just like popular cryptocurrency platforms - Data Bank is also run off a network of nodes where both money and data is stored across the globe. In a traditional banking sense - you can think of these nodes as bank branches or stores that exist around the world.

This regions table contains the region_id and their respective region_name values

| region_id	| region_name |
|-----------|-------------|
| 1	| Africa    |
| 2	| America   |  
| 3	| Asia   |
| 4	| Europe  | 
| 5	| Oceania  |

**Table 2: Customer Nodes**

Customers are randomly distributed across the nodes according to their region - this also specifies exactly which node contains both their cash and data.

This random distribution changes frequently to reduce the risk of hackers getting into Data Bank’s system and stealing customer’s money and data!

Below is a sample of the top 10 rows of the data_bank.customer_nodes

| customer_id	| region_id	| node_id	| start_date |	end_date |
|-------------|-----------|---------|------------|-----------|
|  1	|  3	| 4 |	2020-01-02	| 2020-01-03 |
| 2	| 3	| 5 |	2020-01-03 |	2020-01-17 |
| 3	| 5	| 4 |	2020-01-27	| 2020-02-18 |
| 4	 | 5 |	4 |	2020-01-07	| 2020-01-19 |
| 5 |	3 |	3	| 2020-01-15	| 2020-01-23 |
| 6 |	1 |	1	| 2020-01-11	| 2020-02-06 |
| 7 |	2 |	5	| 2020-01-20	| 2020-02-04 |
| 8 |	1 |	2	| 2020-01-15	| 2020-01-28 |
| 9 |	4 |	5	| 2020-01-21	| 2020-01-25 |
| 10 |	3 |	4 |	2020-01-13	| 2020-01-14 |

**Table 3: Customer Transactions**

This table stores all customer deposits, withdrawals and purchases made using their Data Bank debit card.

| customer_id	| txn_date	| txn_type	 | txn_amount |
|-------------|-----------|------------|------------|
| 429 |	2020-01-21 |	deposit |	82 |
| 155 |	2020-01-10	| deposit	| 712 |
| 398	| 2020-01-01	| deposit	| 196 |
| 255	| 2020-01-14	| deposit	| 563 |
| 185	| 2020-01-29	| deposit	| 626 |
| 309	| 2020-01-13	| deposit	| 995 |
| 312	| 2020-01-20	| deposit	| 485 |
| 376	| 2020-01-03	| deposit	| 706 |
| 188	| 2020-01-13	| deposit	| 601 |
| 138	| 2020-01-11	| deposit	| 520 |

**Case Study Questions**

The following case study questions include some general data exploration analysis for the nodes and transactions before diving right into the core business questions and finishes with a challenging final request!

**A. Customer Nodes Exploration**

How many unique nodes are there on the Data Bank system?

```sql
SELECT COUNT(DISTINCT(node_id)) AS Unique_nodes
              FROM customer_nodes
```

| unique_nodes |
|--------------|
|         5 |

What is the number of nodes per region?

```SQL
SELECT c.region_id,
       r.region_name,
       COUNT(c.node_id) AS Nodes
   FROM customer_nodes c
JOIN regions r ON c.region_id = r.region_id
GROUP BY c.region_id, r.region_name
ORDER BY c.region_id

```

| region_id | region_name |	nodes |
|-----------|-------------|-------|
| 1	| Australia |	                     770 |
| 2	| America  |	                    735 |
| 3	 |Africa  |                       	714 |
| 4	 | Asia	  |                     665 |
| 5	  | Europe |                     616 |

How many customers are allocated to each region?

```SQL
SELECT c.region_id,
       r.region_name,
       COUNT(c.customer_id) AS Customers
           FROM customer_nodes c
JOIN regions r ON c.region_id = r.region_id
GROUP BY c.region_id, r.region_name
ORDER BY c.region_id

```
| region_id | region_name  |  Customers |
|-----------|--------------|------------|
| 1	 |  Australia  |	770 |
| 2	  |  America  |	735   |
| 3	  |  Africa  |  	714   |
|  4	| Asia  |	            665 |
| 5	| Europe |	616  |

How many days on average are customers reallocated to a different node?

```sql
with days_per_node AS (SELECT customer_id, 
                       (end_date-start_date) AS node_days
                  FROM customer_nodes
                  WHERE end_date != '9999-12-31')

SELECT ROUND(AVG(node_days)) AS days_per_node
    FROM days_per_node
```



| days_per_node |
|---------------|
|      15       |

What is the median, 80th and 95th percentile for this same reallocation days metric for each region?

```SQL
with days_per_node AS (SELECT customer_id,
                              region_id, 
                              (end_date-start_date) AS node_days
                     FROM customer_nodes
                     WHERE end_date != '9999-12-31')

SELECT r.region_name,
       PERCENTILE_CONT(0.5) within group (ORDER BY n.node_days) as median,
        PERCENTILE_CONT(0.8) within group (ORDER BY n.node_days) as "80_percentile",
          PERCENTILE_CONT(0.95) within group (ORDER BY n.node_days) as "95_percentile"
FROM days_per_node n
JOIN regions r ON n.region_id = r.region_id
GROUP BY r.region_name 
```

| region_name |	median |	 80_percentile |	95_percentile |
|-------------|--------|-----------------|----------------|
| Africa |                 	15	  |            24	  |                  28  |
| America |	            15	       |       23	       |             28  |
| Asia |                  	15	   |           23	    |                28 |
| Australia |	           15	        |      23	        |            28 |
| Europe |	           15	          |    24	          |          28 |


**B. Customer Transactions**

What is the unique count and total amount for each transaction type?

```SQL
alter table customer_transactions
add id serial;
```

```SQL
SELECT txn_type, 
       COUNT(id) AS total_txn, 
       SUM(txn_amount) AS total_amount
              FROM customer_transactions
               GROUP BY txn_type
```

| txn_type |	total_txn |	total_amount |
|----------|------------|--------------|
| purchase |    1617 |	806537 |
| withdrawal |	     1580 |	793003 |
| deposit |	     2671 |	1359168 |

What is the average total historical deposit counts and amounts for all customers?

```SQL
with totals AS (SELECT customer_id,
                COUNT(txn_type) AS total_txn,
               SUM(txn_amount) AS total_amount
FROM customer_transactions
WHERE txn_type = 'deposit'
GROUP BY customer_id)

SELECT round(AVG(total_txn)) AS avg_txn,
       round(AVG(total_amount)) AS avg_amount
FROM totals
```

| avg_txn |   avg_amount   |
|---------|----------------|
|  5	  |      2718      |

For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?

```SQL
with customer_txn AS
   (with total_txn AS 
	    (with txn_per_month AS (SELECT customer_id, 
         CASE WHEN txn_type = 'deposit' THEN 1
	     ELSE 0 
	     END AS deposits,
	       CASE WHEN txn_type = 'withdrawal' THEN 1 
	       ELSE 0 
	       END AS withdrawals, 
		       CASE WHEN txn_type = 'purchase' THEN 1 
		  	   ELSE 0 
			   END AS purchases,
			      to_char(txn_date, 'MM-YYYY') AS txn_month
	       FROM customer_transactions)
	
SELECT customer_id, 
       txn_month, 
	   SUM(deposits) AS total_deposits, 
	   SUM(purchases) AS total_purchases,
	   SUM(withdrawals) AS total_withdrawals
FROM txn_per_month 
GROUP BY txn_month, customer_id
ORDER BY txn_month)

    SELECT txn_month, customer_id
      FROM total_txn
       WHERE total_deposits >=2
	   AND (total_purchases >=1 OR total_withdrawals >=1)
	   GROUP BY txn_month, customer_id
	   ORDER BY txn_month) 

SELECT txn_month, COUNT(customer_id) AS total_customers
    FROM customer_txn
	GROUP BY txn_month 
	ORDER BY txn_month
```
					   
	
| txn_month |	total_customers |
|-----------|-----------------|
| 01-2020 |	    168 |
| 02-2020 |    181 |
| 03-2020 |    192 |
| 04-2020 |	    70 |


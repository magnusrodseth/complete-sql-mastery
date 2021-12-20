# Code With Mosh: Complete SQL Mastery

## Retrieving Data from a Single Table

### The `SELECT` statement

#### Using a database

Use the `USE` keyword to select a database to perform queries against.

```sql
USE sql_store;
```

#### Selecting data

Use the `SELECT` keyword to select data from a table.

```sql
SELECT *
FROM customers;

SELECT customer_id, first_name
FROM customers;
```

#### Filter data

Use the `WHERE` keyword to filter data.

```sql
SELECT customer_id, first_name
FROM customers
WHERE customer_id = 1;
```

#### Sort result on a given column

Use the `ORDER BY` keyword to sort results on a given column.

```sql
SELECT customer_id, first_name
FROM customers
ORDER BY first_name;
```

#### Comment out segments

Use `--` to comment out queries.

```sql
SELECT customer_id, first_name
FROM customers
-- ORDER BY first_name;
```

### The `SELECT` Clause

#### Arithmetic operations

It is possible to perform arithmetic operations on data.

```sql
SELECT last_name, first_name, points, points + 10
FROM customers

SELECT last_name,
       first_name,
       points,
       (points * 10) + 100
FROM customers;
```

This will print out each customer's `points`, and then their `points + 10`.

#### Aliases

We can give the new column an alias using the `AS` keyword.

```sql
SELECT last_name,
       first_name,
       points,
       (points + 10) * 100 AS discount_factor
FROM customers;
```

#### Remove duplicates

We can remove duplicates using the `DISTINCT` keyword.

```sql
SELECT DISTINCT state
FROM customers;
```

### The `WHERE` Clause

#### Used as a conditional

The `WHERE` statement is used as a conditional when iterating over records.

```sql
SELECT *
FROM customers
WHERE points > 3000;

SELECT *
FROM customers
WHERE state = 'VA';
```

All ordinary comparison operator are included.

#### Filter on dates

We can use the **standard notation for dates** when filtering for dates.

```sql
SELECT *
FROM customers
WHERE birth_date > '1990-01-01';
```

### The `AND`, `OR` and `NOT` operators

We can combine multiple search conditions when filtering data.

```sql
-- Both conditions must be true
SELECT *
FROM customers
WHERE birth_date > '1990-01-01'
  AND points > 1000;

-- One condition must be true
SELECT *
FROM customers
WHERE birth_date > '1990-01-01'
  OR points > 1000;

-- Conditionals can be chained together
SELECT *
FROM customers
WHERE birth_date > '1990-01-01'
   OR (points > 1000 AND state = 'VA');
```

Note that `AND` has a higher precedence than `OR` by default.
We can override this precedence using parenthesis `()`.

```sql
-- Conditionals can be negated
SELECT *
FROM customers
WHERE (NOT birth_date > '1990-01-01')
   OR (points > 1000 AND state = 'VA');
```

### The `IN` operator

Let's say we want to query all customers in Virginia or Florida or Georgia.
We **can** write the query like this:

```sql
SELECT *
FROM customers
WHERE state = 'VA'
   OR state = 'FL'
   OR state = 'GA';
```

However, there is a more efficient way to query this data.

```sql
SELECT *
FROM customers
WHERE state IN ('VA', 'FL', 'GA');
```

We can also negate this conditional.

```sql
SELECT *
FROM customers
WHERE state NOT IN ('VA', 'FL', 'GA');

SELECT *
FROM products
WHERE quantity_in_stock IN (49, 38, 72)
```

### The `BETWEEN` operator

We **can** write a comparison within an interval like this:

```sql
SELECT *
FROM customers
WHERE points >= 1000
  AND points <= 3000;
```

However, there is a syntactically nicer way to query this data:

```sql
SELECT *
FROM customers
WHERE points BETWEEN 1000 AND 3000;

SELECT *
FROM customers
WHERE birth_date BETWEEN '1990-01-01' AND '2000-01-01';
```

This is exactly equivalent to the ordinary comparison above.
This values in `BETWEEN` are inclusive.

### The `LIKE` operator

We can retrieve records that match a given string pattern.

```sql
SELECT *
FROM customers
-- b% - records where last name starts with b, and any character after b
WHERE last_name LIKE 'b%';

SELECT *
FROM customers
-- %b% - records where last name has a b anywhere
WHERE last_name LIKE '%b%';

SELECT *
FROM customers
-- _____y - records where last name has exactly,
-- 6 characters, and where y is the last character
WHERE last_name LIKE '_____y';

SELECT *
FROM customers
WHERE last_name LIKE 'b____y';
```

The `LIKE` operator is an old operator.
We now rather use `REGEXP` to search using regular expressions.

### The `REGEXP` operator

Regular expressions are a powerful tool for string manipulation.

```sql
-- Query customers with 'field' in their last name
SELECT *
FROM customers
WHERE last_name REGEXP 'field';
```

`^` represents the beginning of a string. `$` represents the end of a string.

The pipe operator (`|`) represents a logical `OR`, or multiple search patterns.

```sql
SELECT *
FROM customers
WHERE last_name REGEXP 'field|mac';
```

We can chain these operators.

```sql
-- Last name starts with 'field', or contains 'mac'
SELECT *
FROM customers
WHERE last_name REGEXP '^field|mac';

-- Any last name that contains 'ge', 'ie' or 'me'
SELECT *
FROM customers
WHERE last_name REGEXP '[gim]e';

-- Any last name that contains 'any character from a to p', followed by an 'e'
SELECT *
FROM customers
WHERE last_name REGEXP '[a-p]e';
```

#### Some exercises to get familiar with regular expressions

```sql
-- First names are ELKA or AMBUR
SELECT *
FROM customers
WHERE first_name REGEXP 'elka|ambur';

-- Last names end with EY or ON
SELECT *
FROM customers
WHERE last_name REGEXP 'ey$|on$';

-- Last names start with MY or contains SE
SELECT *
FROM customers
WHERE last_name REGEXP '^my|se';

-- Last names contain B followed by R or U
SELECT *
FROM customers
WHERE last_name REGEXP 'b[ru]';
```

### The `IS NULL` operator

The `IS NULL` operator can be used when a record is missing an attribute.

`NULL` in a record means the absence of an attribute (a column).

```sql
SELECT *
FROM customers
WHERE phone IS NULL;

-- This can be chained with other logical operators, such as NOT
SELECT *
FROM customers
WHERE phone IS NOT NULL;
```

### The `ORDER BY` clause

We can use `ORDER BY` to sort data.

By default, the **primary key** is used when ordering data.

```sql
-- Sort by first name in alphabetical order
SELECT *
FROM customers
ORDER BY first_name;

-- Sort by first name in reverse alphabetical order
SELECT *
FROM customers
ORDER BY first_name DESC;
```

We can also sort by multiple columns.

```sql
-- First sort by state in alphabetical order.
-- If multiple customers are from the same state,
-- then sort on their first name in alphabetical order.
SELECT *
FROM customers
ORDER BY state, first_name

-- Sort by total price in descending order, based on an alias
SELECT *, (quantity * unit_price) AS total_price
FROM order_items
WHERE order_id = 2
ORDER BY total_price DESC;
```

### The `LIMIT` clause

We can use the `LIMIT` clause to limit the amount of data returned from a query.

```sql
-- Return the first 3 customers
SELECT *
FROM customers
LIMIT 3;
```

If the value provided to `LIMIT` is greater than the number of records in the table,
the entire table will be returned.
Let's say we have 10 records in the `customers` table.

```sql
-- Returns the total number of customers, i.e. all 10 records
SELECT *
FROM customers
LIMIT 300;
```

We can add an offset to the limit. This is very useful when we want to **paginate**
the data returned from a query.

```sql
-- Page 1: 1-3
-- Page 2: 4-6
-- Page 3: 7-9
-- Return page 3 by skipping the first 6 records, and then pick 3 records.
SELECT *
FROM customers
LIMIT 6,3;
```

#### A small exercise

```sql
-- Get top 3 customers, based on amount of points
SELECT *
FROM customers
ORDER BY points DESC
LIMIT 3
```

## Retrieving data from multiple tables

### The `INNER JOIN` statement

We can join multiple tables together using the `JOIN` keyword.
For now, we only use `INNER JOIN`, but throughout the course, other type of `JOIN`'s
will be used.

```sql
-- Join the orders and customers table where customer_id in orders equals
-- customer_id in customers.
-- Note that we can give customers an alias 'c'.
SELECT order_id, first_name, last_name
FROM orders
         INNER JOIN customers c ON orders.customer_id = c.customer_id
```

In situations where we have **the same column in multiple tables**,
we need to specify the column.

```sql
-- Here, we specify c.customer_id in order to prevent it being ambiguous
SELECT order_id, c.customer_id, first_name, last_name
FROM orders
         INNER JOIN customers c ON orders.customer_id = c.customer_id
```

These table names are getting quite long.
We can abbreviate them using the first letter like so:

```sql
-- Note that if both tables have the same first letter,
-- we simply use common sense to abbreviate them.
SELECT order_id, c.customer_id, first_name, last_name
FROM orders o
         INNER JOIN customers c ON o.customer_id = c.customer_id
```

### Joining across databases

In the real world, we often need to work with several databases.

In the example below, `sql_inventory.products` is a table from a different database
than the database `order_items` resides in.

```sql
SELECT *
FROM order_items oi
         INNER JOIN sql_inventory.products p ON oi.product_id = p.product_id
```

Note that the prefix before the table name depends on which database we `USE`.

```sql
-- Note that we explicitly use 'sql_inventory' database.
-- This changes how the syntax of getting 'order_items' in 'sql_store'.
USE sql_inventory;

SELECT *
FROM sql_store.order_items oi
         INNER JOIN products p ON p.product_id = oi.product_id
```

### Self joins

In SQL, we can join a table with itself.

As an example, we have employees with a given `employee_id`.
An employee also reports to a given employee in the `reports_to` column,
using an `employee_id`. In this case, we can **self join** the table to query desired
data.

```sql
-- m is shorthand for 'manager'
SELECT e.first_name,
       e.last_name,
       m.first_name AS manager_first_name,
       m.last_name  AS manager_last_name
FROM employees e
         INNER JOIN employees m ON e.reports_to = m.employee_id
```

### Joining multiple tables

We can join **more than 2 tables** when writing a query.

As an example, we want to create a report for the status on our customers' orders.

```sql
SELECT order_id,
       order_date,
       first_name,
       last_name,
       os.name AS status
FROM orders o
         INNER JOIN customers c on o.customer_id = c.customer_id
         INNER JOIN order_statuses os on o.status = os.order_status_id
```

We just joined 3 tables! In the real word, you may need to join 10+ tables.

Another example: We want to create a report for payments executed by client,
with their payment method.

```sql
SELECT p.date,
       p.invoice_id,
       p.amount,
       c.name  AS client_name,
       pm.name AS payment_method
FROM payments p
         INNER JOIN clients c on p.client_id = c.client_id
         INNER JOIN payment_methods pm on p.payment_method = pm.payment_method_id
```

### Compound join conditions

A table may have a **composite primary key**.
A composite primary key contains more than one column.

![Composite Primary Key 1](./img/composite-primary-key/1.png)
![Composite Primary Key 2](./img/composite-primary-key/2.png)

If a table has a composite primary key, we need to be aware when we're
joining this table with other tables.

```sql
SELECT *
FROM order_items oi
         JOIN order_item_notes oin
              ON oi.order_id = oin.order_Id
                  AND oi.product_id = oin.product_id
```

### Implicit `JOIN` syntax

We have the following basic `JOIN` query:

```sql
SELECT *
FROM orders o
         JOIN customers c on o.customer_id = c.customer_id
```

We can rewrite the `JOIN` using the following syntax:

```sql
-- Implicit JOIN syntax
SELECT *
FROM orders o,
     customers c
WHERE c.customer_id = o.customer_id
```

This is called **implicit join syntax**.

Even though MySQL supports this syntax, it is generally not recommended to use it.
This is because if we forget the `WHERE` clause, we get a **cross join**.

Cross joins will be touched on later in the course.

**To summarize: Be aware of implicit join syntax, but use explicit join syntax!**

### Outer joins

Whenever we type `JOIN`, we are really using `INNER JOIN`.

This part of the course focuses on `OUTER JOIN`.

Let's say we want to see all customers, whether they have an order placed or not.

```sql
-- Note that this does not solve the problem above
SELECT c.customer_id, c.first_name, o.order_id
FROM customers c
         JOIN orders o on c.customer_id = o.customer_id
ORDER BY c.customer_id
```

This query does not show all customers whether they have placed an order or not.
This is because some customers do not have an order,
and thus `c.customer_id = o.customer_id` is not true!

We can use an **outer join** to solve this problem!

We have two types of outer joins: `LEFT JOIN` and `RIGHT JOIN`.

#### `LEFT JOIN`

```sql
SELECT c.customer_id, c.first_name, o.order_id
FROM customers c
         LEFT JOIN orders o on c.customer_id = o.customer_id
ORDER BY c.customer_id
```

Using `LEFT JOIN`, all records from the left table are returned whether the condition
is true or not.

In our example, the left table is `customers c`
and the condition is `orders o on c.customer_id = o.customer_id`.

#### `RIGHT JOIN`

```sql
SELECT c.customer_id, c.first_name, o.order_id
FROM customers c
         RIGHT JOIN orders o on c.customer_id = o.customer_id
ORDER BY c.customer_id
```

Using `RIGHT JOIN`, all records from the right table are returned whether
the condition is true or not.

In our example, the right table is `orders o` and the condition
is `orders o on c.customer_id = o.customer_id`.

If we want to use a right join, and still see all customer, the order of
`customers c` and `orders o` must be swapped.

**Note that `RIGHT JOIN` = `RIGHT OUTER JOIN`. The same goes for `LEFT JOIN`.**
We generally don't use this syntax, in order to make our queries cleaner.

### Outer join between multiple tables

Let's say we want to display all customers, regardless of whether they have
placed an order **and** regardless of whether the order has a related shipper.

```sql
SELECT c.customer_id,
       c.first_name,
       o.order_id,
       s.name
FROM customers c
         LEFT JOIN orders o ON c.customer_id = o.order_id
         LEFT JOIN shippers s on o.shipper_id = s.shipper_id
ORDER BY c.customer_id
```

As a rule of thumb, avoid using `RIGHT JOIN` when writing complex queries.
It is just a headache to keep track of which table is joined with which first.
**Thus, you can achieve every join using either `JOIN` or `LEFT JOIN`!**

### An exercise testing inner join, outer join and several `ORDER BY`

The task is to simply write the query returning the result displayed in the course:

![Join Exercise](./img/join-exercise/1.png)

```sql
SELECT o.order_date,
       o.order_id,
       c.first_name,
       s.name  AS shipper,
       os.name AS status
FROM customers c
         JOIN orders o on c.customer_id = o.customer_id
         JOIN order_statuses os on o.status = os.order_status_id
         LEFT JOIN shippers s on o.shipper_id = s.shipper_id
ORDER BY os.order_status_id, o.order_id
```

Note how we first order by `os.order_status_id` and then by `o.order_id`.

### Self outer joins

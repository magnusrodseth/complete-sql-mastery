# Code With Mosh: Complete SQL Mastery

## Introduction

This course really does cover all parts of SQL. Therefore, my notes are split
into several `.md` files, collectively covering the entire course.

I have chosen to split my notes in two: (1) designing databases and (2)
everything else.

[Click here](./designing-databases/README.md) to read more about **designing
databases**. Carry on reading below this point for everything else in the
course.

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

Note that `AND` has a higher precedence than `OR` by default. We can override
this precedence using parenthesis `()`.

```sql
-- Conditionals can be negated
SELECT *
FROM customers
WHERE (NOT birth_date > '1990-01-01')
   OR (points > 1000 AND state = 'VA');
```

### The `IN` operator

Let's say we want to query all customers in Virginia or Florida or Georgia. We
**can** write the query like this:

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

This is exactly equivalent to the ordinary comparison above. This values in
`BETWEEN` are inclusive.

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

The `LIKE` operator is an old operator. We now rather use `REGEXP` to search
using regular expressions.

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

If the value provided to `LIMIT` is greater than the number of records in the
table, the entire table will be returned. Let's say we have 10 records in the
`customers` table.

```sql
-- Returns the total number of customers, i.e. all 10 records
SELECT *
FROM customers
LIMIT 300;
```

We can add an offset to the limit. This is very useful when we want to
**paginate** the data returned from a query.

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

We can join multiple tables together using the `JOIN` keyword. For now, we only
use `INNER JOIN`, but throughout the course, other type of `JOIN`'s will be
used.

```sql
-- Join the orders and customers table where customer_id in orders equals
-- customer_id in customers.
-- Note that we can give customers an alias 'c'.
SELECT order_id, first_name, last_name
FROM orders
         INNER JOIN customers c ON orders.customer_id = c.customer_id
```

In situations where we have **the same column in multiple tables**, we need to
specify the column.

```sql
-- Here, we specify c.customer_id in order to prevent it being ambiguous
SELECT order_id, c.customer_id, first_name, last_name
FROM orders
         INNER JOIN customers c ON orders.customer_id = c.customer_id
```

These table names are getting quite long. We can abbreviate them using the first
letter like so:

```sql
-- Note that if both tables have the same first letter,
-- we simply use common sense to abbreviate them.
SELECT order_id, c.customer_id, first_name, last_name
FROM orders o
         INNER JOIN customers c ON o.customer_id = c.customer_id
```

### Joining across databases

In the real world, we often need to work with several databases.

In the example below, `sql_inventory.products` is a table from a different
database than the database `order_items` resides in.

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

As an example, we have employees with a given `employee_id`. An employee also
reports to a given employee in the `reports_to` column, using an `employee_id`.
In this case, we can **self join** the table to query desired data.

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

As an example, we want to create a report for the status on our customers'
orders.

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

A table may have a **composite primary key**. A composite primary key contains
more than one column.

![Composite Primary Key 1](./img/composite-primary-key/1.png)
![Composite Primary Key 2](./img/composite-primary-key/2.png)

If a table has a composite primary key, we need to be aware when we're joining
this table with other tables.

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

Even though MySQL supports this syntax, it is generally not recommended to use
it. This is because if we forget the `WHERE` clause, we get a **cross join**.

Cross joins will be touched on later in the course.

**To summarize: Be aware of implicit join syntax, but use explicit join
syntax!**

### Outer joins

Whenever we type `JOIN`, we are really using `INNER JOIN`.

This part of the course focuses on `OUTER JOIN`.

Let's say we want to see all customers, whether they have an order placed or
not.

```sql
-- Note that this does not solve the problem above
SELECT c.customer_id, c.first_name, o.order_id
FROM customers c
         JOIN orders o on c.customer_id = o.customer_id
ORDER BY c.customer_id
```

This query does not show all customers whether they have placed an order or not.
This is because some customers do not have an order, and thus
`c.customer_id = o.customer_id` is not true!

We can use an **outer join** to solve this problem!

We have two types of outer joins: `LEFT JOIN` and `RIGHT JOIN`.

#### `LEFT JOIN`

```sql
SELECT c.customer_id, c.first_name, o.order_id
FROM customers c
         LEFT JOIN orders o on c.customer_id = o.customer_id
ORDER BY c.customer_id
```

Using `LEFT JOIN`, all records from the left table are returned whether the
condition is true or not.

In our example, the left table is `customers c` and the condition is
`orders o on c.customer_id = o.customer_id`.

#### `RIGHT JOIN`

```sql
SELECT c.customer_id, c.first_name, o.order_id
FROM customers c
         RIGHT JOIN orders o on c.customer_id = o.customer_id
ORDER BY c.customer_id
```

Using `RIGHT JOIN`, all records from the right table are returned whether the
condition is true or not.

In our example, the right table is `orders o` and the condition is
`orders o on c.customer_id = o.customer_id`.

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

As a rule of thumb, avoid using `RIGHT JOIN` when writing complex queries. It is
just a headache to keep track of which table is joined with which first. **Thus,
you can achieve every join using either `JOIN` or `LEFT JOIN`!**

### An exercise testing inner join, outer join and several `ORDER BY`

The task is to simply write the query returning the result displayed in the
course:

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

Earlier, we wrote the following query to display employees and their manager:

```sql
-- m is shorthand for manager
SELECT e.employee_id,
       e.first_name,
       m.first_name AS manager
FROM employees e
         JOIN employees m ON e.reports_to = m.employee_id
```

However, because the manager itself has no manager, its `reports_to` column is
`null`.

We solve this by using a self outer join.

```sql
-- m is shorthand for manager
SELECT e.employee_id,
       e.first_name,
       m.first_name AS manager
FROM employees e
         LEFT JOIN employees m ON e.reports_to = m.employee_id
```

### The `USING` clause

We have the following query:

```sql
SELECT o.order_id,
       c.first_name
FROM orders o
         JOIN customers c ON o.customer_id = c.customer_id
```

If the column name in the `ON` clause is exactly the same (here: `customer_id`),
we can use the `USING` clause to simplify the syntax:

```sql
SELECT o.order_id,
       c.first_name
FROM orders o
         JOIN customers c USING (customer_id)
```

These two queries are equivalent.

We can use the `USING` clause with both inner joins and outer joins:

```sql
SELECT o.order_id,
       c.first_name,
       s.name
FROM orders o
         JOIN customers c USING (customer_id)
         LEFT JOIN shippers s USING (shipper_id)
```

**What if a table uses a composite primary key?**

We once again look at the `order_item_notes` that uses a composite primary key.
We can query like this:

```sql
SELECT *
FROM order_items oi
         JOIN order_item_notes oin
              ON oi.order_id = oin.order_Id
                  AND oi.product_id = oin.product_id
```

However, this is quite unreadable. We can simplify using the `USING` clause:

```sql
SELECT *
FROM order_items oi
         JOIN order_item_notes oin
              USING (order_id, product_id)
```

These two queries are equivalent.

### An exercise using the `USING` clause

The task is to simply write the query returning the result displayed in the
course:

![`Using` Exercise](./img/using-exercise/1.png)

```sql
SELECT p.date,
       c.name  AS client,
       p.amount,
       pm.name AS payment_method
FROM payments p
         JOIN clients c USING (client_id)
         JOIN payment_methods pm on p.payment_method = pm.payment_method_id
```

Note how we cannot use `USING` on the `payment_methods` table due to the column
name not be exactly the same.

### Natural Joins

Natural joins exist in MySQL, but is not recommended, as it may produce
unexpected results.

```sql
-- Joins 'orders' and 'customers' based on common columns,
-- i.e. columns that have the same name.
SELECT o.order_id,
       c.first_name
FROM orders o
         NATURAL JOIN customers c
```

We're letting the database engine _guess_ the common column name. Thus,
unexpected behavior may occur.

### Cross joins

We use cross join to combine or join every record from the first table with
every record in the second table.

```sql
SELECT c.first_name AS customer,
       p.name       AS product
FROM customers c
         CROSS JOIN products p
ORDER BY c.first_name
```

A real world example of when it makes sense to use `CROSS JOIN` is when we have
table of sizes (e.g. `small`, `medium`, `large`) and a table of colors (e.g.
`red`, `blue`, `green`), and we want to combine all sizes with all colors.

In the example above, we use the explicit syntax for cross join: `CROSS JOIN`.
We can also use the implicit cross join syntax:

```sql
SELECT c.first_name AS customer,
       p.name       AS product
FROM customers c,
     products p
ORDER BY c.first_name
```

Both queries produce the same result.

### Unions

We can also combine **rows** from multiple tables. This is extremely powerful.

Let's say we have many records spanning over several years. We want to create a
report displaying which records are still `'Active'` and which are `'Archived'`
based on a given date (e.g. `2019-01-01`).

We write the following two queries:

```sql
-- Active orders
SELECT order_id,
       order_date,
       'Active' AS status
FROM orders o
WHERE o.order_date >= '2019-01-01';

-- Archived orders
SELECT order_id,
       order_date,
       'Archived' AS status
FROM orders o
WHERE o.order_date < '2019-01-01'
```

We can combine the result of these two queries using the `UNION` operator:

```sql
(-- Active orders
    SELECT order_id,
           order_date,
           'Active' AS status
    FROM orders o
    WHERE o.order_date >= '2019-01-01')

UNION

(-- Archived orders
    SELECT order_id,
           order_date,
           'Archived' AS status
    FROM orders o
    WHERE o.order_date < '2019-01-01')
```

This can of course be done across tables, and across databases.

#### Important to note

The number of columns returned by each `SELECT` must be equal. Else, we get an
error:

```sql
(SELECT first_name, last_name
 FROM customers c)
-- This UNION will result in an error,
-- due to unequal number of columns in each SELECT statement.
UNION
(SELECT name
 FROM shippers s)
```

#### An exercise using `UNION`

Write a query to produce this following report:

![Union Exercise](./img/union-exercise/1.png)

```sql
((SELECT c.customer_id,
         c.first_name,
         c.points,
         'Bronze' AS type
  FROM customers c
  WHERE c.points < 2000)
 UNION
 (SELECT c.customer_id,
         c.first_name,
         c.points,
         'Silver' AS type
  FROM customers c
  WHERE c.points >= 2000
    AND c.points <= 3000)
 UNION
 (SELECT c.customer_id,
         c.first_name,
         c.points,
         'Bronze' AS type
  FROM customers c
  WHERE c.points > 3000)
) ORDER BY first_name
```

## Inserting, Updating and Deleting Data

### Column Attributes

We can inspect a table and its columns:

![Column Attributes 1](./img/column-attributes/1.png)

#### `VARCHAR` and `CHAR`

The type `VARCHAR(50)` tells us that for instance `first_name` can only be 50
characters long. **If it is shorter, only the length of the string gets
stored**.

The type `CHAR(2)` tells us that the length of `state` will always be 2. If a
record is 1, another character wil be added. If a record is 3, a character will
be cut.

Most of the time, `VARCHAR` is used.

In the example above, the primary key `PK` is the `customer_id`.

#### `NOT NULL`

![Column Attributes 2](./img/column-attributes/2.png)

We can decide if columns can be `null` or not. If a column is marked as
`Not null`, it must have content.

#### `AUTO INCREMENT`

![Column Attributes 3](./img/column-attributes/3.png)

A primary key can be marked with `Auto increment`, or `AI`. The primary key will
be calculated based on the previous key, e.g. `1 -> 2 -> 3`.

#### Default values

![Column Attributes 4](./img/column-attributes/4.png)

We can also mark columns with default values. In the example above, `points`
will equal `0` by default, if we don't give it a specific value.

### Inserting a single row

We can insert into a row in the following way, including all columns:

```sql
INSERT INTO customers
VALUES (DEFAULT, -- Let MySQL auto-generate the primary key using AUTO INCREMENT
        'John', -- Required
        'Doe', -- Required
        '1990-01-01', -- Required
        NULL, -- Optional. NULL as default. Passing DEFAULT or NULL is equivalent.
        'Address',
        'City',
        'CA',
        DEFAULT) -- DEFAULT = 0
```

We can also specify which columns we want to insert in, and leave out the rest.
The data has to be inserted in the order they are listed:

```sql
INSERT INTO customers (first_name,
                       last_name,
                       birth_date,
                       address,
                       city,
                       state)
VALUES ('John',
        'Doe',
        '1990-01-01',
        'Address',
        'City',
        'CA')
```

Note that the order of columns can be swapped, but then the data has to match
that order. Pay extra attention to `first_name` and `last_name` in the example
above and below:

```sql
INSERT INTO customers (last_name,
                       first_name,
                       birth_date,
                       address,
                       city,
                       state)
VALUES ('Doe',
        'John',
        '1990-01-01',
        'Address',
        'City',
        'CA')
```

### Inserting multiple rows

Notice the use of parenthesis and comma in the example below:

```sql
INSERT INTO shippers (name)
values ('Shipper 1'),
       ('Shipper 2'),
       ('Shipper3')
```

Because the primary key automatically increments, there's no need to specify a
`shipper_id` when creating a new row.

### Inserting hierarchical rows

We can also insert data into multiple tables.

Let's say we have an `order`, consisting of one or more `order_items`.

```sql
INSERT INTO orders (customer_id, order_date, status)
VALUES (1, -- Must be a valid ID
        '2019-01-02',
        1); -- Must be a valid ID

INSERT INTO order_items(order_id, product_id, quantity, unit_price)
VALUES (LAST_INSERT_ID(), -- SQL function to get the last inserted ID
        1, -- Must be a valid ID
        1,
        2.95),
       (LAST_INSERT_ID(), -- SQL function to get the last inserted ID
        2, -- Must be a valid ID
        1,
        1.5)
```

We have now created a new `order`, consisting of 2 `order_items`.

### Creating a copy of a table

We can copy data between tables.

Let's say we want to create an `orders_archived` table to store all orders
presently recorded.

```sql
CREATE TABLE orders_archived AS
SELECT *
FROM orders
```

Note that when we inspect our new table `orders_archived`, `order_id` is no
longer the primary key and it is not marked as a column to auto-increment.

We refer to the following from the example above as a subquery / sub query:

```sql
SELECT *
FROM orders
```

We can also use a sub query in an `INSERT` statement. This is very powerful.

**Side note:** You can clear all rows without dropping a table using `TRUNCATE`.
As an example, we can `TRUNCATE orders_archived`.

Let's say we want to copy only a subset of the original `orders` table into the
`orders_archived` table. We know how to get a subset of the entire table from
earlier in this course:

```sql
SELECT * FROM orders WHERE order_date < '2019-01-01'
```

We can use this as a sub query when copying rows.

```sql
INSERT INTO orders_archived
SELECT *
FROM orders
WHERE order_date < '2019-01-01'
```

#### An exercise for copying table using `JOIN`, `WHERE` and `IS NOT NULL`

We have our `invoices`. We want to create a copy of this table. This new table
should not have the `client_id`, but rather the name of the client. Include only
the invoices where the payment has been done.

```sql
CREATE TABLE invoices_archived AS (
    SELECT invoice_id,
           number,
           c.name AS client_name,
           invoice_total,
           payment_total,
           invoice_date,
           due_date,
           payment_date
    FROM invoices i
             JOIN clients c ON i.client_id = c.client_id
    WHERE i.payment_date IS NOT NULL)
```

I started selecting everything from `invoices`. Then I singled out the
information I needed using `JOIN` and `WHERE`. When I got my desired result, I
used that as a sub query when creating a new table `invoices_archived`. Nice!

### Updating a single row

We use the `UPDATE` statement to update one or more rows in a table.

```sql
UPDATE invoices
SET payment_total = 10,
    payment_date  = '2019-03-01'
WHERE invoice_id = 1
```

Note that omitting the `WHERE` gives a warning that `UPDATE` will update all
records in the table.

We can of course use `DEFAULT` keyword like usual:

```sql
UPDATE invoices
SET payment_total = DEFAULT,
    payment_date  = DEFAULT
WHERE invoice_id = 1
```

We can also perform arithmetics or reference other column values in a record:

```sql
UPDATE invoices
SET payment_total = (invoice_total * 0.5),
    payment_date  = due_date
WHERE invoice_id = 3
```

### Updating multiple rows

Performing updates on multiple rows is the same as performing updates on single
rows, but it requires a more general `WHERE` clause:

```sql
UPDATE invoices
SET payment_total = (invoice_total * 0.5),
    payment_date  = due_date
WHERE client_id = 3
```

The query above will update all records where `client_id` equals 3.

We can also use the `IN` operator:

```sql
UPDATE invoices
SET payment_total = (invoice_total * 0.5),
    payment_date  = due_date
WHERE client_id IN (3, 4)
```

The query above updates invoices for clients with `client_id` equals 3 or 4.

### Using sub queries in updates

Let's say we want to update the invoice of a client, **but we don't know the ID
of the client**! We only know the name of the client.

We can use a `SELECT` statement as a sub query in the `WHERE` clause in order to
find the desired `client_id`:

```sql
UPDATE invoices
SET payment_total = invoice_total * 0.5,
    payment_date  = due_date
WHERE client_id =
      (SELECT client_id FROM clients WHERE name = 'Myworks');
```

What if the `SELECT` query returns several clients?

```sql
UPDATE invoices
SET payment_total = invoice_total * 0.5,
    payment_date  = due_date
-- We cannot use '=' anymore, because the sub query returns multiple values
WHERE client_id IN
      (SELECT client_id
       FROM clients
       WHERE state IN ('CA', 'NY'));
```

#### Exercise using a sub query

We want to update all `customers` with `points > 3000` with a `comment` equal to
`'Gold Customer'`.

```sql
UPDATE orders
SET comments = 'Gold Customer'
WHERE customer_id IN (SELECT customer_id
                      FROM customers
                      WHERE points > 3000)
```

### Deleting rows

If we know the id, we simply write:

```sql
DELETE
FROM invoices
WHERE invoice_id = 1
```

If we don't know the id, we can use a sub query:

```sql
DELETE
FROM invoices
WHERE client_id = (SELECT *
                    FROM clients
                    WHERE name = 'Myworks')
```

## Summarizing Data

### Aggregate Functions

A function is simply a piece of code that can be reused.

SQL aggregate functions takes a series of values, and aggregate them in order to
**create a single value**.

Examples of aggregate functions include: `MAX()`, `MIN()`, `SUM()`, `AVG()`,
`COUNT()`.

As an example, we can find the max `invoice_total` by querying the following:

```sql
SELECT MAX(invoice_total) AS highest,
       MIN(invoice_total) AS lowest,
       AVG(invoice_total) AS average
FROM invoices
```

We can also use these aggregate functions on values that are not numeric.

```sql
SELECT MAX(payment_date)  AS highest_payment_date,
       MIN(invoice_total) AS lowest,
       AVG(invoice_total) AS average
FROM invoices
```

A complete report may look like this:

```sql
SELECT MAX(invoice_total)   AS highest,
       MIN(invoice_total)   AS lowest,
       AVG(invoice_total)   AS average,
       SUM(invoice_total)   AS total,
       COUNT(invoice_total) AS number_of_invoices
FROM invoices
```

**Aggregate functions only operate on non-null values!**

`payment_date` may have `NULL` values. Thus, a query for `COUNT(payment_date)`
**returns the number of non-null values**.

```sql
SELECT COUNT(payment_date) AS number_of_payments
FROM invoices
```

We can get the total number of records, regardless of whether they are `NULL` or
not using the following query:

```sql
SELECT COUNT(*) AS number_of_records
FROM invoices
```

Most of the time, we use column names as parameters for aggregate functions.
However, we may write expressions and insert that into the aggregate function.

```sql
SELECT MAX(invoice_total)       AS highest,
       MIN(invoice_total)       AS lowest,
       AVG(invoice_total)       AS average,
       SUM(invoice_total * 1.1) AS total,
       COUNT(invoice_total)     AS number_of_invoices,
       COUNT(payment_date)      AS number_of_payments
FROM invoices;
```

We can also apply filters using the `WHERE` clause. The aggregate function is
applied to the records that match the given filter.

```sql
SELECT MAX(invoice_total)       AS highest,
       MIN(invoice_total)       AS lowest,
       AVG(invoice_total)       AS average,
       SUM(invoice_total * 1.1) AS total,
       COUNT(invoice_total)     AS number_of_invoices,
       COUNT(payment_date)      AS number_of_payments
FROM invoices
WHERE invoice_date > '2019-07-01'
```

By default, all aggregate functions accept duplicates. If we do not want to
include duplicates, we need to use the `DISTINCT` keyword in order to only get
unique entries.

```sql
SELECT MAX(invoice_total)        AS highest,
       MIN(invoice_total)        AS lowest,
       AVG(invoice_total)        AS average,
       SUM(invoice_total * 1.1)  AS total,
       COUNT(invoice_total)      AS number_of_invoices,
       COUNT(DISTINCT client_id) AS number_of_clients
FROM invoices
WHERE invoice_date > '2019-07-01'
```

#### Exercise using aggregate functions

Simply generate the following report from the `invoices` table:

![Aggregate functions 1](./img/aggregate-functions/1.png)

We get the following (quite complex) query:

```sql
(
    (-- First half of 2019
        SELECT 'First half of 2019'               AS date_range,
               SUM(invoice_total)                 AS total_sales,
               SUM(payment_total)                 AS total_payments,
               SUM(invoice_total - payment_total) AS what_we_expect
        FROM invoices
        WHERE invoice_date BETWEEN '2019-01-01' AND '2019-06-30')

    UNION

    (-- Second half of 2019
        SELECT 'Second half of 2019'              AS date_range,
               SUM(invoice_total)                 AS total_sales,
               SUM(payment_total)                 AS total_payments,
               SUM(invoice_total - payment_total) AS what_we_expect
        FROM invoices
        WHERE invoice_date BETWEEN '2019-07-01' AND invoice_date <= '2019-12-31')

    UNION

    (-- Total
        SELECT 'Total'                            AS date_range,
               SUM(invoice_total)                 AS total_sales,
               SUM(payment_total)                 AS total_payments,
               SUM(invoice_total - payment_total) AS what_we_expect
        FROM invoices
        WHERE invoice_date BETWEEN '2019-01-01' AND invoice_date <= '2019-12-31')
)
```

I first calculated the desired results for `'First half of 2019'`, and then
`UNION` them with the rest. It's not that complicated when you build the query
step by step.

### The `GROUP BY` clause

#### Grouping by a single column

In the previous section about aggregate functions, we learned how to create
summaries for our data like the following query:

```sql
SELECT SUM(invoice_total) AS total_sales
FROM invoices
```

What if we want to create a summary **per client**, i.e. partitioning our rows?
We need to group the data!

```sql
`SELECT client_id, SUM(invoice_total) AS total_sales
FROM invoices
GROUP BY client_id
ORDER BY total_sales DESC
```

Now, instead of 1 value for `total_sales`, we now see the `total_sales` per
`client_id`. In other words, we now see the total sales each given client has
payed us.

Naturally, we can add keywords to the query:

```sql
SELECT client_id,
       SUM(invoice_total) AS total_sales
FROM invoices
WHERE invoice_date >= '2019-07-01'
GROUP BY client_id
ORDER BY total_sales DESC
```

**Note the order of these keywords!** We get a syntax error if the order is
incorrect.

#### Grouping by multiple columns

We want to see the data from before, but this time **per state and city for our
clients**.

```sql
SELECT state,
       city,
       SUM(invoice_total) AS total_sales
FROM invoices i
         JOIN clients c on i.client_id = c.client_id
GROUP BY state, city
```

Now, we see the `total_sales` for each `city` and `state` combination.

#### An exercise using `GROUP BY`

Produce the following result:

![Group by 1](./img/group-by/1.png)

```sql
SELECT p.date,
       pm.name       AS payment_method,
       SUM(p.amount) AS total_payment
FROM payments p
         JOIN payment_methods pm on p.payment_method = pm.payment_method_id
GROUP BY p.date, p.payment_method
ORDER BY p.date
```

### The `HAVING` clause

We have the following query from before:

```sql
SELECT client_id,
       SUM(invoice_total) AS total_sales
FROM invoices
GROUP BY client_id
```

Let's say this query gives us a client with `total_sales` less than 500, and
**we want to limit our result to client that fulfill a given criteria**.

We cannot use the `WHERE` clause, because we group our data **after** the
`WHERE` clause.

```sql
SELECT client_id,
       SUM(invoice_total) AS total_sales
FROM invoices
WHERE total_sales > 500 -- This gives us a syntax error!
GROUP BY client_id
```

We use the `HAVING` clause to filter data **after** the `GROUP BY` clause.

```sql
SELECT client_id,
       SUM(invoice_total) AS total_sales
FROM invoices
GROUP BY client_id
HAVING total_sales > 500
```

We can create more complex conditionals, of course.

```sql
SELECT client_id,
       SUM(invoice_total) AS total_sales,
       COUNT(*)           AS number_of_invoices
FROM invoices
GROUP BY client_id
HAVING total_sales > 500 AND number_of_invoices > 5
```

**The columns used in the `HAVING` clause has to appear in the `SELECT`
statement**.

#### Exercise using `HAVING` clause

We want to get the customers located in Virginia (VA) that have spent more than
$100.

```sql
SELECT c.customer_id,
       c.first_name,
       c.last_name,
       SUM(quantity * unit_price) AS total_sales
FROM customers c
         JOIN orders o on c.customer_id = o.customer_id
         JOIN order_items oi on o.order_id = oi.order_id
WHERE state = 'VA'
GROUP BY c.customer_id, c.first_name, c.last_name
HAVING total_sales > 100
```

### The `ROLLUP` operator

We can add `WITH ROLLUP` after `GROUP BY` to add an extra row that summarizes
the data.

`ROLLUP` only applies to columns that aggregate values. For instance, it does
not makes sense to `ROLLUP` all `client_id`'s, as they are not aggregated.

```sql
SELECT client_id,
       SUM(invoice_total) AS total_sales
FROM invoices
GROUP BY client_id
WITH ROLLUP
```

`ROLLUP` also works when grouping by multiple columns.

```sql
SELECT state,
       city,
       SUM(invoice_total) AS total_sales
FROM invoices i
         JOIN clients c on i.client_id = c.client_id
GROUP BY state, city
WITH ROLLUP
```

**Note that `ROLLUP` is only available in MySQL. Other database services may
have equivalent operators with different names.**

## Writing Complex Queries

### Sub query / sub queries

We want to write a query that finds products that are more expensive than
'Lettuce'.

First, I write the query to get the `unit_price` for lettuce. Then, I use this
result as a sub query to find all other products that fulfill the given
criteria.

```sql
SELECT *
FROM products
WHERE unit_price >
      (-- unit_price of lettuce
          SELECT unit_price
          FROM products
          WHERE name REGEXP '^Lettuce');
```

#### An exercise using sub queries

Using the `sql_hr` database, find employees that earn more money than average.

```sql
SELECT first_name, last_name, salary
FROM employees
WHERE salary >
      (-- Find average salary of employees
          SELECT AVG(salary)
          FROM employees
      );
```

### More about the `IN` operator

Find the products that have never been ordered.

```sql
SELECT *
FROM products
WHERE product_id NOT IN
      (-- Find products that have been ordered
          SELECT DISTINCT p.product_id
          FROM products p
                   JOIN order_items oi on p.product_id = oi.product_id
      )
```

#### An exercise using the `IN` operator

Find the clients without invoices.

```sql
SELECT *
FROM clients c
WHERE c.client_id NOT IN
      (-- Find clients with invoices
          SELECT DISTINCT client_id
          FROM invoices)
```

### Sub queries versus joins

Oftentimes, we can rewrite a sub query using a join, and vice versa. Look at
these examples:

```sql
-- Using a sub query
SELECT *
FROM clients c
WHERE c.client_id NOT IN
      (-- Find clients with invoices
          SELECT DISTINCT client_id
          FROM invoices)

-- Using a join statement
SELECT *
FROM clients c
         LEFT JOIN invoices i on c.client_id = i.client_id
WHERE invoice_id IS NULL
```

Deciding on which statement to use is a matter of two things, primarily:

1. **Performance**. This is elaborated upon later in the course when talking
   about an "execution plan".
2. **Readability**. Assuming two queries have the same execution time, you
   should always go for the query that is most readable.

In the example above, the statement using a sub query is generally more
intuitive to understand. **Always pay great attention to the readability of the
code**.

#### An exercise on sub queries versus joins

Find customers who have ordered lettuce. Select `customer_id`, `first_name`,
`last_name`.

```sql
-- Using sub queries
SELECT customer_id,
       first_name,
       last_name
FROM customers c
WHERE customer_id IN
      (-- Get customer_id from orders of 'Lettuce'
          SELECT o.customer_id
          FROM order_items
                   JOIN orders o USING (order_id)
          WHERE product_id =
                (-- Get product_id of 'Lettuce'
                    SELECT product_id
                    FROM products
                    WHERE name REGEXP '^Lettuce'
                )
      );

-- Using join statement
SELECT DISTINCT customer_id,
                first_name,
                last_name
FROM customers c
         JOIN orders o USING (customer_id)
         JOIN order_items USING (order_id)
WHERE product_id =
      (-- Get product_id of 'Lettuce'
          SELECT product_id
          FROM products
          WHERE name REGEXP '^Lettuce'
      )
```

The second statement - using `JOIN` - is more readable and also a more natural
way to relate tables together.

### The `ALL` keyword

Select invoices larger than the maximum invoice of client with `client_id = 3`.
This can be solved using ordinary `WHERE` and a sub query, but may also be
solved using the `ALL` keyword.

```sql
-- Using WHERE (without ALL)
SELECT *
FROM invoices i
WHERE invoice_total >
      (-- Get max invoice for client with client_id=3
          SELECT MAX(invoice_total) as max_invoice
          FROM invoices i
          WHERE client_id = 3
      )

-- Using ALL
SELECT *
FROM invoices i
WHERE invoice_total > ALL
      (-- Get max invoice for client with client_id=3
          SELECT invoice_total
          FROM invoices i
          WHERE client_id = 3
      )
```

In the second solution, note how the sub query returns multiple rows.

### The `ANY` keyword

Select clients with at least 2 invoices.

```sql
-- Solution without ANY
SELECT *
FROM clients
WHERE client_id IN (-- Clients with 2 or more invoices
    SELECT client_id
    FROM invoices i
    GROUP BY client_id
    HAVING COUNT(*) >= 2
)

-- Solution with ANY
SELECT *
FROM clients
WHERE client_id = ANY (-- Clients with 2 or more invoices
    SELECT client_id
    FROM invoices i
    GROUP BY client_id
    HAVING COUNT(*) >= 2
)
```

These two solution are equivalent. In other words: **`IN` is equivalent to
`= ANY`**.

### Correlated sub queries

A correlated sub query is a sub query that is related to the outer query. In the
sub query, we reference the alias from the outer table. See example below.

Select employees whose salary is above the average in their office.

```sql
SELECT *
FROM employees e
WHERE salary > (-- Average salary per office
    SELECT AVG(salary) as avg_salary
    FROM employees impl_e
    WHERE e.office_id = impl_e.office_id
)
```

One could write out the pseudo-code for the query above like this:

```txt
- for each employee
    - calculate the average salary for each employee's office
- return the employee if the employee's salary > average salary
```

This is almost like a nested for loop. Thus, these queries may be slow. We can
see that the inner sub query is related to the outer query.

#### An exercise about correlated sub queries

Get the invoices that are larger than the client's average invoice amount.

```sql
SELECT *
FROM invoices i
WHERE invoice_total > (-- Client's average invoice amount
    SELECT AVG(invoice_total)
    FROM invoices _i
    WHERE _i.client_id = i.client_id
)
```

**A note to self:** If you work with correlated sub queries, a nested query
could perhaps have an equal alias, but with an underscore prefix like above.
Nested loops would then look like this:
`alias -> _alias --> __alias --> etc...`.

### The `EXISTS` operator

We want to select clients that have an invoice. There are many ways to achieve
this.

```sql
-- Using IN
SELECT *
FROM clients
WHERE client_id IN (
    SELECT DISTINCT client_id
    FROM invoices
)

-- Using JOIN
SELECT DISTINCT c.client_id
FROM clients c
         JOIN invoices i on c.client_id = i.client_id

-- Using EXISTS
SELECT *
FROM clients c
WHERE EXISTS(
              SELECT i.client_id
              FROM invoices i
              WHERE i.client_id = c.client_id
          )
```

The last example - the one using `EXISTS` - is a correlated sub query.

**The most important part about `EXISTS`**: If the sub query used after the `IN`
operator produces a large result, it is more efficient to use the `EXISTS`
keyword. This is because when we use `EXISTS`, followed by a sub query, the sub
query does not return a result set, but rather a boolean `TRUE` or `FALSE` to
the outer query.

#### An exercise testing `EXISTS`

Find the products that have never been ordered.

```sql
SELECT *
FROM products p
WHERE NOT EXISTS(
              SELECT product_id
              FROM order_items oi
              WHERE oi.product_id = p.product_id
          )
```

### Sub queries in the `SELECT` clause

We want to produce a report that looks like this:

![Sub queries in `SELECT`](./img/sub-queries-in-select/1.png)

We can write the following statement, using sub queries in the `SELECT` clause:

```sql
SELECT invoice_id,
       invoice_total,
       (SELECT AVG(invoice_total)
        FROM invoices)                            AS invoice_average,
       (invoice_total - (SELECT invoice_average)) AS difference
FROM invoices
```

### Sub queries in the `FROM` clause

This is quite straightforward. Just like you can have a sub query in the
`SELECT` clause, you can also put a sub query in the `FROM` clause.

Whenever we use sub query in the `FROM` clause, we need to give that part an
alias. **This is required**.

There is way to **not** use sub queries in the `FROM` clause called views. We
can store the views in our database, and give the view an alias to use later.
We'll look at this later in the course.

A rule of thumb should be: **Only use sub queries in `FROM` when the sub query
is simple**.

## Essential MySQL functions

### Numeric functions

#### `ROUND`

We can round a number using the `ROUND()` function. The function optionally
takes an argument for precision. Examples can be found below.

```sql
SELECT ROUND(5.73); -- > 6
SELECT ROUND(5.73, 1); -- > 5.7
SELECT ROUND(5.73, 2); -- > 5.73
SELECT ROUND(5.73, 0); -- > 6
```

#### `TRUNCATE()`

`TRUNCATE()` does exactly that; it truncates the value with provided precision.

```sql
SELECT TRUNCATE(5.7345, 2); -- > 5.73
SELECT TRUNCATE(6, 2): -- > 6
```

#### `CEILING()`

Returns the smallest integer greater than or equal to the provided number.

```sql
SELECT CEILING(5.1); -- > 6
SELECT CEILING(5); -- > 5
```

#### `FLOOR()`

Returns the largest integer less than or equal to the provided number.

```sql
SELECT FLOOR(5.2) -- > 5
```

#### `ABS()`

Returns the absolute value of the provided number.

```sql
SELECT ABS(-2.5); -- > 2.5
SELECT ABS(3); -- > 3
```

#### `RAND()`

Generates a random floating point number between 0 and 1.

```sql
SELECT RAND()
```

More numeric functions can be found by Googling "MySQL numeric functions".

### String functions

#### `LENGTH()`

Get the number of characters in the string.

```sql
SELECT LENGTH('sky') -- > 3
```

#### `UPPER()`

Converts the string to upper case.

```sql
SELECT UPPER('sky'); -- > 'SKY'
```

#### `LOWER()`

Converts the string to lower case.

```sql
SELECT LOWER('Sky') -- > 'sky'
```

#### Trimming functions

```sql
SELECT LTRIM('   Sky'); -- > 'Sky'
SELECT RTRIM('Sky   '); -- > 'Sky'
SELECT TRIM('  Sky  '); -- > 'Sky'
```

Very straightforward.

#### Substrings

```sql
SELECT LEFT('SQL database', 3); -- > 'SQL'
SELECT RIGHT('SQL database', 8); -- > 'database'
SELECT SUBSTRING('Hello, world!', 8, 12); -- > 'world!'
SELECT REPLACE('Hello, world!', 'world', 'bar'); -- > 'Hello, bar!'
```

#### Find index of character

Note that the search is **case insensitive**.

```sql
SELECT LOCATE('world', 'Hello, world!'); -- > 8
SELECT LOCATE('D', 'Hello, world!'); -- > 12
SELECT LOCATE('z', 'Hello, world!'); -- > 0
```

Note that non-existing characters returns `0`, not `-1`. This way, we cannot
always use `LOCATE` to find a substring in a string.

#### `CONCAT`

Pretty straightforward, but can be very useful in certain tables.

```sql
SELECT CONCAT(first_name, ' ', last_name) AS full_name
FROM customers
```

### Date functions in MySQL

#### Displaying date and time

```sql
-- Get current date and time
SELECT NOW();

-- Get current date, without time
SELECT CURDATE();

-- Get current time, without date
SELECT CURTIME();

-- Get year, month, date
SELECT YEAR(NOW()), MONTH(NOW()), DAY(NOW());

-- Get hour, minute, seconds
SELECT HOUR(NOW()), MINUTE(NOW()), SECOND(NOW());

-- Get name of month, name of day
SELECT MONTHNAME(NOW()), DAYNAME(NOW());
```

#### `EXTRACT()`

This function is part of the general SQL language. If you need to port MySQL to
other database management system, you can use `EXTRACT()`.

```sql
SELECT EXTRACT(DAY FROM NOW())
```

Note that `EXTRACT` takes basically an enum, e.g. `DAY`, `MONTH`, `YEAR_MONTH`,
etc... Read the docs for more information.

### Formatting date and time

```sql
SELECT DATE_FORMAT(NOW(), '%y'); -- > 2 digit representation of the year
SELECT DATE_FORMAT(NOW(), '%Y'); -- > 4 digit representation of the year

SELECT DATE_FORMAT(NOW(), '%m %Y'); -- > 'mm yyyy'
SELECT DATE_FORMAT(NOW(), '%M %Y'); -- > 'month_name yyyy'

SELECT DATE_FORMAT(NOW(), '%M %d %Y'); -- > 'month_name dd yyyy'
SELECT DATE_FORMAT(NOW(), '%M %D %Y'); -- > 'month_name ddth yyyy'
```

For more information about date format and time format, simply read the docs.

### Calculating date and time

```sql
SELECT DATE_ADD(NOW(), INTERVAL 1 DAY); -- > Next day, with the same time
SELECT DATE_ADD(NOW(), INTERVAL 1 YEAR); -- > Next year, with the same date

SELECT DATE_ADD(NOW(), INTERVAL -1 YEAR); -- > Last year, with the same date
SELECT DATE_SUB(NOW(), INTERVAL 1 YEAR); -- > Last year, with the same date

SELECT DATEDIFF(NOW(), '2019-01-01'); -- > Difference between now and beginning of 2019, **in days**

SELECT TIME_TO_SEC(NOW()); -- > Number of seconds elapsed since midnight
SELECT TIME_TO_SEC('01:00'); -- > 3600
SELECT TIME_TO_SEC('00:01'); -- > 60
SELECT (TIME_TO_SEC('09:00') - TIME_TO_SEC('09:01')) -- > -60
```

### The `IFNULL` and `COALESCE` functions

Not all orders have an assigned shipper. We can write a readable query using the
`IFNULL` function.

```sql
SELECT order_id,
       IFNULL(shipper_id, 'Not assigned') AS shipper
FROM orders o
```

Sometimes, it is useful to have a fallback value in a record. This is when we
use the `COALESCE` function.

```sql
SELECT order_id,
       COALESCE(shipper_id, comments, 'Not assigned') AS shipper
FROM orders o
```

In the example above, the `shipper_id` will be used if it is not null. If
`shipper_id` is null, we check if the given `comments` value is null. If not, we
use it. If it is, we display `Not assigned`.

Another example, using string functions too:

```sql
SELECT CONCAT(first_name, ' ', last_name) AS customer,
       IFNULL(phone, 'Unknown')           AS phone
FROM customers
```

### The `IF` function

`IF` takes a boolean statement, what to do if the statement is true, and what to
do if the boolean statement is false.

```sql
SELECT order_id,
       order_date,
       IF(YEAR(order_date) = YEAR(NOW()),
          'Active',
          'Archived') AS category
FROM orders o
```

An example where we want to display the frequency that a product has been
ordered:

```sql
SELECT product_id,
       name,
       COUNT(*)                                   AS orders,
       IF(COUNT(*) > 1, 'Many times', 'One time') AS frequency
FROM products p
         JOIN order_items oi USING (product_id)
GROUP BY p.product_id
```

### The `CASE` operator

If we need several conditionals, we use the `CASE` operator.

```sql
SELECT order_id,
       CASE
           WHEN YEAR(order_date) = YEAR(NOW())
               THEN 'Active'
           WHEN YEAR(order_date) = YEAR(NOW()) - 1
               THEN 'Last year'
           WHEN YEAR(order_date) < YEAR(NOW()) - 1
               THEN 'Archived'
           ELSE 'Future'
           END AS category
FROM orders o
```

#### An exercise using `CASE`

Recreate the following report:

![Case function 1](./img/case-function/1.png)

We get the following:

```sql
SELECT CONCAT(first_name, ' ', last_name) AS customer,
       points,
       CASE
           WHEN points >= 3000
               THEN 'Gold'
           WHEN points >= 2000 AND points < 3000
               THEN 'Silver'
           ELSE
               'Bronze'
           END                            AS status
FROM customers c
ORDER BY points DESC
```

## Views

### Creating Views

Queries with sub-queries can quickly become quite complex and consequently
difficult to read.

We can save a given sub-query in a view, and then reuse it later!

```sql
SELECT
    c.client_id,
    c.name,
    SUM(invoice_total) AS total_sales
FROM clients c
         JOIN invoices i ON c.client_id = i.client_id
GROUP BY c.client_id, name
```

In the future, we might have other queries that are based on this sub-query.
Hence, we make a view out of it:

```sql
CREATE VIEW sales_by_client AS
(
SELECT c.client_id,
       c.name,
       SUM(invoice_total) AS total_sales
FROM clients c
         JOIN invoices i ON c.client_id = i.client_id
GROUP BY c.client_id, name
    )
```

This create a view object, not a result table. We can select data from this
view, but we can also use this view just like a table.

```sql
SELECT * FROM sales_by_client;
```

We can also join this result with a new table.

Views are very powerful, and can greatly improve readability when performing
sub-queries.

**Remember:** Views do not store data. Rather, they provide a _view_ of the
underlying data in the tables.

#### An exercise on views

Create a view to see the balance for each client. The balance is calculated by
taking `invoice_total - payment_total`.

```sql
CREATE VIEW clients_balance AS
(
SELECT c.client_id,
       name,
       (invoice_total - payment_total) AS balance
FROM clients c
         JOIN invoices i ON c.client_id = i.client_id
    )
```

### Altering and Dropping Views

One way to alter a view is to drop it, and create an updated one.

```sql
DROP VIEW clients_balance;
```

The other way is to use the `CREATE OR REPLACE` keyword.

```sql
CREATE OR REPLACE VIEW clients_balance AS
(
SELECT c.client_id,
       name,
       (invoice_total - payment_total) AS balance
FROM clients c
         JOIN invoices i ON c.client_id = i.client_id
    )
```

It is recommended to back up views in a source control system like Git.

### Updatable Views

We can also use views in `INSERT`, `UPDATE` and `DELETE` statements, **but only
under certain circumstances**.

If the view **does not have** any of the following, we refer to that view as an
**updatable view**:

- The `DISTINCT` keyword
- Any aggregate function (e.g. `MIN`, `MAX`, `SUM`, etc...)
- Any `GROUP BY` or `HAVING` keywords
- The `UNION` keyword

This means that we can use that statement in inserts, updates and deletes.

The example below fulfills all criteria mentioned above:

```sql
CREATE OR REPLACE VIEW invoices_with_balances AS
(
SELECT invoice_id,
       number,
       client_id,
       invoice_total,
       payment_total,
       (invoice_total - payment_total) AS balance,
       invoice_date,
       due_date,
       payment_date
FROM invoices
WHERE (invoice_total - payment_total) > 0
    )
```

We can use this to add, update or delete data.

```sql
DELETE
FROM invoices_with_balances
WHERE invoice_id = 1;

UPDATE invoices_with_balances
SET due_date = DATE_ADD(due_date, INTERVAL 2 DAY)
WHERE invoice_id = 2;
```

### The `WITH OPTION CHECK` Clause

When you update or delete data through a view, an affected row might disappear.
There are times where you want to prevent this.

```sql
CREATE OR REPLACE VIEW invoices_with_balances AS
(
SELECT invoice_id,
       number,
       client_id,
       invoice_total,
       payment_total,
       (invoice_total - payment_total) AS balance,
       invoice_date,
       due_date,
       payment_date
FROM invoices
WHERE (invoice_total - payment_total) > 0
    )
WITH CHECK OPTION;
```

**Note the `WITH CHECK OPTION` at the end of the query**. When we apply this
clause, this view will prevent update or delete statements from excluding rows
from the view.

Using `WITH CHECK OPTION`: If we try to modify a row that would cause it to no
longer be part of a given view, you will get an error.

### Other Benefits of Views

We have seen that **views can simplify queries**.

Views can also **reduce the impact of changes**. Imagine you have 10 queries
written for a table. Then you want to change the name of that table, or the
structure of it. You would have to fix all 10 queries for that modified table.
What if you were not allowed to write any queries against this table, making it
"impossible" to update the table? Instead, we can use a view to act as an
**interface** between you and the table.

We can also use views to **restrict access to the data** of the underlying
table. For instance, we can use a `WHERE` clause to only expose the rows of a
table that fulfill a given criteria.

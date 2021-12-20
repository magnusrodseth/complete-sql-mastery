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

TODO: Fill in information here.

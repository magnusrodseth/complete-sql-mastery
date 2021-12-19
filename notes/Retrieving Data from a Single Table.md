# Retrieving Data from a Single Table

## The `SELECT` statement

### Using a database

Use the `USE` keyword to select a database to perform queries against.

```sql
USE sql_store;
```

### Selecting data

Use the `SELECT` keyword to select data from a table.

```sql
SELECT *
FROM customers;

SELECT customer_id, first_name
FROM customers;
```

### Filter data

Use the `WHERE` keyword to filter data.

```sql
SELECT customer_id, first_name
FROM customers
WHERE customer_id = 1;
```

### Sort result on a given column

Use the `ORDER BY` keyword to sort results on a given column.

```sql
SELECT customer_id, first_name
FROM customers
ORDER BY first_name;
```

### Comment out segments

Use `--` to comment out queries.

```sql
SELECT customer_id, first_name
FROM customers
-- ORDER BY first_name;
```

## The `SELECT` Clause

### Arithmetic operations

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

### Aliases

We can give the new column an alias using the `AS` keyword.

```sql
SELECT last_name,
       first_name,
       points,
       (points + 10) * 100 AS discount_factor
FROM customers;
```

### Remove duplicates

We can remove duplicates using the `DISTINCT` keyword.

```sql
SELECT DISTINCT state
FROM customers;
```

## The `WHERE` Clause

### Used as a conditional

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

### Filter on dates

We can use the **standard notation for dates** when filtering for dates.

```sql
SELECT *
FROM customers
WHERE birth_date > '1990-01-01';
```

## The `AND`, `OR` and `NOT` operators

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

## The `IN` operator

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

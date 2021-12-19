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

## The `SELECT` clause

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

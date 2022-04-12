# Indexing for high performance

## Indexes

Indexes are data structures that database engines use to find data quickly. As
an analogy, think of a telephone registry.

Let's say we want to find customers that live in California. As the number of
records in a table grow, the cost of this operation increases drastically.

We speed up the query by creating an index on the `state` column. This is like
creating a collection of customers, sorted by their state.

This index only has a reference to the actual customers table.

![Indexes](./img/indexes.png)

This is way faster than reading every single record.

In many cases, indexes are small enough that they can fit into memory, rather
than live on disk. That is why it is much faster to use them to find data.

![Memory and disk](./img/memory-disk.png)

### Cost of indexes

- **Increase the size of database**, because they need to be permanently store
  next to our tables.
- **Slow down the write operations**, because we need consistent state across
  tables and indexes when writing to the database.

**Because of this, we should reserve indexes for performance critical queries.**

**Design indexes based on your queries, not your tables.** The whole point of
using an index is to speed up a slow query.

### Implementation detail

Internally, indexes are often stored as binary trees. In the TDT4145 course, we
use B+ trees.

![Implementation detail](./img/implementation-detail.png)

## Creating indexes

```sql
EXPLAIN SELECT customer_id
FROM customers c
WHERE state = 'CA'
```

Using the `EXPLAIN` keyword, MySQL explains the executed query.

We see that the `type=ALL` and the total amount of rows (1010). This `SELECT`
query looks through all rows in the `customers` table.

Now, we create an index on the `state` column.

```sql
CREATE INDEX idx_state ON customers (state);
```

When executing the `SELECT` query above, we see something else. We see that the
`type=ref`, `possible_keys=idx_state`, `key=idx_state`. The amount of rows is
now 112. That means we decreased the amount of rows to search through by a
factor of 10!

**This means that by creating the `idx_state`, a query using
`WHERE state='something'` internally uses the newly created index**.

### Exercise: Creating indexes

_Write a query to find customers with more than 1000 points. Use `EXPLAIN` to
see the difference before and after creating the index._

```sql
EXPLAIN
SELECT customer_id
FROM customers
WHERE points > 1000;
```

This searches through all rows.

```sql
CREATE INDEX idx_points ON customers (points);
```

Running the `SELECT` query again, we get `type=range`, `key=idx_points`,
`rows=529`.

## Viewing indexes

```sql
SHOW INDEXES IN customers;
```

![Viewing indexes](./img/viewing-indexes.png)

We see the primary key is an auto-generated index, such that we can quickly look
up records on their index. **This is also called a clustered index**.

Collation represents how data is sorted in the index. `A` means ascending. `B`
means descending.

Cardinality represents an estimated number of unique values in the index. The
cardinality of the primary key naturally equals the number of records in the
table. **Note that this is an estimate**.

To get a more specific number of records, we use `ANALYZE`, followed by
`SHOW INDEXES`.

```sql
-- Updates the state of customers
ANALYZE TABLE customers;

-- This will now display more correct information
SHOW INDEXES IN customers;
```

The other indexes in the table, `idx_state` and `idx_points` are called
**secondary indexes**.

For any secondary index, the primary key is included **in order to reference
back to the correct record in the table**. For instance, for a given amount of
points, we need to be able to trace back the ID that the points belong to.

The index type for all these indexes are `BTREE`, short for binary tree. Again,
note that in the TDT4145 course, we use B+ trees.

### Indexes and foreign keys

Now let's look at the indexes in the `orders` table.

```sql
SHOW INDEXES IN orders;
```

![Indexes in orders](./img/indexes-in-orders.png)

We also have secondary indexes that are placed on the **foreign key columns**.
MySQL automatically adds indexes to the foreign key columns.

## Prefix indexes

If the column we want to index is a string column, e.g. `CHAR`, `VARCHAR`,
`TEXT` or `BLOB`, our index may consume a lot of space and won't perform well.

Hence, smaller indexes are better, because they fit in memory. This makes our
searches faster.

Thus, when indexing string columns, we only want to include a prefix to the
column in order to save space.

```sql
-- Create a prefix index on the first 20 characters of the last_name column
CREATE INDEX idx_last_name ON customers (last_name(20));
```

Choosing the length of the prefix must be seen in context of what we're
indexing. Using 1 character here is no good; many people have different last
names starting with the same character.

### Choosing a good prefix index

We know the amount of rows we have (1010) by executing
`SELECT COUNT(*) FROM customers`.

We want to find a prefix of unique entries that **approaches** this number, with
the minimum amount of characters in the prefix. Observe the following query:

```sql
SELECT COUNT(DISTINCT LEFT(last_name, 1)), -- Returns 25
       COUNT(DISTINCT LEFT(last_name, 5)), -- Returns 966
       COUNT(DISTINCT LEFT(last_name, 10)) -- Returns 996
FROM customers
```

We only see a small improvement from prefix length 5 to prefix length 10.
**Thus, we should use 5 as a prefix here**.

```sql
DROP INDEX idx_last_name ON customers;
CREATE INDEX idx_last_name ON customers (last_name(5));
SHOW INDEXES IN customers;
```

The `idx_last_name` has a `Sub_part` of 5, which is equivalent to the prefix.

## Full-test indexes

Let's say we want to create a blog. The user searches for `"react redux"` to
find posts matching this content. We want to match content appearing in the
title or the body, not necessarily right next to each other.

```sql
CREATE FULLTEXT INDEX idx_title_body ON posts (title, body);
```

We create a full-text index on the two relevant columns.

Now, we use the built-in `MATCH` keyword to match a phrase against the specified
columns.

```sql
SELECT *
FROM posts
WHERE MATCH(title, body) AGAINST('react redux');
```

This returns the following:

![React Redux query](./img/react-redux.png)

Based on several factors, MySQL calculates a relevancy scores for the results,
which is a floating point number between 0 and 1.

We update the query:

```sql
SELECT *,
       MATCH(title, body) AGAINST('react redux') as relevancy
FROM posts
WHERE MATCH(title, body) AGAINST('react redux')
ORDER BY relevancy DESC;
```

This gives the following result:

![Relevancy](./img/relevancy.png)

We can also use `MATCH` in boolean mode to get more fine-grained search.

```sql
SELECT *,
       MATCH(title, body) AGAINST('react redux') as relevancy
FROM posts
WHERE MATCH(title, body) AGAINST('react -redux' IN BOOLEAN MODE)
ORDER BY relevancy DESC;
```

This searches for posts with `react` and not `redux`.

We can also add words as requirements:

```sql
SELECT *,
       MATCH(title, body) AGAINST('react redux') as relevancy
FROM posts
WHERE MATCH(title, body) AGAINST('react -redux +form' IN BOOLEAN MODE)
ORDER BY relevancy DESC;
```

This requires the title or body to include `form`.

## Composite indexes

We can combine indexes for certain queries.

```sql
CREATE INDEX idx_state_points ON customers (state, points);

EXPLAIN SELECT customer_id FROM customers WHERE state='CA' AND points >1000;
```

We can see that MySQL realized that the new composite index does a better job
than any of the previous indexes, e.g. `idx_points` or `idx_state`.

![Composite index](./img/composite-indexes.png)

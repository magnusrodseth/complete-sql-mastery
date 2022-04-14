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

## Order of columns in composite indexes

There are some rules to choosing the order of columns in composite indexes:

1. **Put the most frequently used columns first**.
2. **Put the columns with a higher cardinality first**.

Note that **you should always take your data and your query into account; don't
follow the 2 rules above blindly**.

## Index maintenance

Watch for **duplicate indexes** and **redundant indexes**.

Duplicate indexes are indexes on the same set of columns in the same order, e.g.
`(A,B,C)` and `(A,B,C)`. Sometimes, you may create a duplicate index without
knowing it. This oftentimes happens when you create an index without looking at
the existing indexes.

Redundant indexes are a bit different. Let's say you have an index on columns
`(A,B)`, and then create a new index on column `(A)`, this is **redundant**.
However, if you create an index on columns `(B,A)` or just column `(B)`, this is
not redundant. This also oftentimes happens when you create an index without
looking at the existing indexes.

**Always check existing indexes before creating a new index**.

## ✅ Performance Best Practices

- **Smaller tables perform better**. Don't store the data you don't need. Solve
  today's problems, not tomorrow's future problems that may never happen.

- **Use the smallest data types possible**. If you need to store an age, a
  `TINYINT` is sufficient. No need to use `INT`. Saving a few bytes is not a big
  deal in a small table, but has significant impact in a table with millions of
  records.

- **Every table must have a primary key**.

- **Primary keys should be short**. Prefer `TINYINT` over `INT` if you only need
  to store 100 records.

- **Prefer numeric types to strings for primary keys**. This makes looking up
  records by the primary key faster.

- **Avoid BLOBs**. They increase in size of your database and have a negative
  impact on performance. Store your files on disk if you can.

- **If a table has too many columns, consider splitting it into two related
  tables using a one-to-one relationship**. This is called **vertical
  partitioning**. For instance, you may have a `customers` table with columns
  for storing their `address`. If these columns don't get read frequently, split
  the table into two tables (`customers` and `customers_metadata` or something
  like that).

- **In contrast, if you have several `JOIN` statements in your queries due to
  data fragmentation, consider denormalizing the data**. Denormalizing is the
  opposite of normalization. It involves duplicating a column from one table in
  another table, in order to reduce the number of joins required.

- **Consider creating summary / cache tables for expensive queries**. For
  instance, if the query is to fetch the list of forums, and the number of posts
  in each forum is expensive, create a table called `forums_summary` that
  contains the list of forums and the number of posts in them. You can use
  events to regularly refresh the data in this table. You may also use triggers
  to update the counts every time there is a new post.

- **Full table scans are a major cause of slow queries**. Use `EXPLAIN` and look
  for queries with `type=ALL`. These are full table scans. Use indexes to
  optimize these queries.

- **When designing indexes**:

  1. **Look at the columns in your `WHERE` clauses first**. Those are the first
     candidates for indexes because they help narrow down the searches.
  2. **Next, look at the columns using in the `ORDER BY` clause**. If they exist
     in the index, MySQL can scan your index to return ordered data without
     having to perform a sort operation (filesort).
  3. **Finally, consider adding the columns in the `SELECT` clause to your
     indexes**. This gives you a **covering** index that covers everything your
     query needs. MySQL doesn't need to retrieve anything from your tables.

- **Prefer composite indexes to several single-column indexes**.

- **The order of columns in indexes matter**. Put the most frequently used columns and the columns with higher cardinality first, but always take your queries and your data into account.

- **Remove duplicate, redundant and unused indexes**. Duplicate indexes are the indexes on the same set of columns with the same order. Redundant indexes are unnecessary indexes that can be replaced with existing indexes. For instance, if you have an index on columns `(A,B)` and create another index on column `(A)`, the latter is redundant because the former index can help.

- **Don't create an index before analyzing the existing indexes**.

- **Isolate your columns in your queries, so that MySQL can use your queries**.

- **Avoid `SELECT *`**. Most of the time, selecting all columns ignores your indexes and returns unnecessary columns you may not need. This puts an extra load on your database server.

- **Return only the rows you need**. Use the `LIMIT` clause to limit the number of rows returned.

- **Avoid `LIKE` expressions with a leading wildcard**. For instance, `LIKE '%name'`.

- **If you have a slow query that uses the `OR` operator, consider chopping up the query into 2 queries that utilize separate indexes and combine them using the `UNION` operator**.

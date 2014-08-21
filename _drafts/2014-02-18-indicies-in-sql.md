---
layout: post
title: "Make Your SQL Queries Logarithmically Faster with Indexing"
comments: true
---

*This is the fifth post in a [series of posts]({% post_url 2014-04-17-data-science-sql %})
about doing data science with SQL. The 
[previous post]({% post_url 2014-08-14-filters-joins-aggregations %})
went over the basics of querying in SQL.*

In this post, I will discuss indexing in SQL. Indexing is a caching
mechanism in databases that is often necessary to achieve fast read
performance.

# Sorting a Table in SQL Using an Index

One of the biggest benefits of relational databases is that they
separate the design of the table from 
internal implementation of how the data is stored. 
In addition, SQL abstracts the logical query from implementation
details about how it is performed.

But unfortunately, often times the SQL
[Query optimizer](http://en.wikipedia.org/wiki/Query_optimization)
is unable to naively find an efficient algorithm to query a table
because tables in SQL are inherently unsorted.  So instead, SQL has
to resort to simple and inefficient algorithms like scanning every
row.

One can often significantly improve the efficiency of a particular
query using an index.  An index is a sorted lists of rows in the
table that can be used for more efficient binary searching. Often, an
optimal index can allow the database to perform a query in
logarithmic time.

It is illustrative to consider querying from
a simple table of `people`:

| person_id | First |      Last | gender | birthdate |
| --------- | ----- | --------- | ------ | --------- |
|         0 |  Alex | Garfunkle |      m |  19930527 |
|         1 |  Alex |     Young |      m |  19701201 |
|         2 | Sarah |     Riley |      f |  19890314 |
|         3 |  Alex |  Williams |      m |  19400105 |
|         4 | Sarah |     Smith |      f |  19910317 |
|         5 |  Jane | Fredricks |      f |  19920731 |
|         6 |  Alex |     Young |      f |  19920731 |

Suppose we wanted to compute, for example, all the IDs for people
with a given first name.  The SQL query is straightforward:

```sql
SELECT person_id
  FROM people
 WHERE first = 'Alex'
```

Unfortunately, SQL tables are inherently unsorted so SQL does not
have any good way to know where these rows would be.  The database
has to resort to looking through every row in the table, doing what
it calls a [full table
scan](https://dev.mysql.com/doc/refman/5.5/en/glossary.html#glos_full_table_scan).
This has a run-time efficiency of O(n) where n is the number of rows in the table.
For large tables, this can be prohibitively expensive.

To improve the query efficiency, we could imagine pre-sorting our
table based on the first names and then [binary
searching](http://en.wikipedia.org/wiki/Binary_search_algorithm) through
the rows to find what we are looking for.  This would have a run-time
efficiency of O(log(n)) and would be significantly faster.

Instead of sorting the table, typically we create an index which
create an accompanying sorted list of rows in the table.  For our
example above, we would need to create an index on first name.  For
our example, the syntax is

```sql
CREATE INDEX people_first_indx
ON people (last)
```

This creates an accompanying index sorted by first name:

| row_id | First |
|------- | ----- |
|      0 |  Alex |
|      1 |  Alex |
|      3 |  Alex |
|      6 |  Alex |
|      5 |  Jane |
|      4 | Sarah |
|      2 | Sarah |

Here, the row ID column refers to the location in the original table
where the row exists.  Note that the index also contains the column
that is being indexed on to allow for fast searching.

Given this index, we could binary search for people of a given first
name. Given the matching rows, we could look up accompanying
information in the original table.

Before we move on, I will mention that some databases allow for
sorting the underlying tables.  This is called a Clustered Index
in Oracle SQL or Index-organized Tables in Microsoft SQL Server.
But these are uncommon because a good sort order for one popular
query might be bad for another. Instead, people typically use
accompanying indices and leave the original table unsorted.

# Upsides and Downsides of Indexing

Indices have several benefits.  An index can make a particular query
much faster.  Because indices are not part of the underlying table
data, multiple indices can be created to facilitate different kinds
of queries. Finally, indices are automatically updated whenever the
table is updated so they don't require additional work to maintain.

On the other hand, indices have to be automatically updated whenever
tables are modified. This overhead harms the performance of commands
that alter the table.  In addition, they take up additional space
since they have to store a copy of much of the original table.  So
the best practice is to only use indices for queries where better
read performance is needed.

# Multi-column Indices in SQL


SQL can index on multiple columns. This can be used to make queries
with multiple columns in the WHERE clause more efficient.

For example, we might be interested in the ID of all people with a
particular first name, last name, and gender:

```sql
SELECT recipe_id
  FROM people
 WHERE first='Alex'
   AND last='Young'
   AND gender='f'
```

For this query, our index from above (on first name) is not optimal
because the rows for all users with first name 'Alex' are unsorted.

SQL allows for indexing on multiple columns, which will sort
by successive columns for rows that have identical values of preceding
columns. For our example, we could index on first name, then last name,
and then gender:

```sql
CREATE INDEX people_first_last_gender_indx
ON people (first,last,gender)
```

This will create the index:

|     index | first |      last | gender |
| --------- | ----- | --------- | ------ |
|         0 |  Alex | Garfunkle |      m |
|         3 |  Alex |  Williams |      m |
|         6 |  Alex |     Young |      f |
|         1 |  Alex |     Young |      m |
|         5 |  Jane | Fredricks |      f |
|         2 | Sarah |     Riley |      f |
|         4 | Sarah |     Smith |      f |

Given this sorting, we can then directly binary search for
the rows required by the query.

One important question is in which order to list columns in an
index.  The best index order typically depends on what other queries
we expect to run.  The index on (first,last,gender) works well for
queries which filter only on the first name.  On the other hand,
if we planned to filter only for users with a given last name:

```sql
SELECT recipe_id
  FROM people
 WHERE last='Young'
```

Then it would be better to index on (last,first,gender)

| person_id | last      | first | gender |
| --------- | --------- | ----- | ------ |
|         5 | Fredricks |  Jane |      f |
|         0 | Garfunkle |  Alex |      m |
|         2 |     Riley | Sarah |      f |
|         4 |     Smith | Sarah |      f |
|         3 |  Williams |  Alex |      m |
|         6 |     Young |  Alex |      f |
|         1 |     Young |  Alex |      m |

Note that there is a common belief that it is better to index first
on the most selective column (with the highest cardinality).
This is considered to be mostly a myth.  See
[here](http://use-the-index-luke.com/sql/myth-directory/most-selective-first)
for a detailed explanation.

# Index-Only Scan

If columns in the SQL query reference columns that do not exist in
the index, then SQL will have to look for data in the original table
to find the required information. On the other hand, if all the columns
are part of the filter, then SQL can perform the query without
referencing the original table (with what is called an Index-Only
Scan).

For the example query from above:

```sql
SELECT recipe_id
  FROM people
 WHERE first='Alex'
   AND last='Young'
   AND gender='f'
```

The best index would also include the recipe_id column:

```sql
CREATE INDEX people_first_last_gender_recipe_id_indx
ON people (first,last,gender,recipe_id)
```

On the other hand, adding additional columns
to the index increases the memory required
to store it and also the difficulty of keeping it
up to date. So unless performance is critical, it is
often not necessary to index on too many columns.


# Primary Key Indices

In SQL, [primary
keys](http://dev.mysql.com/doc/refman/5.5/en/optimizing-primary-keys.html)
also act as indices on a table.  The reason for this is that SQL
needs an efficent way to tell if a new row is a duplicate. Keeping
an index allows for this test very efficiently.

The primary key should be picked based on the logical correctness
of a table. But it is good to order columns in the index based on
the queries which are most common.

# Indexing on Inequalities

Indexing on inequalities is somewhat different.  For example,
supposed we wanted to find all people with first name Alex born
after 1967. The query is:

```sql
SELECT *
  FROM people
 WHERE first = 'Alex'
   AND birthdate > 19670000
```

If our index was on (first, birthdate), we would create
the index:

| index |    birthdate |    First |
| ----- | ------------ | -------- |
|     3 |     19400105 |     Alex |
| **1** | **19701201** | **Alex** |
|     2 |     19890314 |    Sarah |
|     4 |     19910317 |    Sarah |
| **6** | **19920731** | **Alex** |
|     5 |     19920731 |     Jane |
| **0** | **19930527** | **Alex** |

The rows which would be selected are in bold.  This index is not
particularly efficient because the requires rows are not together in
the table.

Instead, it would be more efficient to index on (first,birthdate):

| index |    First |    birthdate |
| ----- | -------- | ------------ |
|     3 |     Alex |     19400105 |
| **1** | **Alex** | **19701201** |
| **6** | **Alex** | **19920731** |
| **0** | **Alex** | **19930527** |
|     5 |     Jane |     19920731 |
|     2 |    Sarah |     19890314 |
|     4 |    Sarah |     19910317 |

Given this index, we could easily select all people named Alex and,
for those people, quickly scan for people in the desired age range.

The general rule is that when you are indexing for an inequality,
it is better to but the inequality column at the end of the index.


# Function-based Indexing

Indexing when the were clause contains function of parameters can complicated.
To SQL, it has no general way to know how the sorting of a function of a column
relates to the sorting of the underlying column, and therefore
the function of a column will naturally break the sorting.

As a concrete example, we could imagine trying to find all people who are at least
21 years old. This is easy to do by adding 21 years to the birthday and seeing if
that is after today. In MySQL, the command would be:

```
SELECT *
FROM people
WHERE date(birthdate) + interval 21 YEAR < DATE()
```

Assuming we had an index which began with birthdate, you might
expect this query to be bast. But because it computes a function of
the column, SQL cannot use the index.

The easiset way to solve this problem would be to shuffle
the logic to the other side of the equation:


```
SELECT *
FROM people
WHERE birthdate < CAST(DATE() - interval 21 YEAR AS UNSIGNED INTEGER)
```

Although logically the same, this query can be much more efficient
becaues the right hand side is only computed once and the index on
birthday can properly be used.


As a similar challenege, imagine if we wanted to find all people
born in 1981. We could do so by running a query like:

```sql
SELECT *
FROM people
WHERE FLOOR(birthdate/10000) = 1981
```

A few solutions in SQL:

* If possible, we could rebuild our table to have year as its own
column and then index on that column.
* We could require the query as an inequality match:

  ```sql
  SELECT *
  FROM people
  WHERE birthdate > 19810000 AND birthdate < 19820000
  ```
* In some databases, you can actually index on functions of
  columns. For exmple, in PostgreSQL, the syntax would 
  look like:

  ```sql
  CREATE INDEX birthyear_people
  ON people FLOOR(birthdate/10000)
  ```

  Function-based indexing is supported by Oracle, PostgreSQL,
  and SQL Server, but not by MySQL. 

More stuff:

* Non-deterministic Functions -> Can't index on the age of somebody...
* Queries With Pattern Matching
  
  ...
  Find people whose first name starts with the 'Al'
  is easy. Finding people whose names end with 'Ax'
  is hard.

# Indexing for the ORDER BY Clause

SQL databases

Proper index for the `ORDER BY` clause is straightforward.

# Indexing for the GROUP BY Clause

# Indexing for the JOIN Clause



# Database Implementation of Indicies in SQL

Despite giving all the right intuition,
the simple picture of an index being a sorted array is a bit
idealized. It gives all the right intiution for
building the best index for a given query. 

But it is illustrative to undertsand how actually work.
The operations requried of a binary tree are

1. Being able to binary search for the desired value
2. Being able to linearly search (for finding other keys of the same value)
3. Fast insertion/deletion of rows when the table is modified.

Storing the data in a static array would accomplish 1 and 2.
A linked list would allow 2 and 3. And a balanced binary search
tree would acomplish 1 and 3.

To get all three benefits, SQL has to resport to a more complicated
data structure called a balanced B-tree where each of the 
notes is 

# Indexing in SQL Cheat Sheet

Here are the major takeaways about indexing in SQL:

1. An index is a sorted list of columns in a database.
2. Indexing in SQL improves query efficiency at the expense
   of increased storage space and difficult in table modification.
4. Indexing on all of the columns used in the `WHERE` clause
   allow for binary searching for the required rows.
5. When indexing on an inequality, it is most efficient to 
   put the inequality column as the last column in the index.
6. When indexing for a `GROUP BY` query, make the column
   being aggregated come in the index after the columns in the
   `WHERE` clause.
7. When indexing for a `JOIN` query, make the column
   being joined come in the index after the columns in the `WHERE`
   clause.
8. When possible, avoid querying on the functions of parameter
   because this will break indexing.

# Further resources

* http://use-the-index-luke.com/

{% include twitter_plug.html %}

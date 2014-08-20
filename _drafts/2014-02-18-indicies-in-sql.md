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
performance on a database.

# Sorting a Table in SQL Using an Index

One of the biggest benefits of relational databases is that they
separate the layout of our data from the implementation of
how the data should be stored. Similarly, they abstract
away from the SQL query infomration about how the query should be performed.

But unfortunately, often times the SQL
[Query optimizer](http://en.wikipedia.org/wiki/Query_optimization)
is unable to naively find an efficient algoirthm to query the data
because tables in SQL are inherently unsorted.
So instead, SQL has to resort to simple and innefficently algorithms like
scanning every row.

For a particular query, one can often significantly improve the
efficiency of the querying using an index.  An index is a sorted
lists of rows in the table that can be used for more efficient
querying.  Typically, an appropriate index can convert an algorithm
to run in logarithmic time.  What is convenient is that the index
is stored aside from the original table since it is only a computational
aid.  In addition, you can add as many indices as are needed.

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

Unfortunately, SQL tables are inherently unsorted, so SQL does not
have any good way to know where these rows are. So the database
would have to look through every row in the table, doing what it
calls a [full table
scan](https://dev.mysql.com/doc/refman/5.5/en/glossary.html#glos_full_table_scan).
This has a run-time efficiency of O(n) where n is the number of rows.
For large tables, this can be prohibatively expensive.

To improve the query efficiency, we could imagine pre-sort our table based on
the last names and then [binary
searching](http://en.wikipedia.org/wiki/Binary_search_algorithm) through
the rows to find what we are looking for.  This would have a run-time
efficiency of O(log(n)) which would be significantly faster.

Some databases allow for sorting the underlying tables
(see Clustered Index in Oracle SQL or
Index-organized table in Microsoft SQL Server).  But in particular,
this is uncommon because a good sort order for one popular query
might be bad for another.

For the most part, sorting data in SQL is costly and impractical
and would only speed up one kind of query.
Instead, typically SQL tables are left unsorted and an accompanying
index stores a sorted list of all the rows in the table.
This allows for the original table to be unmodified while allowing
accompanying information to allow for faster querying.

For our example above, we would need a list of rows sorted by last name,
so we would index on last name.

The syntax to create an index is:

```sql
CREATE INDEX index_name
ON table_name (column_name)
```

For our example, the command would be

```sql
CREATE INDEX people_first_indx
ON people (last)
```

This creates an accompanying table
sorted by first name:

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
where the row exists.  Note that the index also contains all of the
columns from the table that exist in the index to allow for quick
searching. But finding other information from the original table
would require searching in the original table by row_id.

Given this index, we could binary search for people of a given first
name, and then linearly step through the index to find all following
rows matching the query.

# Upsides and Downsides of Indexing

Indices have several benefits.  For a common type of query, they
can make the query run much faster.  In addition, because they are
not part of the underlying table data, multiple indices can be
created to facilitate different kinds of queries.  Indices are
automatically updated whenever the table is updated so they don't
require extra book keeping by the user.  And finally, they are
stored separately from the original table, which allows for a logical
separation of the actual data from particular information about how
the data will be queried.

On the other hand, indices have to be automatically updated whenever
tables are modified. This overhead will degrade the performance of
modifying the database.  In addition, they can take up a lot of
space since they have to store a copy of much of the original table.
So the best practice is to only use indices for queries which are
slow and which you plan to run frequently.


# Multi-column Indices in SQL

For queries with multiple conditions in the WHERE clause, SQL can
be more efficient by indexing on multiple columns.

For example, we might be interested in the ID of all people with a
particular first name, last name, and gender.  We could retrieve
this using a query like:

```sql
SELECT recipe_id
  FROM people
 WHERE first='Alex'
   AND last='Young'
   AND gender='f'
```

For this query, our index from above (on last name) would work.  
But it is not  optimal because there is no inherent sorting to
all of the rows in the index with a given first name.  So given the
index, we would have to step through all of the rows to find the
required IDs. This could be very inefficinet if there were many
rows with first name Alex.

To avoid this, SQL allows for indexing on multiple columns. When
you index on multiple columns, it will sort by the first column,
and then for similar values by successive columns.

Therefore, we could create an index on all three columns used in
the WHERE clause:

```sql
CREATE INDEX people_first_last_gender_indx
ON people (first,last,gender)
```

This will create a index which sorts first on first name,
then on last name for equal first names, and finally on gender.

|     index | first |      last | gender |
| --------- | ----- | --------- | ------ |
|         0 |  Alex | Garfunkle |      m |
|         3 |  Alex |  Williams |      m |
|         6 |  Alex |     Young |      f |
|         1 |  Alex |     Young |      m |
|         5 |  Jane | Fredricks |      f |
|         2 | Sarah |     Riley |      f |
|         4 | Sarah |     Smith |      f |

Given this sorting, we can then directly do a binary search for
rows with a given first name, last name, and gender.

One important question that arises is in which order we should list
columns in the index.  The benefit of indexing on (first,last,gender)
is that this index will also work for queries which filter
only on first name or only on first and last name.
But if on the other hand if we knew we were going to filter for
rows only with a given last name:

```sql
SELECT recipe_id
  FROM people
 WHERE last='Young'
```

Then it would be better to index on (last,first,gender)
so that the index would be more versatile.

| person_id | last      | first | gender |
| --------- | --------- | ----- | ------ |
|         5 | Fredricks |  Jane |      f |
|         0 | Garfunkle |  Alex |      m |
|         2 |     Riley | Sarah |      f |
|         4 |     Smith | Sarah |      f |
|         3 |  Williams |  Alex |      m |
|         6 |     Young |  Alex |      f |
|         1 |     Young |  Alex |      m |

Note that there is a common belief
that it is better to index first on the most
selective column first (with the highest cardinality).
This is considered to be mostly a myth. 
See [here](http://use-the-index-luke.com/sql/myth-directory/most-selective-first)
for a more detailed explanation.

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
also perform as indicies on a table. This makes sense when you
realize that a primary key constrains duplicate rows from being
inserted into a row, which requires quickly looking for duplicate
rows. Although the columns in a primary key are dictated by teh
logical correctness of a table, it is good to think about the best
order to place the columns based on the queries which you plan to
run and the cardianity of the columns.

# Indexing on Inequalities

Indexing on inequalities requires a somewhat different logic than
equality indexing.  For example, supposed we wanted to find all
people with first name Alex born after 1967.  We would run the
query:

```sql
SELECT *
  FROM people
 WHERE first = 'Alex'
   AND birthdate > 19670000
```

Given the rules about cardinality from above and the fact that
birthdate has the highest cardinality, we might expect the optimal
index for this query to be (first, birthdate).

| index |    birthdate |    First |
| ----- | ------------ | -------- |
|     3 |     19400105 |     Alex |
| **1** | **19701201** | **Alex** |
|     2 |     19890314 |    Sarah |
|     4 |     19910317 |    Sarah |
| **6** | **19920731** | **Alex** |
|     5 |     19920731 |     Jane |
| **0** | **19930527** | **Alex** |

As you can see from this example, this is not especially helpful.
We can easily find all of the people who were born after 1967, but
people with first name Alex are spread throughout the table.

Instead, it would be more efficient to index first on Alex, and
second on birthdate.

| index |    First |    birthdate |
| ----- | -------- | ------------ |
|     3 |     Alex |     19400105 |
| **1** | **Alex** | **19701201** |
| **6** | **Alex** | **19920731** |
| **0** | **Alex** | **19930527** |
|     5 |     Jane |     19920731 |
|     2 |    Sarah |     19890314 |
|     4 |    Sarah |     19910317 |

Given this index, we could easily select all people named Alex and
then for those people their birthdates are all contiguous so we
could easily scan through to find all the people born after 1967.

The general rule here is that when you are indexing for an inequality,
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

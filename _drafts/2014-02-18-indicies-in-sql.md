---
layout: post
title: "Indexing in SQL for the Rest of Us"
comments: true
---

<!--
* [Table Indicies](http://en.wikipedia.org/wiki/Database_index) are a concept in
  database design where sorted indicies of a table can be cached to
  improve performance of particular queries on a table.
-->

In this post, I will discuss indexing in SQL. Indexing is a caching
mechanism that is often necessary to obtain acceptable read performance
on a database.

One of the biggest benefits of relational databases is that they
separate layout of our data from the underlying implementation of
how the data should be stored.

Furthemore, they separate the logic of the questions (queries) you
want to ask of the database from details about how the queries
should be run.  Databases then use the the [Query
optimizer](http://en.wikipedia.org/wiki/Query_optimization)/[Query
execution plan](http://en.wikipedia.org/wiki/Query_plan) to find
an efficient way to write the query.

But this picture is a bit naieve because SQL often does not have
enough information to find a fast way to run a query and often
resorts to inefficient things like scaning the entire table.

To improve the efficiency of querying data from a database, one can
use indices which are separate sorted lists of rows in the table
that can be used for more efficient query algorithms.

## Sorting a Table in SQL Using an Index

An index in SQL is just a sorted list of all of our data in a table
where the index specifies the column order in which to sort rows.

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

PUT A NOTE ABOUT HOW DATES ARE STORED HERE.

Suppose we wanted to compute, for example, all the people with a
given last name.  The query is straightforward:

```sql
SELECT *
  FROM people
 WHERE last = 'Young'
```

But SQL does not automatically have any great way to run this query
since tables in SQL are inherently unsorted, so the database would
have to inefficiently look through every row in the table.  This is
called a [full table
scan](https://dev.mysql.com/doc/refman/5.5/en/glossary.html#glos_full_table_scan).
It has a run-time efficiency of O(n) where n is the nubmer of rows,
so it become increasly costly for larger tables.

To improve query efficiency, we could pre-sort our table based on
the last names and then [binary
search](http://en.wikipedia.org/wiki/Binary_search_algorithm) through
the names for the rows we are looking for.  Because binary searching
is O(log(n)), this algorithm woudl be significantly faster than a
full table scan.

In practice sorting the actual data would be very costly and would
limit this algorithmic improvement to only one kind of query.
Instead, it would be better to leave the underlying table in place
and instead store a sorted list of all of the row indicies.  This
sorted list coudl be used for algorithm improvements without altering
the underlyibgn data.

This is exactly what an index in SQL does.
For our example, we would index on the `last` column to
sorted list the row indicies sorted by last name.
For our example, the index would look like:

|          index (last) |
| --------------------- |
|         5 (Fredricks) |
|         0 (Garfunkle) |
|             2 (Riley) |
|             4 (Smith) |
|          3 (Williams) |
|             1 (Young) |
|             6 (Young) |


Note that the index in SQL only stores row indicies and nothing
about the content in the rows.  I have included the last names
paranthetically just for clarity.  For rows with the same last name,
there is no guarantee about their sorted order.

Given this index, we could binary search for people of a given last
name.  At each point in the search, we would have to look up in the
acutal table to see the particular last name for a given row index.

The syntax to create an index is:

```sql
CREATE INDEX index_name
ON table_name (column_name)
```

For our exmaple, the query would be
```sql
CREATE INDEX people_last_name_indx
ON people (last)
```

## Benefits of SQL indices

Indices have several benefits.  First, they don't store any of the
underlying underlying data so they are fairly memory-efficient.
Second, you can have multiple indicies on a table to improve different
kinds of querieies.  Third, they are automatically updated whenever
the table is updated so they don't require any additional bookkeeping.
And finally, they are stored separately from the original table,
which allows for a logical separation of the actual data from
particular information about how the data will be queriied.

On the other hand, indicies have to be automatically updated whenever
tables are modified. This overhead will degrate the performance of
modifying the database.  So indicies should only be added as necesary.

## Multi-column Indices in SQL

Indexing for querieis with 
multiple conditions in the `WHERE` 
cluase requires a little more though.

For example, we might be looking for a
person with a particular first name, last name, and gender
using a query like:

```sql
SELECT *
  FROM people
 WHERE first='Alex'
   AND last='Williams'
   AND gender='m'
```

SQL allows for indexing on multiple columns. When you index on
multiple columns, it first sort the rows by the
first column in the index. In situations where there are
identical rows with that column, it will then sort by
the second column.

first column and then for rows
with the same value by it will sort by the second column, etc.

In order to efficiently run this query, we can build an index which
sorts on (first, last, gender). What this means is we first sort
on the last name.  Whenver the last name is the same, we then sort
on first name. Whenever both are the same, we finally sort on gender.

```sql
CREATE INDEX amount_name
ON recipe_ingredients (first,last,gender)
```

This will create a index which sorts first on first name,
then on last name, and finally on gender.

| index (first,last,gender) |
|-------------------------- |
|      0 (Alex,Garfunkle,m) |
|       3 (Alex,Williams,m) |
|          6 (Alex,Young,f) |
|          1 (Alex,Young,m) |
|      5 (Jane,Fredricks,f) |
|         2 (Sarah,Riley,f) |
|         4 (Sarah,Smith,f) |

As was discussed above, indicies only contain the row number and
not any data from the table rows. But I include the data in
this table paranthetically for clarity.

This leads to a natural question. When we define our index, in what
order should we list the columns in the index. 

For our previous query (finding the male named Alex Williams), 
it is somewhat painful to index 
first on first name because there are a lot more people
with first name alex than last name Williams.

Since there is much more variety in last names,
a more efficent index would be (last,first,gender):

| person_id(last,first,gender) |
| ---------------------------- |
|         5 (Fredricks,Jane,f) |
|         0 (Garfunkle,Alex,m) |
|            2 (Riley,Sarah,f) |
|            4 (Smith,Sarah,f) |
|          3 (Williams,Alex,m) |
|             6 (Young,Alex,f) |
|             1 (Young,Alex,m) |

That way, finding all people will last name Williams immediately
shrinks the list to only one person!

The general rule here is that we want to order our columns from the
column with the most distinct values to the column with the fewest
distinct values.  This allows the databae to narrow down the list
of potential matches as fast as possible.

In SQL, we refer to the amount of distint values as the
[cardinality](http://en.wikipedia.org/wiki/Cardinality_(SQL_statements)) of
a table. Because last names has higher cardinality than first names,
and first names has higher cardinality than gender, the most efficient
index is (last,first,gender).

Before we move on, we note that it is perfectly harmless to add
additional rows to an index which are not used in a query.  Conversely,
a query will can only use an index if the first columns in the index
are constrained in the query. For example, you can not index on
(first,last) and then use that index to search for people of a given
last name.

## Primary Key Indices

In SQL, [primary
keys](http://dev.mysql.com/doc/refman/5.5/en/optimizing-primary-keys.html)
also perform as indicies on a table. This makes sense when you
realize that a primary key constrains duplicate rows from being
inserted into a row, which requires quickly looking for duplicate
rows. Although the columns in a primary key are dictated by teh
logical correctness of a table, it is good to think about the best
order to place the columns based on the queries which you plan to
run and the cardianity of the columns.

## Indexing on Inequalities

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

## Function-based Indexing

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

## Indexing the ORDER BY Clause

Proper index for the `ORDER BY` clause is straightforward.

## Indexing the GROUP BY Clause

## Indexing the JOIN Clause



## Database Implementation of Indicies in SQL

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

## Indexing in SQL Cheat Sheet

Here are the major takeaways about indexing in SQL:

1. An index is a sorted list of columns in a database.
2. Indexing in SQL improves query efficiency at the expense
   of table alteration efficiency.
3. When querying on multiple columns, it is most efficient to
   make an index on all of them, sorting the columns
   from highest to lowest cardinality.
4. A query can use all the columns in an index until a column
   in the index which is not part of the query.
5. When indexing on an inequality, it is most efficient to 
   put the inequlaity expression as the last column in the index.
6. When possible, avoid querying on functions of parameters 
   becuase they break indexing. When necessary,
   some databases allow function-based indexing

## Further resources

* http://use-the-index-luke.com/

<--

## Query Optimization

Now that we have seen several examples of the `SELECT` statement,
I will mention one final benefit of using relational databases
compared to programming languages.  As we saw above, a SQL query
is a logical description of what should be done to the data, not
a description of how or in what order to perform the operations 
needed to get the desired data.  

Because of this, SQL databases have [query
optimizers](http://en.wikipedia.org/wiki/Query_optimization) which
will logically inspect the query, think of different ways that the
query could be executed, and guess at the most efficient way to
perform the calculation. This is especially powerful because it is
often not clear to a user the fastest way to perform a calculation.
Furthermore, the SQL implementation has the benefit that best method
could change over time as the size of the database evolves.

## Further SQL Reading

* SQL databases go through great lenghts to deal with `NULL` values
  in a sensible way. [Here](http://dev.mysql.com/doc/refman/5.0/en/working-with-null.html)
  is some documentation in the way MySQL handles `NULL` values.
* [Views](http://dev.mysql.com/doc/refman/5.0/en/create-view.html) in
  SQL act as temporary tables, able to both simplify queries in MySQL
  as well as abstract the end user from the underlying implementation
  of a database.
* Beyond [MySQL](http://www.mysql.com/), there are some really great
  high-performance parallel databases like
  [Terradata](http://www.teradata.com/) and
  [Vertica](http://www.vertica.com/). They allow for large data
  sets then can traditionally be stored in MySQL.
* For data sets of a large enough size, [hadoop](http://hadoop.apache.org/),
  the [Hadoop Distributed File System (HDFS)](http://hadoop.apache.org/docs/r1.2.1/hdfs_design.html),
  and [MapReduce](https://hadoop.apache.org/docs/r1.2.1/mapred_tutorial.html) 
  are typically used to store and analyze the data. 
  [Apache Hive](http://hive.apache.org/) is an implementation of
  SQL on top of MapReduce capable of analyzing these exceptionally-large
  data sets. [Apache Pig](https://pig.apache.org/) is a similar SQL-like
  langauge which runs on top of MapReduce.
* If you are interested in learning more about the implementaion of query
  optimizers inf SQL, Bill Howe's coursera class 
  [Introduction to Data Science](https://www.coursera.org/course/datasci)
  has a great discussion of database implmeenations in his lectures on
  "[Relational Databases, Relational Algebra](https://class.coursera.org/datasci-001/lecture/preview)".
-->


{% include twitter_plug.html %}

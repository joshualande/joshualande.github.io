---
layout: post
title: Indexing in SQL for the Rest of Us
comments: true
---

In this post, we will discuss indexising in SQL which is a caching
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

# Sorting a Table in SQL Using an Index

An index in SQL is just a sorted list of all of our data in a table
where the index specifies the column order in which to sort rows.

It is illustrative to consider querying from
a simple table of `people`:

| person_id | First |      Last | gender |  birthdate |
| --------- | ----- | --------- | ------ | ---------- |
|         0 |  Alex | Garfunkle |      m |   19850527 |
|         1 |  Alex |     Young |      m |   19701201 |
|         2 | Sarah |     Riley |      f |   19890314 |
|         3 |  Alex |  Williams |      m |   19400105 |
|         4 | Sarah |     Smith |      f |   19910317 |
|         5 |  Jane | Fredricks |      f |   19920731 |
|         6 |  Alex |     Young |      f |   19920731 |

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

# Benefits of SQL indices

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

# Multi-column Indices in SQL

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

# Indexing on Inequalities

Indexing on inequalities requires a somewhat different logic than
equality indexing.  For example, supposed we wanted
to find all people with first name Alex born after 1970.
We would run the query:

```sql
SELECT *
  FROM people
 WHERE first = 'Alex'
   AND birthdate > 19700000
```

Given our rules about cardinality and the fact that birthdate has
the highest cardinality, we might expect the optimal
index for this query to be (first, birthdate).

...

Despite the rule from above that there are more ages than genders,
it is better to make age the final index.  The reason is 
for this index, after filering on men all of the ages are
already sorted in the index so the inequality can be easily applied.

The alternative index would require first going through
each of the allowed ages and for each of them applying another
search for the other query terms.

# Function-based Indexing

SQL has troulbe using indicies when there
are functions of parameters in the query.

Maybe a query which pulls out the year from the table.
Find people born in 1980.

Find query to do this.

# Primary Key Indices

Put a note about how a PK is also an index.
How this makes sense for quickly deciding if a 
PK constraint has been violated.

"It has an associated index, for fast query 
performance" -- http://dev.mysql.com/doc/refman/5.5/en/optimizing-primary-keys.html


## Deterministic Functions

A simple query we might be interested in is finding
all the people born after 1981.

First, we would have to index on birthdate:

```sql
CREATE INDEX birthdate_name
ON recipe_ingredients (first,last,age,gender)
```

To do that, we migth be tempted to run the query:


```sql
SELECT *
FROM people
WHERE date(birthdate) > date('1981-00-00')
```

Or atlernately, we could do 

```sql
SELECT *
FROM people
WHERE ROUND(birthdate/10000) > 1981
```

Neither of thse correctly work because MySQL
because functions of indexed columns
are not proprely indexed. That is because
the database is not smart enough to
determine any relation between the sorting
of the underlying data and the sorting
of the function of the data.

For our example above, the right query
would be:

```sql
SELECT *
FROM people
WHERE birthdate > 19810000
```

Two alternative solutions

* Function-based indexing
* Chagning the table to store date objects
* Creating a dimension table mapping date ints to date objects
  and join on that first.


## Non-deterministic Functions

Can't index on the age of somebody...

## Queries With Pattern Matching

...
Find people whose first name starts with the 'Al'
is easy. Finding people whose names end with 'Ax'
is hard.

# Indexing the GROUP BY Clause

# Indexing the JOIN Clause

# Indexing the ORDER BY Clause



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

# Further resources

* http://use-the-index-luke.com/

{% include twitter_plug.html %}

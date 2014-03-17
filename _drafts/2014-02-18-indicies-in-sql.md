---
layout: post
title: Indexing in SQL for the Rest of Us
comments: true
---

One of the biggest benefits of relational databases is that they
abstract the logical layout of our data from the underlying
implementation of how the data should be stored. Furthemore, they
abstract the queries you ask of the database from the details about
how the queriies shoudl be run.  Databases then use the the [Query
optimizer](http://en.wikipedia.org/wiki/Query_optimization)/[Query
execution plan](http://en.wikipedia.org/wiki/Query_plan) to find
an efficient way to write the query.

But this picture is a bit naieve because SQL cannot without help
find an efficient algorithm for every query you can could come up with.
In pactice, SQL quierie scan often be very slow, and one has
to introduce indicies to speed up queries and improve read
performance of the database.

# Sorting a Table in SQL Using an Index

An index in SQL is just a sorted list of all of our data in a table
where the index specifies in the column order in which to sort rows.

It is illustrative to consider querying from
a simple table of people:

| person_id | First |  Last | gender |  birthdate |
| --------- | ----- | ----- | ------ | ---------- |
|         0 |  Alex | Smith |      m | 1985-05-27 |
|         1 | Steve | Smith |      m | 1991-03-57 |
|         2 |  Alex | Young |      m |            |
|         3 |  Alex | Smith |      m |            |
|         4 | Sarah | Young |      f |            |

PUT A NOTE ABOUT HOW DATES ARE STORED HERE.

Suppose we wanted to compute, for example, all
the people with a given last name.
The query for this is straightforward:

```sql
SELECT *
  FROM people
 WHERE last = 'Young'
```

Given that SQL does not internally sort the table by any particular
column, the only way that SQL can issue this query is to loop over
every row in the table and check the amount value.  This is called
a [full table
scan](https://dev.mysql.com/doc/refman/5.5/en/glossary.html#glos_full_table_scan)
and is not very efficient. It has a run-time efficiency of O(n)
where n is the nubmer of rows, so it become increasly costly for
larger tables.

Instead, we could imagine creating an index on the amount column.
The index is just a list of the indicies for each row in the table
sorted by column specified. For our example, it would look like:

| index |
| ----- |
|   ... |

Note that the index does not contain any information about the value
of the column at that index. In addition, for columns
with the same value of `amount`, there is no guarantee about their
order in the index.

Given this index, we could use it to [binary
searching](http://en.wikipedia.org/wiki/Binary_search_algorithm)
our table for the values we want.  For each index during our search,
we could look up the row with that index in the `recipe_ingredients`
table and check compare our desired value to the value of `amount`
in that table.

For our particular example, the binary search would first check the
middle of our index table. The index points to 1st row, which has
`amount=2`, which is less than our desird amount.  Therefore, we
would check the middle of the top half of the table.  This row
points to the 5th row in the table, whcih has `amount=2`.  We have
to  look higher, so we would next try the 8th row in our index,
which points to the 3rd row in the table, which has the desired
amount=3! 

Because at each stage of the search we half the number of rows we
need to check, this algorithm is O(log(n)). For example, if there
were a million rows, the log of a million is only six.  So this is
a huge efficiency gain over the full table scan.


The syntax to create an index on this column is:

```sql
CREATE INDEX index_name
ON table_name (column_name)
```

For our exmaple, the query would be
```sql
CREATE INDEX amount_name
ON recipe_ingredients (amount)
```

# Primary Key Indices

Put a note about how a PK is also an index.
How this makes sense for quickly deciding if a 
PK constraint has been violated.

"It has an associated index, for fast query 
performance" -- http://dev.mysql.com/doc/refman/5.5/en/optimizing-primary-keys.html

# Benefits of SQL indices

Indices are great for several reasons.  First, they don't store any
of the underlying underlying data so they are fairly memory-efficient.
Second, they are automatically updated when the table is updated,
so they don't have to be explicilty handled by a developer.  And
finally, they are stored separately from the original table, which
allows for a logical separation of the data from particular information
about how the data will be queriied.

On the other hand, indicies have to be automatically updated by the
database when tables are modified, and this additional overhead
will degrate write performance.  So they should only be added as
necesary when performance is an issue.

# Multi-column Indices in SQL

Next, we will go over the indices
rquired for harder SQL queries.

First, we will learn what to do
when there are multiple 
conditions in the `WHERE` clause.
For example, we might be looking for a
row in `recipe_ingredients` with an known
recipe ID, ingredient ID, and amount.
If so, we could run a query like:

```sql
SELECT *
  FROM people
 WHERE first='Steve'
   AND last='Smith'
   AND gender='m'
```

In order to efficiently run this query, we can build an index which
sorts on (last, first, gender). What this means is we first sort
on the last name.  Whenver the last name is the same, we then sort
on first name. Whenever both are the same, we finally sort on gender.

```sql
CREATE INDEX amount_name
ON recipe_ingredients (first,last,age,gender)
```

And will create an index like:

| index |
| ----- |
|   ... |

Given that we have sorted by all four columns, it is easy binary
search or data for people of a given first name, last name, age, and gender.
Similarly,


This leads to a natural question. When we define our index, in what
order should we list the columns in the index. The rule here is
that we want to order our columns from the column with
the most distinct values to the column with the fewest
distinct values.  The reason for this is because we can
narrow down the possible search results fastest if we 
fist select only rows matching the fastest-varying column.

As a simple analogy to understand this, suppose you were to design
a phone book for quickly looking up a person.  Would you first sort
the book by first name and then, for people with the same first
name, sort by last name? Or would you first sort by last name and
then by first name? Because there are typically a lot more last
names than first names, it is usually going to be faster to first
find everybody with a specific last name and then, for all the
people with that given last name, find the person with the first
name you are looking for. 

For our example, we assume there are many more first
names than last names, many more last names than ages,
and many more ages than genders.

In SQL, we refer to the amount of distint values as the
[cardinality](http://en.wikipedia.org/wiki/Cardinality_(SQL_statements)) of
a table. The rule is therefore to put columns with higher cardinailty
first in an index.


# Indexing on Inequalities

Indexing on inequalities requires a somewhat different logic than
equality indexing.  For example, if we wanted to find all the men
who were older than 25, we would use the query

```sql
SELECT *
  FROM people
 WHERE gender = 'm'
   AND age > 25
```

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

## Deterministic Functions

A simple query we might be interested in is finding
all the people born after 1980.
To do that, we migth be tempted to run the query:

## Non-deterministic Functions

Can't index on the age of somebody...

## Queries With Pattern Matching

...
Find people whose first name starts with the 'Al'
is easy. Finding people whose names end with 'Ax'
is hard.

# Indexing for the GROUP BY clause

# Indexing for the JOIN clause

# Indexing for the ORDER BY clause



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

---
layout: post
title: Indexing in SQL for the Rest of Us
comments: true
---

As was discussedin XXX, the benefit of relational databases is that
they abstract the logical layout of our data from the underlying
implemntation of how the data should be stored. Furthemroe, they
abstract the logical question you want to ask of the data (the
query) from the the details of finding an efficent way to get this
data (the [Query
optimizer](http://en.wikipedia.org/wiki/Query_optimization)/[Query
execution plan](http://en.wikipedia.org/wiki/Query_plan)).

But this picture is a bit naieve because there SQL cannot naturally
find an efficient algorithm for every query you can throw at it,
and will often in practice issue queries signficiatly slower that
a more custom-made application.  In this post, I will introduce the
topic of indexing in SQL, which is a tool for caching intermediate
data

# The Structure of an Index in SQL

An index in SQL is just a sorted list of all of our data in a table
where the index specifies in the column order in which to sort rows.

It is illustrative to begin with a simple example of our.
`recipe_ingredients` table from the post XXX.

| recipe_id | ingredient_id | amount |
| --------- | ------------- | ------ |
|         0 |             0 |      1 |
|         0 |             1 |      2 |
|         0 |             2 |      2 |
|         0 |             3 |      3 |
|         0 |             4 |      1 |
|         1 |             2 |      2 |
|         1 |             5 |      1 |
|         2 |             4 |      1 |
|         2 |             6 |      2 |

Suppose we wanted to compute, for example, all
of the recipies which required three of a particular ingredient (`amount=3`). 
The query for this is straightforward:

```sql
SELECT *
  FROM recipe_ingredients
 WHERE amount = 3
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
|     0 |
|     4 |
|     6 |
|     7 |
|     1 |
|     2 |
|     5 |
|     8 |
|     3 |

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

PUT a note about how a PK is also an index.
HOW this makes sense for quickly deciding if a 
PK constraint has been violated.

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


Despite giving all the right intuition

# Multi-column Indices in SQL

Now that we have seen the benefits of indexing,
we will go over examples of finding
indices for more complciated SQL queiries.

First is the situation where we have
multiple conditions in the `WHERE` clause.
For example, we might be looking for row
in `recipe_ingredients` with a known
recipe ID, ingredient ID, and amount.
If so, we could run a query like:

```sql
SELECT *
  FROM recipe_ingredients
 WHERE ingredient_id = 3
   AND recipe_id = 0
   AND amount = 3
```

In oder to efficiently run this query,
we can build an index which sorts on (ingredient_id, recipie_id, amount).
What this means is we first sort on the ingredient ID.
Whenver the ingredient ID is the same, we then sort by the recipie ID.
And finally, whenever the ingredietn and recpies IDs are the same,
when then sort on the amount column.
The syntax to do this is straightforward:

```sql
CREATE INDEX amount_name
ON recipe_ingredients (ingredient_id, recipe_id, amount)
```

And will create a tablet like

| index |
| ----- |
|   ... |



This leads to a natural question, which is in what order to set the index.

# Advanced Indexing


# Database Implementation of Indicies in SQL

The simple picture of an index being a sorted array is a bit
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


# Advanced topics:

* Function-based indexing
* The `ORDER BY` clause

# Indexing in SQL Cheat Sheet


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

Note that the index does not contain any information about hte value
of the column at that index.

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
need to check, this algorithm is O(log(n)) which is a huge efficiency
gain over the full table scan.


# Benefits of SQL indices

Indices are great for several reasons.  First, they don't store any
of the underlying underlying data so they are fairly memory-efficient.
Second, they are automatically updated when the table is updated,
so they don't have to be explicilty handled by a developer.  And
finally, they are stored separately from the original table, which
allows for a logical separation of the data from the implmentation
of fast optimization.

# Database Implementation of SQL Indices

# Advanced Indexing

Advanced topics:

* Function-based indexing

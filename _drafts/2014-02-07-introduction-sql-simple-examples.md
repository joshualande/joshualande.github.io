---
layout: page
title: An Introduction to SQL With Simple Examples
comments: true
---

In this post, my intention is to broadly go over the purpose of
relational databases like SQL and describe how they are different
from other data structures or formats. I will discuss the key topics
of database normalization, the basic databases operators like joining
and aggregating tables, and a few advanced topics like using indices
to speed up queries. To get you up to speed as quickly as possible, I will
teach all of this using a simple examples.

Of course, databases are an enormous topic for which there
is [an entire profession](http://en.wikipedia.org/wiki/Database_administrator),
so this will be whirlwind tour of the key concepts, 
enough to convey the value and get you started.

<!--
In a followup post, I discuss some more advanced SQL queries which require cleaver
use of the basic building blocks of SQL.

And in another post, I introduce the Apache Pig programming language, which is
very similar to SQL but allows for analysis of big data.

Before you start reading this post, I would recommend setting
up SQL on your computer. I wrote instructions for doing so on a Mac here XXX.
Once you have done that you can create the tables used in this example using
the section linked from below XXX
-->

# Background

During my Physics PhD, I never understood the purpose of 
[SQL](http://en.wikipedia.org/wiki/SQL) and
similar [relational database management systems 
(RDBMs)](http://en.wikipedia.org/wiki/Relational_database_management_system).
As I prepared to transition to a career as a data scientist, I was
confused why so much data science was done inside of these databases.

After all, physicists (and scientists more broadly) love to reinvent the wheel. 
Working in astrophysics, I had used all sorts of file formats like
[ROOT Trees](http://en.wikipedia.org/wiki/ROOT), 
[Fits Files](http://en.wikipedia.org/wiki/FITS),
[HDF5 Files](http://en.wikipedia.org/wiki/Hierarchical_Data_Format),
and [pandas DataFrames](http://en.wikipedia.org/wiki/Pandas_(software). 
I wans't sure why I needed to learn SQL.

And besides, SQL seemed so limited. Everything had to be these flat
tables, and the syntax to deal with theme seemes archane. 

Now that I have been a data scientist for half a year, I have grown
to really love databases.  They do what they are designed to do
well they do better than anything else.  And their use case in
business and data science applications is almost limitless.

Now that I am on the other side of the table 
helping other academics transition to become become data scientists, I see that
most of them have very similar confusion about the purpose and benefit of these
databases. 


# A Simple Example SQL Database

It is instructive to begin with a simple example.
The most important takeaway about SQL databases
is that a database is a collection of two-dimensional tables.
Each table has a fixed number of rows and columns. This may
seem like a very limited design, but as you will see later this simplification,
along with the necessary opeations, becomes incredibly powerful.

As a simple example of a relational database, supposed 
we want to describe a schema for a database of recipes.
The first thing we might want to do is to describe a list of all
the recipes. All recipes have a name which we call recipe_name and
they all have a description which we will call recipe_description.
In addition, we will associated a unique id for each recipe which
we will call recipe_id.

| recipe_id |    recipe_name |        recipe_description |
| --------- | -------------- | ------------------------- |
|         0 |          Tacos |        Mom's famous Tacos |
|         1 |    Tomato Soup |      Homemade Tomato soup |
|         2 | Grilled Cheese | Delicious Cheese Sandwich |

But each recipe is built out of ingredients, so 
we might want an ingredients table describing all of the ingredients 
that we can build food out of. For this simplified example,
we assume that all ingredients have a price which is stored
in the ingredient_price column.

| ingredient_id | ingredient_name | ingredient_price |
| ------------- | --------------- | ---------------- |
|             0 |            Beef |                5 |
|             1 |         Lettuce |                1 |
|             2 |        Tomatoes |                2 |
|             3 |      Taco Shell |                2 |
|             4 |          Cheese |                3 |
|             5 |            Milk |                1 |
|             6 |           Bread |                2 |

  
Finally, we need some way of listing all of the ingredients in each recipe.
Although we might naturally want to put this information into the recipe table,
we will see that it is advantageous to use a third table to store this information.
For every recipe, we will need to know all of the ingredients in that table.
This information can nautrally be stored in a third table which we will call `recipe_ingredients`:

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

Note that we put in this table in addition how much of each ingredient is needed in each recipe.

Of course, we could imagine that a real database, like the one
curated by [yummly.com](http://yummly.com), would have lots more information in it.
You might want more accurate list of steps involved in preparing the recipe.
But this simplified example should get the point across.

# SQL Database Design

Moving from the general to the concreate, here is
the SQL code to create the database we described above:

First, we can create the `recipes` table:

```sql
CREATE TABLE recipes (
  recipe_id int(11) NOT NULL, 
  recipe_name varchar(30) NOT NULL,
  recipe_description varchar(30) NOT NULL
);
INSERT INTO recipes 
    (recipe_id, recipe_name, recipe_description) 
VALUES 
    (0,"Tacos","Mom's famous Tacos"),
    (1,"Tomato Soup","Homemade Tomato soup"),
    (2,"Grilled Cheese","Delicious Cheese Sandwich");
```

Next we can reate the `ingredients` table:

```sql
CREATE TABLE ingredients (
  ingredient_id int(11) NOT NULL, 
  ingredient_name varchar(30) NOT NULL,
  ingredient_price int(11) NOT NULL
);
INSERT INTO ingredients
    (ingredient_id, ingredient_name, ingredient_price)
VALUES 
    (0, "Beef", 5),
    (1, "Lettuce", 1),
    (2, "Tomatoes", 2),
    (3, "Taco Shell", 2),
    (4, "Cheese", 3),
    (5, "Milk", 1),
    (6, "Bread", 2);
```

And finally, we can create the recipe-ingredient-mapping table:

```sql
CREATE TABLE recipe_ingredients (
  recipe_id int(11) NOT NULL, 
  ingredient_id int(11) NOT NULL, 
  amount int(11) NOT NULL
);
INSERT INTO recipe_ingredients 
    (recipe_id, ingredient_id, amount)
VALUES
    (0,0,1),
    (0,1,2),
    (0,2,2),
    (0,3,3),
    (0,4,1),
    (1,2,2),
    (1,5,1),
    (2,4,1),
    (2,6,2);
```

XXXXXX NOTE ABOUT SIMILAR BLOG POST ABOUT HOW TO SETUP MYSQL DATABASE
AND PLAY ALONG WITH THE EXAMPLES IN THIS POST XXXXXX

# Similar python Implementation

Comming from a python background, my natural tendency is to feel that this
is totally overengineering the problem. Instead, I would imagine using
a nested dictionary to define this data:

```python
recipes = {
  "Tacos": ["Beef", "Lettuce", "Tomatoes", 
            "Taco Shell", "Cheese" ],
  "Tomato Soup": [ "Tomatoes", "Milk" ],
  "Grilled Cheese": [ "Cheese", "Bread" ]
}
```
That way, if I wanted to find, for example, all of the ingredients associated
with Tomato Soup, I could use the code:

```python
ingredients = recipes["Tomato Soup"]["ingredients"]
```

Although this seems works well, there are several issues associated with this approach.
The first is that organizing our data in this particular way makes
other queries about the data particularly hard.
For example, it is much more cumbersome to get a list of all recipes 
which contain tomatoes. You could imagine a similar data structure optimized for that 
kind of query:

```python
recipes = {
    "Beef": [ "Tacos" ],
    "Lettuce": [ "Tacos" ],
    "Tomatoes": [ "Tacos", "Tomato Soup"],
    "Taco Shell": [ "Tacos"],
    "Cheese": [ "Tacos", "Grilled Cheese"]
    "Milk": [ "Tomato Soup" ]
    "Bread": [ "Grilled Cheese" ]
}
```

I am leaving out some of the other metadata to make my point clear.
But this data structure makes it cumbersome to find what ingredients are
required to make a paritcular recipe.
This may seem like a contrived example, but in real life it is very often
the case that that you later on have to query your data in ways you did not
originally intent.

As you will see, SQL solves this problem by dividing up our data into multiple tables.

The second issue with the approach listed above is that there are a lot of duplciate data
which shows up in mulitple places. 




# THE SELECT, FROM, and WHERE Statements in SQL

Given our recipe schema above, there are many kind of calculations we could imagine doing
on this databse.

Imagine that we wanted to find all the recipes in "Tomato Soup".
As a first step, we could figure out the recipe_id for "Tomato Soup"
with a simple SQL query using the `SELECT` statement:

```sql
SELECT recipe_id 
FROM recipes
WHERE recipe_name="Tomato Soup"
```

This querie says that we take the the `recipes` table,
and we filter only on a particular kind of row (where the
`recipe_name` is "Tomato Soup") and the filter for 
a particular column (the `recipe_id` column).

This query returns the table

| recipe_id |
| --------- |
|         1 |

Now, we can get a list of the `ingredient_id` using a similar query
on the `recipe_ingredients` table:

```sql
SELECT ingredient_id
FROM recipe_ingredients
WHERE recipe_id = 1
```

Which returns the table

| ingredient_id |
| ------------- |
| 2             |
| 5             |

# The ORDER BY operator in SQL

The next command we will cover is `ORDER BY`.
`ORDER BY` can be used to sort the rows based on
a particular column. For example, if we wanted
to sort the ingredients by how expensive they are
in order of descending price, we could run the query:

```
SELECT *
FROM ingredients
ORDER BY ingredient_price DESC
```

Which returns 

| ingredient_id | ingredient_name | ingredient_price |
| ------------- | --------------- | ---------------- |
| 0             |            Beef |                5 |
| 4             |          Cheese |                3 |
| 6             |           Bread |                2 |
| 3             |      Taco Shell |                2 |
| 2             |        Tomatoes |                2 |
| 1             |         Lettuce |                1 |
| 5             |            Milk |                1 |

If we wanted to sort columns of the same price alphabetically by name, we could
use a similar query:

```
SELECT *
FROM ingredients
ORDER BY ingredient_price DESC,ingredient_name
```

This creates the table:

| ingredient_id | ingredient_name | ingredient_price |
| ------------- | --------------- | ---------------- |
| 0             |            Beef |                5 |
| 4             |          Cheese |                3 |
| 6             |           Bread |                2 |
| 3             |      Taco Shell |                2 |
| 2             |        Tomatoes |                2 |
| 1             |         Lettuce |                1 |
| 5             |            Milk |                1 |

# The LIMIT operator in SQL

We can use the `LIMIT` operator to limit the number of results returend
by the query. For example, to get only the most expensive ingredient, we could use
the query:

```sql
SELECT *
FROM ingredients
ORDER BY ingredient_price DESC
LIMIT 1
```

This returns the table:

| ingredient_id | ingredient_name | ingredient_price |
| ------------- | --------------- | ---------------- |
| 0             |            Beef |                5 |


# The JOIN operator in SQL

Having to perform multiple quieries on
multiple tables is quite cumbersome,
and would require unnecessary additional
logic outside of the database.

Instead, if we wanted to directly get the
`ingredient_id` knowing the `recipe_name`,
we could `JOIN` together

the way to do this is to join the 
two tables together when the `recipe_id`
is equal:

```sql
SELECT *
FROM recipes
JOIN recipe_ingredients
ON recipes.recipe_id = recipe_ingredients.recipe_id
```

The `JOIN` command will combine all the rows
from the one table with all the rows from the other
table when the condition of the `ON` clause is true,
there that the recipe_id is equal.

For this example, we get the table:

| recipe_id | recipe_name    | recipe_description        | recipe_id | ingredient_id | amount |
| --------- | -------------- | ------------------------- | --------- | ------------- | ------ |
|         0 | Tacos          | Mom's famous Tacos        | 0         |             0 | 1      |
|         0 | Tacos          | Mom's famous Tacos        | 0         |             1 | 2      |
|         0 | Tacos          | Mom's famous Tacos        | 0         |             2 | 2      |
|         0 | Tacos          | Mom's famous Tacos        | 0         |             3 | 3      |
|         0 | Tacos          | Mom's famous Tacos        | 0         |             4 | 1      |
|         1 | Tomato Soup    | Homemade Tomato soup      | 1         |             2 | 2      |
|         1 | Tomato Soup    | Homemade Tomato soup      | 1         |             5 | 1      |
|         2 | Grilled Cheese | Delicious Cheese Sandwich | 2         |             4 | 1      |
|         2 | Grilled Cheese | Delicious Cheese Sandwich | 2         |             6 | 2      |

Now, we want to filter this table out by only getting the ingredient_id 
when the recipe_name is "Tomato Soup":

```sql
SELECT recipe_ingredients.ingredient_id
FROM recipes
JOIN recipe_ingredients
ON recipes.recipe_id = recipe_ingredients.recipe_id
WHERE recipes.recipe_name = 'Tomato Soup'
```

This returns exactly the table we were looking for

| ingredient_id |
| ------------- |
| 2             |
| 5             |

# JOINing Three Tables in SQL

What's really cool about SQL is all of the commands can e stacked.
So if we wanted to directly return the name of the ingredients
in Tomatoe Soup, all we have to do is JOIN the
ingredients table to the other two tables joined above.
The query ends up being:

```sql
SELECT ingredients.ingredient_name 
FROM recipes AS a
JOIN recipe_ingredients AS b
ON a.recipe_id = b.recipe_id
JOIN ingredients 
ON b.ingredient_id = ingredients.ingredient_id
WHERE
    a.recipe_name = "Tomato Soup"
```

Which returns the table

| ingredient_name |
| --------------- |
| Tomatoes        |
| Milk            |

Sometimes people find it cumbersome that
the table name shows up so many times
in a SQL query, so as a short hand
you can give your tables nicknames.
A more conventional query might look something like:

```sql
SELECT c.ingredient_name 
FROM recipes AS a
JOIN recipe_ingredients AS b
ON a.recipe_id = b.recipe_id
JOIN ingredients AS c
ON b.ingredient_id = c.ingredient_id
WHERE
    a.recipe_name = "Tomato Soup"
```

Now at this point, you are probably thinking
that having to join three tables together is way
too much work compared to the simple data structure
we wrote in python above.

But what's great about SQL is that once we have
laid out our data in this abstract table representation,
it becomes no harder to do all sorts of other
queries against our database. For example,
to find all the recipes that include tomatoes
is just as easy as the above query:

```sql
SELECT a.recipe_name
FROM recipes AS a
JOIN recipe_ingredients AS b
ON a.recipe_id = b.recipe_id
JOIN ingredients AS c
ON b.ingredient_id = c.ingredient_id
WHERE
    c.ingredient_name = "tomatoes"
```

This returns the expected tables

| recipe_name |
| ----------- |
| Tacos       |
| Tomato Soup |

# The GROUP BY Operator In SQL

Now, we have seen the `SELECT`, `FROM`, `JOIN`, `ON`, and
`WHERE` commands in SQL.

There are only a couple more commands.
Most importantly, the `GROUP BY` command is used to aggregate
down a larger table to a smaller table, gaining insight
from the aggregation.

Supposed for example that we wanted to make a list of the number
of ingredients for each recipe. 
To do that, 
we can group all of the recipe_id rows in the recipe_ingredients
and then count all of the rows in each group. To do this in SQL:

```sql
SELECT recipe_id, 
  COUNT(ingredient_id) as num_ingredients
FROM recipe_ingredients
GROUP BY recipe_id
ORDER BY num_ingredients DESC
```

This code says to group all of the rows by common `recipe_id`s and to 
count all of the ingredient_id for each recipe.

The resulting table is

| recipe_id | num_ingredients |
| --------- | --------------- |
|         0 |               5 |
|         1 |               2 |
|         2 |               2 |

Similarly, we can combine the `GROUP BY` and `JOIN` operators in a single query.
To compute in addition the price of each recipe, we would need to figure
out the price of each ingredient by joining with the ingredients table.
This query would look like:

```sql
SELECT recipe_id, 
    COUNT(a.ingredient_id) as num_ingredients, 
    SUM(a.amount*b.ingredient_price) as total_price
FROM recipe_ingredients as a
JOIN ingredients as b
ON a.ingredient_id = b.ingredient_id
GROUP BY a.recipe_id
```

And this returns

| recipe_id | num_ingredients | total_price |
| --------- | --------------- | ----------- |
|         0 |               5 |          20 |
|         1 |               2 |           5 |
|         2 |               2 |           7 |

Similarly, if we want to make plot the nicer recipe name  
we could also JOIN with the recipes tables:

```sql
SELECT c.recipe_name, 
    COUNT(a.ingredient_id) as num_ingredients, 
    SUM(a.amount*b.ingredient_price) as total_price
FROM recipe_ingredients as a
JOIN ingredients as b
ON a.ingredient_id = b.ingredient_id
JOIN recipes AS c
ON a.recipe_id = c.recipe_id
GROUP BY a.recipe_id
```

This returns a nicer formated table:

|    recipe_name | num_ingredients | total_price |
| -------------- | --------------- | ----------- |
|          Tacos |               5 |          20 |
|    Tomato Soup |               2 |           5 |
| Grilled Cheese |               2 |           7 |

# The HAVING Operator in SQL

The `HAVING` clause in SQL is almost exactly like the `WHERE`
clause, but filters the table after the aggregation has been
performed.

Suppose we wanted to find only recipes with 2 ingredients in it.
We could use the `HAVING` clause:

```sql
SELECT recipe_id, 
  COUNT(ingredient_id) AS num_ingredients
FROM recipe_ingredients
GROUP BY recipe_id
HAVING num_ingredients = 2
```

This creates the table

| recipe_id | num_ingredients |
| --------- | --------------- |
|         1 |               2 |
|         2 |               2 |

# Subqueries in SQL

These tables we created above would be nicer if they had the recipe
name in them. One way to do this in SQL would be to
create a new table in SQL to store the results.

```sql
CREATE TABLE ingredient_counts AS
SELECT recipe_id, 
  COUNT(ingredient_id) AS num_ingredients
FROM recipe_ingredients
GROUP BY recipe_id
```

Then we can join this table with the recipes table
to create a nicer name:

```sql
SELECT  b.recipe_name, a.num_ingredients
FROM ingredient_counts AS a
JOIN recipes AS b
ON a.recipe_id = b.recipe_id
```

| recipe_name    | num_ingredients |
| -------------- | --------------- |
| Tacos          |               5 |
| Tomato Soup    |               2 |
| Grilled Cheese |               2 |

But it is cubmersome to have to create this temporary table.  In
addition, anytime we modified the recipe_ingredients table, we would
have to recreate the ingredient_counts table.  Another issue is
that this command requires that you have write access on the database
and permission to create new tables. In enterprise situations, you
generally are (for good reason) now allowed to modify the database
unless you really need to.

SQL does support [temporary tables](http://dev.mysql.com/doc/refman/5.1/en/create-table.html)
which could also be used in this situation.


This leads us naturally into the next topic in SQL
which is subqueries.  The idea behind subqueries is both simple and
incredibly powerful. Because every query in SQL
returns a table, you can use SQL queries as tables inside of other
queries.

So for our example above, conceptually all we have to do is
join the ingredient count table with the recipe table on
recpie_id to get the  names of all of the recipes.

The synax in SQL would 

Now, you might think

```sql
SELECT  b.recipe_name, a.num_ingredients
FROM
  (
    SELECT recipe_id, 
      COUNT(ingredient_id) AS num_ingredients
    FROM recipe_ingredients
    GROUP BY recipe_id
  ) AS a
JOIN
  recipes AS b
ON
  a.recipe_id = b.recipe_id
```

This returns a much nicer table then before:

| recipe_name    | num_ingredients |
| -------------- | --------------- |
| Tacos          |               5 |
| Tomato Soup    |               2 |
| Grilled Cheese |               2 |

What's really cool about SQL is that it is incredibly
flexible about the struture of queries. Similar to how we
can JOIN as many tables together as needed, we can also
nest multiple subquieries together.

# Example harder querie: get a list of the name
and price of all recipies that include tomatoes

MAKE SOME FANCY JOINING...

# TEMP

To compute the cost of each recipe


# Views in SQL


Another way is to create views. These are discussed below. XXX

You m

# INNER, LEFT, and RIGHT join.

# Harder SQL queries

Surprising as it might seem, the few
SQL commands described above go an enormous way
in data analysis.

Just to give you an idea of how powerful SQL queries can become,
we could imagine a command:

Other things you might want:

* A list of users in the system.
* A mapping of what users have eaten what recipes...
* FIT DATA INSIDE MEMORY.
 
# Benefits of SQL Over Other Data Structures

* Future proofign yourself against changing business requirements.
* Letting SQL decide the fastest way to do run the query

The fundamental ideas begind a database are that
[Database normalization](http://en.wikipedia.org/wiki/Database_normalization),
which is the process of minimizing the amount of redundant data
in a database by creating lots of related tables. The next idea is




# Advanced SQL: 

How do we actually build these tables:

# SQL Indicies For Faster Queries


# Wrapup 

The key concepts
we went over in this post are:


Thats about it. If you can get get you mind around the idea of table normalization, as well as
as the benefits of joining tables together to produce more complicated queries, everything
else should be easy.

# Closing Words

* Terradata/Vertica for distributed databases.
* HIVE/Pig to run parallel Map/Reduce queries.

# Relevant links

This was of course just a whirlwind tour of what's so great
about SQL databases. There is tons of topics left to explore:

* [MySQL](http://www.mysql.com/) is a popular SQL implementation.
* [Sequel Pro](http://www.sequelpro.com) is a great MySQL client.
* Bill Howe's coursera class (https://www.coursera.org/course/datasci). The class Relational Databases, Relational Algebra (https://class.coursera.org/datasci-001/lecture).

If you liked this post, feel free to share this
with your followers!


Or you can follow me on Twitter [@joshualande](http://twitter.com/joshualande).


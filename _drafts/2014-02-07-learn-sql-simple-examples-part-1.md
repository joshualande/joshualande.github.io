---
layout: post
title: Learn SQL With Simple Examples (Part 1)
comments: true
---

In this post, I will go over many of the benefits of relational
databases like [SQL](http://en.wikipedia.org/wiki/SQL) and describe
the ways in which they are better than other programming languages
and data formats. 

Then, by way of a simple to understand example, I will describe the
basics of how to store data in a database and the benefits of
database normalization.  Using this example, I will work
through successively harder SQL queries to introduce the basic
operators in SQL like filtering, joining and aggregating.

This post is intended as more of a why than a how.  Despite 
lacking throughness, by the end of the post you 
should dave a good overview of the key benefits and major topics
of SQL. In this post, I will show commands for the 
[MySQL](http://www.mysql.com/) database, but the commands and 
concepts are easily transferable to

# A Simple Example SQL Database

We begin with a simple example.  Suppose for that we want to store
in a database information about recipes in a cookbook. 

The fundamental building block of SQL databases are two-dimensional
tables. This may seem like a limited design, but as you will
see, this limitation is incredibly powerful.

In order to store our recipes into the database, we can first create
a table listing the names of all of our recipes.  In addition, we
will associated a unique id for each recipe.  The purpose of the
recipe id, as you will see soon, is to connect rows in this table
to rows in other tables.

For our particular example, we will call this table `recipes` and
it will have the following data:

| recipe_id |    recipe_name |
| --------- | -------------- |
|         0 |          Tacos |
|         1 |    Tomato Soup |
|         2 | Grilled Cheese |

Each recipe is built out of ingredients, so 
we can now make a table containing
all of the ingredinets that are in our recipes.

For our simplified example, we assume every recipe has
a price and we define the `ingredients` table as
follows:

| ingredient_id | ingredient_name | ingredient_price |
| ------------- | --------------- | ---------------- |
|             0 |            Beef |                5 |
|             1 |         Lettuce |                1 |
|             2 |        Tomatoes |                2 |
|             3 |      Taco Shell |                2 |
|             4 |          Cheese |                3 |
|             5 |            Milk |                1 |
|             6 |           Bread |                2 |

  
Finally, we need some way of listing what ingredients are in each
recipe.
Although we might naturally want to put this information into the recipe table,
it is advantageous to use a third table to store this information.

This table needs to contain a mapping from recipes to ingredients.
Although it might seem cumbersome, a straightfowrd way to do this
is to have a table with all the (recipie, ingredient) pairs.

To make the following examples more entertaining, we will also
list an amount for each ingredinet to allow different ingredients to
have more or less of anything. Our
`recipe_ingredients` table now looks like:

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

Of course, this example is very simplified.  We could imagine that
a real database like the one curated by [yummly.com](http://yummly.com)
would have lots more information in it.  But this example should
be sufficient to allow for very interesting questions to be asked
of the database.

At this point, I would expect you to be confused about why we laid
out the table in in this manner. We will go over the rationale for
this design in the next two sections, but before moving on I hope
you understand the logical correctness of the design.

Before we move on, I will mention two bits of terminology used when
discussing databases.  A
[schema](http://en.wikipedia.org/wiki/Database_schema) is the
structure of the tables in the database.  A
[query](http://en.wikipedia.org/wiki/SQL#Queries) in SQL is a command
which retrivies data from a database.



# Normalize Your Database

One natural question would be why
we needed three tables to store what seemingly isn't very much
information.  Another question is why `recipe_ingredients` stores
these strange IDs instead of just storing the names of the recipes
and ingredients.  And finally, one might wonder why each recipe-ingredient
pair takes up its own row in the `recipe_ingredients` table.

This naturally leads us to the concept of [database
normalization](http://en.wikipedia.org/wiki/Database_normalization).
The goal of database normalization is to design
the database which avoid having any duplicate data in the
database.

If we stored recipie and ingredient names
directly in the `recipe_ingredients` table,
we would end up with rows like

| recipe_name | ingredient_name | amount |
| ----------- | --------------- | ------ |
|       Tacos |            Beef |      1 |
|       Tacos |         Lettuce |      2 |

At some point in the future, somebody might
decide that they wanted the Tacos to have Chicken
instead of beef. In the process of updating
the table, they migth accidently
change the name of the recipe.

| recipe_name | ingredient_name | amount |
| ----------- | --------------- | ------ |
|       tacos |         Chicken |      1 |
|       Tacos |         Lettuce |      2 |

But if the recipe\_name wasn't changed consistently, the table
itself could become inconsistent.  Finding and fixing these sorts
of bugs can be costly and time consuming.  And as the size of a
database grows (more tables and more rows), these errors become
even harder to spot and avoid.  So it is a huge win if we can design
our database to prevent this sort of error.

In addition, database normalization has the added benefit that
updating the name of an ingredient or recipie requires only chaning
one thing in one place. This is useful because the end user
of your database will most likely always want to change things.

Finally, another another benefit of this modular table design 
is that it scales nicely to adding additonal information.
Suppose we wanted to store information about the steps needed to
build the recipe. We could just create a new table called
`recipe_instructions` which linked back to the recipes by recipe\_id:

| step_id | recipe_id | step_number |                 step_description |
| ------- | --------- | ----------- | -------------------------------- |
|       0 |         0 |           0 |   Put the Taco Shells in an oven |
|       1 |         0 |           1 | Cook the beef in a pan on medium |

Being able to add new data easily without altering the orignal
tables is very useful because it is less error prone and doesn't
require modifying any of the existing code and queries which read
and write from the database.

Finally, if the extra data only existed for a limited number of
rows, this would be another win in space since otherwise
the table would have to have lots of `NULL` values in it for
recipes without directions.

# Why Do Tables Have to be Flat?

This limitation that SQL tables have to be flat may at first seem
strange.  Although it should seem clear at this point that there
is some way of flattening out your data using keys to fit into flat
tables, it is probably unclear why this is enforced.

If you are used to programming in scripting langauges
like ruby, python, or perl, or if you are used to dealing with JSON
data, you may be confused why tables in SQL have to be flat.  A
common paradme is to next data in a format which is more human
readable.  So in python, I could imagine storing our data in a
dictionary of sets:

```python
recipes = {
  "Tacos": ("Beef", "Lettuce", "Tomatoes", 
            "Taco Shell", "Cheese"),
  "Tomato Soup": ("Tomatoes", "Milk"),
  "Grilled Cheese": ("Cheese", "Bread")
}
```

Using this dictionary, it would be easy to find all the
ingredients for a particular recipe:

```python
ingredients = recipes["Tomato Soup"]["ingredients"]
```

But the problem with laying out the data in this way is it
makes this paritcular query easy at the expense of making other kinds
of queries hard. 

It would be cumbersome for example to 
to list all recipes that have a particular ingredient.
We could imagine building a data structure optimized to
solve this question:

```python
recipes = {
    "Beef": ("Tacos"),
    "Lettuce": ("Tacos"),
    "Tomatoes": ("Tacos", "Tomato Soup"),
    "Taco Shell": ("Tacos"),
    "Cheese": ("Tacos", "Grilled Cheese"),
    "Milk": ("Tomato Soup" )
    "Bread": ("Grilled Cheese")
}
```

But this structure this makes the first query more difficult!

Although it seems strange at first, having to use flat tables is
actually a good thing.  By forcing you to use a flat table, SQL
ensures that our table isn't bias towards (and also against) certain
kinds of query.

The designers of SQL can have optimized the database so that our
table can be efficient for all sorts of SQL queries.  There is a
very good chance that what you think will be interesting to do now
with your data today will be quite different from a year from now.
So having his power is a huge win.

# Creating Tables in SQL

Before getting further,
I would highly recommend [setting up
MySQL](https://dev.mysql.com/tech-resources/articles/mysql_intro.html) on
your local machine and creating a test database to
run these examples in. Everything in this blog post
should be self contained, so you can will be able to
create all of the tables and run all of the queries. 
If you are using an Apple computer,
I recommend using the program [Sequel Pro](http://www.sequelpro.com/)
which provides a great graphical interface for interacting
with MySQL.

First, we can create the `recipes` table
using the `CREATE TABLE` command:

```sql
CREATE TABLE recipes (
  recipe_id int(11) NOT NULL,
  recipe_name varchar(30) NOT NULL,
  PRIMARY KEY (recipe_id)
  UNIQUE (recipe_name)
);

INSERT INTO recipes 
    (recipe_id, recipe_name) 
VALUES 
    (0,"Tacos"),
    (1,"Tomato Soup"),
    (2,"Grilled Cheese");
```

The purpose of the `PRIMARY KEY` is to 
avoid any possiblity of having duplicate
rows in the table. The `PRIMARY KEY`
forces every row to have a different value.
of the `PRIMARY KEY`. In addition, we
can use the `UNIQUE (recipe_name)` 
command to ensure that no two
rows have the same recipe name since
this could introduce bugs.

Similaryly, we can reate the `ingredients` table:

```sql
CREATE TABLE ingredients (
  ingredient_id int(11) NOT NULL, 
  ingredient_name varchar(30) NOT NULL,
  ingredient_price int(11) NOT NULL,
  PRIMARY KEY (ingredient_id),  
  UNIQUE (ingredient_name)
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

Note that we do not ensure uniqueness of the `ingredient_price`
since multiple recipes could have the same price.

Finally, we can create the recipe-ingredient-mapping table:

```sql
CREATE TABLE recipe_ingredients (
  recipe_id int(11) NOT NULL, 
  ingredient_id int(11) NOT NULL, 
  amount int(11) NOT NULL
  PRIMARY KEY (recipe_id,ingredient_id),  
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

Here, our `PRIMARY KEY` is for `recipe_id`, `ingredient_id`
pairs because two rows can have the same `recipe_id` as long
as they have a different `ingredient_id` (and vice versa).

One last thing to mention is there is an `AUTO_INCREMENT` command
which can be used to let SQL automatically
pick the `recipe_id` and `ingredient_id`
values to ensure that each ID is one larger than the last one
added to database.  [Here](http://dev.mysql.com/doc/refman/5.0/en/example-auto-increment.html)
is the documentation on this command.

# THE SELECT, FROM, and WHERE Statements in SQL

Given our recipe database, there are all kinds
of interesting questionts we could
ask using our database.

Imagine that we wanted to find all the ingredients in "Tomato Soup".
As a first step, we could figure out the recipe_id for "Tomato Soup".
Whenever we want to query information from the database,
we use the `SELECT` statement:

```sql
SELECT recipe_id 
FROM recipes
WHERE recipe_name="Tomato Soup"
```

This query says take the `recipes` table,
filter for rows where `recipe_name` is "Tomato Soup", 
and select only the clumn `recipe_id`.
This query returns the table

| recipe_id |
| --------- |
|         1 |

We can get the `ingredient_id` values using a similar query
on the `recipe_ingredients` table:

```sql
SELECT ingredient_id
FROM recipe_ingredients
WHERE recipe_id = 1
```

This returns

| ingredient_id |
| ------------- |
| 2             |
| 5             |

Getting the ingredient names is similar:

```sql
SELECT ingredient_name
FROM ingredients
WHERE ingredient_id=2 OR ingredient_id=5
```

This returns:

| ingredient_name |
| --------------- |
| Tomatoes        |
| Milk            |


# The JOIN operator in SQL

Because our data is spread across three
tables, having to perform multiple successive
queries is a hassle.
To avoid this, we can join the tables
to ask more complicated questions
about our database.

When we join two tables on a column, it will create a row in the
joined table whenever that value is the same in both tables.  For
example, if we joined `recipes` with `recipe_ingredients`:

```sql
SELECT *
FROM recipes
JOIN recipe_ingredients
ON recipes.recipe_id = recipe_ingredients.recipe_id
```

We get the table:

| recipe_id | recipe_name    | recipe_id | ingredient_id | amount |
| --------- | -------------- | --------- | ------------- | ------ |
|         0 | Tacos          | 0         |             0 | 1      |
|         0 | Tacos          | 0         |             1 | 2      |
|         0 | Tacos          | 0         |             2 | 2      |
|         0 | Tacos          | 0         |             3 | 3      |
|         0 | Tacos          | 0         |             4 | 1      |
|         1 | Tomato Soup    | 1         |             2 | 2      |
|         1 | Tomato Soup    | 1         |             5 | 1      |
|         2 | Grilled Cheese | 2         |             4 | 1      |
|         2 | Grilled Cheese | 2         |             6 | 2      |

Getting back to our example from above,
we can compute the ingredient IDs for 
for 'Tomato Soup' by joining 
`recipes` with `recipe_ingredients` on
the recipe ID.

```sql
SELECT recipe_ingredients.ingredient_id
FROM recipes
JOIN recipe_ingredients
ON recipes.recipe_id = recipe_ingredients.recipe_id
WHERE recipes.recipe_name = 'Tomato Soup'
```

This returns:

| ingredient_id |
| ------------- |
| 2             |
| 5             |

As an aside, having to use the table names over
and over again in a SQL query is cumbersome, and
there is a shorthand where we can give each table
a nickname:

```sql
SELECT b.ingredient_id
FROM recipes AS a
JOIN recipe_ingredients AS b
ON a.recipe_id = b.recipe_id
WHERE a.recipe_name = 'Tomato Soup'
```
This is functionally equivalent, but somewhat easier to understand.


# JOIN Three Tables in SQL

What's  cool about SQL is that we can JOIN multiple tables.
To find the ingredient names for the 'Tomato Soup' recipe,
all we have to do is join all three tables on recipe ID:

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

This returns the table:

| ingredient_name |
| --------------- |
| Tomatoes        |
| Milk            |

Another great about SQL is that once we have laid out data across
three normalized tables, it is just as easy to ask many other
questions about our data.

For example, to find all the recipes that include 'tomatoes':

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

As expected, this returns:

| recipe_name |
| ----------- |
| Tacos       |
| Tomato Soup |


# The GROUP BY Operator In SQL

The next major command to learn in SQL is `GROUP BY`, which can be
used to aggregate rows in a table.

Supposed for example that we wanted to list the number
of ingredients in each recipe. We could do this by 
grouping the rows in the `recipe_ingredients` table by the
recipe ID and count the number or grouped rows.

```sql
SELECT recipe_id, 
  COUNT(ingredient_id) as num_ingredients
FROM recipe_ingredients
GROUP BY recipe_id
ORDER BY num_ingredients DESC
```

The code returns:

| recipe_id | num_ingredients |
| --------- | --------------- |
|         0 |               5 |
|         1 |               2 |
|         2 |               2 |

We can combine the `GROUP BY` and `JOIN` operators in a single query.
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

PUT A NOTE ABOUT HAVING IS THE SAME AS A FILTER AFTER THE SUBQUERY

# Subqueries in SQL

As an example of a harder
SQL query, we could imaging trying to
make a list of the number of ingredients for
all recipes that include tomatoes.

To do this, we first would need to
find all the recipes which include tomatoes
and then count the number of ingredients
for each of those recipes.

We could imagine doing this in two steps.
First, we find the recipes that
have tomatoes in it:

```
SELECT a.recipe_id
FROM recipe_ingredients AS a
JOIN ingredients AS b
ON a.ingredient_id = b.ingredient_id
WHERE b.ingredient_name = 'Tomatoes' 
```

This creates the table:

| recipe_id |
| --------- |
|         0 |
|         1 |

Next, we could joining this table with
the ingredients count table from above
to filter out the recipes that aren't in
this table.

This leads us to the idea of subqueries.
In SQL. Because every SQL query returns a table,
so wherever a table can be used in SQL,
instead a subquery can be used.

Therefore, we can compute the desired table
as follows

```sql
SELECT b.recipe_name, 
    COUNT(a.ingredient_id) as num_ingredients
FROM recipe_ingredients as a
JOIN recipes AS b
ON a.recipe_id = b.recipe_id
JOIN 
    (
        SELECT c.recipe_id
        FROM recipe_ingredients AS c
        JOIN ingredients AS d
        ON c.ingredient_id = d.ingredient_id
        WHERE d.ingredient_name = 'Tomatoes' 
    ) as e
ON b.recipe_id = e.recipe_id
GROUP BY a.recipe_id
```

This returns the expected table

| recipe_name | num_ingredients |
| ----------- | --------------- |
| Tacos       |               5 |
| Tomato Soup |               2 |

What's cool about SQL is that it is incredibly
flexible about the structure of queries. Similar to how we
can JOIN as many tables together as needed, we can also
nest multiple subquieries together.

# The DISTINT Operator

In SQL, `DISTINCT` can be used to
find all of the unique elements in a set.

For example, to find all the recipes that include
either beef or cheese, we could use the SQL query below:

```sql
SELECT DISTINCT recipe_name
FROM recipe_ingredients AS a
JOIN ingredients AS b
ON a.ingredient_id = b.ingredient_id
JOIN recipes AS c
ON a.recipe_id = c.recipe_id
WHERE b.ingredient_name = 'Cheese' OR b.ingredient_name = 'Beef'
```

Note that here the `DISTINT`
keyword is required because otherwise two rows would
be returned for tacos since they contain bhot
cheese and beef.

TODO: PUT A NOTE ABOUT 
```sql
COUNT(DISTINCT XX)
```
WHICH IS SURPRISINGLY USEFUL


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


# Query Optimization

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

# Thanks for Reading

In this post, we went over the key concepts of good database design
and dateabase normalization. Then, we went over the topics of how
to issue queries on a database using the `SELECT` statement. Finally,
we saw the power of filtering, joining, and aggregating multiple
tables in order to ask advanced questions about your data.

# Links

This was just a whirlwind tour of what's so great
about SQL databases. There are tons of advanced topics left to explore.
Here is some further reading to get you started:

* SQL databases go through great lenghts to deal with `NULL` values
  in a sensible way. [Here](http://dev.mysql.com/doc/refman/5.0/en/working-with-null.html)
  is some documentation in the way MySQL handles `NULL` values.
* SQL has the concept of `INNER`, `LEFT`, `RIGHT`, and `OUTER` joins,
  which behave different in the situation where you are joining on 
  columns with `NULL` values in them.
  [Here](http://dev.mysql.com/doc/refman/5.0/en/join.html)
  is some documentation on different joins.
* XXXXXXXXXXX TODO: NOTE ABOUT THE `UNION` AND `INTERSECTION` COMMAND.
* [Table Indicies](http://en.wikipedia.org/wiki/Database_index) are a concept in
  database design where sorted indicies of a table can be cached to
  improve performance of particular queries on a table.
* [Views](http://dev.mysql.com/doc/refman/5.0/en/create-view.html) in
  SQL act as temporary tables, able to both simplify queiries in MySQL
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

{% include twitter_plug.html %}

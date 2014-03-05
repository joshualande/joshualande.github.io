---
layout: post
title: Find What You Need to Know By Querying in SQL
comments: true
---

In the previous post, I went over the benefits of database normalization
for correctly organizing data in a database.  In this post, I will
take the example recipes database worked out in the previous post
and use it to ask successfuly harder SQL queriies.  This querieis
will allow me to introduce the basic operators in SQL like filtering,
joining and aggregating.

# THE SELECT, FROM, and WHERE Statements in SQL

In our recipes database,
we had three tables. One
had a list of recipes, one with a list of ingredients, and one mapped
ingredients to recipies.  Given this schema, there are all kinds
of interesting questions we could ask using our recipies.

If we wanted to find all the ingredients in the recipe for "Tomato Soup",
we could first figure out the recipe ID for "Tomato Soup".
We always query information from the database using the `SELECT` statement:

```sql
SELECT recipe_id 
FROM recipes
WHERE recipe_name="Tomato Soup"
```

This query says take the table of recipe names,
filter for rows where the name is "Tomato Soup", 
and select the recipe ID column.
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
* SQL supports set operations like `UNION` and `INTERSECTION` XXXXXXX FIND LINK
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

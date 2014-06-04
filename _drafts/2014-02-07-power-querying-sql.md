---
layout: post
title: Uncover the Power of Querying in SQL
comments: true
permalink: "power-querying-sql"
---

*This is the fourth post in a [series of posts]({% post_url 2014-04-17-data-science-sql %})
about doing data science with SQL. The 
[previous post]({% post_url 2014-04-28-create-tables-sql %})
went over the commands
required to set up the example recipes database from the 
[first post]({% post_url 2014-04-18-database-normalization %}) 
in this series.*

In this post, I will use the example recipes database
from the
[first post]({% post_url 2014-04-18-database-normalization %})
to go over the basics of quering in SQL with the `SELECT` statement.
I will start with the basic operators like filtering, joining, and
aggregating. Then I will show how these simple commands
can be combined to create very powerful queries.
By the end of this post, you will be able to write advanced SQL queries.

## SELECT, FROM, and WHERE in SQL

We can use the `SELECT` statement in SQL to query data from a
database.  For example, we might be interested in finding
all of the ingredients in the "Tomato Soup" recipe
from the 
recipies database from the 
[first post]({% post_url 2014-04-18-database-normalization %}) 
in this series. 

This query is non-trivial because this
information is spread across three tables.
As a first, step we could query for the recipe ID of this
recipe with:

```sql
SELECT recipe_id 
  FROM recipes
 WHERE recipe_name="Tomato Soup"
```

This says to take the `recipes` table and take the `recipe_id`
column for all rows where the `recipe_name` column has a
particular value.  This query returns the table

| recipe_id |
| --------- |
|         2 |

Given this recipe ID, we can get the ingredient IDs
using a similar query on the recipe-ingredients-mapping table:

```sql
SELECT ingredient_id
  FROM recipe_ingredients
 WHERE recipe_id = 2
```

This returns

| ingredient_id |
| ------------- |
| 3             |
| 6             |

Finally, we can pull out the ingredient names knowing the ingredient
IDs:

```sql
SELECT ingredient_name
  FROM ingredients
 WHERE ingredient_id=3
    OR ingredient_id=6
```

This returns:

| ingredient_name |
| --------------- |
| Tomatoes        |
| Milk            |

## The JOIN Operator in SQL

Because our data is spread across three tables, it is cubmersome
and error-prone to have to run multiple queries to find the information
we want.  We can avoid this by joining the tables together.

When we join two tables on a column in SQL, it will create a row
in the joined table whenever that value in the join column is the
same in both tables.  For example, if we joined `recipes` with
`recipe_ingredients` on the recipe ID:

```sql
SELECT *
  FROM recipes
  JOIN recipe_ingredients
    ON recipes.recipe_id = recipe_ingredients.recipe_id
```

We get the table:

| recipe_id | recipe_name    | recipe_id | ingredient_id | amount |
| --------- | -------------- | --------- | ------------- | ------ |
|         3 | Grilled Cheese | 3         | 5             | 1      |
|         3 | Grilled Cheese | 3         | 7             | 2      |
|         1 |          Tacos | 1         | 1             | 1      |
|         1 |          Tacos | 1         | 2             | 2      |
|         1 |          Tacos | 1         | 3             | 2      |
|         1 |          Tacos | 1         | 4             | 3      |
|         1 |          Tacos | 1         | 5             | 1      |
|         2 |    Tomato Soup | 2         | 3             | 2      |
|         2 |    Tomato Soup | 2         | 6             | 1      |

This joined table includes the recipe names along with the recipe
IDs for each recipe-ingredient pair.

Getting back to our example from above, we can compute the ingredient
IDs for 'Tomato Soup' by joining `recipes` with `recipe_ingredients`
on the recipe ID.

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
| 3             |
| 6             |

In the next section, we will show how we can also join with the
`ingredients` table to directly get the ingredient names.

Having to use the full table names repeatedly in our query is
cumbersome.  SQL provides a convenient shorthand where
we can give each table a nickname:

```sql
SELECT b.ingredient_id
  FROM recipes AS a
  JOIN recipe_ingredients AS b
    ON a.recipe_id = b.recipe_id
 WHERE a.recipe_name = 'Tomato Soup'
```

This is the same query as before, but slightly less verbose.

## JOINing Multiple Tables in SQL

SQL allows us to join multiple tables for even more powerful queries.
Getting back to our original example, we can find the ingredient
names for the ingreidents in 'Tomato Soup' by joining all three
tables together:

```sql
SELECT c.ingredient_name 
  FROM recipes AS a
  JOIN recipe_ingredients AS b
    ON a.recipe_id = b.recipe_id
  JOIN ingredients AS c
    ON b.ingredient_id = c.ingredient_id
 WHERE a.recipe_name = "Tomato Soup"
```

As expected, this returns the table:

| ingredient_name |
| --------------- |
| Tomatoes        |
| Milk            |

What is great about SQL is that by joining tables together,
we can ask many diverse questions about our data.
For example, finding all the recipes that include 'tomatoes'
is just as easy:

```sql
SELECT a.recipe_name
  FROM recipes AS a
  JOIN recipe_ingredients AS b
    ON a.recipe_id = b.recipe_id
  JOIN ingredients AS c
    ON b.ingredient_id = c.ingredient_id
 WHERE c.ingredient_name = "tomatoes"
```

This returns:

| recipe_name |
| ----------- |
| Tacos       |
| Tomato Soup |


## The GROUP BY Operator In SQL

The next important concept in SQL is aggregating rows.
This is done with the `GROUP BY` command.

Supposed for example that we wanted to list the number
of ingredients in each recipe. We could do this by 
grouping the rows in the `recipe_ingredients` table by the
recipe ID and counting the number or grouped rows:

```sql
  SELECT recipe_id, 
         COUNT(ingredient_id) AS num_ingredients
    FROM recipe_ingredients
GROUP BY recipe_id
ORDER BY num_ingredients DESC
```

The code returns:

| recipe_id | num_ingredients |
| --------- | --------------- |
|         1 |               5 |
|         2 |               2 |
|         3 |               2 |

We can combine the `GROUP BY` and `JOIN` operators in a single query.
To compute in addition the price of each recipe, we would need to figure
out the price of each ingredient by joining with the ingredients table.
This query would look like:

```sql
  SELECT recipe_id, 
         COUNT(a.ingredient_id) AS num_ingredients, 
         SUM(a.amount*b.ingredient_price) AS total_price
    FROM recipe_ingredients as a
    JOIN ingredients as b
      ON a.ingredient_id = b.ingredient_id
GROUP BY a.recipe_id
```

This returns

| recipe_id | num_ingredients | total_price |
| --------- | --------------- | ----------- |
|         1 |               5 |          20 |
|         2 |               2 |           5 |
|         3 |               2 |           7 |

Similarly, if we want to make the table the display recipe names,
we could also JOIN with the recipes tables:

```sql
  SELECT c.recipe_name, 
         COUNT(a.ingredient_id) AS num_ingredients, 
         SUM(a.amount*b.ingredient_price) AS total_price
    FROM recipe_ingredients AS a
    JOIN ingredients AS b
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

## Aggregation Functions in SQL

As you saw above, SQL can apply different
aggregation functions. This query demonstrates
more of them:

```sql
SELECT COUNT(ingredient_price) as count,
       AVG(ingredient_price) as avg,
       SUM(ingredient_price) as sum,
       MIN(ingredient_price) as min,
       MAX(ingredient_price) as max,
       STDDEV(ingredient_price) as stddev,
       SUM(ingredient_price) as sum
  FROM ingredients
```

This returns

| count |    avg | sum | min | max |             stddev | sum |
| ----- | ------ | --- | --- | --- | ------------------ | --- |
|     7 | 2.2857 |  16 |   1 |   5 | 1.2777531299998797 |  16 |

Note here that when you leave out the `GROUP BY` class, but include
an aggregation function, SQL assumes that you want to group all
rows together.

You can find the full list of aggregation functions in MySQL
[here](http://dev.mysql.com/doc/refman/5.0/en/group-by-functions.html).

## The HAVING Operator in SQL

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
|         2 |               2 |
|         3 |               2 |

As you will see below, `HAVING`
is really just a convenient shorthand to
avoid having to use a subquery.

## Subqueries in SQL

A more challenging query would be to make a list of the number of
ingredients but only for recipes that include tomatoes.

To do this, we first would need to find all the recipes which include
tomatoes and then count the number of ingredients for each of those
recipes.

We could imagine doing this in two steps.  First, we find the recipes
that have tomatoes in it:

```sql
SELECT a.recipe_id
  FROM recipe_ingredients AS a
  JOIN ingredients AS b
    ON a.ingredient_id = b.ingredient_id
 WHERE b.ingredient_name = 'Tomatoes' 
```

This creates the table:

| recipe_id |
| --------- |
|         1 |
|         2 |

Next, we could joining this table with the ingredients count table
from above to filter out the recipes that aren't in this table.

This leads us to the idea of subqueries.  In SQL. Because every SQL
query returns a table, inside of a query a subquery can be used
instead of a table.

Therefore, we can compute this as follows

```sql
  SELECT b.recipe_name, 
         COUNT(a.ingredient_id) AS num_ingredients
    FROM recipe_ingredients AS a
    JOIN recipes AS b
      ON a.recipe_id = b.recipe_id
    JOIN (
             SELECT c.recipe_id
             FROM recipe_ingredients AS c
             JOIN ingredients AS d
             ON c.ingredient_id = d.ingredient_id
             WHERE d.ingredient_name = 'Tomatoes' 
         ) AS e
      ON b.recipe_id = e.recipe_id
GROUP BY a.recipe_id
```

This returns the expected table

| recipe_name | num_ingredients |
| ----------- | --------------- |
| Tacos       |               5 |
| Tomato Soup |               2 |

What's cool about SQL is that it is very
flexible about the structure of queries. Similar to how we
can JOIN as many tables together as needed, we can also
nest multiple subquieries together to create more powerful
queries.

## The DISTINT Operator

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
 WHERE b.ingredient_name = 'Cheese' 
    OR b.ingredient_name = 'Beef'
```

This creates

|    recipe_name |
| -------------- |
| Grilled Cheese |
|          Tacos |

Note that here the `DISTINT`
keyword is required because otherwise two rows would
be returned for tacos since they contain both
cheese and beef.

We can count the number of distinct recipes by
putting the `COUNT` keyword outside the `DISTINCT`
keyword:

```sql
SELECT COUNT(DISTINCT recipe_name) AS num_recipes
  FROM recipe_ingredients AS a
  JOIN ingredients AS b
    ON a.ingredient_id = b.ingredient_id
  JOIN recipes AS c
    ON a.recipe_id = c.recipe_id
 WHERE b.ingredient_name = 'Cheese' 
    OR b.ingredient_name = 'Beef'
```

This returns:

| num_recipes |
| ----------- |
|           2 |


## The ORDER BY operator in SQL

The next command we will cover is `ORDER BY`.
`ORDER BY` can be used to sort the rows based on
a particular column. For example, if we wanted
to sort the ingredients by how expensive they are
in descending order of price, we could run the query:

```sql
  SELECT *
    FROM ingredients
ORDER BY ingredient_price DESC
```

This returns

| ingredient_id | ingredient_name | ingredient_price |
| ------------- | --------------- | ---------------- |
| 1             | Beef            | 5                |
| 5             | Cheese          | 3                |
| 3             | Tomatoes        | 2                |
| 4             | Taco Shell      | 2                |
| 7             | Bread           | 2                |
| 2             | Lettuce         | 1                |
| 6             | Milk            | 1                |


If we wanted to sort columns of the same price alphabetically by name, we could
use a similar query:

```sql
  SELECT *
    FROM ingredients
ORDER BY ingredient_price DESC,
         ingredient_name
```

This creates the table:

| ingredient_id | ingredient_name | ingredient_price |
| ------------- | --------------- | ---------------- |
| 1             | Beef            | 5                |
| 5             | Cheese          | 3                |
| 7             | Bread           | 2                |
| 4             | Taco Shell      | 2                |
| 3             | Tomatoes        | 2                |
| 2             | Lettuce         | 1                |
| 6             | Milk            | 1                |


## The LIMIT operator in SQL

We can use the `LIMIT` operator to limit the number of results returned
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
| 1             |            Beef |                5 |

# Inequality And Self Joins

So far, all of our examples were equality joins, where we join two
tables only on the condition of a row being equal. And so far,
we have only joined different tables together.

A challenging
List Most expensive ingredient for each recipe.

```
SELECT *
FROM recipe_ingredients AS a
JOIN recipe_ingredients AS b
ON a.recipe_id = b.recipe_id AND a.amount < b.amount
```



{% include twitter_plug.html %}

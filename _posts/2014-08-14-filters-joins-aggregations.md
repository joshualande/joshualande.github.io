---
layout: post
title: "Filters, Joins, Aggregations, and All That: A Guide to Querying in SQL"
comments: true
permalink: "filters-joins-aggregations"
---

*This is the fourth post in a [series of posts]({% post_url 2014-04-17-data-science-sql %})
about doing data science with SQL. The 
[previous post]({% post_url 2014-04-28-create-tables-sql %})
went over the commands
required to set up the example recipes database from the 
[first post]({% post_url 2014-04-18-database-normalization %}) 
in this series.*

In this post, I will use the example recipes database from the
[first post]({% post_url 2014-04-18-database-normalization %}) to
go over the basics of querying in SQL with the `SELECT` statement.
I will start with the basic operators of filtering, joining, and
aggregating. Then I will show how these simple commands can be
combined to create powerful queries.  By the end of this post, you
should be able to write advanced SQL queries.

## SELECT, FROM, and WHERE in SQL

We can use the `SELECT` statement in SQL to query data from a
database.  For example, we might be interested in finding
all of the ingredients in the "Tomato Soup" recipe
(from the 
recipes database described in the
[first post]({% post_url 2014-04-18-database-normalization %}) 
in this series). 

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

Given this recipe ID, we can get the ingredient IDs for the recipe
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

Finally, we can find the ingredient names knowing their IDs:

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

Because our data is spread across three tables, it is cumbersome
and error-prone to have to run multiple queries to find the information
we want.  We can avoid this by joining the tables together.

When we join two tables on a column in SQL, it will create every
possible combination of rows in the output table where the condition
holds For example, if we joined `recipes` with `recipe_ingredients`
on the recipe ID:

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

Having to use the full table names repeatedly in a SQL query is
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
Getting back to our original example, we can directly find the ingredient
names for the ingredients in 'Tomato Soup' by joining all three
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
we can ask very diverse questions about our data.
For example, finding all the recipes that include 'tomatoes'
is just as straightforward:

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

Supposed for example that we wanted to find the number
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

Similarly, if we want to make the table display recipe names,
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

Finally, as a shorthand, SQL allows you to 
refer to the columns in the `SELECT`
clause by numbering 

```sql
  SELECT c.recipe_name, 
         COUNT(a.ingredient_id) AS num_ingredients, 
         SUM(a.amount*b.ingredient_price) AS total_price
    FROM recipe_ingredients AS a
    JOIN ingredients AS b
      ON a.ingredient_id = b.ingredient_id
    JOIN recipes AS c
      ON a.recipe_id = c.recipe_id
GROUP BY 1
```

Some people consider this to be more elegant
and less error prone.

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

As you will see below, `HAVING` is just a convenient shorthand to
avoid using a subquery.

## Subqueries in SQL

A more challenging query would be to make a list of the number of
ingredients, but only for recipes that include tomatoes.

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

Next, we could joining this table with the ingredient count table
from the query above to filter out the recipes that aren't in this table.

This leads us to the idea of subqueries. Because every SQL
query returns a table, another SQL query can be used instead of a table
inside of another SQL query.

The final query is:

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

As expected, this returns

| recipe_name | num_ingredients |
| ----------- | --------------- |
| Tacos       |               5 |
| Tomato Soup |               2 |

What's cool about SQL is that it is very flexible and can allow
multiple subqueries to be nested together.

## The DISTINCT Operator

In SQL, `DISTINCT` operator can be used to find all of the unique
rows.

For example, to find all the recipes that include
either beef or cheese, we could use the SQL query:

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

Note that here the `DISTINCT`
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

`ORDER BY` can be used to sort the output rows based on a particular
column. For example, if we wanted to sort the ingredients by how
expensive they are in descending order of price, we could run the
query:

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
use a similar query but perform a second sort based on the price:

```sql
  SELECT *
    FROM ingredients
ORDER BY ingredient_price DESC, ingredient_name
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

This returns just one result:

| ingredient_id | ingredient_name | ingredient_price |
| ------------- | --------------- | ---------------- |
| 1             |            Beef |                5 |

# Self AND Inequality Joins

The final concepts we will learn about is self and equality joins.  As
a concrete example, supposed that we wanted to compute the number
of shared ingredients for all pairs of recipes.

To compute this, we can join the ingredient-recipe mapping table
with itself and select for rows that have the same ingredient.
This will create a row for ever matching ingredient in every pair
of recipes:

```sql
  SELECT a.recipe_id, b.recipe_id, a.ingredient_id
    FROM recipe_ingredients AS a
    JOIN recipe_ingredients AS b
      ON a.ingredient_id = b.ingredient_id
     AND a.recipe_id != b.recipe_id
ORDER BY a.recipe_id, b.recipe_id
```

Note that we have to filter for `a.recipe_id != b.recipe_id` to avoid
matching recipe with themselves. Joins with an inequality
condition are unsurprisingly called inequality joins.

This returns

| recipe_id | recipe_id | ingredient_id |
| --------- | --------- | ------------- |
| 1         |         2 |             3 |
| 1         |         3 |             5 |
| 2         |         1 |             3 |
| 3         |         1 |             5 |

This table shows the recipe 1 ("Tacos") and recipe 2 ("Tomato Soup")
share ingredient 3 ("Tomatoes"). Similarly, recipe 1 ("Tacos") and
recipe 3 ("Grilled Cheese") share ingredient 5 ("Cheese").

One issue with this query is that it matches every pair of ingredients
twice. To avoid this, we can modify the query to 
return only rows when the first recipe ID is less than
the second.

Finally, we can can aggregate over the recipe IDs
to compute the count of shared ingredients:

```sql
  SELECT a.recipe_id, 
         b.recipe_id, 
         COUNT(*) as num_shared
    FROM recipe_ingredients AS a
    JOIN recipe_ingredients AS b
      ON a.recipe_id < b.recipe_id
     AND a.ingredient_id = b.ingredient_id 
GROUP BY a.recipe_id, b.recipe_id
```

As expected, this returns

| recipe_id | recipe_id | num_shared |
| --------- | --------- | ---------- |
|         1 |         2 |          1 |
|         1 |         3 |          1 |


We can include the recipe names by also joining with the recipes
table:

```sql
  SELECT c.recipe_name AS recipe_1, 
         d.recipe_name AS recipe_2, 
         COUNT(*) AS num_shared
    FROM recipe_ingredients AS a
    JOIN recipe_ingredients AS b
      ON a.recipe_id < b.recipe_id
     AND a.ingredient_id = b.ingredient_id 
    JOIN recipes AS c
      ON a.recipe_id = c.recipe_id
    JOIN recipes AS d
      ON b.recipe_id = d.recipe_id
GROUP BY a.recipe_id, b.recipe_id
ORDER BY recipe_1, recipe_2
```

This returns:

| recipe_1 |       recipe_2 | num_shared |
| -------- | -------------- | ---------- |
|    Tacos | Grilled Cheese |          1 |
|    Tacos |    Tomato Soup |          1 |

From these examples, I hope you see that the simple SQL operators
can be combined to perform very powerful queries.

{% include twitter_plug.html %}

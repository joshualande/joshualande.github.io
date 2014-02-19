---
layout: page
title: An Introduction to Analyzing Big Data With Apache Pig Using Simple Examples
comments: true
---

In [my blog post](XXXXX), I introduced the basics
of analyzing data using SQL.

In this post, we will see how a similar work can be done
on [big data](http://en.wikipedia.org/wiki/Big_data)
using [Apache Pig](https://pig.apache.org/).

For the most part, there is a direct mapping
from a SQL query to PigLatin code.

# Example Data Set

I will introduce the major topics in Pig by way of
a simple example.

First we need to create a sample data set to play with.
For this post, we will use the same tables
from the recipe example in the other post.
We have one table of recipies, one of ingredients,
and one mapping recipies to ingredients.

For our example, I will assume you have the three
tables stored as [CSV](http://en.wikipedia.org/wiki/Comma-separated_values).
files on your computer: 

First, we have the recipes data source. 
The data in this table is recipe_id, recipe_name, and recipe_description:

```
$ cat recipes.csv
0,Tacos,Mom's famous Tacos
1,Tomato Soup,Homemade Tomato soup
2,Grilled Cheese,Delicious Cheese Sandwich
```

Second, we have the ingredients data set.  The data in this table
is ingredient_id, ingredient_name, and ingredient_price:

```
$ cat ingredients.csv 
0,Beef,5
1,Lettuce,1
2,Tomatoes,2
3,Taco Shell,2
4,Cheese,3
5,Milk,1
6,Bread,2
```

Finally, we have the recipe-ingredients mapping data source:

```
$ cat recipe_ingredients.csv 
0,0,1 
0,1,2 
0,2,2 
0,3,3 
0,4,1 
1,2,2 
1,5,1 
2,4,1 
2,6,2 
```

What's great about Pig is it doesn't force data to 
be stored in any particular schema. Pig can read data
in whatever format you want. Pig also has built-in readers
for simple file formats
like [CSV](http://en.wikipedia.org/wiki/Comma-separated_values).

In typical real-life situations, we expect the files themselves
to be very large and unable to fit on any single computer.
The data itself is typically stored on the 
[Hadoop Distributed File System (HDFS)](http://en.wikipedia.org/wiki/Apache_Hadoop).
And our Pig code would perform a distributed analysis
of this data over the comptuer cluster using distributed algoirhtms.

But for our examples, we will learn the basics of PigLatin, the
language of Apache Pig using our small data sources run in local mode.

```
$ pig -x local
```

# Key Differences Between SQL and Pig

Note that HAVING doens't exist since it isn't needed.
You can just folloup up a `GROUP BY` with a `FILTER`.

No such thing as subquieries b/c you can procedurally
create tables.


Also note about how 

# Loading Data in Pig

As our first script,

```
recipes = LOAD 'recipes.csv' 
    USING PigStorage(',') 
    AS (recipe_id:int, recipe_name:chararray, 
        recipe_description:chararray);

ingredients = LOAD 'ingredients.csv' 
    USING PigStorage(',') 
    AS (ingredient_id:int, ingredient_name:chararray, 
        ingredient_price:float);

recipe_ingredients = LOAD 'recipe_ingredients.csv' 
    USING PigStorage(',') 
    AS (recipe_id:int, ingredient_id:int, 
        amount:int);
```

Now that we have loaded the data, we can easily look at the shema of any table

```
DESCRIBE  recipes;
```

Which returns the schema Pig has for the table:

```
recipes: {recipe_id: int,recipe_name: chararray,recipe_description: chararray}
```

Similar, we can dump the values in any table with the command:

```
DUMP recipes
```

This returns the output:

```
(0,Tacos,Mom's famous Tacos)
(1,Tomato Soup,Homemade Tomato soup)
(2,Grilled Cheese,Delicious Cheese Sandwich)
```

This shows that the table itself contains:

| recipe_id |    recipe_name |        recipe_description |
| --------- | -------------- | ------------------------- |
|         0 |          Tacos |        Mom's famous Tacos |
|         1 |    Tomato Soup |      Homemade Tomato soup |
|         2 | Grilled Cheese | Delicious Cheese Sandwich |

# Saving out tables to a file

XXX.

# The FILTER and FOREACH Operator in Apache Pig

Suppose we want to find the recipe_id for the recipe named "Tomato Soup".
To do this, we would issue the Pig command:

```
soup_recipe = FILTER recipes 
    BY recipe_name == 'Tomato Soup';
```

The `soup_recipe` table contains

| recipe_id | recipe_name |   recipe_description |
| --------- | ----------- |   ------------------ |
|         1 | Tomato Soup | Homemade Tomato soup |

If we wanted to select only the `user_id` from this table,
we could use the `FOREACH` and `GENERATE` command:

```
soup_recipe_id = FOREACH soup_recipe 
    GENERATE recipe_id;
```

The `soup_recipe_id` contains the `recipe_id` column, as expected:

| recipe_id |
| --------- |
|         1 |

# The ORDER BY Operator in Pig

Suppose we wanted to sort the ingredients table
by price in descending order. In Pig we could use
the ORDER BY Operator:

```
ingredients_sorted = ORDER ingredients BY ingredient_price DESC;
```

The `ingredients_sorted` table contains:

| ingredient_id | ingredient_name | ingredient_price |
| ------------- | --------------- | ---------------- |
| 0             |            Beef |                5 |
| 4             |          Cheese |                3 |
| 2             |        Tomatoes |                2 |
| 3             |      Taco Shell |                2 |
| 6             |           Bread |                2 |
| 1             |         Lettuce |                1 |
| 5             |            Milk |                1 |

# The LIMIT Operator

Similarly, if we wanted to find only the most expensive ingredient,
we could

```
most_expensive = LIMIT ingredients_sorted 1;
```

The `most_expensive` table contains only the first row:

| ingredient_id | ingredient_name | ingredient_price |
| ------------- | --------------- | ---------------- |
| 0             |            Beef |                5 |

# Joins in Pig

Joins in Pig are very similar to SQL.
When we join the `recipes` table with the `recipe_ingredients` table, we get:

```
recipe_ingredients_by_name = JOIN recipes BY recipe_id, recipe_ingredients BY recipe_id;
```

When we `DESCRIBE` the `recipe_ingredients_by_name` table, we see that the schema is

```
recipe_ingredients_by_name: {
    recipes::recipe_id: int,
    recipes::recipe_name: chararray,
    recipes::recipe_description: chararray,
    recipe_ingredients::recipe_id: int,
    recipe_ingredients::ingredient_id: int,
    recipe_ingredients::amount: int}
```

If we wanted to broadcast out only the columns we cared about, we could do so using the `FOREACH` command:

```
nicer_recipe_ingredients_by_name = FOREACH recipe_ingredients_by_name 
    GENERATE
    recipes::recipe_name as recipe_name,
    recipes::recipe_id as recipe_id,
    recipes::recipe_description as recipe_description,
    recipe_ingredients::ingredient_id as ingredient_id,
    recipe_ingredients::amount as amount;
```

The table `nicer_recipe_ingredients_by_name` now contains:

|    recipe_name | recipe_id |        recipe_description | ingredient_id | amount |
| -------------- | --------- | ------------------------- | ------------- | ------ |
|          Tacos |         0 |        Mom's famous Tacos |             0 |      1 |
|          Tacos |         0 |        Mom's famous Tacos |             1 |      2 |
|          Tacos |         0 |        Mom's famous Tacos |             2 |      2 |
|          Tacos |         0 |        Mom's famous Tacos |             3 |      3 |
|          Tacos |         0 |        Mom's famous Tacos |             4 |      1 |
|    Tomato Soup |         1 |      Homemade Tomato soup |             2 |      2 |
|    Tomato Soup |         1 |      Homemade Tomato soup |             5 |      1 |
| Grilled Cheese |         2 | Delicious Cheese Sandwich |             4 |      1 |
| Grilled Cheese |         2 | Delicious Cheese Sandwich |             6 |      2 |

What is cool about Pig is that unlike SQL where more advanced
queries become more cumbersome requirng nested subquiries and mutiple joins, in Pig compliated quieries
just reuqire chaining the basic commands together.

If we wanted to find the ingredient names for 
recipies which contain "Tomato Soup", all we have to do is 
`FILTER` our table, `JOIN` it with the `ingredients` table,
and then broadcast out the column we want.

In Pig, this becomes

```
recipe_ingredients_filtered = FILTER nicer_recipe_ingredients_by_name 
    BY recipe_name == 'Tomato Soup';

recipe_ingredients_recipe_name = JOIN 
    recipe_ingredients_filtered BY ingredient_id, 
    ingredients BY ingredient_id;

tomato_soup_ingredients = FOREACH recipe_ingredients_recipe_name 
    GENERATE ingredient_name;
```

As expected, the table `tomato_soup_ingredients` table contains

| ingredient_name |
| --------------- |
|        Tomatoes |
|            Milk |

# The GROUP BY Operator in Apache Pig


# The DISTINCT Operator

# Set Operations: UNION and INTERSECTION

# Advanced Pig:

* Skew `JOIN`
* replicated `JOIN`

# Links

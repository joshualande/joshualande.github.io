---
layout: post
title: An Introduction to Analyzing Big Data With Apache Pig Using Simple Examples
comments: true
---

In [an earlier blog post](XXXXX), I introduced the basics of analyzing
data using SQL.  In this post, we will see how the same sorts of
analysis done in SQL can be done on [big
data](http://en.wikipedia.org/wiki/Big_data) stored in HDFS using
using [Apache Pig](https://pig.apache.org/).

By using Pig, it is surprisingly easy to get started using to analyze
easily tens of terrabytes of data using distributed map-reduce
algorithms.  In this post, I will go over the basics of [Pig
Latin](http://pig.apache.org/docs/r0.7.0/piglatin_ref2.html), the
programming language used to write Pig job.  And I will do all of
this by way of a simple example which you can easily run on your
local computer.

## Introduction to Hadoop, HDFS, MapReduce, and Apache Pig

In this section, I will give an overview of hadoop, MapReduce, and
the situaions where Apache Pig.  If you are already farmilliar with
Hadoop or just want to get started, feel free to skip to the next
section.

In a typical 'big data' situations, we are dealing with data so
large that it cannot fit on any single machine. The data itself is
so large that it has to be distributed over hundreds of different
machines.  [Apache Hadoop](http://hadoop.apache.org/) provides The
[Hadoop Distributed File System
(HDFS)](http://en.wikipedia.org/wiki/Apache_Hadoop) is a pogram
which automates the managment of these very large files, storing
them a distirbuted manner in a computing cluster. In addition, at
this scale the probability of data loss becomes non-neglegible and
to avoid this HDFS stores typically three copies of each file on
independent machines
([here](http://hadoop.apache.org/docs/r1.2.1/hdfs_design.html) is
a detailed description of HDFS).

On top of HDFS, Apache Hadoop provides a distributed data analysis
and data reduction programming language called
[MapReduce](http://wiki.apache.org/hadoop/MapReduce).  MapReduce
limits an analysis to a series of mapper or reducer functions.  But
MapReduce has the benefit that the algorithm is completely parallizable.
In addition, as much as possible MapReduce moves the analysis code
to the machines storing the data to avoid costly data transfers.
And finally, the algorithm itself is [fault
tolerant](http://developer.yahoo.com/hadoop/tutorial/module4.html#tolerence).
If any of the machines fail, MapReduce will automatically rerun
the work on another machine. This is especially important because
the probability of any machine failing increases as the number
of machines increases.

Conversely, writing MapReduce jobs is cubmersome and the map-reduce
framework doesn't always naturally map to the sorts of analysis
people want to do with their data.

To improve developer productivity and make analysizng large data
sets easier, Apache Pig was created as a simpler data analysis
langauge.  On the surface, Pig looks very similar to SQL.  For the
most part, there is a direct mapping from a SQL commands to Pig
code, and Pig provides all the farmilliar functions like filtering,
aggregations, and joining, sorting, deduping, etc.  But under the
hood, Pig converts these commands to a series of MapReduce jobs
which are run in successuion.

In short, Pig then provides all the benefits of MapReduce, but is
significantly easier to use.  In addition, just like with SQL, by
writing your analysis in a high-level descriptive language, you can
allow Pig to do its best to optimize the order of analyzing data
to find a very efficient algoirhtm.


## Key Differences Between SQL and Pig

Although conceptually similar, SQL and Pig are have several practical
differences from SQL. In this section, I will describe how it is
different. But if you want to get started or aren't farmilliar with
SQL, feel free to skip ahead.


* Simpler commands, written one after another.
* Conceptually, the code creates temporary tables, more similar to programming languages.
* Nested tables.
* More complicated data types like tuples, bags, etc. Why is this useful in the big data error.
* Schema on read/not write.

Whereas

Besides the fact that Pig runs on top of HDFS
and convers your analysis code into parralleized
map-reduce jobs, Pig itself has several
practical differences from SQL.

Note that HAVING doens't exist since it isn't needed.
You can just folloup up a `GROUP BY` with a `FILTER`.

No such thing as subquieries b/c you can procedurally
create tables.


Also note about how

## Example Data Set

In this post, I will introduce the
major synax constructs of PigLatin using a simple example.
Of course, the major benefits of Pig come into play
when the data itself is so big that it cannot be
stored on one computer.

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

## Loading Data in Pig

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

## Saving out tables to a file

XXX.

## The FILTER and FOREACH Operator in Apache Pig

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

## The ORDER BY Operator in Pig

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

## The LIMIT Operator

Similarly, if we wanted to find only the most expensive ingredient,
we could

```
most_expensive = LIMIT ingredients_sorted 1;
```

The `most_expensive` table contains only the first row:

| ingredient_id | ingredient_name | ingredient_price |
| ------------- | --------------- | ---------------- |
| 0             |            Beef |                5 |

## Joins in Pig

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


## Nested Data Structures in Pig

So far, everything we have seen about Pig probably seems fairly similar
to SQL. 
But one of the biggest ways in which Pig diverges from
SQL is that it allows for nested data structure.

For example, if we wanted, we could
define our recipes in a totally 
[denormalized](http://en.wikipedia.org/wiki/Denormalization)
data structure where the ingredients are nested within
the rows for each recipe.
commas separating items.

Despite having some obvious distadvantages, there are a
few advantages to having this denormalized data structure.

* Costly joins

To do that, we can define the file as:

```
$ cat recipes_denormalized.tsv
Tacos	{(Beef),(Lettuce),(Tomatoes),(Taco Shell),(Cheese)}
Tomato Soup		{(Tomatoes),(Milk)}
Grilled Cheese	{(Cheese),(Bread)}
```

I will mention that here that I switch from the CSV to TSV file format 
because pig gets confused between the commas separating items
in the bag from commas separating items. 
NOte that the charater between the recipe name
and the bag is a tab!

```
recipes_denormalized = LOAD 'recipes_denormalized.tsv'
    USING PigStorage('\t')
    AS (recipe:chararray, ingredients: {(name:chararray)});
```

When we `DESCRIBE` this table, we see that each row has a character array recipie
and a bag of ingredients:

```
recipes_denormalized: {
    recipe: chararray,
    ingredients: {
        (name: chararray)
    }
}
```

Similarly, when we `DUMP` this table:

```
(Tacos,{(Beef),(Lettuce),(Tomatoes),(Taco Shell),(Cheese)})
(Tomato Soup,{(Tomatoes),(Milk)})
(Grilled Cheese,{(Cheese),(Bread)})
```

Note that the tsv fileformat is wonkly, but Pig also
supports a more conventional JSON format for data.
To read the same data in JSON format, we could
define:


<!--
```
$ cat recipes_denormalized.json
{"recipe":"Tacos","ingredients":["Beef","Lettuce","Tomatoes","Taco Shell","Cheese"]}
{"recipe":"Tomato Soup","ingredients":["Tomatoes","Milk"]}
{"recipe":"Grilled Cheese","ingredients":["Cheese","Bread"]}
```
-->

```
$ cat recipes_denormalized.json
{"recipe":"Tacos",
 "ingredients": [
    {"name":"Beef"},
    {"name":"Lettuce"},
    {"name":"Tomatoes"},
    {"name":"Taco Shell"},
    {"name":"Cheese"}
]}
{"recipe":"Tomato Soup",
 "ingredients": [
    {"name":"Tomatoes"},
    {"name":"Milk"}
]}
{"recipe":"Grilled Cheese",
 "ingredients": [
    {"name":"Cheese"},
    {"name":"Bread"}
]}
```

And load the file using:

```
recipes_denormalized = LOAD 'recipes_denormalized.json' 
    USING JsonLoader('recipe:chararray, ingredients: {(name:chararray)}');
```

Note that because

```
recipes_denormalized = LOAD 'recipes_denormalized.json' USING JsonLoader()}');
```

...

```
STORE recipes_denormalized INTO 'recipes_denormalized.json' USING JsonStorage();
```


---

```
recipe_ingredients_denormalized = FOREACH recipes_denormalized
    GENERATE recipe,
    FLATTEN(ingredients.name) as ingredient;
DUMP recipe_ingredients_denormalized;
```

Creates:

```
(Tacos,Beef)
(Tacos,Lettuce)
(Tacos,Tomatoes)
(Tacos,Taco Shell)
(Tacos,Cheese)
(Tomato Soup,Tomatoes)
(Tomato Soup,Milk)
(Grilled Cheese,Cheese)
(Grilled Cheese,Bread)
```


## The GROUP BY Operator in Apache Pig

```
grouped_ingredients = GROUP recipe_ingredients 
    BY (recipe_id);
```

When we `DESCRIBE` the `grouped_ingredients` table:

```
grouped_ingredients: {
    group: int,
    recipe_ingredients: {
        (recipe_id: int,
         ingredient_id: int,
         amount: int)
    }
}
```

And when we `DUMP` the `grouped_ingredients` table:

```
(0,{(0,0,1),(0,1,2),(0,2,2),(0,3,3),(0,4,1)})
(1,{(1,2,2),(1,5,1)})
(2,{(2,4,1),(2,6,2)})
```

temp:

```
temp = FOREACH grouped_ingredients GENERATE
    group AS recipe_id,
    recipe_ingredients.ingredient_id AS ingredient_id;
```

When we `DUMP` temp


```
(0,{(0),(1),(2),(3),(4)})
(1,{(2),(5)})
(2,{(4),(6)})
```


```
STORE INTO 'temp2' USING PigStorage(',');
```

```
0,{(0),(1),(2),(3),(4)}
1,{(2),(5)}
2,{(4),(6)}
```

```
temp3 = LOAD 'temp2'
    USING PigStorage(',')
    AS (recipe_id:chararray, ingredient_id: {(name:chararray)});
```


## The DISTINCT Operator

## Set Operations: UNION and INTERSECTION

## Advanced Pig:

* Ternary expressions
* Skew `JOIN`
* replicated `JOIN`

## Links

...

{% include twitter_plug.html %}

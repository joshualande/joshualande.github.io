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


First we need to create a sample data set to play with.
For this post, we will use the same tables
from the recipe example in the other post.
We have one table of recipies, one of ingredients,
and one mapping recipies to ingredients.

For our example, I will assume you have the three
tables stored as [CSV](http://en.wikipedia.org/wiki/Comma-separated_values).
files on your computer: 

First, we have the recipes data source:

```
$ cat recipes.csv
0,Tacos,Mom's famous Tacos
1,Tomato Soup,Homemade Tomato soup
2,Grilled Cheese,Delicious Cheese Sandwich
```

Second, we have the ingredients data set:

```
$ cat ingredients.csv 
0,Beef,5
1,Lettuce,1
2,Tomatoes,2
3,Taco,Shell,2
4,Cheese,3
5,Milk,1
6,Bread,2
```

Finally, we have the recipe-ingredients mapping data source:

```
recipe_id,ingredient_id,amount
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

Similar, we can dump the values in any table:

```
(0,Tacos,Mom's)
(1,Tomato,Soup)
(2,Grilled,Cheese)
```

# Filtering rows in Apache Pig

Suppose we want to find the recipe_id for the recipe named "Tomato Soup".
To do this, we would issue the Pig command:

```
soup_recipe = FILTER recipes 
    BY recipe_name == 'Tomato Soup';

soup_recipe_id = FOREACH soup_recipe 
    GENERATE recipe_id;
```

When we dump the `soup_recipe_id` table, it is just has one column recpie_id and one
row with the value 1, as expected:




# Advanced Pig:

Skew Join, replicated JOIN

# Links

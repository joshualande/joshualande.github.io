---
layout: page
title: An Introduction to SQL for Data Scientists
comments: true
---

During my Physics PhD program, I never understood the purpose of 
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

In this post, my intention is to go over broadly what the purpose
of databases are, and how they are different from similar file
formats. My hope is to touch on the key topics of [Database
normalization](http://en.wikipedia.org/wiki/Database_normalization), 
the basic databases operators like joining and aggregating
over tables, and developing indices on tables to speed up queries.

Of course, databases are an enormous topic for which tehre
is [an entire profession](http://en.wikipedia.org/wiki/Database_administrator),
so this will be at most a whirlwind tour of the key concepts, the
purpose being to convey the value of databases
and to get you excited about them.

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
For every recipe 

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


Of course, we could imagine that a real database, like the one
curated by [yummly.com](http://yummly.com), would have lots more information in it.
You might want more accurate list of steps involved in preparing the recipie.
But this simplified exmaple should get the point across.

## Similar python Implementation

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
required to make a paritcular recipie.
This may seem like a contrived example, but in real life it is very often
the case that that you later on have to query your data in ways you did not
originally intent.

As you will see, SQL solves this problem by dividing up our data into multiple tables.

The second issue with the approach listed above is that there are a lot of duplciate data
which shows up in mulitple places. 




# Example SQL Queries

## The SELECT Statement

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
`recipie_name` is "Tomato Soup") and the filter for 
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


## The JOIN statement

Having to perform multiple quieries on
multiple tables is quite cumbersome,
and would require unnecessary additional
logic outside of the database.

Instead, if we wanted to directly get the
`ingredient_id` knowing the `recipie_name`,
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
when the recipie_name is "Tomato Soup":

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

## Tripple Joins of SQL Tables

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
to find all the recipies that include tomatoes
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

# Advanced SQL queries

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




# Advanced SQL: 

How do we actually build these tables:

* Indicies
* Terradata/Vertica for distributed databases.
* HIVE/Pig to run parallel Map/Reduce queries.

# Wrapup 

Thats about it. If you can get get you mind around the idea of table normalization, as well as
as the benefits of joining tables together to produce more complicated queries, everything
else should be easy.

# Relevant links

This was of course just a whirlwind tour of what's so great
about SQL databases. There is tons of topics left to explore:

* [MySQL](http://www.mysql.com/) is a popular SQL implementation.
* [Sequel Pro](http://www.sequelpro.com) is a great MySQL client.

If you liked this post you can follow me on Twitter [@joshualande](http://twitter.com/joshualande).


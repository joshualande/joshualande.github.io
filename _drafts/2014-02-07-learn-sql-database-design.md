---
layout: post
title: Learn SQL With Simple Examples (Part 1) - Robust Database Design
comments: true
---

In this post, I will go over the benefits of laying out data in a
relatinoal database like [SQL](http://en.wikipedia.org/wiki/SQL)
compared to other data formats.

By way of a simple example, I will then go over the basics of how
to design a robust database and the concepts of [database
normalization](http://en.wikipedia.org/wiki/Database_normalization).
Finally, I will go over the commands required to set up this
database in MySQL.


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

{% include twitter_plug.html %}

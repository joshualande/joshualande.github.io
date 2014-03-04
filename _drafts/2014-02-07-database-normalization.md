---
layout: post
title: "Design a Better Database With Database Normalization "
comments: true
---

In this post, I will go over the benefits of laying out data in a
relational database like [SQL](http://en.wikipedia.org/wiki/SQL).
By way of a simple example, I will then go over the basics of how
to design a robust database and the concept of [database
normalization](http://en.wikipedia.org/wiki/Database_normalization).
Finally, I will go over the commands required to set up this database
in MySQL.

In the next post in this series, I will discuss the commands
required to query information from a database.

# A Simple Example SQL Database

We begin with a simple example.  Suppose that we wanted to store
information about recipes in a cookbook.  Each recipe will have a
list of ingredients.

The fundamental building block of SQL databases are two-dimensional
tables. This may seem like a limited design, but as you will
see, this limitation will become incredibly powerful.

For our example,
in order to store our recipes into the database we can first create
a table listing recipe names.  
Our `recipes` table looks like:

| recipe_id |    recipe_name |
| --------- | -------------- |
|         0 |          Tacos |
|         1 |    Tomato Soup |
|         2 | Grilled Cheese |

We associate a unique id with each recipe so that we can connect
rows in this table to rows in other tables.

Next, we need a table listing all the ingredients.
To make later examples more interesting,
we will also also assume that each ingreident
has a price. For our `ingredients` table looks like:

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

Although it might seem cumbersome, a straightfowrd way to store
this information is to make a list of all (`recipe_id`, `ingredient_id`) pairs.

To make the later examples more intersting, we will also
list a quanitity of each ingredient required in each recipie.
Our `recipe_ingredients` table looks like:

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
a real database, like the one curated by [yummly.com](http://yummly.com),
would have a lot more information in it.  But this example should
be sufficient to convey the major concepts of database design.

Before we move on, I will mention two bits of terminology common
to databases.  A
[schema](http://en.wikipedia.org/wiki/Database_schema) is the
structure of the tables in the database. So for our example, 
the three tables are the scheme of our recipies.

A [query](http://en.wikipedia.org/wiki/SQL#Queries) in SQL is a command
which retrivies data from a database. For example, we might
want to query the number of recipies in our database.

# Database Normalization

At this point, I would expect you to be confused about why we laid
out the database across three tables. 
Another question is why we need all these 
IDs when we already have names.
And finally, you might be wondering
why each recipe-ingredient
pair takes up its own row in the `recipe_ingredients` table.

This leads us to the concept of [database
normalization](http://en.wikipedia.org/wiki/Database_normalization).
Our database design above is normalized to avoid
having duplicate data.
stored in the database.

If we stored recipe and ingredient names
directly in the `recipe_ingredients` table,
we would end up with rows like

| recipe_name | ingredient_name | amount |
| ----------- | --------------- | ------ |
|       Tacos |            Beef |      1 |
|       Tacos |         Lettuce |      2 |

At some point in the future, somebody might
decide that they wanted the Tacos to have Chicken
instead of beef. In the process of updating
the table, they might accidently
change the name of the recipe.

| recipe_name | ingredient_name | amount |
| ----------- | --------------- | ------ |
|        Taco |         Chicken |      1 |
|       Tacos |         Lettuce |      2 |

when they set `recipe_name` to 'Taco', or ' taco', or 'tacos'.
This row would then become inconsistent with
the other rows in the table, which could break future SQL 
queries.

Finding and fixing these sorts of bugs is costly, time consuming,
and frustrating. By instead using recpie and ingredient IDs, it
should be totally unambigious and much less error prone to update
the tables.

In addition, database normalization has the added benefit that
updating the name of a recipe requires only changing
one thing in one place. 
You could imagine, for example, trying to change the name
'Tacos' to 'Chicken Tacos' everywhere in the table
only to have your code crash after updating half the rows.
To avoid this, we store the name in only one place in 
the `recipes` table to avoid this problem.

Finally, another another benefit of this modular table design 
is that it scales nicely to adding additonal information.
Suppose we wanted to store information about the steps needed to
build the recipe. We could just create a new table called
`recipe_instructions` which linked back to the recipes by their ID:

| step_id | recipe_id | step_number |                 step_description |
| ------- | --------- | ----------- | -------------------------------- |
|       0 |         0 |           0 |  Put the Taco Shells in the oven |
|       1 |         0 |           1 | Cook the beef in a pan on medium |

Being able to add new data easily without altering the orignal
tables is very useful because it is less error prone and doesn't
require modifying any existing tables or existing code that queries
the tables.

Finally, if the extra data only existed for a limited number of
rows, this would be another win becasue we wouldn't have to store
`NULL` values for all the recipies missing the additional data.

# Why Do Tables Have to be Flat?

The limitation that SQL tables have to be flat may at first very
seem strange.  Although it should be clear at this point that there
is most likley some way of flattening out any data into some schema
design using flat tables, it is probably unclear why this is better
than allowing nested data structures.

If you are used to programming in scripting langauges like
[ruby](https://www.ruby-lang.org/), [python](http://www.python.org/),
or [perl](http://www.perl.org/), or if you are used to dealing with
[JSON](http://json.org/)-formatted data, you may be
very used to nested data.

A common design you might expected in these langauges would
be to store our recipes as a 
dictionary of sets. In python, it would look like:

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

But the downside of laying out the data in this way is that it
makes this query easy at the expense of making other kinds
of queries hard. 
For example, to find all the recipies with a given ingredient
would require innefficiently looking through all recipies:

```python
recipies = [ i for i in recipies.keys() if "Tomaotes" in recipies[i]]
```

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
    "Milk": ("Tomato Soup")
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

As a final note, some newer databases like
[MongoDB](http://www.mongodb.com/) nativly support nested
[JSON](http://json.org/)-like data structures.  Despite the caveats
mentioned above, in real-world situations it is sometimes significantly
more convenient to store data in this structure.

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

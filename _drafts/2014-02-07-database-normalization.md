---
layout: post
title: "Design a Better Database With Database Normalization"
comments: true
---

In this post, I will go over the benefits of laying out data in a
relational database like [SQL](http://en.wikipedia.org/wiki/SQL).
By way of a simple example, I will then go over the basics of how
to design a robust database and the concept of [database
normalization](http://en.wikipedia.org/wiki/Database_normalization).
Finally, I will go over the actual commands required to set up a
simple database in MySQL.

<!--
In the next post in this series, I will discuss the commands
required to query information from a database.
-->

# A Simple Example SQL Database

We begin with a simple example.  Suppose that we wanted to store
information about recipes in a cookbook.  

The fundamental building block of SQL databases are two-dimensional
tables. This may seem like a limited design, but as you will
see, this limitation will become incredibly powerful.

For our cookbook, each recipe will have a list of ingredients.  To
store our recipes, we can first create a table of recipe names.
Our `recipes` table is:

| recipe_id |    recipe_name |
| --------- | -------------- |
|         0 |          Tacos |
|         1 |    Tomato Soup |
|         2 | Grilled Cheese |

We associate a unique id with each recipe so that we can connect
rows in this table to rows in other tables.

Next, we need a table listing all the ingredients.
To make later examples more interesting,
we will also also assume that each ingredient
has a price. Our `ingredients` table is:

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

Although it might seem cumbersome at first, a straightforwd way to store
this information is to make another table listing all (`recipe_id`, `ingredient_id`) pairs.

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
to databases.  A [schema](http://en.wikipedia.org/wiki/Database_schema)
is the structure of the tables in the database. For our example,
our three tables make up the scheme of our recipies database.

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
pair has to take up an entire row in the `recipe_ingredients` table.

This leads us to the concept of [database
normalization](http://en.wikipedia.org/wiki/Database_normalization).
Our database design above is normalized to avoid
having duplicate data stored in the database.

If we stored recipe and ingredient names
directly in the `recipe_ingredients` table,
we would end up with rows like

| recipe_name | ingredient_name | amount |
| ----------- | --------------- | ------ |
|       Tacos |            Beef |      1 |
|       Tacos |         Lettuce |      2 |

At some point in the future, somebody might
decide that they wanted the Tacos to be made
of Chicken instead of beef. In the process of updating
the table, they might accidently
change the name Tacos to Taco:

| recipe_name | ingredient_name | amount |
| ----------- | --------------- | ------ |
|        Taco |         Chicken |      1 |
|       Tacos |         Lettuce |      2 |

A this point, our table is inconsistent and
many downstream queries might break.
Similar problems could happen if we
tried to rename the recipie from 'Tacos' to 'Chicken Tacos'
but forgot to update the name in ever row.

Finding and fixing these sorts of bugs is costly, time consuming,
and frustrating. By storing the name in only one table
and using a presumably-never-changing recipie ID in ever
other location, we avoid any possibility of several costly errors.

Another another benefit of this modular table design 
is that it scales nicely to adding additonal information.
Suppose we wanted to store information about the steps needed to
build the recipe. We could just create a new table called
`recipe_instructions` which linked back to the recipes by their ID:

| step_id | recipe_id | step_number |                 step_description |
| ------- | --------- | ----------- | -------------------------------- |
|       0 |         0 |           0 |  Put the Taco Shells in the oven |
|       1 |         0 |           1 | Cook the beef in a pan on medium |

Being able to add new data easily without altering the original
tables is useful because it is less error prone and allows for
rapidly adding new features without altering production tables/code.

Finally, if the extra data only existed for a limited number of
rows, this would be another win becasue we wouldn't have to store
additional `NULL` rows for all the missing data.

# Why Do Tables Have to be Flat?

The limitation that SQL tables have to be flat may at first seem very
seem strange.  Although it should be clear that there
is probably some way way to flattening out any data structure
to fit into flat tables, it is probably unclear why this is better
than allowing nested data structures.

Scripting langauges like [ruby](https://www.ruby-lang.org/),
[python](http://www.python.org/), or [perl](http://www.perl.org/)
make it especially easy to have data structures with very nested
data. In python, a common design for our recipies data
might look like:

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

But the downside of this is that it would be expensive to run other
queries on our data For example, finding all the recipies with a
given ingredient would require looking through each recipie:

```python
recipies = [ i for i in recipies.keys() if "Tomaotes" in recipies[i]]
```

We could build a different dat structure to make this other query easier:

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

But this structure would make the first query more difficult!

Although it seems strange at first, flattening out the tables ensures
that our data is not biased towards (or againts) any kinds of query.
Instead, the designers of SQL optimized it to perform all kinds of
different queries efficienty assuming that all the tables are flat.
There is a good chance that what will want to querying tomorrow is
different from today, so this is a huge advantage.

As a final note, some newer databases like
[MongoDB](http://www.mongodb.com/) nativly support storing nested
[JSON](http://json.org/)-like data structures in databases.  Despite
the caveats mentioned above, for pragmatic reasons sometimes in
real-world situations it is more convenient to store data inside
in nested structures.

# Setting up SQL On Your local Machine

Before getting further, I would recommend setting up SQL on your
local computer. My examples will assume you are using the free and
open-source [MySQL](http://www.mysql.com/) database program.  Here
is some documentation for installing MySQL on
[Windows](https://dev.mysql.com/doc/refman/5.0/en/windows-installation.html)
or 
[Mac](https://dev.mysql.com/doc/refman/5.0/en/macosx-installation.html).

If you are using an Apple computer, I recommend using the free and
open-source graphical program [Sequel Pro](http://www.sequelpro.com/)
to test out the MySQL commands in this example.

# Creating the Recipies Database in SQL

To wrap up this post, I will
go through the commands required to create
the recipipes database described above.

First, we have to create a database to work in

```sql
CREATE DATABASE recipes_database
```

Next, we have to enter the database

```sql
USE recipes_database
```

To create the recipes table, we can use the `CREATE TABLE` command:

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

The purpose of the `PRIMARY KEY` is to avoid the possiblity of
having duplicate rows in the table. The `PRIMARY KEY` forces every
row to have a different value.  In addition, we use the `UNIQUE`
keyword to ensure that no two rows have the same recipe name.

Similarly, we can create the ingredients table:

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

Here, our `PRIMARY KEY` is for `(recipe_id,ingredient_id)` pairs
because two rows should not have both the same recipie and ingreident.

One last thing to mention is that there is an `AUTO_INCREMENT` command
which can be used to let SQL automatically
pick the `recipe_id` and `ingredient_id`
values to ensure uniqueness.
[Here](http://dev.mysql.com/doc/refman/5.0/en/example-auto-increment.html)
is the documentation on this command in MySQL.

<!--
# Next Time: Querying the Database

In the next post, I will discuss the commands required to ask
very sophistical questions about data in this database.
-->

{% include twitter_plug.html %}

---
layout: post
title: "Design a Better SQL Database With Database Normalization"
comments: true
permalink: database-normalization
---

*This is the first post in a [series of posts]({% post_url 2014-04-17-data-science-sql %})
about doing data science with SQL.*

In this post, I will go over the benefits of laying out data in a
relational database like [SQL](http://en.wikipedia.org/wiki/SQL).
By way of a simple example, I will then go over the basics of how
to design a robust database and the concept of [database
normalization](http://en.wikipedia.org/wiki/Database_normalization).  These
topics are essential in being able to design and interact effectively
with databases.

## A Simple Example Recipes Database

We will introduce these topics through a simple example.  Suppose
that we wanted to store information about recipes in a cookbook.

The fundamental building block of SQL databases are two-dimensional
tables. This may seem like a limited design. But as you will
see, this limitation will become incredibly powerful.

For our cookbook, each recipe will have a name. So we can begin by
creating a table of recipe names.  Our `recipes` table will be:

| recipe_id |    recipe_name |
| --------- | -------------- |
|         0 |          Tacos |
|         1 |    Tomato Soup |
|         2 | Grilled Cheese |

We associate a unique ID with each recipe so that we can connect
rows in this table to rows in other tables (more on this soon).

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

  
Again, the ingredient IDs are to connect rows in this table to other
tables.  Finally, we need some way of listing what ingredients are
in each recipe.  Although we might naturally want to put this
information into the `recipe` table, it is advantageous to use a third
table to store this information.

Although it might seem cumbersome at first, a straightforward way
to store this information is to make another table listing all
(`recipe_id`, `ingredient_id`) pairs. I will discuss
in the next section why this is advantageous.

To make the later examples more interesting, we will also
list an amount of each ingredient required in each recipe.
Our `recipe_ingredients` table is:

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
would have a lot more information (more tables, more columns, and
more rows).  But this example should be sufficient to allow for
interesting examples later.

Before we move on, I will mention two bits of terminology common
to databases.  A [schema](http://en.wikipedia.org/wiki/Database_schema)
is the structure of the tables in the database. For our example,
our three tables make up the scheme of our recipes database.

A [query](http://en.wikipedia.org/wiki/SQL#Queries) in SQL is a command
which retrieves data from a database. For example, we might
want to query the number of recipes in our database.

## How to Normalize Your Database

At this point, you should be confused about why we laid out the
database across three tables and why we need all these strange IDs.
And finally, you might be wondering why each recipe-ingredient pair
has to take up an entire row in the `recipe_ingredients` table.

This leads us to the concept of [database
normalization](http://en.wikipedia.org/wiki/Database_normalization).
Database normalization is the process of designing a database so
that every piece of information shows up in only one place in the
database.  This is the most important concept for designing effective
databases.

In principle, we could directly cram all
three tables into one larger table:

| recipe_name | ingredient_name | amount | price |
| ----------- | --------------- | ------ | ----- |
|       Tacos |            Beef |      1 |     5 |
|       Tacos |         Lettuce |      2 |     1 |
|         ... |             ... |    ... |   ... |

This would be a denormalized table. 

But although it is easier to read, it is very fragile.  For example,
at some point in the future somebody might decide that they wanted
the tacos to be made of Chicken instead of beef. In the process of
updating the table, they might accidentally change the name `Tacos`
to `Taco` in only one of the rows:

| recipe_name | ingredient_name | amount | price |
| ----------- | --------------- | ------ | ----- |
|        Taco |            Beef |      1 |     5 |
|       Tacos |         Lettuce |      2 |     1 |
|         ... |             ... |    ... |   ... |

A this point, our table would no longer be self consistent and this
could break all sorts of downstream queries.  Similar problems could
happen if our code crashed after updating half of the
rows. In addition, this schema also allows for the same ingredient to have
different prices in different recipes which shouldn't be possible.

Finding and fixing these sorts of bugs is costly, time consuming,
and frustrating. The benefit of having small tables linked by IDs
is that the IDs can be assumed to never change since they are only
used inside the database. On the other hand, important information
like names only show up in one place, avoiding inconsistencies and
allowing for easier and less-error-prone modifications.

Another benefit of database normalization is that it scales nicely
to adding new kinds of information. For example, suppose we wanted
to store additional information about the steps needed to build the
recipe.  We could just create a new table which linked back to the
recipes by their ID. The `recipe_instructions` table might look
like:

| recipe_id | step |                 step_description |
| --------- | ---- | -------------------------------- |
|         0 |    0 |  Put the Taco Shells in the oven |
|         0 |    1 | Cook the beef in a pan on medium |

Being able to add new data without changing the existing
tables is much less error prone and potentially costly.

Finally, if the extra data only existed for a limited number of
rows, this would be another win because we wouldn't have to store
empty rows for all the missing data.

## Why Do Tables Have to be Flat?

The limitation that SQL tables have to be flat may at first seem
very strange.  Although it should be clear that there is probably
some way to flattening out any data structure to fit into flat
tables, it is probably unclear why this is better than just allowing
nested data.

Scripting languages like [ruby](https://www.ruby-lang.org/),
[python](http://www.python.org/), or [perl](http://www.perl.org/)
make it especially easy to nest data using dictionaries.  In python,
a common design for our recipes data might look like:

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
soup_ingredients = recipes["Tomato Soup"]
```

But the downside of this is that it would be expensive to ask other
questions about the data. For example, finding all the recipes with
a given ingredient would require looking through each recipe:

```python
tomato_recipes = [i for i in recipes.keys() \
            if "Tomatoes" in recipes[i]]
```

We could build a different data structure to make this other query
easier:

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
that our data layout is not biased towards (or against) any particular
kind of query.  Instead, the designers of SQL optimized their
database to perform all kinds of different queries against flat
tables efficiently.  There is a good chance that what we will want
to query tomorrow is different from today, so this is a huge
advantage.

As a final note, some newer databases like
[MongoDB](http://www.mongodb.com/) natively support storing nested
[JSON](http://json.org/)-like data structures.  Despite the caveats
mentioned above, in real-world situations sometimes this is a
necessity.

*In the [next post]({% post_url 2014-04-25-install-mysql %}) 
in this [series of posts]({% post_url 2014-04-17-data-science-sql %}), 
I will explain how to install MySQL on your local machine to test out 
SQL commands.*

{% include twitter_plug.html %}

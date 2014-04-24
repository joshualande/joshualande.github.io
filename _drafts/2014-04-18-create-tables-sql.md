---
layout: post
title: "How to Create Tables and Insert Data into SQL Databases"
comments: true
permalink: create-tables-sql
---

*This is the second post in a [series of posts]({% post_url 2014-04-17-data-science-sql %})
about doing data science with SQL.

I will then go over the
commands required to set up the example recpie database from the
[previous post]({% post_url 2014-04-18-database-normalization %}).
By the end of this post, you will have a working database!



XXX Other GUI Sequel Programs...

## Creating the Recipies Database in SQL

To create the recipies database, we need
to first create a database to work in:

To run SQL commands, you can either connect to SQL
through the command line with the command



```sql
CREATE DATABASE recipes_database
```

Having multiple databases is useful to separate
tables in different projects.

Next, we have to select this particular database

```sql
USE recipes_database
```

To create the recipes table, we can use the `CREATE TABLE` command:

```sql
CREATE TABLE recipes (
  recipe_id int(11) NOT NULL,
  recipe_name varchar(30) NOT NULL,
  PRIMARY KEY (recipe_id),
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
  amount int(11) NOT NULL,
  PRIMARY KEY (recipe_id,ingredient_id)
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

# Browse the Tables

To view the table
```sql
SELECT * FROM recipe_ingredients
```

![Sequel Pro Content Tab](/assets/sequel_pro_content_tab.jpg)


<!--
## Next Time: Querying the Database

In the next post, I will discuss the commands required to ask
very sophistical questions about data in this database.
-->

{% include twitter_plug.html %}

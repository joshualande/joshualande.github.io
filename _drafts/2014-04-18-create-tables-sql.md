---
layout: post
title: "How to Create Tables and Insert Data into SQL Databases"
comments: true
permalink: create-tables-sql
---

*This is the third post in a [series of posts]({% post_url 2014-04-17-data-science-sql %})
about doing data science with SQL. The 
[previous post]({% post_url 2014-04-25-install-mysql %})
covered the steps required to install MySQL on on your
local machine.*

In this post, I will 
go over the commands required to set up the example recpie database from the
[first post]({% post_url 2014-04-18-database-normalization %}) in
this series.  By the end of this post, you will have a working
database to run SQL queries on!

## Creating the Recipies Database in SQL

To create the recipies database, we need
to first create a database to work in.
To do that, we run the SQL command:

```sql
CREATE DATABASE recipes_database;
```

Having multiple databases is useful to separate
tables in different projects. 

But for now, we
will always use this particular database.
We select this database with the command:

```sql
USE recipes_database;
```

# Creating Tables in SQL

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
    (1,"Tacos"),
    (2,"Tomato Soup"),
    (3,"Grilled Cheese");
```

The `PRIMARY KEY` forces every row to a unique value, stoppign the
recipe from having multiple rows with the same recipe_id.  In
addition, we use the `UNIQUE` keyword to ensure that no two rows
have the same recipe name.

To create the ingredients table:

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
    (1, "Beef", 5),
    (2, "Lettuce", 1),
    (3, "Tomatoes", 2),
    (4, "Taco Shell", 2),
    (5, "Cheese", 3),
    (6, "Milk", 1),
    (7, "Bread", 2);
```

Similar to before, we ensure that ingredient names are unique to
avoid ambiguity. But we do not ensure uniqueness of `ingredient_price`
since multiple recipes could have the same price.

Finally, to create the recipe-ingredient-mapping table:

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
    (1,1,1),
    (1,2,2),
    (1,3,2),
    (1,4,3),
    (1,5,1),
    (2,3,2),
    (2,6,1),
    (3,5,1),
    (3,7,2);
```

The last row says, for example, that the recipe with an ID of 3
("Grilled Cheese") has an amount 2 of the ingredient with ID 7
("Bread").

Here, our `PRIMARY KEY` is for `(recipe_id,ingredient_id)` pairs.
This means that the combination of `recipe_id` and `ingredient_id`
must be unique. This is required because each recipie can only have
an ingredient once.

One final thing to mention is that there is an `AUTO_INCREMENT`
command which can be used to let SQL automatically pick the `recipe_id`
and `ingredient_id` values to ensure uniqueness.
[Here](http://dev.mysql.com/doc/refman/5.0/en/example-auto-increment.html)
is the documentation on this command in MySQL.

Using this command, we could have created the same `recipes`
table as above in a less error-prone way:

```sql
CREATE TABLE recipes (
  recipe_id int(11) NOT NULL AUTO_INCREMENT,
  recipe_name varchar(30) NOT NULL,
  PRIMARY KEY (recipe_id),
  UNIQUE (recipe_name)
);

INSERT INTO recipes 
    (recipe_name) 
VALUES 
    ("Tacos"),
    ("Tomato Soup"),
    ("Grilled Cheese");
```

## Browse the Tables

If we want to browse tables in the
database, we can use the `SELECT`
command.

```sql
  SELECT * 
    FROM recipes
ORDER BY recipe_id;
```

From the command line, this query will will return

```
+-----------+----------------+
| recipe_id | recipe_name    |
+-----------+----------------+
|         1 | Tacos          |
|         2 | Tomato Soup    |
|         3 | Grilled Cheese |
+-----------+----------------+
```

We can also use the content table in Sequel Pro
to graphically browse the tables:

![Sequel Pro Content Tab](/assets/sequel_pro_content_tab.jpg)


*In the [next post](...) in this series, I will
go over the basics of quering for data in the database.*

{% include twitter_plug.html %}

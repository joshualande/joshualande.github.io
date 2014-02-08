---
layout: page
title: An Introduction to SQL for Data Scientists
comments: true
---

During my Physics PhD program, I really never understood the purpose of SQL and
similar [relational database management systems 
(RDBMs)](http://en.wikipedia.org/wiki/Relational_database_management_system).
And so as I prepared to transition to a career as a data scientist, I was
confused why so much data science was done inside of databases.

After all, physicists (and scientists more broadly) love to reinvent the wheel. 
Working in astrophysics, I was used to all sorts of file formats like
[ROOT Trees](http://en.wikipedia.org/wiki/ROOT), 
[Fits Files](http://en.wikipedia.org/wiki/FITS),
[HDF5 Files](http://en.wikipedia.org/wiki/Hierarchical_Data_Format),
and [pandas DataFrames](http://en.wikipedia.org/wiki/Pandas_(software). 
I wans't sure why I needed to learn SQL.

And besides, SQL seemed so limited. Everything had to be these flat tables, and the syntax
to deal with theme seemes really archane. I didn't get it.

Now that I have been a data scientist for half a year, I have grown to really love databases.
What they are designed to do well they do expetionally better than anything else.
And the use case for them in business and data science applications is almost limitless.

Now that I am on the other side of the table 
helping other academics transition to become become data scientists, I see that
most of them have very similar confusion about the purpose and benefit of these
databases. So my hope in this post is do describe
the benefits of an SQL database, go over the basic synax of SQL,

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
|             1 |        Tomatoes |                2 |
|             1 |      Taco Shell |                2 |
|             1 |          Cheese |                3 |
|             1 |            Milk |                1 |
|             1 |           Bread |                2 |

  
Finally, we need some way of listing all of the ingredients in each recipe.
Although we might naturally want to put this information into the recipe table,
we will see that it is advantageous to use a third table to store this information.
For every recipe 

| recipe_id | ingredient_id | amount |
| --------- | ------------- | ------ |
|         0 |             0 |      . |
|         0 |             1 |      . |
|         0 |             2 |      . |
|         0 |             3 |      . |

Of course, we could imagine that a real database, like the one
curated by [yummly.com](http://yummly.com), would have lots more information in it.
You might want more accurate list of steps involved in preparing the recipie.
But this simplified exmaple should get the point across.

## Similar python Implementation

Comming from a python background, my natural tendency is to feel that this
is totally overengineering the problem. Instead, I would imagine using
a nested dictionary to define this data:

```python
recipies = {
  "Tacos": { ingredients: ["Beef", "Lettuce", "Tomatoes", 
                           "Cheese", "Taco Shell" ], 
             description: "Mom's famous Tacos" },
  "Tomato Soup": { ingredients: [ "Tomatoes", "Milk" ], 
                   description: "Homemade Tomato soup" },
  "Grilled Cheese": { ingredients: [ "Cheese", "Bread" ],
                      description: "Delicious Cheese Sandwich" }
```
That way, if I wanted to find, for example, all of the ingredients associated
with Tomato Soup, I could use the code:

```python
ingredients = recipies["Tomato Soup"]["ingredients"]
```

Although this seems works well, there are several issues associated with this approach.
The first is that organizing our data in this particular way makes
other queries about the data particularly hard.
For example, it is much more cumbersome to get a list of all recipies 
which contain tomatoes. You could imagine a similar data structure optimized for that 
kind of query:
```python
recipies = {
    "Beef": [ "Tacos" ],
    "Lettuce": [ "Tacos" ],
    "Tomatoes": [ "Tacos", "Tomato Soup"],
    "Taco Shell": [ "Tacos"],
    "Cheese": [ "Tacos", "Grilled Cheese"]
    "Milk": [ "Tomato Soup" ]
    "Bread": [ "Grilled Cheese" ]
}
```
But this data structure makes it cumbersome to find what ingredients are
required to make a paritcular recipie.
This may seem like a contrived example, but in real life it is very often
the case that that you later on have to query your data in ways you did not
originally intent.

As you will see, SQL solves this problem by dividing up our data into multiple tables.

The second issue with the approach listed above is that there are a lot of duplciate data
which shows up in mulitple places. 




## Example SQL Queries

Given our recipe schema above, there are many kind of calculations we could imagine doing
on this databse.

```sql
SELECT ingredient_name 
FROM 
```

Other things you might want:
* A list of users in the system.
* A mapping of what users have eaten what recipies...
* FIT DATA INSIDE MEMORY.
 
To get a list of all ingredients for the taco recipe by ingredient name:
Some SQL queries
* How 

# Advanced SQL: 

* Indicies
* Terradata/Vertica for distributed databases.
* HIVE/Pig to run parallel Map/Reduce queries.

# Wrapup 

Thats about it. If you can get get you mind around the idea of table normalization, as well as
as the benefits of joining tables together to produce more complicated queries, everything
else should be easy.

If you liked this post you can follow me on Twitter [@joshualande](http://twitter.com/joshualande).

# Relevant links

* 
and finally to work out a concrete exmaple which should get at the purpose of

# The Need for Relational Database

Common use cases in business:
* Data sets which down fit into

Denormalized database.

I was used to 

---
layout: page
title: An Introduction to SQL for Academics
comments: true
---

During my Physics PhD program, I really never understood the purpose of SQL and
similar [relational database management systems 
(RDBMs)](http://en.wikipedia.org/wiki/Relational_database_management_system).
And so as I began to prepare to transition to a career as a data scientist, I was
confused why so much data science was done inside of databases.

After all, physicists (and scientists more broadly) love to reinvent the wheel. 
Working in astrophysics,
I was used to all sorts of file formats like
[ROOT Trees](http://en.wikipedia.org/wiki/ROOT), 
[Fits Files](http://en.wikipedia.org/wiki/FITS),
[HDF5 Files](http://en.wikipedia.org/wiki/Hierarchical_Data_Format),
and [pandas DataFrames](http://en.wikipedia.org/wiki/Pandas_(software)),

What was the point of SQL databases? everything had to be these flat tables, and the syntax
to deal with theme seemes really archane!

Now that I have been a data scientist for half a year, I have grown to really love databases.
What they are designed to do well they do expetionally better than anything else.
And now that I am on the other side of the table 
helping other academics transition to become become data scientists, I see that
most of them have very similar confusion about the purpose and benefit of databases.

# The Need for Relational Database

Common use cases in business:
* Data sets which down fit into

Denormalized database.

I was used to 

# A Simple Example Of a Database

As a simple example of a relational database, supposed 
we want to describe a schema for a recepie book.

The first thing we would like to do is describe a
list of all the recipies.

| recipe_id |    recipe_name |        recipe_description |
| --------- | -------------- | ------------------------- |
|         0 |          Tacos |        Mom's famous Tacos |
|         1 |   Tomatoe Soup |      Homemade Tomato soup |
|         2 | Grilled Cheese | Delicious Cheese Sandwich |

But each recipie is built out of ingredients.

| ingredient_id | ingredient_name | ingredient_price |
| ------------- | --------------- | ---------------- |
|             0 |            Beef |                5 |
|             1 |         Lettuce |                1 |
|             1 |        Tomatoes |                2 |
|             1 |      Taco Shell |                2 |
|             1 |          Cheese |                3 |
|             1 |            Milk |                1 |
|             1 |           Bread |                2 |

  
Findally, we chould create a table to describe all of the ingredients in all of the tables:

| recipe_id | ingredient_id |
| --------- | ------------- |
|         0 |             0 |
|         0 |             1 |
|         0 |             2 |
|         0 |             3 |

Comming from a python background, my natural tendency is to believe that this
is totally overengineering the problem. Instead, I would imagine using
a hashtable to define this data:

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
But there are several downsides of this approach.

The way to read

Other things you might want:
* A list of users in the system.
* A mapping of what users have eaten what recipies...

To get a list of all ingredients for the taco recipie by ingredient name:

```sql
SELECT ingredient_name 
FROM 
```
 
Some SQL queries
* How 

# Advanced SQL: 

* Indicies
* Terradata/Vertica for distributed databases.
* HIVE/Pig to run parallel Map/Reduce queries.

If you liked this post you can follow me on Twitter [@joshualande](http://twitter.com/joshualande).

# Relevant links

* 

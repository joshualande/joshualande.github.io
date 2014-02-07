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

Create a table of all recepies
| recipe_id | recipe_name | recipe_description |
|         0 |       tacos | Mom's famous Tacos |

| ingredient_id | ingredient_name | ingredient_price |
|             0 |            beef |                5 |
|             1 |         Lettuce |                1 |
|             1 |        Tomatoes |                2 |
|             1 |          Cheese |                3 |
|             1 |            Milk |                1 |
|             1 |           Bread |                2 |

  
A table of ingredients in recepies:
| recipe_id | ingredient_id |
|       ... |           ... |

Other things you might want:
* A list of users in the system.
* A mapping of what users have eaten what recipies...
 
Some SQL queries
* How 

# Advanced SQL: 

* Indicies
* Terradata/Vertica for distributed databases.
* HIVE/Pig to run parallel Map/Reduce queries.

If you liked this post you can follow me on Twitter [@joshualande](http://twitter.com/joshualande).

# Relevant links

* 

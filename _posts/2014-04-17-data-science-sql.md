---
layout: post
title: What Every Data Scientist Needs to Know about SQL
comments: true
permalink: data-science-sql
---

One of the most important tools required to be a successful data
scientist is relational databases like
[SQL](http://en.wikipedia.org/wiki/SQL).  The majority of data
stored by businesses is in these relational databases. And in
addition, they are exceptionally good at storing complicated business
data sets as well as allowing for efficient information retrieval.
So having a strong understanding of relational databases is essential
to being an effective data scientist.

In this series of posts, I will provide a broad overview of the key
topics required to successfully work with databases to do effective
data science. My hope is to take a breath-first search approach to
teaching by introducing the most important topics first.  Whenever
possible, I will attempt to avoid theory in favor of practicality
with the goal of getting you up to speed as quickly as possible.

In these posts, I will focus on the [MySQL](http://www.mysql.com/)
database because it is popular, free, open source, and easy to get started
with. But the concepts and most of the commands are easily transferable
to other databases.

The posts in this series are:

1. **[Design a Better Database With Database Normalization]({% post_url 2014-04-18-database-normalization %})**

   In this post, we will go over the design decisions behind
   databases, how they are different from other programming languages
   and data formats, and how to lay out our data so that it will
   fit into a SQL database.  By way of a simple example, we will
   go over the basics of database normalization, the best-practice
   for correctly storing complex data in a database.  Despite being
   a bit abstract and theoretical, these topics are absolutely
   essential to understanding how databases work and why they are
   so good at what they do. So this is a must read.

2. **[How to Install MySQL On Your Local Machine]({% post_url 2014-04-25-install-mysql %})**

   In this post, we will describe how to install MySQL on our local
   machine. This will allow you to test out SQL by running SQL
   queries against a test database. By the end of this post,
   you will have MySQL running on your local machine.

3. **[How to Create Tables and Insert Data into SQL Databases]({% post_url 2014-04-28-create-tables-sql %})**

   In this post, we will go over the SQL commands required to create
   tables in MySQL and insert data into them.  I will go over the
   specific commands required to create the example database from
   the [first post]({% post_url 2014-04-18-database-normalization %}) 
   of the series. By the end of this post, you will have an
   example database on you computer to run SQL queries against.

4. **["Filters, Joins, Aggregations, and All That: A Guide to Querying in SQL"]({% post_url 2014-08-14-filters-joins-aggregations %})**

   In this post, we will go over the basics of querying data from
   a database. Using the example from above, I will work through
   successively harder queries show how the simple operations can
   be combined write complicated queries.

<!--
3. **"Indexing in SQL for the Rest of Us"**

4. **"How to Handle Missing Data in SQL Using NULL Values"**

-->


## Further SQL Reading

* SQL databases are designed to deal with `NULL` values
  in a sensible way. [Here](http://dev.mysql.com/doc/refman/5.0/en/working-with-null.html)
  is some documentation about the way MySQL handles `NULL` values.
* Views in
  SQL act as temporary tables able to simplify queries 
  as well as abstract a user from the underlying table implementations. 
  [Here](http://dev.mysql.com/doc/refman/5.0/en/create-view.html) 
  is some documentation about how they are handled by MySQL.
* Beyond [MySQL](http://www.mysql.com/), there are several
  high-performance parallel databases designed to deal with large data sets.
  Two popular ones are [Terradata](http://www.teradata.com/) and
  [HP Vertica](http://www.vertica.com/).
* For very large data sets, [hadoop](http://hadoop.apache.org/),
  the [Hadoop Distributed File System (HDFS)](http://hadoop.apache.org/docs/r1.2.1/hdfs_design.html),
  and [MapReduce](https://hadoop.apache.org/docs/r1.2.1/mapred_tutorial.html) 
  are typically used to store and analyze these large data sets. 
  [Apache Hive](http://hive.apache.org/) is an implementation of
  SQL on top of MapReduce which brings the power of SQL to hadoop.
  [Apache Pig](https://pig.apache.org/) and [Scalding](https://github.com/twitter/scalding)
  are similar competitors.
* Bill Howe's coursera class 
  [Introduction to Data Science](https://www.coursera.org/course/datasci)
  has a good discussion of SQL query optimizers in his lectures on
  "[Relational Databases, Relational Algebra](https://class.coursera.org/datasci-001/lecture/preview)".

{% include twitter_plug.html %}

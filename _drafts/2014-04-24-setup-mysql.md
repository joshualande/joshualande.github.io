---
layout: post
title: "How to Setup MySQL"
comments: true
permalink: setup-mysql
---

*This is the second post in a [series of posts]({% post_url 2014-04-17-data-science-sql %})
about doing data science with SQL. The [previous post]({% post_url 2014-04-18-database-normalization %})
described the topic of database normalization and good database design.*

In this post, I will describe how to setup [MySQL](http://www.mysql.com/)
on your local machine.  MySQL is great because it is popular, open
source, and easy to get started with.  I will then go over the
commands required to set up the example recpie database from the
[previous post]({% post_url 2014-04-18-database-normalization %}).
By the end of this post, you will have a working database!

## Setting up SQL On Your local Machine

Here is some documentation for installing MySQL on
[Windows](https://dev.mysql.com/doc/refman/5.0/en/windows-installation.html)
or 
[Mac](https://dev.mysql.com/doc/refman/5.0/en/macosx-installation.html).
In particular, you can get the installer
[here](http://dev.mysql.com/downloads/mysql/).

In the process of setting up MySQL, you will need to create a root
account. SQL is built to have robust permissions around who can do
what, but for testing, it is fine to use your root account.

To connect to the MySQL dateabase, you can issue the

```bash
$ mysql -u root --host=localhost --password
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 22
Server version: 5.6.12 MySQL Community Server (GPL)

Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> ...
```

From the terimal, you can issue SQL commands.

# Sequel Pro on OS X

If you are using an Apple computer, I recommend using the free and
open-source graphical program [Sequel Pro](http://www.sequelpro.com/).
Sequel Pro allows you to interact with the database through
a friendly graphical user interface instaed of the commandline.
It also simplifies the process of viewing tables and inspecting
other database properties.

Once MySQL is set up, you can connect to it with your root
username and password:

{% include twitter_plug.html %}

---
layout: post
title: "How to Install MySQL On Your Local Machine"
comments: true
permalink: install-mysql
---

*This is the second post in a [series of posts]({% post_url 2014-04-17-data-science-sql %})
about doing data science with SQL. The [previous post]({% post_url 2014-04-18-database-normalization %})
described the topic of database normalization and good database design.*

In this post, I will describe how to setup [MySQL](http://www.mysql.com/)
on your local machine.  MySQL is great for learning about SQL and
relational databases because it is popular, open source, and easy
to get started with.  By the end of this post, you will be able to
issue SQL queries against a MySQL database on your local machine.

## Setting up SQL On Your local Machine

[mysql.com](mysql.com) documents the steps required
to install MySQL on
[Windows](https://dev.mysql.com/doc/refman/5.0/en/windows-installation.html)
and
[Mac](https://dev.mysql.com/doc/refman/5.0/en/macosx-installation.html).
To install MySQL, you can get the installer
[here](http://dev.mysql.com/downloads/mysql/).
On a Mac, it is as easy as downloading a disk image, double clicking to install,
and then stepping through the installation process.

The Mac disk image also comes with a startup package which will
automatically launch MySQL when you start your computer.  Additionally,
it adds a convenient menu in the System Preferences for managing
MySQL:

![MySQL System Preferences](/assets/mysql_system_preferences.jpg)

After you setup MySQL, you can set the root MySQL account 
from the command line by issuing the command:

```bash
$ mysqladmin -u root password: XXXXXXXXXXXX
```

SQL has robust permissions so that different accounts can have
different permissions. This is good in production environments, but
for now logging in as root with full permissions is fine.

You can connect to the MySQL database from the command line:

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

Now you are ready to issue SQL commands!

# The Sequel Pro SQL Client on OS X

If you are using an Apple computer, I recommend using the free and
open-source graphical program [Sequel Pro](http://www.sequelpro.com/)
to connect to the database and to run SQL commands.  Sequel Pro also
simplifies the process of viewing tables and inspecting other
database properties. The sign-in screen looks like:

![Sequel Pro Connect Tab](/assets/sequel_pro_connect_tab.jpg)

Once you install Sequel Pro, you connect to it with your user name
and password as above.

Inside of Sequel Pro, there is a query menu
where you can issue SQL commands against the database:

![Sequel Pro Query Tab](/assets/sequel_pro_query_tab.jpg)

# Other MySQL Clients

If you prefer, there are several other programs for connecting to 
MySQL databases.

* MySQL Workbench ([link](http://dev.mysql.com/downloads/tools/workbench/))
* DBVisualizer ([link](http://www.dbvis.com/))
* phpMyAdmin ([link](http://www.phpmyadmin.net/home_page/))
* HeidiSQL ([link](http://www.heidisql.com/))

*In the [next post]({% post_url 2014-04-28-create-tables-sql %}) in this
[series of posts]({% post_url 2014-04-17-data-science-sql %}), 
we will go over the SQL commands
required to set up the example recipes database from the 
[first post]({% post_url 2014-04-18-database-normalization %}) 
in this series.*

{% include twitter_plug.html %}

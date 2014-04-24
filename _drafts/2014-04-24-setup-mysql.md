---
layout: post
title: "How to Setup MySQL On Your Local Machine"
comments: true
permalink: setup-mysql
---

*This is the second post in a [series of posts]({% post_url 2014-04-17-data-science-sql %})
about doing data science with SQL. The [previous post]({% post_url 2014-04-18-database-normalization %})
described the topic of database normalization and good database design.*

In this post, I will describe how to setup [MySQL](http://www.mysql.com/)
on your local machine.  MySQL is great for learning about SQL because
it is popular, open source, and easy to get started with.

## Setting up SQL On Your local Machine

Here is some documentation for installing MySQL on
[Windows](https://dev.mysql.com/doc/refman/5.0/en/windows-installation.html)
or 
[Mac](https://dev.mysql.com/doc/refman/5.0/en/macosx-installation.html).
In particular, you can get the installer
[here](http://dev.mysql.com/downloads/mysql/).

On a Mac, you can download a DMG, double click to install,
and then step through the installation process.
Also, the Mac DMG comes with a startup package which will automatically
launch MySQL when you turn on your computer and adds
a convenient menu in the System Preferences for managing MySQL:

![MySQL System Preferences](/assets/mysql_system_preferences.jpg)

After you setup MySQL, you can set up the root MySQL account 
from the terminal using the command:

```
$mysqladmin -u root password: XXXXXXXXXXXX
```

SQL has robust permissions so that different accounts can have
different access/privealges. This is good in a production
environment, but for now logging in as root is fine.

You can connect to the MySQL database from the commandline:

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

From the terimal, you are ready to issue SQL commands.

# The Sequel Pro SQL Client on OS X

If you are using an Apple computer, I recommend using the free and
open-source graphical program [Sequel Pro](http://www.sequelpro.com/).
Sequel Pro allows you to interact with the database through
a friendly graphical user interface instaed of the commandline.
It also simplifies the process of viewing tables and inspecting
other database properties.

![Sequel Pro Connect Tab](/assets/sequel_pro_connect_tab.jpg)

Once you install Sequel Pro, you connect to it with your username
and password as above.

Inside of SQL Pro, there is a query menu
where you can issue SQL commands.

![Sequel Pro Query Tab](/assets/sequel_pro_query_tab.jpg)

# Other MySQL Programs

If you prefer, there are several other programs for connecting to 
a MySQL database.

* MySQL Workbench ([link](http://dev.mysql.com/downloads/tools/workbench/))
* DBVisualizer ([link](http://www.dbvis.com/))
* phpMyAdmin ([link](http://www.phpmyadmin.net/home_page/))
* HeidiSQL ([link](http://www.heidisql.com/))

{% include twitter_plug.html %}

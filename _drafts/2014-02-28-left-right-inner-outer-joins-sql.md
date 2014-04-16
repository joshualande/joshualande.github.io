---
layout: post
title: Left, Right, Inner, and Outer Joins in SQL
comments: true
---

<!--
* SQL has the concept of `INNER`, `LEFT`, `RIGHT`, and `OUTER` joins,
  which behave different in the situation where you are joining on 
  columns with `NULL` values in them.
  [Here](http://dev.mysql.com/doc/refman/5.0/en/join.html)
  is some documentation on different joins.
-->

SQL has several different JOIN operators (INNER, LEFT, RIGHT, and
OUTER) which behave differently in situations where there is data
missing in either the left or right table.

## JOINing with a Simple Exmaple

The easiest way to understand this is by way of example.

Suppse we have a database storing information about people. For each
person, we know their name, their height, and their age. But for
our example, we will imagine our data to be spread across three
tables:

First, we have a `people` table

| person_id |   name |
| --------- | ------ |
|         0 | George |
|         1 |  Alice |
|         2 |  Steve |
|         3 |    Ted |
|         4 |  Sarah |

Next, we have a `heights` table:

| person_id | height |
| --------- | ------ |
|         0 |    6.1 |
|         1 |    5.3 |

Finally, we have a `ages` table:

| person_id |    age |
| --------- | ------ |
|         1 |     26 |
|         2 |   NULL |
|         4 |     37 |

Before we get into the examples, I will
provide self-contained SQL syntax so
you can play with these examples on your computer:

```sql
CREATE TABLE people (
  person_id int(11), 
  name varchar(30)
);
INSERT INTO people (person_id, name)
VALUES (0, "George"), 
       (1, "Alice"), 
       (2, "Steve"), 
       (3, "Ted"),
       (4, "Sarah");

CREATE TABLE heights (
  person_id int(11), 
  height float
);
INSERT INTO heights (person_id, height)
VALUES (0, 6.1), 
       (1, 5.3);

CREATE TABLE ages (
  person_id int(11), 
  age float
);
INSERT INTO ages (person_id, age)
VALUES (1, 26), 
       (2, NULL),
       (4, 37);
```

## Inner Joins

Suppose we want to get a table with
the heights and ages of everybody
in our table. To do this, we could
`JOIN` the tables together:

```sql
SELECT *
FROM heights AS a
JOIN ages AS b
ON a.person_id = b.person_id;
```

| a.person_id | a.height | b.person_id | b.age |
|-------------|----------|-------------|-------|
|           1 |      5.3 |           1 |    26 |


## Left and Right Joins

A left join:

```sql
SELECT *
FROM heights AS a
LEFT JOIN ages AS b
ON a.person_id = b.person_id;
```

| a.person_id | a.height | b.person_id | b.age |
|-------------|----------|-------------|-------|
|           0 |      6.1 |        NULL |  NULL |
|           1 |      5.3 |           1 |    26 |


Similarly, a right join

```sql
SELECT *
FROM heights AS a
RIGHT JOIN ages AS b
ON a.person_id = b.person_id;
```

| a.person_id | a.height | b.person_id | b.age |
|-------------|----------|-------------|-------|
|           1 |      5.3 |           1 |    26 |
|        NULL |     NULL |           2 |  NULL |
|        NULL |     NULL |           4 |    37 |



## Outer Join

Conceptually, an OUTER JOIN is...

It is not supported by MySQL.

It can always be emulated by unioning a
left join with a right join where the right
join filteres out rows which show up in the
left join:

```sql
(
    SELECT *
    FROM heights AS a
    LEFT JOIN ages AS b
    ON a.person_id = b.person_id
)
UNION ALL 
(
    SELECT *
    FROM heights AS a
    RIGHT JOIN ages AS b
    ON a.person_id = b.person_id
    WHERE a.person_id IS NULL
)
```


| a.person_id | a.height | b.person_id | b.age |
|-------------|----------|-------------|-------|
|           0 |      6.1 |        NULL |  NULL |
|           1 |      5.3 |           1 |    26 |
|        NULL |     NULL |           2 |  NULL |
|        NULL |     NULL |           4 |    37 |


We might want to query, for example,
the hieght of all people who have...







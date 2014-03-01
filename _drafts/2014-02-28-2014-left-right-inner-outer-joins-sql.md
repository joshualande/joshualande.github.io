---
layout: post
title: Left, Right, Inner, and Outer Joins in SQL
comments: true
---

SQL has the concept of INNER, LEFT, RIGHT, and OUTER joins.
They are all similar, but provide slightly different behavior
in the situation where rows are missing in a table.
The easiest way to understand this is by way of another example.

Suppse we have a database storing information about people.

First, we have a `people` table

| person_id |   name |
| --------- | ------ |
|         0 | George |
|         1 |  Alice |
|         2 |  Steve |

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

We might want to query, for example,
the hieght of all people who have...

# Inner Joins

Suppose we want to get a table with
the heights and ages of everybody
in our table. To do this, we could
`JOIN` the tables together:

```sql
SELECT *
FROM heights
JOIN person
```

But this returns

# Left and Right Join

# Approximating a Full Outer Join








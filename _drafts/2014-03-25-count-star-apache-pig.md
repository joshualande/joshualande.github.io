---
layout: post
title: "Avoid a Common Gotchas in Apache Pig: COUNT vs COUNT_STAR"
comments: true
permalink: common-cotcha-apache-pig-count-star
---

The `COUNT` and `COUNT_STAR` functions in Pig are especially confusing
to people new to Apache Pig.  They are especially confusing becuase
they behave different than what you would expect in SQL.

In Apache Pig, `COUNT` computes the number of rows with a non-null
first column. `COUNT_STAR` computes all rows.

As a concreate example, imagine this csv file `data.csv`.

```bash
$ cat > data.csv
first,second
third,
,
,fourth
fifth,final
```

When we use `PigStorage`,
Pig interprets empty columns as NULLs.
We can read in this data using the command:

```
data = LOAD 'data.csv' 
    USING PigStorage(',') 
    AS (col1:chararray, col2:chararray);
```

Next, we want to try `COUNT` and `COUNT_STAR` for
the rows.

```
group_all = GROUP data all;
nrows_count = FOREACH group_all GENERATE COUNT(data);
nrows_count_star = FOREACH group_all GENERATE COUNT_STAR(data);
```

As expected, because there are 3 rows with non-null first columns,
`nrows_count` has a value of 3. because there are 5 rows,
`nrows_count_star` has a value of 5.

# SQL Implementation

In SQL

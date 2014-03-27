---
layout: post
title: "One of the Biggest Gotchas in Apache Pig! COUNT vs COUNT_STAR"
comments: true
permalink: count-star-apache-pig
---

The `COUNT` and `COUNT_STAR` functions in Pig are especially confusing
to people new to Apache Pig. 
They are especially confusing becuase they behave different than what you would expect in SQL.

In this post I will go over their the difference between them using a simple exmaple.

```bash
$ cat > data.csv
first,second
one,
,
last,final
```

```
data = LOAD 'data.csv' 
    USING PigStorage(',') 
    AS (col1:chararray, col2:chararray);
```

Data is

|  col1 |   col2 |
| ----- | ------ |
| first | second |
|   one |        |
|       |        |
|  last |  final |


```
group_all = GROUP data all;
```



```
nrows_count = FOREACH group_all GENERATE COUNT(data);
```

`nrows_count` has a value of 3.

```
nrows_count_star = FOREACH group_all GENERATE COUNT_STAR(data);
```

`nrows_count_star` has a value of 4

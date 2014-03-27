---
layout: post
title: "COUNT vs COUNT_STAR in Apache Pig"
comments: true
permalink: count-star-apache-pig
---

The `COUNT` and `COUNT_STAR` functions in Pig are especially confusing
because they behave differently than what you expect in SQL.

```bash
$ cat > data.dat
first row
\N
third row
\N
not really null
```

```
data = LOAD 'data.dat' AS (row);

group_all = GROUP data all;
nrows_count = FOREACH group_all GENERATE COUNT(data);
nrows_count_star = FOREACH group_all GENERATE COUNT_STAR(data);
```

`nrows_count`
`nrows_count_star`

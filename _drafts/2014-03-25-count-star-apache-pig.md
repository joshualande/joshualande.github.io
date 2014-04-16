---
layout: post
title: "Avoid a Common Gotchas in Apache Pig: COUNT vs COUNT_STAR"
comments: true
permalink: count-star-pig
---

One common cotcha with Apache Pig is mixing up
the `COUNT` and `COUNT_STAR` functions.
They are especially confusing because
`COUNT` in Apache Pig works different than in SQL.

## COUNT in Apache Pig

In Apache Pig, `COUNT` computes the number of rows with a non-null
first column. `COUNT_STAR` computes all rows.

As a concreate example, imagine this csv file `data.csv`.

```bash
$ cat > data_with_nulls.csv
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
data_with_nulls = LOAD 'data_with_nulls.csv' 
    USING PigStorage(',') 
    AS (col1:chararray, col2:chararray);
```

Before testing out the aggregation, we need to group
the data together. Pig provides a conveenient function
to group the entire table into one bag:

```
group_all = GROUP data_with_nulls all;
```

`group_all` becomes the table

| group |                                     data\_with\_nulls |
| ----- | ----------------------------------------------------- |
|   all | {(first,second),(third,),(,),(,fourth),(fifth,final)} |

Next, we want to try `COUNT` and `COUNT_STAR` for
the rows.

```
nrows_count = FOREACH group_all GENERATE COUNT(group); -- returns 3

nrows_count = FOREACH group_all GENERATE COUNT(data.col1); -- returns 3
nrows_count = FOREACH group_all GENERATE COUNT(data.col2); -- returns 3
```

As expected, because there are 3 rows with non-null first columns,
`nrows_count` has a value of 3. Surprisingly, this behavior is
indepdnent of wheter you computer `COUNT(data)`, `COUNT(data.col1)`

Next, we can try again with `COUNT_STAR`:
```
nrows_count_star = FOREACH group_all GENERATE COUNT_STAR(data); - returns 
```

because there are 5 rows,
`nrows_count_star` has a value of 5.

## Count in SQL



```sql
CREATE TABLE data_with_nulls (
  col1 varchar(10),
  col2 varchar(10)
);
INSERT INTO data_with_nulls
    (col1, col2)
VALUES
    ("first","second"),
    ("third",NULL),
    (NULL,NULL),
    (NULL,"fourth"),
    ("fifth","final")
```

```sql
SELECT COUNT(*)
FROM data_with_nulls
```

-> Returns ???


```sql
SELECT COUNT(col1)
FROM data_with_nulls
```

-> returns ???


```sql
SELECT COUNT(col2)
FROM data_with_nulls
```

-> Returns ???

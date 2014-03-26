---
layout: post
title: "COGROUP in Apache Pig"
comments: true
permalink: cogroup-in-pig
---

The `COGROUP` command in Apache Pig is somewhat confusing,
because it is half way between the `JOIN` and `GROUP` command.

In its simplest form, `COGROUP` is exactly the same as the `GROUP`. 
It groups on the desired data and creats bags for all the input.

For exmaple, given a sample ages data set:

```bash
$ cat ages.csv
adam,24
alex,23
alice,24
steve,23
```

We can group on age:

```
ages = LOAD 'ages.csv' 
    USING PigStorage(',')
    AS (name:chararray,age:int);

grouped = COGROUP ages BY age;
DUMP grouped;
```

This returns a list of ages and for each age,
a bags of rows containing the ages.

| ages | {name: chararray,age: int} |
| ---- | -------------------------- |
| 23   |    {(alex,23),(steve,23)}) |
| 24   |    {(adam,24),(alice,24)}) |

Where `COGROUP` gets fancier than GROUP is
you can perform a GROUP and then JOIN of multiple
tables at once.

For example, if we had a table of eye colors

```bash
$ cat eyecolor.csv
adam,blue
alex,green
alice,blue
steve,blue
```

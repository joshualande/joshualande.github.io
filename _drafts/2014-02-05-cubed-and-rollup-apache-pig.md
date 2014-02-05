---
layout: page
title: "Cube and Rollup: Two Apache Pig Functions That You Need to Learn"
comments: true
---

I recently found two really incredible functions in Apache Pig
called Cubed and Rollup for computing multi-level aggregations of
data sets. I found the documentation for these examples to be
confusing, so I will try to explain how these functions
work with simple examples.

```
a,c,e,1
a,d,f,2
b,d,e,3
```

```
data = LOAD 'test.csv' USING PigStorage(',') 
    AS (col1:chararray, col2:chararray, col3:chararray, val:int);
DUMP data;
```
Which returns
```
(a,c,e,1)
(a,d,f,2)
(b,d,e,3)
```

Next, we can cube up the data based on column two and three:

```
cubed = CUBE data BY CUBE(col2, col3);
DUMP cubed;
```

This returns:

```
((c,e),{(c,e,a,1)})
((c,),{(c,,a,1)})
((d,e),{(d,e,b,3)})
((d,f),{(d,f,a,2)})
((d,),{(d,,a,2),(d,,b,3)})
((,e),{(,e,a,1),(,e,b,3)})
((,f),{(,f,a,2)})
((,),{(,,a,1),(,,a,2),(,,b,3)})
```

We can flatten out all of the rows:

```
flat = FOREACH cubed GENERATE FLATTEN(cube); 
DUMP flat;
```

This returns:

```
(c,e,a,1)
(c,,a,1)
(d,e,b,3)
(d,f,a,2)
(d,,a,2)
(d,,b,3)
(,e,a,1)
(,e,b,3)
(,f,a,2)
(,,a,1)
(,,a,2)
(,,b,3)
```

This is kind of confusing:

```
x = FOREACH flat GENERATE
    (col1 is not NULL ? col1 : '*') as col1,
    (col2 is not NULL ? col2 : '*') as col2,
    (col3 is not NULL ? col3 : '*') as col3,
    val;
dump x;
```

Which returns:

```
(a,c,e,1)
(a,c,*,1)
(b,d,e,3)
(a,d,f,2)
(a,d,*,2)
(b,d,*,3)
(a,*,e,1)
(b,*,e,3)
(a,*,f,2)
(a,*,*,1)
(a,*,*,2)
(b,*,*,3)
```

# Advanced Cubed

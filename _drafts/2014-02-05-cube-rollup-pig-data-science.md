---
layout: page
title: "CUBE and ROLLUP: Two Pig Functions That Every Data Scientist Should Know"
comments: true
---

I recently found two incredible functions in 
[Apache Pig](https://pig.apache.org) called 
[CUBE and ROLLUP](https://pig.apache.org/docs/r0.11.0/api/org/apache/pig/newplan/logical/relational/LOCube.html).
These functions allow for computing multi-level
aggregations of a data set. I found the documentation for these
examples to be confusing, so in this post I
am going to explain how they work with simple examples.

# Aggregating in Pig Using the GROUP Operator

Before we get into CUBE and ROLLUP, I will describe how simple
aggregations work using the GROUP BY operator in pig.  If this is
familiar to you, feel free to skip ahead to the next section.

For this post, lets imagine a simple data set of people.  For
each person, we know their name, the country they live in, their
gender, the sport they play, and their height.

```
$ cat people.csv
Steve,US,M,football,6.5
Alex,US,M,football,5.5
Ted,UK,M,football,6.0
Mary,UK,F,baseball,5.5
Ellen,UK,F,football,5.0
```

A common task in analytics is to compute the average
value (or some other aggregate quantity like median or maximum)
for people in different groups.
For example, we might be interested in computing
the average height of people in a given country.

We can do this using Apache Pig fairly easily.
First, we load in the data:

```
people = LOAD 'people.csv' USING PigStorage(',') 
    AS (name:chararray, country:chararray, 
    gender:chararray, sport:chararray, height:float);
```


You can follow along by running pig in local mode with with command
`pig -x local`.

First, lets `DUMP` the `people` table to make sure we
loaded the data correctly:

```
(Steve,US,M,football,6.5)
(Alex,US,M,football,5.5)
(Ted,UK,M,football,6.0)
(Mary,UK,F,baseball,5.5)
(Ellen,UK,F,football,5.0)
```

In order to compute the average height for people of a given gender,
we have to first group the data by gender:

```
grouped = GROUP people BY gender;
```

In Pig,
the `GROUP` command returns a table of each gender and
for each gender a bag 
containing every row of people with that gender.
So the `grouped` table is:

```
(F,{(Mary,UK,F,baseball,5.5),(Ellen,UK,F,football,5.0)})
(M,{(Steve,US,M,football,6.5),(Alex,US,M,football,5.5),(Ted,UK,M,football,6.0)})
```

To compute the average height as well as number of people,
we can then use the `FOREACH`, `COUNT`, and `AVG` command:

```
heights = FOREACH grouped GENERATE
    group AS gender,
    COUNT(people) AS num_people,
    AVG(people.height) AS avg_height;
```

The `heights` table is:

```
(F,2,5.25)
(M,3,6.0)
```

This shows that there are 3 men with an average height of 6.0 and two 
women with an average height of 5.25.

If we wanted to compute the average height for people 
of a particular gender and sport, we could again 
expand upon the `GROUP BY` command from above:

```
grouped = GROUP people BY (gender,sport);
heights = FOREACH grouped GENERATE
    FLATTEN(group) AS (gender,sport),
    COUNT(people) AS num_people,
    AVG(people.height) AS avg_height;
```

The `heights` table now contains:

```
(F,baseball,1,5.5)
(F,football,1,5.0)
(M,football,3,6.0)
```

This says, for example, that there are three men who play football with
an average height of 6.0.

# Multi-Level Aggregations in Pig the Hard Way

What if both aggregations were important, i.e. we wanted
to know both the average height by gender as well as the average
height by gender and sport.  One way to solve this is to simply
create two tables, one for each kind of aggregation.

Another option would be to create a duplicate set of rows in our
`people` table where the gender is hard coded to a default value
(like '*').  Then, when we computed average height by gender and
sport, we would automatically also compute average height by just
gender.

To do this, we can create a new table and `UNION` it
with the original table:

```
agg_people = FOREACH people GENERATE
    name, 
    country, 
    gender, 
    '*' AS sport, 
    height;

unioned_people = UNION people, agg_people;

grouped = GROUP unioned_people BY (gender,sport);
heights = FOREACH grouped GENERATE
    FLATTEN(group) AS (gender,sport),
    COUNT(unioned_people) AS num_people,
    AVG(unioned_people.height) AS avg_height;
```

The `heights` table now contains

```
(F,*,2,5.25)
(F,baseball,1,5.5)
(F,football,1,5.0)
(M,*,3,6.0)
(M,football,3,6.0)
```

As you can see, this table nicely combines both tables from above.
We can compute the average height of people with a certain gender
and sport as well as the average height for all people with a certain
gender (where sport is equal to '*').


# CUBE And ROLLUP For Multi Level Aggregations

This `UNION` approach of tables works, but is not very inelegant. It
suffers from the limitations that it requires a lot of extra code
to write, does not generalize easily to performing aggregations
over multiple dimensions, and is not well optimized internally by
Pig.

Fortunately, as of version 0.11, Apache Pig provides the `CUBE` and
`ROLLUP` function which can perform this kind of calculation much
more efficiently and elegantly.

Assuming we would are using the same people data set as above, but
we want to compute all levels of aggregation 
for gender, and sport, we can use the following code in pig.


```
cubed = CUBE people BY CUBE(gender, sport);
```

`CUBED` returns:

```
((F,baseball),{(F,baseball,Mary,UK,5.5)})
((F,football),{(F,football,Ellen,UK,5.0)})
((F,),{(F,,Mary,UK,5.5),(F,,Ellen,UK,5.0)})
((M,football),{(M,football,Alex,US,5.5),(M,football,Ted,UK,6.0),(M,football,Steve,US,6.5)})
((M,),{(M,,Alex,US,5.5),(M,,Steve,US,6.5),(M,,Ted,UK,6.0)})
((,baseball),{(,baseball,Mary,UK,5.5)})
((,football),{(,football,Ted,UK,6.0),(,football,Steve,US,6.5),(,football,Alex,US,5.5),(,football,Ellen,UK,5.0)})
((,),{(,,Mary,UK,5.5),(,,Alex,US,5.5),(,,Steve,US,6.5),(,,Ellen,UK,5.0),(,,Ted,UK,6.0)})
```

Althrough this is hard to look at, you will see that the first entry
is are the values for all of the columns which are being cubed over,
and the second column are bags of all the rows.
This is almost identical to the way `GROUP BY` works, except there
are additional columns with `NULL` values corresponding to buckets
which aggregate over that quantity.

Now, to compute the average height, we can use thecode:

```
heights = FOREACH cubed GENERATE 
    FLATTEN(group) AS (gender, sport), 
    COUNT(cube) AS num_people,
    AVG(cube.height) AS avg_height;  
``` 

The `heights` table now contains:

```
(F,baseball,1,5.5)
(F,football,1,5.0)
(F,,2,5.25)
(M,football,3,6.0)
(M,,3,6.0)
(,baseball,0,5.5)
(,football,0,5.75)
(,,0,5.7)
```

Note that by default, `CUBE` sets the
aggregate value to `NULL`. I find this
confusing and prefer to use a a star 
('*') to represent the aggregate value.
We can fix this using one more line of code:

```
heights  = FOREACH heights GENERATE
    (gender is not NULL ? gender : '*') as gender, 
    (sport is not NULL ? sport : '*') as sport,
    avg_height;
```

The `heights` table finally contains:

```
(F,baseball,5.5)
(F,football,5.0)
(F,*,5.25)
(M,football,6.0)
(M,*,6.0)
(*,baseball,5.5)
(*,football,5.75)
(*,*,5.7)
```

This computes the average height
for any gender and any sport, 
for any sport (with gender equal to '\*'),
and for any gender (and sport equal to '\*').
This is super cool!

# CUBE Over Multiple Dimensions


Looking again to our example from above, 
if we `CUBE` on (country, gender, sport),
we would compute aggregations for all
(country, gender, and sports) pairs. But we would
also find the aggregation for all (country, gender) pairs,
all (country, sports) pairs, and all (gender, sports) pairs.
We would also compute all aggregations for any
country, gender, and sport. And we would be able to compute
the average aggregating over all countries, genders, and sports.

For our example, when we `CUBE` over all pairs:

```
cubed = CUBE people BY CUBE(country,gender,sport);

heights = FOREACH cubed GENERATE 
    FLATTEN(group) AS (country,gender,sport),
    AVG(cube.height) AS avg_height;  

heights = FOREACH heights GENERATE
    (country is not NULL ? country : '*') as country, 
    (gender is not NULL ? gender : '*') as gender, 
    (sport is not NULL ? sport : '*') as sport,
    avg_height;
```

The heights table now contains the average height for all
possible aggregations

```
(UK,F,baseball,5.5)
(UK,F,football,5.0)
(UK,F,*,5.25)
(UK,M,football,6.0)
(UK,M,*,6.0)
(UK,*,baseball,5.5)
(UK,*,football,5.5)
(UK,*,*,5.5)
(US,M,football,6.0)
(US,M,*,6.0)
(US,*,football,6.0)
(US,*,*,6.0)
(*,F,baseball,5.5)
(*,F,football,5.0)
(*,F,*,5.25)
(*,M,football,6.0)
(*,M,*,6.0)
(*,*,baseball,5.5)
(*,*,football,5.75)
(*,*,*,5.7)
```

For example, the average height of all baseball players is 5.5,
the average height of all people in the US is 6.0, and
the average height of all people is 5.7.

# The ROLLUP Operator

The `ROLLUP` operator is similar to `CUBE`,
but will only perform hirearchitcal
aggregations. For our example,
`ROLLUP` of (country,gender,sport)
will aggregate over sport, then gender and sport,
and then country, gender and sport.

Lets `ROLLUP` our example from above:

```
rolledup = CUBE people BY ROLLUP(country,gender,sport);

heights = FOREACH rolledup GENERATE 
    FLATTEN(group) AS (country,gender,sport),
    AVG(cube.height) AS avg_height;  

heights = FOREACH heights GENERATE
    (country is not NULL ? country : '*') as country, 
    (gender is not NULL ? gender : '*') as gender, 
    (sport is not NULL ? sport : '*') as sport,
    avg_height;
```

When we `ROLLUP` by country, gender, and sport we get the following
aggregations:

```
(UK,F,baseball,5.5)
(UK,F,football,5.0)
(UK,F,*,5.25)
(UK,M,football,6.0)
(UK,M,*,6.0)
(UK,*,*,5.5)
(US,M,football,6.0)
(US,M,*,6.0)
(US,*,*,6.0)
(*,*,*,5.7)
```

Although this might seem artificial, there are some situations where
this is very useful.  For example, if the dataset contained 
hirearctical columns 
(like continent, country, and city), then these would be the only
sensible aggregations.

# Not Aggregating Over Certain Columns

The last thing I wanted to cover was situations where you may
not want to aggregation over a certain column.
For our original people table above, supposed that we wanted to 
compute the average height of people with a given 
country and gender, but also of a particular sport. But
for whatever reason, we don't want to aggregate over all 
countries, as above.

We can do this by cubing the data and then flattening out all of the bags:

```
cubed = CUBE people BY CUBE(gender, sport);

flattened_bags = FOREACH cubed GENERATE 
    FLATTEN(cube);

flattened_bags = FOREACH flattened_bags GENERATE
    name as name,
    country as country,
    (gender is not NULL ? gender : '*') as gender, 
    (sport is not NULL ? sport : '*') as sport,
    height as height;
```

The `flattened_bags` table contains:

```
(Mary,UK,F,baseball,5.5)
(Ellen,UK,F,football,5.0)
(Mary,UK,F,*,5.5)
(Ellen,UK,F,*,5.0)
(Alex,US,M,football,5.5)
(Ted,UK,M,football,6.0)
(Steve,US,M,football,6.5)
(Alex,US,M,*,5.5)
(Steve,US,M,*,6.5)
(Ted,UK,M,*,6.0)
(Mary,UK,*,baseball,5.5)
(Ted,UK,*,football,6.0)
(Steve,US,*,football,6.5)
(Alex,US,*,football,5.5)
(Ellen,UK,*,football,5.0)
(Mary,UK,*,*,5.5)
(Alex,US,*,*,5.5)
(Steve,US,*,*,6.5)
(Ellen,UK,*,*,5.0)
(Ted,UK,*,*,6.0)
```

Now that we have generated the flattend bags,
we can group by the columns we were originally
interested in:

```
grouped = GROUP flattened_bags BY (country,gender,sport);

heights = FOREACH grouped GENERATE
    FLATTEN(group) AS (country,gender,sport),
    AVG(flattened_bags.height) AS avg_height;
```

This computes the average height for people of a given
country, gender, and sport. 

```
(UK,*,*,5.5)
(UK,*,baseball,5.5)
(UK,*,football,5.5)
(UK,F,*,5.25)
(UK,F,baseball,5.5)
(UK,F,football,5.0)
(UK,M,*,6.0)
(UK,M,football,6.0)
(US,*,*,6.0)
(US,*,football,6.0)
(US,M,*,6.0)
(US,M,football,6.0)
```

But it also aggregates over gender, sport, and gender and sport.

But note that this does not compute aggregations over all countries.

# Links

* [Prasanth Jayachandran](http://prasanthj.info) created this function 
as a [Google Summer of Code project](http://www.google-melange.com/)
in 2012. See [PIG-2167](https://issues.apache.org/jira/browse/PIG-2167) for
the information on the development of htis feature.
* [Here](https://blogs.apache.org/pig/entry/apache_pig_it_goes_to)
is the officila release of Apache Pig 0.11 which included CUBE and ROLLUP.
* The idea behind the CUBE and ROLLUP operator originated in the
idea of a [OLAP cube](http://en.wikipedia.org/wiki/OLAP_cube) and
exist in other databases (like 
[Microsoft SQL Server](http://technet.microsoft.com/en-us/library/ms175939)).


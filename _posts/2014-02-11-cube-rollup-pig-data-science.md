---
layout: post
title: "CUBE and ROLLUP: Two Apache Pig Functions That Every Data Scientist Should Know"
comments: true
redirect_from: "/2014/02/11/cube-rollup-pig-data-science/"
permalink: cube-rollup-pig-data-science
---

![The Apache Pig Logo](/assets/pig_logo.jpg)

I recently found two incredible functions in 
[Apache Pig](https://pig.apache.org) called 
[CUBE and ROLLUP](https://pig.apache.org/docs/r0.11.0/api/org/apache/pig/newplan/logical/relational/LOCube.html)
that every data scientist should know.
These functions can be used to compute multi-level
aggregations of a data set. I found the documentation for these
functions to be confusing, so I will work through
a simple example to explain how they work.

## Aggregating in Pig Using the GROUP Operator

Before we get into `CUBE` and `ROLLUP`, I will describe how to do simple
aggregations using the `GROUP BY` operator in pig.  If this is
familiar to you, feel free to skip ahead to the next section.

Imagine a simple data set of people where for
each person we know their name, the country they live in, their
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

We can do this using Pig fairly easily.
First, we load in the data:

```
people = LOAD 'people.csv' USING PigStorage(',') 
    AS (name:chararray, country:chararray, 
    gender:chararray, sport:chararray, height:float);
```


You can follow along by running pig in local mode with with the 
command:

```
pig -x local
```

First, we should `DUMP` the `people` table to make sure we
loaded it correctly:

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

The `GROUP` operator returns, for each gender,
a bag 
containing all of the rows for people of that gender.
For our example, the `grouped` table is:

```
(F,{(Mary,UK,F,baseball,5.5),(Ellen,UK,F,football,5.0)})
(M,{(Steve,US,M,football,6.5),(Alex,US,M,football,5.5),(Ted,UK,M,football,6.0)})
```

To compute the average height,
we can then use the `FOREACH` and `AVG` command.
While we are at it, we can compute the
number of people with the `COUNT` command:

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
of a particular gender who played a particular sport, we could again 
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

## Multi-Level Aggregations in Pig the Hard Way

What if both aggregations were important. Suppose we wanted
to know both the average height by gender as well as the average
height by gender and by sport.  One way to solve this is to simply
create two tables, one for each aggregation. But
this is cumbersome, especially if there were many interesting
dimensions to aggregate over. 

Another thing we could do is a weighted average of
the average by height and gender to compute the average by gender.
But this is inelegant, will slow down any future queries on the
data set by requiring an additional aggregation, and will not work
for more sophisticated aggregate quantities like median or quantiles.

Another option would be to create a duplicate set of rows in our
`people` table where the gender is hard coded to a default value
like a star.  Then, when we computed the average height by gender and
by sport we would automatically also compute the average height by just
gender (where sport is equal to a star).

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

As you can see, this table nicely combines both data sets from above.
The average height by gender for people of any sport
can be found by looking for rows where sport equals '*'.

## CUBE And ROLLUP For Multi-Level Aggregations

This `UNION` approach above works, but it is not very elegant. It
suffers from the limitations that it requires a lot of extra code, does not 
generalize easily to performing aggregations
over multiple dimensions, and is not well optimized by Pig.

Fortunately, as of version 0.11, Apache Pig provides the `CUBE` and
`ROLLUP` function which can perform this kind of calculation much
more efficiently and elegantly.

For the same `people` table from before, we
can compute the different possible aggregations by
gender and sport using the code:

```
cubed = CUBE people BY CUBE(gender, sport);
```

The `cubed` table is:

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

Although this is hard to look at, you will see that the first column
contains the possible values for cubed columns.
The second column contains bags of all the rows matching that
aggregation.
This is almost identical to the way `GROUP BY` works except that 
some of the rows now contain `NULL` values (corresponding 
to aggregations over that column.

To compute the average height as above, we can again use the `FOREACH` and `AVG` operators:

```
heights = FOREACH cubed GENERATE 
    FLATTEN(group) AS (gender, sport), 
    COUNT_STAR(cube) AS num_people,
    AVG(cube.height) AS avg_height;  
``` 

The `heights` table now contains:

```
(F,baseball,1,5.5)
(F,football,1,5.0)
(F,,2,5.25)
(M,football,3,6.0)
(M,,3,6.0)
(,baseball,1,5.5)
(,football,4,5.75)
(,,5,5.7)
```

This table is the same as above, but also
computes the average per sport for all genders
and the average over all genders and all sports.

Note that in this example we changed
from using the `COUNT` to `COUNT_STAR`
function. The reason is that the
`COUNT` function in Pig will not count
rows with `NULL` values in it whereas
`COUNT_STAR` includes those rows.

Unlike the example above, `CUBE` sets the
aggregate value to `NULL` instead of a star.
I find this annoying and prefer to use the star 
symbol. We can fix this in Pig:

```
heights  = FOREACH heights GENERATE
    (gender is not NULL ? gender : '*') as gender, 
    (sport is not NULL ? sport : '*') as sport,
    num_people,
    avg_height;
```

The `heights` table finally contains:

```
(F,baseball,1,5.5)
(F,football,1,5.0)
(F,*,2,5.25)
(M,football,3,6.0)
(M,*,3,6.0)
(*,baseball,1,5.5)
(*,football,4,5.75)
(*,*,5,5.7)
```

## CUBEing Over Multiple Dimensions

What is really great about `CUBE` is
the ability to cube over multiple dimensions.
It isn't much harder using `CUBE`
to compute
all possible aggregations
of country, gender, and sport:

```
cubed = CUBE people BY CUBE(country,gender,sport);

heights = FOREACH cubed GENERATE 
    FLATTEN(group) AS (country,gender,sport),
    COUNT_STAR(cube) As num_people,
    AVG(cube.height) AS avg_height;  

heights = FOREACH heights GENERATE
    (country is not NULL ? country : '*') as country, 
    (gender is not NULL ? gender : '*') as gender, 
    (sport is not NULL ? sport : '*') as sport,
    num_people,
    avg_height;
```



The `heights` table now contains the average height for all
possible aggregations of the three columns:

```
(UK,F,baseball,1,5.5)
(UK,F,football,1,5.0)
(UK,F,*,2,5.25)
(UK,M,football,1,6.0)
(UK,M,*,1,6.0)
(UK,*,baseball,1,5.5)
(UK,*,football,2,5.5)
(UK,*,*,3,5.5)
(US,M,football,2,6.0)
(US,M,*,2,6.0)
(US,*,football,2,6.0)
(US,*,*,2,6.0)
(*,F,baseball,1,5.5)
(*,F,football,1,5.0)
(*,F,*,2,5.25)
(*,M,football,3,6.0)
(*,M,*,3,6.0)
(*,*,baseball,1,5.5)
(*,*,football,4,5.75)
(*,*,*,5,5.7)
```

Here `CUBE` computed all aggregations for a
particular country, gender, and sport. But it also computed all
aggregations for a particular country and gender (but any sport),
country and sport (but any gender), and gender and sport (but any
country).  It also computed all aggregations for a particular country
(but any gender and sport), a particular gender (but any country
and sport), and for a particular sport (but any gender and country)
And finally, it computed the aggregation over all people.

From this table, we see that the average height of all football
players is 5.75, the average height of all people in the US is 6.0,
and the average height of all football players in the US is 6.0.

## The ROLLUP Operator

The `ROLLUP` operator is similar to `CUBE`,
but will only perform hierarchical
aggregations. For our example,
`ROLLUP` of country, gender, and sport
will perform all aggregations for a particular
country, gender, and sport.
Then it will perform all aggregations for
a particular country and gender (but any sport).
Then it will perform all aggregations for a particular
country (but for any gender and sport). And
finally it will compute an aggregation for all
people.

To `ROLLUP` our example from above:

```
rolledup = CUBE people BY ROLLUP(country,gender,sport);

heights = FOREACH rolledup GENERATE 
    FLATTEN(group) AS (country,gender,sport),
    COUNT_STAR(cube) AS num_people,
    AVG(cube.height) AS avg_height;  

heights = FOREACH heights GENERATE
    (country is not NULL ? country : '*') as country, 
    (gender is not NULL ? gender : '*') as gender, 
    (sport is not NULL ? sport : '*') as sport,
    num_people,
    avg_height;
```

The `heights` table now contains:

```
(UK,F,baseball,1,5.5)
(UK,F,football,1,5.0)
(UK,F,*,2,5.25)
(UK,M,football,1,6.0)
(UK,M,*,1,6.0)
(UK,*,*,3,5.5)
(US,M,football,2,6.0)
(US,M,*,2,6.0)
(US,*,*,2,6.0)
(*,*,*,5,5.7)
```

Although this might seem artificial, 
it is very useful if the columns in the data set are 
hierarchical. For example, this
would be useful if the data had the columns 
continent, country, and city.

## Not Aggregating Over Certain Columns

The last situation is where you may
not want to aggregation over a certain column.
This may be useful if one of the columns cannot logically
be aggregated over.

For our example data set above, supposed that we wanted to 
cube over gender and sport. But what if we wanted
this quantity only for a particular country,
not aggregated over all countries as before.

We can do this with `CUBE` followed by `FLATTEN`:

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

The `flattened_bags` table contains all of the rows
from the bags created by `CUBE`:

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

Finally, we can group the flattened bags
by any columns we want to aggregate over:

```
grouped = GROUP flattened_bags BY (country,gender,sport);

heights = FOREACH grouped GENERATE
    FLATTEN(group) AS (country,gender,sport),
    COUNT(flattened_bags) AS num_people,
    AVG(flattened_bags.height) AS avg_height;
```

The `heights` table now contains:

```
(UK,*,*,3,5.5)
(UK,*,baseball,1,5.5)
(UK,*,football,2,5.5)
(UK,F,*,2,5.25)
(UK,F,baseball,1,5.5)
(UK,F,football,1,5.0)
(UK,M,*,1,6.0)
(UK,M,football,1,6.0)
(US,*,*,2,6.0)
(US,*,football,2,6.0)
(US,M,*,2,6.0)
(US,M,football,2,6.0)
```

This table has all possible combinations of
aggregations over gender and sport, but does not aggregate over
country.

## Links

* [Prasanth Jayachandran](http://prasanthj.info) created the `CUBE`
and `ROLLUP` functions
as a [Google Summer of Code project](http://www.google-melange.com/)
in 2012. See [PIG-2167](https://issues.apache.org/jira/browse/PIG-2167) for
the development of these commands.
* The official release of [Apache Pig 0.11](https://blogs.apache.org/pig/entry/apache_pig_it_goes_to)
that announced the `CUBE` and `ROLLUP` functions.
* The idea behind the `CUBE` and `ROLLUP` function originated in the
idea of a [OLAP cube](http://en.wikipedia.org/wiki/OLAP_cube).
* Several databases (like 
[Microsoft SQL Server](http://technet.microsoft.com/en-us/library/ms175939)) implement
`CUBE` and `ROLLUP`.
* Cube and Rollup are implemented in [Apache Hive](http://hive.apache.org/). See [here](https://cwiki.apache.org/confluence/display/Hive/Enhanced+Aggregation,+Cube,+Grouping+and+Rollup#EnhancedAggregation,Cube,GroupingandRollup-CubesandRollups) for documentation.

{% include twitter_plug.html %}

---
layout: page
title: "CUBE and ROLLUP: Two Pig Functions That Every Data Scientist Needs to Know"
comments: true
---

I recently found two really incredible functions in Apache Pig
called CUBE and ROLLUP.
These functions allow for computing multi-level aggregations of
data sets. I found the documentation for these examples to be
confusing, so I will explain how these functions
work with simple examples.

# Aggregating in Pig Using the GROUP Operator

Before we get into CUBE and ROLLUP, I will
describe how simple aggregations work using
the GROUP BY operator in pig.
If this is already familliar to you, skip ahead
to the next section.

For this post, I will imagine a simple data
set of people.  For each person, we know 
their name, the country they live in,
their gender, the sport they play, and
their height.

```bash
$ cat people.csv
Steve,US,M,football,6.5
Alex,US,M,football,6.2
Ted,UK,M,baseball,6.2
Mary,US,F,baseball,5.2
Ellen,UK,F,football,5.4
```

A common task in analytics is to compute the average
(or some other quantity like median)
value for people in different groups. 
For exmaple, we might be interested in computing
the average height of people in a given country.

We could compute the average in Pig using the code:

```
people = LOAD 'people.csv' USING PigStorage(',') 
    AS (name:chararray, country:chararray, gender:chararray, sport:chararray, height:int);
DUMP people;
```

This computes

```
(Steve,US,M,football,6)
(Alex,US,M,football,6)
(Ted,UK,M,baseball,6)
(Mary,US,F,baseball,5)
(Ellen,UK,F,football,5)
```

Note, you can follow along my example by running pig in local mode with with command:

```bash
pig -x local.
```


When, we group our data by country:

```
grouped = GROUP people BY country;
DUMP grouped;
```

The group command returns a list of each country and, for each country, a bag containing every row
for people in that country.

```
(UK,{(Ted,UK,M,baseball,6),(Ellen,UK,F,football,5)})
(US,{(Steve,US,M,football,6),(Alex,US,M,football,6),(Mary,US,F,baseball,5)})
```

It is easy to aggregate over each bag computing the average height:

```
heights = FOREACH grouped GENERATE
    group AS country,
    AVG(people.height) AS avg_height;
DUMP heights;
```
Which returns

```
(UK,5.5)
(US,5.666666666666667)
```

Similarly, if we wanted to compute the average height for people in a particular
country, gender, and sport, we could again use the GROUP command in pig

```
grouped = GROUP people BY (country,gender);
heights = FOREACH grouped GENERATE
    FLATTEN(group) AS (country,gender),
    AVG(people.height) AS avg_height;
DUMP heights;
```

Which returns

```
(UK,F,5.0)
(UK,M,6.0)
(US,F,5.0)
(US,M,6.0)
```

# Multi-Level Aggregations in Apache Pig

What if, on the other hand, we wanted to compute the average
height both for people in a country as well as people in
a country of a particular gender.

One way to solve this is to simply create a separate output 
table for the different aggregations. 

Another option would be to create a duplicate set of rows
in the table where the gender is hard coded as a star ('*'),
and then to aggregate over both country and gender.

```
agg_people = FOREACH people GENERATE
    name, 
    country, 
    '*' AS gender, 
    sport, 
    height;

unioned_people = UNION people, agg_people;

grouped = GROUP unioned_people BY (country,gender);
heights = FOREACH grouped GENERATE
    FLATTEN(group) AS (country,gender),
    AVG(unioned_people.height) AS avg_height;
DUMP heights;
```

This computes

```
(UK,*,5.5)
(UK,F,5.0)
(UK,M,6.0)
(US,*,5.666666666666667)
(US,F,5.0)
(US,M,6.0)
```

This computes the average height both for people in a
particular country as well as people of a particular country
and a particular gender.

And what's best is that the data itself fits into one simple table.
When we specify that the gender is 'M', we get the average height for men, when
we specify gender is 'F', we get the average height for women, and
when we specify gender is '*', we get the average height aggregated
over men and women.

# CUBE And ROLLUP For Multi Level Aggregations

This approach of unioning tables is sufficent, but inelegant.  It
suffers from the limitations that it requires a lot of extra code
to write, does not generalize easily to performing aggregations
over multiple dimensions, and is not well optimized internally by
Pig.

Fortunately, as of version 0.11, Apache Pig provides the CUBE and
ROLLUP function which can perform this kind of calculation much
more efficiently and elegantly.

Assuming we would are using the same people data set as above, but
we want to compute all levels of aggregation 
for gender, and sport, we can use the following code in pig.


```
cubed = CUBE people BY CUBE(gender, sport);
DUMP cubed;
```

This computes

```
((F,baseball),{(F,baseball,Mary,US,5)})
((F,football),{(F,football,Ellen,UK,5)})
((F,),{(F,,Ellen,UK,5),(F,,Mary,US,5)})
((M,baseball),{(M,baseball,Ted,UK,6)})
((M,football),{(M,football,Steve,US,6),(M,football,Alex,US,6)})
((M,),{(M,,Steve,US,6),(M,,Alex,US,6),(M,,Ted,UK,6)})
((,baseball),{(,baseball,Ted,UK,6),(,baseball,Mary,US,5)})
((,football),{(,football,Alex,US,6),(,football,Steve,US,6),(,football,Ellen,UK,5)})
((,),{(,,Alex,US,6),(,,Ellen,UK,5),(,,Mary,US,5),(,,Steve,US,6),(,,Ted,UK,6)})
```

Althrough this is hard to look at, you will see that the first entry
is are the values for all of the columns which are being cubed over,
and the second column are bags of all the rows.
This is almost identical to the way GROUP BY works, except there
are additional columns with NULL values corresponding to buckets
which aggregate over that quantity.

Now, to compute the average height, we can use thecode:

```
heights = FOREACH cubed GENERATE 
    FLATTEN(group) AS (gender, sport), 
    AVG(cube.height) AS avg_height;  
DUMP heights; 
``` 

This computes

```
(F,baseball,5.0)
(F,football,5.0)
(F,,5.0)
(M,baseball,6.0)
(M,football,6.0)
(M,,6.0)
(,baseball,5.5)
(,football,5.666666666666667)
(,,5.6)
```

Note that by default, CUBE sets the
aggregate value to NULL. I find this
confusing and prefer to use a a star 
('*') to represent the aggregate value.
We can fix this using one more line of code:

```
heights  = FOREACH heights GENERATE
    (gender is not NULL ? gender : '*') as gender, 
    (sport is not NULL ? sport : '*') as sport,
    avg_height;
dump heights; 
```

Finally, we have the table we desire:

```
(F,baseball,5.0)
(F,football,5.0)
(F,*,5.0)
(M,baseball,6.0)
(M,football,6.0)
(M,*,6.0)
(*,baseball,5.5)
(*,football,5.666666666666667)
(*,*,5.6)
```

This computes the average height
for any gender and any sport, 
for any sport (with gender equal to '\*'),
and for any gender (and sport equal to '\*').
This is super cool!

# CUBE Over Multiple Dimensions


Looking again to our example from above, 
if we CUBE on (country, gender, sport),
we would compute aggregations for all
(country, gender, and sports) pairs. But we would
also find the aggregation for all (country, gender) pairs,
all (country, sports) pairs, and all (gender, sports) pairs.
We would also compute all aggregations for any
country, gender, and sport. And we would be able to compute
the average aggregating over all countries, genders, and sports.

For our exmaple, when we cube over all pairs:
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
dump heights; 
```

We get the average over all aggregations:

```
(UK,F,football,5.0)
(UK,F,*,5.0)
(UK,M,baseball,6.0)
(UK,M,*,6.0)
(UK,*,baseball,6.0)
(UK,*,football,5.0)
(UK,*,*,5.5)
(US,F,baseball,5.0)
(US,F,*,5.0)
(US,M,football,6.0)
(US,M,*,6.0)
(US,*,baseball,5.0)
(US,*,football,6.0)
(US,*,*,5.666666666666667)
(*,F,baseball,5.0)
(*,F,football,5.0)
(*,F,*,5.0)
(*,M,baseball,6.0)
(*,M,football,6.0)
(*,M,*,6.0)
(*,*,baseball,5.5)
(*,*,football,5.666666666666667)
(*,*,*,5.6)
```


# The ROLLUP Operator

The ROLLUP operator is similar to CUBED,
but will only perform hirearchitcal
aggregations. For our example,
ROLLUP of (country,gender,sport)
will aggregate over sport, then gender and sport,
and then country, gender and sport.

We can ROLLUP our example from above:

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
dump heights; 
```

When we ROLLUP by country, gender, and sport we get the following
aggregations:

```
(UK,F,football,5.0)
(UK,F,*,5.0)
(UK,M,baseball,6.0)
(UK,M,*,6.0)
(UK,*,*,5.5)
(US,F,baseball,5.0)
(US,F,*,5.0)
(US,M,football,6.0)
(US,M,*,6.0)
(US,*,*,5.666666666666667)
(*,*,*,5.6)
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
DUMP flattened_bags;
```

This flattens out all of the bags:

```
(Mary,US,F,baseball,5)
(Ellen,UK,F,football,5)
(Ellen,UK,F,*,5)
(Mary,US,F,*,5)
(Ted,UK,M,baseball,6)
(Steve,US,M,football,6)
(Alex,US,M,football,6)
(Steve,US,M,*,6)
(Alex,US,M,*,6)
(Ted,UK,M,*,6)
(Ted,UK,*,baseball,6)
(Mary,US,*,baseball,5)
(Alex,US,*,football,6)
(Steve,US,*,football,6)
(Ellen,UK,*,football,5)
(Alex,US,*,*,6)
(Ellen,UK,*,*,5)
(Mary,US,*,*,5)
(Steve,US,*,*,6)
(Ted,UK,*,*,6)
```

Now that we have generated the flattend bags,
we can group by the columns we were originally
interested in:

```
grouped = GROUP flattened_bags BY (country,gender,sport);

heights = FOREACH grouped GENERATE
    FLATTEN(group) AS (country,gender,sport),
    AVG(flattened_bags.height) AS avg_height;
DUMP heights;
```

This computes the average height for people of a given
coutnry, gender, and sport. 

```
(UK,*,*,5.5)
(UK,*,baseball,6.0)
(UK,*,football,5.0)
(UK,F,*,5.0)
(UK,F,football,5.0)
(UK,M,*,6.0)
(UK,M,baseball,6.0)
(US,*,*,5.666666666666667)
(US,*,baseball,5.0)
(US,*,football,6.0)
(US,F,*,5.0)
(US,F,baseball,5.0)
(US,M,*,6.0)
(US,M,football,6.0)
```

But it also aggregates over gender, sport, and gender and sport.

# Links

* The developer who wrote this function: http://prasanthj.info
* The Apache relese of Cube & Rollup: https://blogs.apache.org/pig/entry/apache_pig_it_goes_to

* 

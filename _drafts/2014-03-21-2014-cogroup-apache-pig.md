---
layout: post
title: "COGROUP in Apache Pig"
comments: true
permalink: cogroup-in-pig
---

The `COGROUP` command in Apache Pig is somewhat confusing
because it is sort of both a `GROUP` and `JOIN`.

In its simplest form, `COGROUP` is exactly the same as `GROUP`. 
It groups rows based on a column or set of column, and
creats bags for each group.

For example, assume we have a data set of animal owners:

```bash
$ cat > owners.csv
adam,cat
adam,dog
alex,fish
alice,cat
steve,dog
```

We can group on age using the Pig code:

```
owners = LOAD 'owners.csv' 
    USING PigStorage(',')
    AS (owner:chararray,pet:chararray);

grouped = COGROUP owners BY pet;
DUMP grouped;
```

This returns a list of ages. For each age,
Pig groups the matchign rows into bags.
The resulting `grouped` table is:

| group |                   owners |
| ----- | ------------------------ |
|   cat | {(adam,cat),(alice,cat)} |
|   dog | {(adam,dog),(steve,dog)} |
|  fish |            {(alex,fish)} |


Where `COGROUP` gets fancy is that you can `COGROUP`
on two tables at the same time.

For example, assume we also had a data set of pet names:

```bash
$ cat > pets.csv
nemo,fish
fido,dog
rex,dog
paws,cat
wiskers,cat
```

Given this table, we could compare, for example, all the people 
with cats to all the names of cats.
To do this, we cogroup based on the pet:

```
owners = LOAD 'owners.csv' 
    USING PigStorage(',')
    AS (owner:chararray,pet:chararray);

pets = LOAD 'pets.csv' 
    USING PigStorage(',')
    AS (animal:chararray,pet:chararray);

grouped = COGROUP owners BY pet, pets by pet;
DUMP grouped
```

This will group each table based on the `pet` column.
For each pet, it will a bag for all the matches in each column.
For this example, we would get:

| group |                      owners |                       pets | 
| ----- | --------------------------- | -------------------------- | 
|  cat  |    {(adam,cat),(alice,cat)} | {(paws,cat),(wiskers,cat)} |
|  cat  |    {(adam,cat),(alice,cat)} | {(paws,cat),(wiskers,cat)} |
|  dog  |    {(adam,dog),(steve,dog)} |     {(fido,dog),(rex,dog)} |
|  fish |               {(alex,fish)} |              {(nemo,fish)} |

Given this table, we can compare, for example, all the people 
with cats to all the names of cats.

{% include twitter_plug.html %}

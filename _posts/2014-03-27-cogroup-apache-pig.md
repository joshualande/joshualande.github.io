---
layout: post
title: "A Simple Explanation of COGROUP in Apache Pig"
comments: true
permalink: cogroup-in-pig
---

The `COGROUP` command in Apache Pig is somewhat confusing because
it is sort of both a `GROUP` and a `JOIN`.

## COGROUP One Table

In its simplest form, `COGROUP` is exactly the same as `GROUP`.  It
groups rows based on a column, and creates bags for each group.

For example, assume we have a data set of animal owners:

```bash
$ cat > owners.csv
adam,cat
adam,dog
alex,fish
alice,cat
steve,dog
```

We could `COGROUP` on animal using the Pig code:

```
owners = LOAD 'owners.csv' 
    USING PigStorage(',')
    AS (owner:chararray,animal:chararray);

grouped = COGROUP owners BY animal;
DUMP grouped;
```

This returns a list of animals. For each animal, Pig groups the
matching rows into bags.  The resulting table `grouped` is:

| group |                   owners |
| ----- | ------------------------ |
|   cat | {(adam,cat),(alice,cat)} |
|   dog | {(adam,dog),(steve,dog)} |
|  fish |            {(alex,fish)} |

## COGROUP Two Tables

Where `COGROUP` gets fancy is that you can `COGROUP` on two tables
at once. Pig will group the two tables and then join the two tables
on the grouped column.  For example, assume we also had a data set
of pet names:

```bash
$ cat > pets.csv
nemo,fish
fido,dog
rex,dog
paws,cat
wiskers,cat
```

Given this table, we could compare for example all the people with
a given animal to all the names of that animal.  The `COGROUP`
command is:

```
owners = LOAD 'owners.csv' 
    USING PigStorage(',')
    AS (owner:chararray,animal:chararray);

pets = LOAD 'pets.csv' 
    USING PigStorage(',')
    AS (name:chararray,animal:chararray);

grouped = COGROUP owners BY animal, pets by animal;
DUMP grouped;
```

This will group each table based on the `animal` column.  For each
animal, it will create a bag of matching rows from both tables.  For
this example, we get:

| group |                      owners |                       pets | 
| ----- | --------------------------- | -------------------------- | 
|  cat  |    {(adam,cat),(alice,cat)} | {(paws,cat),(wiskers,cat)} |
|  dog  |    {(adam,dog),(steve,dog)} |     {(fido,dog),(rex,dog)} |
|  fish |               {(alex,fish)} |              {(nemo,fish)} |

In summary, you can use `COGROUP` when you need to group two tables
by a column and then join on the grouped column.

{% include twitter_plug.html %}

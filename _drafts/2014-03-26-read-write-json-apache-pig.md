---
layout: post
title: "How to Read and Write JSON-formatted Data Using Apache Pig"
comments: true
permalink: read-write-json-apache-pig
---

In this post, I will explain how to use the `JsonStorage` and
`JsonLoader` objects in Apache Pig to read and write 
[JSON](www.json.org)-formatted data.

# Loading JSON-Formatted Data With JsonLoader

The basic schema required by Pig to read and write data is faily straightforward.
Each row in the file should be a JSON object where the keys specify the column name
and the value specifies the value of the object.

For example, a schema specifying only a single column might look like:

The file: `first_table.json`:

```json
{"col1":"Tacos"}
{"col1":"Tomato Soup"}
{"col1":"Grilled Cheese"}
```

We could load the file using the `JsonLoader`. Note, we need to specify
the schema of the data we are reading into Pig. For our example, the schema
is `col1:chararray`.

```
first_table = LOAD 'first_table.json' USING JsonLoader('col1:chararray');
```

This creates the table:

|           col1 |
| -------------- |
|          Tacos |
|    Tomato Soup |
| Grilled Cheese |


The file `second_table.json`

```json
{"food":"Tacos", "person":"Alice", "amount":3}
{"food":"Tomato Soup", "person":"Sarah", "amount":2}
{"food":"Grilled Cheese", "person":"Alex", "amount":5}
```

We can read this in by specifying a slightly more complicated schema:

```
second_table = LOAD 'second_table.json' USING JsonLoader('food:chararray, person:chararray, amount:int');
```

This creates the table:

|           food | person | amount |
| -------------- | ------ | ------ |
|          Tacos |  Alice |      3 |
|    Tomato Soup |  Sarah |      2 |
| Grilled Cheese |   Alex |      5 |


# Reading Nested Data

The next is: `third_table.json`:

```json
{"recipe":"Tacos",
 "ingredients": [
    {"name":"Beef"},
    {"name":"Lettuce"},
    {"name":"Cheese"},
  "inventor": {"name":"Alex, "age": 25}
]}
{"recipe":"Tomato Soup",
 "ingredients": [
    {"name":"Tomatoes"},
    {"name":"Milk"}
  "inventor": {"name":"Steve, "age": 23}
]}
```

```
recipes_denormalized = LOAD 'recipes_denormalized.json'
    USING JsonLoader('recipe:chararray, ingredients: {(name:chararray)}, inventor: (name:chararray, age:int)');
```


# Writing JSON-Formatted Data in Pig

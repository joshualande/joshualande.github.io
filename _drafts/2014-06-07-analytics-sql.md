---
layout: post
title: "Analytics in SQL: Practice Your Skills"
comments: true
permalink: "analytics-sql"
---

## Example Engagement Database Design

We can imagine an example schema for a database which contains user engagement data. Our database has a list of clients, user actions, and users, and a table showing every engagement event for each users.

First, we have a table of all clients (`dim_clients`):

| client_id | client_name |
| --------- | ----------- |
|         0 |      iphone |
|       ... |         ... |

Next, we have a table of all user actions (`dim_action`):

| action_id | action_name |
| --------- | ----------- |
|         0 |  front_page |
|       ... |         ... |

Next, we have a table of all users (`dim_users`):

| user_id | user_name |
| ------- | --------- |
|       0 |     Steve |
|     ... |       ... |


Finally, we have a table listing all engaements for all users (`fact_engagement`):

|  date_id | user_id | client_id | action_id |
| -------- | ------- | --------- | --------- |
| 20140503 |       2 |         2 |         1 |
|      ... |     ... |       ... |       ... |

## Create these tables:

Below are the commands to create this database in MySQL.

First we have to create the database to work in:

```sql
CREATE DATABASE engagement_database
USE engagement_database
```

Next, we can create the four tables we need:

```sql
CREATE TABLE dim_clients (
  client_id INT NOT NULL,
  client_name VARCHAR(30) NOT NULL,
  PRIMARY KEY (client_id)
);

CREATE TABLE dim_actions (
  action_id INT NOT NULL,
  action_name VARCHAR(30) NOT NULL,
  PRIMARY KEY (action_id)
);

CREATE TABLE dim_users (
  user_id INT NOT NULL,
  user_name VARCHAR(30) NOT NULL,
  PRIMARY KEY (user_id)
);

CREATE TABLE fact_engagement (
  date_id INT NOT NULL,
  user_id INT NOT NULL,
  client_id INT NOT NULL,
  action_id INT NOT NULL
)
```
Note, there is no PK on `fact_engagement`. This implies that a user can 
perform an action multiple times per day. Typically, there would also be a 
timestamp column and the other columns would be part of a PK to 
allow for duplicate actions in a day without allowing totally duplicate rows

Finally, we can populate this table with made up data using the commands from [here](https://gist.github.com/joshualande/d194f84f1ce80e1a4e2e)

## Practice Analytics Questions

Below are some practice analytics questions to get you into the habbit of thinking in terms of SQL:

### Compute The Most Popular Action


<!--
SELECT b.action_name, COUNT(a.action_id) AS num_action
FROM fact_engagement AS a
JOIN dim_actions AS b
ON a.action_id = b.action_id
GROUP BY a.action_id

---
action_name	num_action
front_page	1881
search	789
profile	330
-->

Over all time, what actions are most popular.

### Compute Most Active User 

Compute Most Active User in a Given Day

<!--

SELECT date_id, user_id
### Compute Daily Active Users (DAUs)

A user is active if they perform any engagement in a given day
Compute this number for every day

<!--
SELECT date_id, COUNT(distinct user_id)
FROM fact_engagement
GROUP BY date_id
--->

### Engagement Summary Table

For each action, compute number of actions per day and number of unique users who compute that action

<!--
SELECT date_id, action_id, COUNT(*), COUNT(DISTINCT user_id)
FROM fact_engagement
GROUP BY date_id, action_id
ORDER BY date_id DESC
---
date_id	action_id	COUNT(*)	COUNT(DISTINCT user_id)
20130228	0	81	8
20130228	1	45	5
20130228	2	12	3
-->

### Compute Monthly Active Users (MAUs)

A user counts as a MAU if they performed any engagement in any
day in the last 30 days. Compute this number for the most recent
day in the database.


<!--
SELECT COUNT(distinct user_id)
FROM fact_engagement
WHERE date_id <= 20130228 AND date_id > 20130228 - 30
---
MAUs
10
-->

Next, compute this for every day separately using one SQL query:

<!--
SELECT a.date_id, COUNT(DISTINCT b.user_id)
FROM fact_engagement AS a
JOIN fact_engagement AS b
ON b.date_id <= a.date_id
AND a.date_id < CAST(DATE(b.date_id) + INTERVAL 30 DAY AS UNSIGNED INTEGER)
GROUP BY a.date_id
ORDER BY a.date_id DESC
---
20130228	10
20130227	10
20130226	10
...
20130111	10
20130110	10
20130109	10
20130108	9
20130107	9
20130106	9
20130105	9
20130104	9
20130103	8
20130102	7
20130101	3
-->


### Number of Login Days

For each user in the last month, compute the number of days that the user logged in.

<!--
SELECT user_id, COUNT(DISTINCT date_id)
FROM fact_engagement
WHERE date_id <= 20130228 AND date_id > 20130228 - 30
GROUP BY user_id
-->

Next, for each rolling 30 day period, compute the number of login days.


### Compute Signups

For each User, compute the day they signed up.
This is defined as the first day they logged in.

<!--
SELECT DISTINCT user_id, MIN(date_id) as signup
FROM fact_engagement
GROUP BY user_id
ORDER BY user_id DESC
---
user_id	signup
9	20130101
8	20130102
7	20130102
6	20130109
5	20130103
4	20130102
3	20130104
2	20130102
1	20130101
0	20130101
--->

### Multiple Clients

For each client pair, compute number of users who have used both clients

<!--
SELECT a.client_id, b.client_id, COUNT(distinct a.user_id)
FROM fact_engagement AS a
JOIN fact_engagement AS b
ON a.user_id = b.user_id
AND a.client_id <= b.client_id
GROUP BY a.client_id,b.client_id

---

client_id	client_id	n_users
0	0	10
0	1	10
0	2	10
1	1	10
1	2	10
2	2	10
-->


### Compute Top Client For Each User

For the top client for each user for each day

<!--
SELECT *
FROM fact_engagement
GROUP BY (date_id, user_id)
-->

### Compute Churn

Compute all fraction of users who in in the first month but didn't
log in in the second month.

If you can get this to work for static dates, compute a rolling
count of churn (for every day, the fraction of users in the previous
60-30 day period who didn't log in in the last 30 days).


### Filter Buggy Data

...



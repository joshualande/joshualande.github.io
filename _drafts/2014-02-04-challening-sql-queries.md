---
layout: post
title: "Challenging SQL Questions That Will Make You Better With Databases"
comments: true
---

In this post, I will ask some questions that require issuing challenging
SQL queries. These questions should challenge your ability to
join, filter, aggregate, and use subqueries when querying
databases. By working through these examples, you should
be able to improve your SQL skills.

## Database Schema

To be concrete,
for this post we will use the `world` database provided on
the MySQL website. Setting up the database is described 
[here](http://dev.mysql.com/doc/world-setup/en/index.html)
and the data can be downloaded 
[here](http://downloads.mysql.com/docs/world_innodb.sql.zip).


Before we begin, lets take a look at the database.
You should see that there are
three tables:


| Tables\_in\_world |
| --------------- |
|            City |
|         Country |
| CountryLanguage |

The schema of the `City` table is:

```
CREATE TABLE `City` (
  `ID` int(11) NOT NULL AUTO_INCREMENT,
  `Name` char(35) NOT NULL DEFAULT '',
  `CountryCode` char(3) NOT NULL DEFAULT '',
  `District` char(20) NOT NULL DEFAULT '',
  `Population` int(11) NOT NULL DEFAULT '0',
  PRIMARY KEY (`ID`),
  KEY `CountryCode` (`CountryCode`),
  CONSTRAINT `city_ibfk_1` FOREIGN KEY (`CountryCode`) REFERENCES `Country` (`Code`)
) ENGINE=InnoDB AUTO_INCREMENT=4080 DEFAULT CHARSET=latin1
```

The schema of the `Country` table is:

```
CREATE TABLE `Country` (
  `Code` char(3) NOT NULL DEFAULT '',
  `Name` char(52) NOT NULL DEFAULT '',
  `Continent` enum('Asia','Europe','North America','Africa','Oceania','Antarctica','South America') NOT NULL DEFAULT 'Asia',
  `Region` char(26) NOT NULL DEFAULT '',
  `SurfaceArea` float(10,2) NOT NULL DEFAULT '0.00',
  `IndepYear` smallint(6) DEFAULT NULL,
  `Population` int(11) NOT NULL DEFAULT '0',
  `LifeExpectancy` float(3,1) DEFAULT NULL,
  `GNP` float(10,2) DEFAULT NULL,
  `GNPOld` float(10,2) DEFAULT NULL,
  `LocalName` char(45) NOT NULL DEFAULT '',
  `GovernmentForm` char(45) NOT NULL DEFAULT '',
  `HeadOfState` char(60) DEFAULT NULL,
  `Capital` int(11) DEFAULT NULL,
  `Code2` char(2) NOT NULL DEFAULT '',
  PRIMARY KEY (`Code`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1
```

And the schema of the `CountryLanguage` table is:

```
CREATE TABLE `CountryLanguage` (
  `CountryCode` char(3) NOT NULL DEFAULT '',
  `Language` char(30) NOT NULL DEFAULT '',
  `IsOfficial` enum('T','F') NOT NULL DEFAULT 'F',
  `Percentage` float(4,1) NOT NULL DEFAULT '0.0',
  PRIMARY KEY (`CountryCode`,`Language`),
  KEY `CountryCode` (`CountryCode`),
  CONSTRAINT `countryLanguage_ibfk_1` FOREIGN KEY (`CountryCode`) REFERENCES `Country` (`Code`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1
```

Notice how the threet ables are related by the `Country.Code`,
`CountryLanguage.CountryCode` and `City.CountryCode` columns.



## Question 1: Minimum Surface Area

For each region, find the minimum surface area
for countries that are in the top five for best life expectancy.

[Click Here for the answer](https://gist.github.com/joshualande/9166779)

## Question 2: Number of shared langauges.

Find the top ten countries countries with the most shared languages.
Return the full country names as well as the number of shared
languages. Sort in descending order by the number of shared lanuages.

[Answer ...]

## Question 3: Calculate the number of countries in life expectancy age bins

In buckets of 0-9 years, 10-19 years, 20-29 years, etc, calculate
the number of countries in each age bucket.

{% include twitter_plug.html %}

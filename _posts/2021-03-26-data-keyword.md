---
title: "Big Data, Database, Data Warehouse and Data Lake"
search: true
categories:
  - research 
tags:
  - database
  - big data 
  - web 
last_modified_at: 2021-03-26T22:51:00
---

# [Research] Big Data, Database, Data Warehouse and Data Lake

Meanings of the words that are used in database engineering.

## Big Data
It refers a large amount of data that is defined by 4 V's characteristics.
- Volume
- Variety 
- Velocity
- Veracity

Also, it may refer the technology to achieve value from the huge data set by managing and analyzing in proper way.

## Database
It is a collection of data that is controlled by certain system(DBMS, DataBase Management Systems).
Data is stored and managed by certain structure, like schema in RDBMS(Relational DBMS) or key-value pair.. etc.


## Data Warehouse
As its name says, it is "warehouse" of whole data that data analysts can access and use them.
From it, many technologies to store and analyze the huge amount of data has been developed.
Dividing one-big data warehouse to smaller *Data mart*, ETL(Extract, Transform, Load) tool, Columnar Database and hardware improvement etc.


## Data Lake

In the sense of storing whole data, it is somewhat similar to DW but the core concepts of Data Lake are:
> - Data maintains its own structure and format
> - There are multiple groups of users who use the data

As there's no structure and the data source can be diverse, it is important to maintain the data searchable to collect by *Data cataloging*.

## Reference
1. Alex Gorelik, *The Enterprise Big Data Lake*, O'Reilly

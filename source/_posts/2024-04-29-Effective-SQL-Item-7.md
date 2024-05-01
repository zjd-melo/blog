---
title: Effective SQL Item 7
date: 2024-04-29 10:27:22
updated: 2024-04-29 10:27:22
categories: DataBase
tags: [SQL]
description: Data Model Design Define Be Sure Your Table Relationships Make Sense
---

## 简介
You can, in theory, create any relationship you want between two tables as long as the data types of each pair of related columns are the same. But just because you can do something does not mean you should.

## Things to Remember
- Carefully examine whether it really makes sense to combine tables that appear to contain similar columns in order to simplify relationships.
- You can create a join between columns in two tables as long as the data types match (or can be implicitly casted), but a relationship is valid only if the columns are in the same domain. However, it is optimal to have the same data types on both sides of the join.
- Check whether you are in fact dealing with structured data before including it in your data model. If the data is semistructured, make the necessary provisions.
- It is usually helpful to clearly identify the goals of a data model to help you assess whether a given design justifies the added complexity or anomalies due to simplifications and the design of the applications using the data model.



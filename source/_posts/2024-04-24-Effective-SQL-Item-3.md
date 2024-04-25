---
title: Effective SQL Item 3
date: 2024-04-24 09:13:48
updated: 2024-04-24 09:13:48
categories: DataBase
tags: [SQL]
description: Data Model Design 重复分组问题 Repeating groups
---

## 简介
### repeating group pattern
一中有类似月份（Jan, Feb, Mar）、或多列（Quantity1, ItemDescription1, Price1, Quantity2, ItemDescription2, Price2...），
这种情况就被认为是重复组模式。这种模式很难查询，并且当需要添加或减少类似属性时，当前的设计只能增/删表中的列，并且也需要修改基于表的查询。

需要牢记的是：**Columns are expensive. Rows are cheap.**

>A red flag should be raised in your mind if the table design requires adding or removing columns to accommodate future data requirements with similar data. A much better design involves adding or removing rows as needed.

repeating group 最早是有 E.F.Codd 提出的，本意是指在一列中包含了多个类似的值，比如列中存放了数组，这违背了 1NF，现在被数据库设计者非正式的引用为一张表中有属性值相同的多个列。
后者并不违背 1NF，没列只有一个单一的值。后者违背 DRY 原则。

[关于repating group的一些讨论](https://stackoverflow.com/questions/23194292/normalization-what-does-repeating-groups-mean)

## Union
可以使用 union 查询处理重复分组的表。在不能规范化表的情况下（没权限修改）可以通过只读视图来达到规范化的效果。
```sql
SELECT ID AS DrawingID, Predecessor_1 AS Predecessor
FROM Assignments WHERE Predecessor_1 IS NOT NULL
UNION
SELECT ID AS DrawingID, Predecessor_2 AS Predecessor
FROM Assignments WHERE Predecessor_2 IS NOT NULL
UNION
SELECT ID AS DrawingID, Predecessor_3 AS Predecessor
FROM Assignments WHERE Predecessor_3 IS NOT NULL
UNION
SELECT ID AS DrawingID, Predecessor_4 AS Predecessor
FROM Assignments WHERE Predecessor_4 IS NOT NULL
UNION
SELECT ID AS DrawingID, Predecessor_5 AS Predecessor
FROM Assignments WHERE Predecessor_5 IS NOT NULL
ORDER BY DrawingID, Predecessor;
```

`union` 关键字要求每个 `select` 字段需要相同的类型和顺序，只需要在第一个 `select` 语句后添加 `as` 就可以了，每个 `select` 都可以有自己不同的 `where` 查询，
只需要在最后添加 `order by`。

## Things to Remember
- A goal of database normalization is the elimination of repeating groups of data and minimizing the schema change.
- By eliminating repeating groups of data, you can use indexing to prevent accidental duplication of data, and you greatly simplify any queries needed.
- Removing repeating groups of data makes the design more flexible because adding a new group simply requires adding another row of data, not changing the table design to add more columns.

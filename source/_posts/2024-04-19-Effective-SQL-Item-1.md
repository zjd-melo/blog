---
title: Effective SQL Item 1
date: 2024-04-19 16:04:40
updated: 2024-04-19 16:04:40
categories: DataBase
tags: [SQL]
description: Data Model Design 确保所有表都有主键
---

## 简介
为了遵守关系模型的要求，数据库系统能够区分表中的单行和所有其他行，因此每个表都应该有一列或一组列作为主键。
主键的值在所有行中必须唯一（unique）且非空（not null）。如果没有主键，在过滤查询时没办法确保到底匹配到了
多少行数据。当然没有主键的表也是合法的。事实上在一列或者多列上设置非空并且保证唯一性并不意味着数据库引擎
高效的利用这些列，我们必须显式的指定 `primary key`，更深一点，没有主键更谈不上在表之间建立关联关系。

如果一张表没有主键会引起很多问题，**重复**且**不一致**的数据、**慢查询**、**不准确**的报表等。

## 怎么选择主键
- Must hold unique values
- Can never be null
- Should be stable
  - 不需要或很少更新
- Should be as simple as possible 
  - 整数比浮点数和字符串好
  - 单列比多列好

达到这些目的的通常做法是使用自增的物理主键。

## Referential Integrity 
在关系型数据库中**参照完整性**是一个很重要的概念，它要求一个 child table 中的每一个非空外键在 parent 
table 中都必须有一条记录与之对应。

## Numeric vsersus text-based primary keys
一直是一个争论。

## Things to Remember
- All tables should have a column (or a set of columns) designated as a pk
- 如果担心非 key 列有重复的值，可以定义一个 unique index 确保完整性。
- 使用较少更新、值简单的列作为主键

## Primary key, unique key, foreign key

| Features | pk | uk | fk |
| --- | --- | --- | --- |
|Number of keys| One pk in a parent table | One or more than one, in parent or child tables | Tables can have multiple fks |
|Values|Must have a value, cannot be null|Can be null|Can be null|
|Use|Identify every item in a table|Identify items in a table when they cannot have duplicate values| Relate to parent table |
|Ease|Can not be removed, difficult to change|Can be removed or changed easily| Can be remove from table|
|Indexes| Clustered index | Non-clustered index | None |
|In tables| Part of a parent table | Part of a table | Part of a child table, always links back to a pk in a parent table |

## Nature key、surrogate key
Nature key 自然主键/业务主键来源于业务字段，surrogate key 物理主键/代理主键/逻辑主键一般是自增序列。说法不一样。
###聚集索引
聚集索引基于数据行的键值在表内排序和存储这些数据行。每个表只能有一个聚集索引，因为数据行本身只能按一个顺序存储。
主键必须是聚集索引（即便是没有主键或者唯一约束字段，会自动生成一个主键做聚集索引），这一点没有商量的余地，因此，在MySQL中，“为什么InnoDB表最好要有自增列做主键？”这一点就显得更加有必要。
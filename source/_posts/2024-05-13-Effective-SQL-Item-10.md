---
title: Effective SQL Item 10
date: 2024-05-13 14:38:56
updated: 2024-05-13 14:38:56
categories: DataBase
tags: [SQL]
description: Programmability and Index Design Factor in Nulls When Creating Indexes
---

> 不能假设逻辑上设计良好的表结构会让你写出高效的 SQL，必须在表的物理结构上进行优化，该主题介绍索引。

## 简介

1. `NULL` 与其他任何值都不相等，包括自身，判断是否为 `NULL` 要用 `IS NULL` 或者 `IS NOT NULL` 判断。

2. 在给数据库表中某列建立索引时，需要事先了解该列是否包含 `NULL` 值，并了解使用的数据库在建立索引时如何对待 `NULL`。

3. 如果一个列中存在大量的 `NULL`，在建立索引时会占用很多空间，有些数据库允许在建索引时排除 `NULL`；有些数据库会把空字符串当作 `NULL`。

4. ISO SQL 标准中规定主键不能包含 `NULL`。

5. 某些数据库在使用 unique key时，如果有两条记录的相同列包含 `NULL`，会认为是重复，可以手动忽略 `NULL` 值。

## Things to Remember

- Consider whether a column you want to index will contain null values.
- If you want to search for null values, but the majority of values in the column are likely to be NULL, it is better not to index the column. It may be also an indication that redesign of the table may be warranted.
- When you want to be able to search for values on a column more quickly, but the majority of the values will be NULL, build the index without null values if your database supports it.
- Every database system supports null values in indexes differently. Be sure you understand the options for your database system before considering building an index on a column that may contain null values.

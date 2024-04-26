---
title: Effective SQL Item 4
date: 2024-04-25 09:22:31
updated: 2024-04-25 09:22:31
categories: DataBase
tags: [SQL]
description: Data Model Design Store Only One Property per Column
---

## 简介
>In relational terminology, a relation (table) should describe one and only one subject or action. Attributes (columns) contain the data pertaining to one and only one property (often referred to as “atomic” data) that describes the subject defined by the relation. An attribute can also be a foreign key containing an attribute from another relation, and this foreign key provides the relationship to some tuple (row) in another relation.

列只保存原子数据，比如地址信息，需要拆分为多列（省、市、区等）。

单列中存多属性的值并不是一个好的设计，不利于查询和聚合。

## 例子
比如一张表中有存外国人的名字（first name last name） 和地址信息会导致如下问题：
- 很难搜索 last name 是 smith 的，使用低效的 `like` 查询可能也会返回 SmithA 等信息
- 使用 `like` 查询 first name 可能会使用前缀索引提升效率，但是如果是 `Mr Smith` 呢，就不能使用 trailing wildcard 了，会退化为 data scan
- join 数据时很难用省去关联

## 分解
此时我们就需要把列分解为更加具体的原子数据，至于如何区分粒度，则要根据具体应用而定，这样存储、查询都会变的高效，如果想提取具体信息则可以使用字符串函数拼接数据。

## Things to Remember
- Correct table design assigns each individual property to its own column, because when a column contains multiple properties, searching and grouping become difficult if not impossible.
- For some applications, the need to filter the parts in columns such as address or phone number may dictate the level of granularity.
- When you need to reassemble properties for a report or a printed listing, use concatenation.

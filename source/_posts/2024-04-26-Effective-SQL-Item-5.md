---
title: Effective SQL Item 5
date: 2024-04-26 09:52:15
updated: 2024-04-26 09:52:15
categories: DataBase
tags: [SQL]
description: Data Model Design Understand Why Storing Calculated Data Is Usually a Bad Idea
---

## 简介
You might sometimes **be tempted to** store calculated data, especially when the calculation depends on data in a related table.

## 举例
订单表中有个字段来是有详细订单和其他表计算而来的字段，这种设计看上去很好，我们每次查总额时不需要关联其他子表进行计算了。

calculated fields 在数据仓库中可能没什么问题，但是在 OLTP 数据库中会带来很大的性能问题。并且这种设计很难维护数据完整性，子表的每次增、删、改都需要重新计算该字段并且反映到主表上。

## 触发器
好消息是现代数据库系统允许你在定义子表时写触发器来计算这种字段，但是触发器是很重的并且编写困难。

## Potentially 比触发器更好选择
现代的某些数据库系统允许你在定义表的同时定义计算字段，为什么说这种做法比触发器好呢，在定义表时定义计算字段比触发器来的简单，减少了很多复杂性（complex code of triggers）。

如果计算字段来源于同一张表可以使用表达式直接定义，如果需要关联表的字段则需要定一个函数来处理，

Note that because the function depends on data from another table, it is **nondeterministic**, so you cannot build an index on the calculated field.


## Deterministic versus Nondeterministic
对于相同的输入，确定性函数每次调用都会返回相同的结果，非确定性函数返回结果会不一样。

## 缺点
使用非确定性函数作为计算列/虚拟列是一个很不好的设计，这个列并不能像其他列一样持久化存储，并且不能对它构建索引（有些数据库可以），也会遇到很多性能问题。

## Things to Remember
- 很多数据库系统允许在建表的时候使用求值字段，必须考虑性能问题，尤其是使用了非确定性函数的时候
- 可以把求值字段当作普通列存储，然后使用触发器去维护，但是触发器代码很难维护
- 求值字段会给系统带来额外开销，只有在明确这样做的收益超过开销时才能这样做
- 大多数时候，会希望在计算列上创建索引，以获得一些好处，以换取增加的存储空间和较慢的更新速度
- Using views to define calculations is often a desirable alternative to actually storing calculations on a table for cases where indexing does not apply

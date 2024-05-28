---
title: Effective SQL Item 12
date: 2024-05-22 15:56:06
updated: 2024-05-22 15:56:06
categories: DataBase
tags: [SQL]
description: Programmability and Index Design Use Indexes for More than Just Filtering
---

## 简介
数据库索引是数据库中不同的数据结构。每个索引都需要单独的磁盘空间，因为它维护了被索引表的数据，从某种意义上来说完全是
冗余的，但是这种冗余是可以接受的，它避免了每次检索查询都查找表中的每一行数据。但是索引还有其他用途。

SQL 语句中的 `WHERE` 子句定义了查询的条件，因此会利用索引的核心功能，快速定位数据。

列是否被索引会影响表之间联接的执行效率。本质上，`JOIN` 操作允许将规范化数据模型中的数据转换为
非规范化形式，以实现特定的处理目的。由于 `JOIN` 操作组合了分散在许多表中的数据，因此需要从不同
页面进行更多读取，因此它们对磁盘查找延迟特别敏感，因此正确的索引会对响应时间产生很大影响。

## JOIN
查询时使用三种常见的 `JOIN` 算法（nested loops、hash join 和 ），但所有算法的相似之处在于它们一次仅处理两个表。

涉及更多表的 SQL 查询需要多个步骤。首先，通过连接两个表形成一个中间表，然后再与另外一张表关联，以此类推。

## Nested loop join
nested loop join 是最基本的 join 算法。将其视为两个嵌套查询：外部（驱动）查询从一个表中获取
结果，第二个查询从另一个表中为驱动查询的每一行获取相应的数据。因此，嵌套循环连接最适合与要连接
的列上的索引配合使用。如果驱动查询返回较小的结果集，则嵌套循环联接可提供良好的性能。否则，
优化器可能会选择不同的连接算法。

## Hash join
哈希连接将候选记录从连接的一侧加载到哈希表中，可以非常快速地从连接的另一侧探测每一行。调整哈希连
接需要与嵌套循环连接完全不同的索引方法。由于连接是使用哈希表完成的，因此无需对要连接的列建立索引。
唯一可以提高散列连接性能的索引是连接的 WHERE 谓词或 ON 谓词中的列；事实上，这是哈希连接使用索引
的唯一时刻。实际上，哈希连接的性能是通过水平（更少的行）或垂直（更少的列）减小哈希表的大小来实现的。

## Sort merge join
排序合并连接要求连接的两侧都按连接谓词进行排序。然后它像拉链一样将两个排序列表组合起来。在许多方面，
排序合并连接与散列连接类似。单独为连接谓词建立索引是没有用的，但是应该为独立条件建立一个索引，
以便一次性读取所有候选记录。然而，排序合并连接有一个独特的方面：连接顺序没有任何区别，甚至对于性能
也没有影响。对于其他算法，外连接的方向（左或右）意味着连接顺序。但是，排序合并连接的情况并非如此。
排序合并连接甚至可以同时进行左外连接和右外连接（所谓的全外连接）。尽管排序合并连接在输入排序后表现良好，
但很少使用，因为对两侧进行排序非常昂贵。但是，如果有一个与排序顺序相对应的索引，则可以完全避免排序操作，
并且排序合并连接会大放异彩。否则，因为哈希连接只需要预处理连接的一侧，所以在许多情况下它是优越的。

MySQL 只支持 nested loop join。

## 索引的其他使用
使用索引的另一种方式是通过数据clustering。集群数据意味着将连续访问的数据紧密存储在一起，因此访问它需要更少的 I/O 操作。
换句话说，性能依赖于查询的数据的物理存储。

### order by
索引也会影响 ORDER BY 子句的效率。排序是资源密集型的。虽然它通常是 CPU 密集型的，但主要问题是
数据库必须临时缓冲结果：在生成第一个输出之前必须读取所有输入。索引提供索引数据的有序表示。事实上，
索引以预先排序的方式存储数据。这允许我们使用索引来避免排序操作来满足 ORDER BY 子句。

与 join 可以使用“流水线”（中间结果中的每一行可以立即流水线到下一个 JOIN 操作，从而不需要存储
中间结果集）来减少内存使用不同，完整的排序操作必须完成在它产生第一个输出之前。

由于索引（尤其是 B 树索引）提供了索引数据的有序表示，因此我们可以将索引视为以预排序方式存储数据。
这意味着索引可用于避免满足 ORDER BY 子句所需的排序操作。事实上，有序索引不仅可以节省排序开销，
还能在不需要处理所有数据时返回第一个结果，起到了流水线的作用。

请注意，数据库可以双向读取索引。这意味着即使扫描的索引范围与 ORDER BY 子句指定的顺序完全相反，
也可以使用管道 ORDER BY。这不会影响索引对于 WHERE 子句的可用性。然而，排序方向在包含多列的索
引中可能很重要。

**MySQL ignores ASC and DESC modifiers in index declarations.**

## Things to Remember
- Whether or not columns in WHERE clauses are included in indexes has an impact on the performance of the query.
- Whether or not columns in SELECT clauses are indexed can also affect the efficiency of the query.
- Whether or not a column is indexed can affect how efficiently joins between tables get executed.
- Indexes can also have an impact on the efficiency of ORDER BY clauses.
- The existence of multiple indexes can have an impact on write operations.


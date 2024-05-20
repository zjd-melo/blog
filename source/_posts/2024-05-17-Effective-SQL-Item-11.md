---
title: Effective SQL Item 11
date: 2024-05-17 17:24:53
updated: 2024-05-17 17:24:53
categories: DataBase
tags: [SQL]
description: Programmability and Index Design Carefully Consider Creation of Indexes to Minimize Index and Data Scanning
---

## 简介
尽管增加硬件资源可能是改善查询性能的一个方法，可以通过修改查询以花更少的钱来提升性能。通常这种问题是由没有索引或者不正确的索引导致的。

通常称为 `index scan and table scans`。

> An index scan or table scan occurs when the database engine has to scan the index or data pages to find the appropriate records, as opposed to a seek, where an index is used to pinpoint the records that are needed to satisfy the query. The more data that exists, the more time index scans can take to complete.


## Index seek
`SELECT * FROM Customers WHERE CustomerID = 1` 和 `SELECT CustomerID FROM Customers WHERE CustomerID = 1`，假设 CustomerID 列有**唯一**索引，第一个查询通过 index seek 定位到数据，然后回表查询数据，第二个直接返回不需要回表。
## Index scan
`SELECT * FROM Customers WHERE CustState = 'TX'`，假设 CustState 列有索引但不是唯一索引，这就意味着，查询必须查看整个索引来找到所有满足 where 条件的行，这就是一个 `index scan`，并且查询的列不在索引中，需要 go back to table。
## Table scan
`SELECT CustomerID FROM Customers WHERE CustAreaCode = '905'` CustAreaCode 列没有索引，db engine 会查找表中的每一行。

## 说明
There might not seem to be much difference between a table scan and an index scan in many instances, because it is necessary to search through all entries in an object to find a particular value. However, the index is usually much smaller and specially designed to be scanned, so it is generally much faster to do an index scan if you want only a small proportion of the rows in the table.

但有时 table scan 有更好的查询性能，这依赖于具体情况，比如返回的行很多时。

There is a risk, though, of assuming that indexes are the solution to all data retrieval problems.

Many indexes do not speed up retrieval and can actually slow down updates. A problem is that whenever you update an indexed column, you force an update to one or more “index tables”—meaning more disk reads and writes. Because indexes are highly organized, making updates in them is often more expensive than the update to the table.

OLTP 有大量的更新，建立索引时需谨慎，而 OLAP 更新不频繁，可以使用更多的索引改善查询。

However, simply applying indexes is not a panacea.

## B-tree
B-tree 索引对查询性能贡献成都取决于索引类型，有两种索引方式：clustered index and nonclustered。

A clustered index physically sorts the table’s contents in the order of whichever columns were specified when the index was created. 
Because it is not possible to order the rows in a table in more than one way, you can have only one clustered index per table. 
In SQL Server, at least, usually a clustered index has leaf nodes that contain data directly. 
A nonclustered index has the same index structure as a clustered index, but with two important differences:
- Nonclustered indexes may be sorted differently from the table’s physical order.
- A nonclustered index’s leaf level consists of an index key plus a bookmark that points to the data, rather than containing the data.

聚簇索引叶子结点包含数据，一般一张表只有一个聚簇索引，一般是主键。

非聚簇索引性能是否比table scan 更好取决于 table size、row’s storage pattern、rows length and the percentage of rows the query returns。

A table scan often starts to perform better than a nonclustered index access when at least 10% of the rows are selected. A clustered index usually performs better than a table scan even when the percentage of returned rows is high.

## 其他因素
如果一个列很少出现在 where 中，加索引收益很小，如果一列基数很低，即大量的行拥有相同的数据，同样建力索引收益很小。

If an index will not result in the database engine reading less than a minimum percentage of the table, the engine will not use the index.(是回表查询导致走索引比直接table scan 慢吗)

此外，只有当表很大时索引才有意义，大多数数据库直接把小的表加载到内存里面。

Once a table is in memory, searching it goes quickly, no matter what you do or do not do. What **small** means depends on the number of rows, the size of each row, how it fits into a page, and how much memory your database server has available.

复合索引同样重要，如果查询需要多个列，可以考虑创建复合索引，建复合索引时的顺序很重要。

## Things to Remember
- Analyze your data so that the appropriate indexes are created to improve performance.
- Ensure that the indexes you have created are, in fact, going to be used.

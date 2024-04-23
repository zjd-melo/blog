---
title: Effective SQL Item 2
date: 2024-04-23 09:29:22
updated: 2024-04-23 09:29:22
categories: DataBase
tags: [SQL]
description: Data Model Design 消除数据冗余存储
---

## 简介
数据的冗余存储会引起很多问题，如数据一致性，增、删、改带来的异常情况，并且浪费了磁盘空间。
**Normalization is a process that involves dividing information by subject to help eliminate problems associated with storing duplicate data.**

## 举例
|SalesId|CustFirstName|CustLastName|Address|City|Phone|PurchaseDate|ModelYear|Model|SalesPerson|
|---|---|---|---|---|---|---|---|---|---|
|1|Tom|Smith|7453 BA|SH|131123|2019-01-01|2018-12-01|BMW|Dk|
|1|Tom|Smith|7435 BA|SH|131123|2019-01-01|2018-12-01|Audi|Dk|

### Inconsistent
地址信息不对
### Insertion anomaly
对于一个新上市的 Car，只有等到有人购买才能录入信息。还有其他字段存在冗余，这些冗余/重复数据会浪费
磁盘空间、内存、网络带宽甚至浪费工作人员的时间来录入这些信息，同时也带来了数据错误的风险，如上述编码错误。
### Update anomaly
比如销售人员结婚并更换了 lastName，此时就需要对整个表里面属于该销售人的记录进行更新，只有在没有数据一致性问题且没有重名的情况下才能成功，
也会带来性能问题。
### Deleteion anomaly
删除一行数据可能会删除不想删的信息。

## 解决
可以把表拆分为4张表，来解决上述问题。
1. Customers
2. Employees
3. AutomobileModels
4. SalesTransactions

|CustomerId|CustFirstName|CustLastName|Address|City|Phone|
|---|---|---|---|---|---|

|EmployeeId|SalesPerson|
|---|---|

|ModelId|ModelYear|Model|
|---|---|---|

|SalesId|CustomerId|ModelId|SalesPersonId|PurchaseDate|
|---|---|---|---|---|

通过这种设计，结构更简洁，消除了冗余数据，可以通过视图来重建原始的表。

```sql
SELECT st.SalesID, c.CustFirstName, c.CustLastName, c.Address,
       c.City, c.Phone, st.PurchaseDate, m.ModelYear, m.Model,
       e.SalesPerson
FROM SalesTransactions st
INNER JOIN Customers c
    ON c.CustomerID = st.CustomerID
INNER JOIN Employees e
    ON e.EmployeeID = st.SalesPersonID
INNER JOIN AutomobileModels m
    ON m.ModelID = st.ModelID;
```

## Things to Remember
- A goal of database normalization is the elimination of redundant data and minimizing resource use when processing data.
- By eliminating redundant data, you eliminate insert, update, and delete anomalies.
- By eliminating redundant data, you minimize the occurrence of inconsistent data.

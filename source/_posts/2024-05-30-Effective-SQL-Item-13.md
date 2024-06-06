---
title: Effective SQL Item 13
date: 2024-05-30 20:06:46
updated: 2024-05-30 20:06:46
categories: DataBase
tags: [SQL]
description: Programmability and Index Design Don’t Go Overboard with Triggers
---

绝大多数情况下都不要使用触发器。

以下情况可能适合使用触发器：
- Maintenance of duplicate or derived data
- Complex column constrains
- Complex defaults
- Inter-database referential integrity：关联关系跨越了不同的数据库系统

**note**:
In those cases where triggers are used, it might be preferable to create the triggers on views, not on the table. This can make things easier, because you may not want triggers fired during bulk import/export operations, but need them to fire when used in an application.
在 view 上使用触发器。

## Things to Remember
DRI = **Declarative Referential Integrity（声明性参照完整性)**， 即约束。
- Because performance is usually better with DRI provided through the use of constraints and with calculated columns using built-in features when you create a table, we recommend that constraints or the built-in features for calculated columns be the default approach.
- Triggers are generally not portable: it is difficult to create a trigger for one DBMS and expect it to run without modifications on another DBMS.
- Use triggers only when absolutely necessary. If possible, ensure that the triggers are idempotent.

---
title: StarRocks-注意事项
date: 2025-05-14 15:37:44
updated: 2025-05-14 15:37:44
tags: SR
categories: SR
description: SR 注意事项
---

## 注意事项

- 在 StarRocks 中，字段名不区分大小写，表名区分大小写。
- 建表时，DISTRIBUTED BY 为必填字段。
- 排序列在建表时应定义在其他列之前。
- 索引创建对表模型和列有要求。
- StarRocks 默认会给 Key 列创建稀疏索引加速查询。

## DDL

```sql
SHOW ALTER TABLE COLUMN\G;
CANCEL ALTER TABLE COLUMN FROM table_name\G;
```

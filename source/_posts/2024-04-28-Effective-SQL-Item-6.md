---
title: Effective SQL Item 6
date: 2024-04-28 09:32:14
updated: 2024-04-28 09:32:14
categories: DataBase
tags: [SQL]
description: Data Model Design Define Foreign Keys to Protect Referential Integrity
---

## 简介

>When you design a database schema correctly, you have foreign keys in many of your tables that contain the primary key value of the related parent table.

## Things to Remember

- Making foreign keys explicit helps ensure data integrity between related tables by ensuring that no child row exists without a matching parent row.
- Attempting to add a FOREIGN KEY constraint to tables that contain data will fail if data exists that violates the constraint.
- In some systems, the performance of joins may be improved because defining a FOREIGN KEY constraint automatically builds indexes. On other systems, you must take care to create an index to cover the FOREIGN KEY constraint. Even without indexes, some system's optimizer may treat a column differently and produce better query plans.

2019-10-08  2020-10-08
2020-10-08  2021-10-08   5  天假
2021-10-08  2022-10-08   6  天假
2022-10-08  2023-10-08   7  天假
2023-10-08  2024-10-08   8  天假
2024-10-08  2025-10-08   9  天假
2025-10-08  2026-10-08   10 天假

---
title: Effective SQL Item 9
date: 2024-05-13 11:23:22
updated: 2024-05-13 11:23:22
categories: DataBase
tags: [SQL]
description: Data Model Design Use Denormalization for Information Warehouses
---

## 简介
数仓环境可能需要 Denormalization。存储计算字段等。数仓环境一般用于查询，写入一般没有很高的要求。

## Things to Remember
- Decide what data to duplicate and why.
- Plan how to keep the data in sync.
- Refactor the queries to use the denormalized fields.

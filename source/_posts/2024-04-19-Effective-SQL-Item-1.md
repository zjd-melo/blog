---
title: Effective SQL Item 1
date: 2024-04-19 16:04:40
updated: 2024-04-19 16:04:40
categories: DataBase
tags: [SQL]
description: Data Model Design 之所有表都需要有主键
---

为了遵守关系模型的要求，数据库系统能够区分表中的单行和所有其他行，因此每个表都应该有一列或一组列作为主键。
主键的值在所有行中必须唯一（unique）且非空（not null）。
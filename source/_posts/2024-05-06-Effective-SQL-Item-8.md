---
title: Effective SQL Item 8
date: 2024-05-06 19:43:48
updated: 2024-05-06 19:43:48
categories: DataBase
tags: [SQL]
description: Data Model Design Define When 3NF Is Not Enough, Normalize More
---

## 简介
A common myth is the idea that third normal form is usually sufficient for most applications. Many practitioners have heard and quoted that “3NF is usually enough,” or maybe “Normalize until it hurts, then denormalize until it works.

## Things to Remember
- Higher normal forms are likely to be already achieved in most data models. Therefore, you need to watch for cases where higher normal forms are explicitly violated. It is more likely for tables that have composite keys or participate in several many-to-many relationships.
- Fourth normal form can be violated by the special case where all possible combinations of two unrelated attributes on an entity must be enumerated for that entity.
- Fifth normal form deals with ensuring that all join dependencies are implied by candidate keys, meaning that you should be able to constrain what are valid values for a candidate key based on the individual elements. This can happen only if the key is composite.
- Sixth normal form deals with reducing the relations to only one non-key attribute generally, thus resulting in an explosion of tables, but enabling us to never need to define a nullable column.
- Testing for lossless decomposition can be an effective tool for detecting if your table violates higher normal forms.

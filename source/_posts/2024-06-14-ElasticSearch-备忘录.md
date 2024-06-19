---
title: ElasticSearch 备忘录
date: 2024-06-14 16:03:42
updated: 2024-06-14 16:03:42
categories: Search
tags: [ElasticSearch]
description: ES 备忘录
---

## ES 不会平衡单节点上的数据
> 
Elasticsearch does not balance shards across a node’s data paths. High disk usage in a single path can trigger a high disk usage watermark for the entire node. If triggered, Elasticsearch will not add shards to the node, even if the node’s other paths have available disk space. If you need additional disk space, we recommend you add a new node rather than additional data paths.


---
title: ElasticSearch 备忘录
date: 2024-06-14 16:03:42
updated: 2024-06-14 16:03:42
categories: Search
tags: [ElasticSearch]
description: ES 备忘录
---

## ES 不会平衡单节点上的数据
> Elasticsearch does not balance shards across a node’s data paths. High disk usage in a single path can trigger a high disk usage watermark for the entire node. If triggered, Elasticsearch will not add shards to the node, even if the node’s other paths have available disk space. If you need additional disk space, we recommend you add a new node rather than additional data paths.

ES 节点在存在多个数据目录时并不会进行自动平衡操作，所以当一个数据路径达到一定的值时会触发一些机制，即使某些数据路径依然有大量存储空间，ES 也不会往那个节点分配分配了，所以不建议在节点上增加磁盘，更好的做法是增加节点。

## 集群级别的分片分配和路由设置
分片配置是指把索引分片分配到各节点上的过程，这个过程会发生在初始化恢复、副本分配、rebalance或者有节点加入/离开集群时。

Master 节点的一个主要功能就是决定哪个分片应该分配到哪个节点上、什么时候在节点之间移动分片来达到集群的平衡。

有如下几个设置可以控制分片分配的过程：
- 集群级别的分片分配设置控制分配和再平衡操作
- 基于磁盘的分片分配设置解释了 ES 如何考虑到磁盘可用空间来控制分片分配
- 分片分配感知强制感知控制分片如何在不同的机架或空间/区域上分配
- 集群级别的分配分配过滤允许某些节点或者一组节点不参与分片分配，好让这些节点退役


## 集群级别的分片分片设置
可以用以下设置来控制分片的分配和恢复
- cluster.routing.allocation.enable
  - all 默认值 允许所有类型的分片的分配
  - primaries 只允许主分片分配
  - new_primaries 只允许新创建的索引的主分片分配
  - none 不允许任何索引分片进行分片分配


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
### cluster.routing.allocation.enable
- all 默认值 允许所有类型的分片的分配
- primaries 只允许主分片分配
- new_primaries 只允许新创建的索引的主分片分配
- none 不允许任何索引分片进行分片分配
  
### cluster.routing.allocation.node_concurrent_incoming_recoveries
一个节点允许发生多少并发写入分片恢复。传入恢复是在节点上分配目标分片（很可能是副本，除非分片正在重新定位）的恢复。

### cluster.routing.allocation.node_concurrent_outgoing_recoveries
一个节点上允许发生多少并发传出分片恢复。传出恢复是在节点上分配源分片（很可能是主分片，除非分片正在重新定位）的恢复。

### cluster.routing.allocation.node_concurrent_recoveries
一个设置以上两个参数的快捷方式参数。

### cluster.routing.allocation.node_initial_primaries_recoveries
虽然副本的恢复通过网络进行，但节点重新启动后未分配的主节点的恢复使用本地磁盘中的数据。这些应该很快，以便更多的初始主恢复可以在同一节点上并行发生。

### cluster.routing.allocation.same_shard.host
允许根据主机名和主机地址执行检查，以防止在单个主机上分配同一分片的多个实例。默认为 false，表示默认不执行任何检查。仅当在同一台计算机上启动多个节点时，此设置才适用。

## 分片平衡设置
可以通过以下参数控制集群中分片平衡过程。
### cluster.routing.rebalance.enable
为特定类型的分片启用或禁用重新平衡
- all 允许所有类型的分片参与平衡
- primaries 只允许主分片参与平衡
- replicas 只允许副本分片参与平衡
- none 不允许任何索引及任何分片参与平衡

### cluster.routing.allocation.allow_rebalance
用来控制rebalance触发条件
- always  始终允许重新平衡；
- indices_primaries_active  仅在所有主分片都成功被分配后；
- indices_all_active （默认）仅当所有分片都成功被分配后；

### cluster.routing.allocation.cluster_concurrent_rebalance
允许集群级别的并发分片平衡数，这个参数只限制由于集群中分片不平衡导致的再平衡过程，并不限制由于分配过滤和强制感知导致的分片再分配。

## 分片平衡启发式设置
以下设置一起使用来确定每个分片的分配位置。当没有允许的重新平衡操作可以使任何节点的权重更接近任何其他节点的权重超过 balance.threshold 时，集群达到平衡。

### cluster.routing.allocation.balance.shard
定义节点上分配的分片总数的权重因子（浮点数）。默认为 0.45f。提高这个值会增加集群中所有节点的分片数量均衡的趋势（更加积极）。
### cluster.routing.allocation.balance.index
定义在特定节点上分配的每个索引的分片数量的权重因子（浮点数）。默认为 0.55f。提高这个值会增加集群中所有节点上每个索引的分片数量均衡的趋势。
### cluster.routing.allocation.balance.threshold
应执行的操作的最小优化值（非负浮点数）。默认为 1.0f。提高这个值将导致集群在优化分片平衡方面不那么积极。

### 举例
公式=`weight(node) = (number_of_shards_on_node * cluster.routing.allocation.balance.shard) + (number_of_indices_on_node * cluster.routing.allocation.balance.index)`
假设有三个节点A、B、C，节点上分别有以下分片和索引数量：
- 节点A：10个分片，5个索引
- 节点B：8个分片，5个索引
- 节点C：12个分片，6个索引

假设使用默认配置则：
- 节点A的权重 = (10 * 0.45) + (5 * 0.55) = 4.5 + 2.75 = 7.25
- 节点B的权重 = (8 * 0.45) + (5 * 0.55) = 3.6 + 2.75 = 6.35
- 节点C的权重 = (12 * 0.45) + (6 * 0.55) = 5.4 + 3.3 = 8.7

计算标准差得到: 0.9680，比 1 小不会触发rebalancing，如果超过阈值，则会把权重高的节点往权重低的节点迁移。

## 基于磁盘的分片分配设置
Elasticsearch 在决定是向该节点分配新分片还是主动将分片从该节点重新定位之前，会考虑该节点上的可用磁盘空间。
### cluster.routing.allocation.disk.threshold_enabled
默认true，开启基于磁盘的使用量来决定是否分配分片

### cluster.routing.allocation.disk.watermark.low 低水位线
控制磁盘使用的低水位线。默认为 85%，这意味着 Elasticsearch 不会将分片分配给磁盘使用率超过 85% 的节点。它还可以设置为绝对字节值（例如 500mb），以防止 Elasticsearch 在可用空间小于指定数量时分配分片。此设置对新创建索引的主分片没有影响，但会阻止分配其副本。
### cluster.routing.allocation.disk.watermark.high 高水位线
控制高水位线。它默认为 90%，这意味着 Elasticsearch 将尝试将分片重新分配到磁盘使用率低于 90% 的节点上。它还可以设置为绝对字节值（类似于低水位线），以便在节点的可用空间小于指定数量时将分片重新定位到远离节点的位置。此设置会影响所有分片的分配，无论之前是否已分配。
### cluster.routing.allocation.disk.watermark.flood_stage 范洪水位线
控制洪水阶段水位线，默认为95%。 Elasticsearch 在节点上分配了一个或多个分片且至少有一个磁盘超过泛洪阶段的每个索引上强制执行只读索引块 (index.blocks.read_only_allow_delete)。此设置是防止节点耗尽磁盘空间的最后手段。当磁盘利用率低于高水位线时，索引块会自动释放。
会阻止事件写入，可能会对一些系统索引产生影响导致集群不可用。

百分比和具体值不能混用，百分比是已使用空间，具体值表示剩余空间。


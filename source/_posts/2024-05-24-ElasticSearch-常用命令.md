---
title: ElasticSearch 常用命令
date: 2024-05-24 15:03:06
updated: 2024-05-24 15:03:06
categories: Search
tags: [ElasticSearch]
description: ES 常用命令
---

```shell
# 集群配置
GET _cluster/settings?include_defaults&flat_settings

GET _cluster/settings
# 查看索引
GET _cat/indices?v&s=index:desc

# 查看scroll context
GET /_nodes/stats/indices/search

# 查看搜索线程池使用情况
GET /_cat/thread_pool/search?v&h=id,name,active,queue,rejected,completed,host,psz,qs,type

# 查看fielddata使用情况
GET _cat/fielddata?v&s=size:desc

# 删除某个索引的fielddata
POST event_item/_cache/clear?fielddata=true

# 删除集群中某个字段的filelddata
POST _cache/clear/?fields=_id&fielddata=true

# 查看各个节点熔断器的情况
GET _nodes/stats/breaker?human=true

# 查看segmets
GET _cat/segments/event_app?v

# 查看所有索引的fielddata
GET _stats/fielddata?human=true&format=json

GET event_loan_2021/_stats/fielddata?human=true

GET _cat/segments/event_loan?v

GET _cat/segments/event_loan_2021?v&h=index,shard,segment,size,size.memory

GET _cat/nodes/?help=true
GET _cat/nodes/?v&h=rcm,rti

GET _cluster/state/metadata?pretty&filter_path=**.stored_scripts

```

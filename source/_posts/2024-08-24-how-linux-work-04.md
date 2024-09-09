---
title: how linux work 04
date: 2024-08-24 13:42:04
updated: 2024-08-24 13:42:04
categories: 操作系统
tags: [linux]
description: how linux work 04 - Disks and filesystems 
---

## 概要
![disk](disk.png)

分区是整个磁盘的细分。在 Linux 上，它们在整个块设备后面用数字表示，因此它们的名称类似于 /dev/sda1 和 /dev/sdb3。内核将每个分区呈现为块设备，就像整个磁盘一样。分区是在磁盘的一个小区域上定义的，称为分区表（也称为磁盘标签）。是否要分区根据实际情况定，最好系统一个分区，方便后续的升级。

## Partitioning Disk Devices
有很多类型的分区表，关于分区表并没有什么特殊地方，它仅仅是一些数据描述了磁盘上的 blocks 是如何划分的。

传统的表可以追溯到 PC 时代，是在主引导记录 Master Boot Record (MBR) 中找到的表，它有很多限制。大多数较新的系统都使用全局唯一标识符分区表 Globally Unique Identifier Partition Table（GPT）。

下面是一些 Linux 分区工具：
- parted partition editor 支持 MBR 和 GPT 的基于文本的分区工具
- gparted parted 的图形化界面版本
- fdisk 文本的命令行工具，支持 MBR 和 GPT

fdisk 是交互式的，在真正变更前，不会对系统进行更改。

分区和文件系统操作之间存在一个关键区别：分区表定义了磁盘上的简单边界，而文件系统是一个复杂得多的数据系统。因此，要使用单独的工具来分区和创建文件系统。

### 查看一个分区表
`parted -l` 或者 `fdisk -l` 查看分区表。

```shell
Model: Virtio Block Device (virtblk)
Disk /dev/vda: 85.9GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos  # MBR
Disk Flags: 

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  85.9GB  85.9GB  primary  ext4         boot
                                主分区
```

### MBR 分区表
MBR 分区表，包括 priamry、extended 和 logical 分区，主分区就是普通的磁盘上的分区。MBR 只支持 4 个主分区，如果想要更多的分区，就要把其中的一个主分区设计为扩展分区，在扩展分区内划分逻辑分区。
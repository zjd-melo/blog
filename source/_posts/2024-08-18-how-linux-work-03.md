---
title: how linux work 03
date: 2024-08-18 14:03:54
updated: 2024-08-18 14:03:54
categories: 操作系统
tags: [linux]
description: how linux work 03 - devices
---

> 本章是对正常运行的 Linux 系统中内核提供的设备基础结构的基本介绍。纵观 Linux 的历史，内核向用户呈现设备的方式发生了许多变化。我们首先查看传统的设备文件系统，了解内核如何通过 `sysfs` 提供设备配置信息。我们的目标是能够提取有关系统上设备的信息，以便了解一些基本操作。后面的章节将更详细地介绍与特定类型设备的交互。了解新设备出现时内核如何与用户空间交互非常重要。 `udev` 系统使用户空间程序能够自动配置和使用新设备。将了解内核如何通过 `udev` 向用户空间进程发送消息的基本工作原理，以及进程如何处理该消息。

## Device files 设备文件
在 Unix 系统上操作大多数设备都很容易，因为内核将许多设备 I/O 接口作为文件呈现给用户进程。一切皆文件。这些设备文件有时称为设备节点 `device nodes`。程序员不仅可以使用常规文件操作来处理设备，而且某些设备也可以通过标准程序（例如 cat）访问，因此不必是程序员亦可使用设备。但是，使用文件接口可以执行的操作是有限的，因此并非所有设备或设备功能都可以通过标准文件 I/O 访问。Linux 使用与其他 Unix 风格相同的设备文件设计。设备文件位于 /dev 目录中，运行 ls /dev 会显示 /dev 中的多个文件。那么如何使用设备呢？

`echo hello > /dev/null`，输出到文件，这个文件代表一个设备，由内核决定如何处理。

## 设备类型
`ls -l /dev` 查看第一个字符，标识了设备类型。
### Block device
Programs access data from a `block device` in fixed chunks。程序以固定块的形式从块设备访问数据。磁盘就是典型的块设备文件，磁盘可以轻松分割成数据块。由于块设备的总大小是固定的并且易于索引，因此进程可以在内核的帮助下随机访问设备中的任何块。

### Character device
Character devices work with data streams。只能从字符设备读取字符或向字符设备写入字符，如前面使用 `/dev/null` 那样。字符设备是没有大小的，当从字符设备读写时，内核通常向设备发起读写命令。直接连接到计算机的打印机由字符设备表示。需要注意的是，在字符设备交互期间，内核在将数据传递到设备后无法备份和重新检查数据流。不可重复读写多次数据流。

### Pipe device
Named pipes 命名管道就像字符设备，I/O 流的另一端有另一个进程而不是内核驱动程序。

### Socket device
套接字是经常用于进程间通信的特殊用途接口。它们通常位于 /dev 目录之外。套接字文件代表 Unix 域套接字。

Unix Domain Sockets (UDS) 是一种在同一台主机上的进程之间进行通信的机制。与通过网络进行的通信不同，Unix Domain Sockets 是专为在同一台机器上运行的进程间通信（IPC）设计的。它们提供了一种非常高效的通信方式，不需要通过网络协议栈，因此比基于 TCP/IP 的通信速度更快，开销更小。

并非所有设备都有设备文件，因为块和字符设备 I/O 接口并不适合所有情况。例如，网络接口没有设备文件。理论上可以使用单个字符设备与网络接口进行交互，但由于这非常困难，因此内核使用其他 I/O 接口。

## sysfs device path
传统的 Unix `/dev` 目录是用户进程引用和连接内核支持的设备的一种便捷方式，但它也是一个非常简单的方案。`/dev` 中的设备名称告诉我们有关该设备的一些信息，但不是很多。另一个问题是内核按照设备被发现的顺序分配设备，因此设备在重新启动之间可能有不同的名称。

为了根据设备的实际硬件属性提供统一的视角，Linux 内核通过文件和目录系统提供 `sysfs` 接口，`/sys/devices`。比如 `devices/pci0000:00/0000:00:06.0/virtio1/block/vda/vda1`。

正如所看到的，与同样是目录的 `/dev/vda` 文件名相比，该路径相当长。但你无法真正比较这两条路径，因为它们有不同的目的。 `/dev` 文件的存在是为了让用户进程可以使用设备，而 `/sys/devices` 路径用于查看信息并管理设备。

这个目录中的文件是给程序使用的而不是用户。`cat longpath/dev` 可以看到和 `/dev/vda` 一样的设备号。 

`/sys` 目录中有一些快捷目录，比如 `/sys/block` 包含所有的块设备，其实就是一些软连接。

在 /dev 中找到设备的 sysfs 位置可能很困难。使用 udevadm 命令显示路径和其他属性：
`udevadm info --query=all --name=/dev/vda`。

### sysfs
sysfs 是 Linux 内核中的一个虚拟文件系统，旨在通过文件和目录的形式向用户空间提供系统内核对象的信息和配置接口。它将内核内部的各种子系统、设备、内核模块等信息以层次化的方式暴露出来，使得用户和系统管理工具能够方便地查看和修改系统的状态。

sysfs 的特点：
- 虚拟文件系统：sysfs 是一个虚拟文件系统，类似于 /proc 和 /dev 文件系统。它并不占用磁盘空间，而是内核在内存中动态生成的。
- 结构化信息：sysfs 以目录树的形式组织信息，反映了内核对象的层次结构。例如，PCI 设备、USB 设备、块设备等都可以在 sysfs 中找到它们对应的信息。
- 可读可写：sysfs 中的大部分文件是可读的，有些文件是可写的，这意味着你可以直接通过文件操作来修改内核对象的属性（需要有足够的权限）。

sysfs 的挂载点。sysfs 通常挂载在 /sys 目录下。你可以通过以下命令确认它是否已挂载： `mount | grep sysfs` 或者手动挂载 `mount -t sysfs sysfs /sys`

sysfs 的用途
- 查看系统硬件信息：可以在 /sys 目录中查看系统中的设备信息。例如，在 /sys/class/net/ 下可以看到所有的网络接口信息。
- 设备管理：sysfs 提供了与设备相关的详细信息和控制接口。比如在 /sys/block/ 目录下，你可以查看和管理系统中的块设备（如硬盘）。
- 驱动程序与模块：sysfs 还可以用于查看加载的内核模块、驱动程序及其属性。例如，你可以在 /sys/module/ 目录下查看加载的内核模块。
- 电源管理： 通过 /sys/class/power_supply/ 目录，你可以查看系统电源管理相关的信息，包括电池状态、AC 电源状态等。
- 控制设备行为：可以通过写入特定的文件来改变设备的行为。例如，写入 /sys/class/leds/led0/brightness 可以控制 LED 的亮度。

sysfs 和 udev，sysfs 通常与 udev 一起使用。udev 通过 sysfs 提供的设备信息来管理 /dev 目录中的设备节点文件。sysfs 提供的详细设备信息让 udev 能够根据设备属性创建和删除设备节点。

总结：sysfs 是一个虚拟文件系统，主要用于以文件系统的形式向用户空间暴露内核对象的信息和控制接口。它提供了系统中几乎所有设备、驱动程序、内核模块的详细信息，并允许用户在某些情况下通过修改 sysfs 中的文件来控制系统行为。sysfs 通常与 udev 协同工作，用于动态管理设备文件。通过 sysfs，用户和系统管理员可以直接访问内核对象的信息，进行系统配置和设备管理，这是 Linux 内核对外提供的重要接口之一。

## dd and devices
程序 dd 在使用块设备和字符设备时非常有用。该程序的唯一功能是从输入文件或流中读取并写入输出文件或流，可能会在此过程中进行一些编码转换。

`dd if=/dev/zero of=new_file bs=1024 count=1`，可以看到 dd 的参数命令风格和其他的不一样，它使用 `=` 号而不是 `-`。

## /dev/zero
`/dev/zero` 是 Linux 和其他类 Unix 操作系统中的一个特殊文件，它是一个伪设备文件，用于生成无限量的零字节流（0x00 字节）。读取 `/dev/zero` 的时候，系统会返回无限个零字节，而写入 `/dev/zero` 则会被系统丢弃。

主要用途：**初始化文件或内存**。

dd 命令可以通过读取 `/dev/zero` 来创建一个指定大小的文件并将其全部填充为零。这在初始化存储空间或测试磁盘性能时非常有用。`dd if=/dev/zero of=emptyfile bs=1M count=10` 这个命令会创建一个名为 emptyfile 的文件，大小为 10 MB，文件内容全部为零。

分区和文件系统初始化： 在创建新的分区或文件系统时，`/dev/zero` 可以用于清除磁盘上的数据，确保没有残留的旧数据影响新的文件系统。`dd if=/dev/zero of=/dev/sdX bs=1M` 这个命令会将 /dev/sdX 磁盘上的所有内容都写为零，通常在完全擦除磁盘时使用。

内存分配：程序可以通过 mmap 将 `/dev/zero` 映射到内存，从而快速分配和初始化一块全为零的内存区域。许多低层次的编程场景下，`/dev/zero` 可以用于内存管理。

虚拟内存分页：/dev/zero 有时也用作虚拟内存分页的一个源，来分配一块初始化为零的匿名内存。

`/dev/zero` 与 `/dev/null` 的区别
- /dev/zero：读取时返回零字节流，写入时数据被丢弃。
- /dev/null：读取时返回 EOF（表示文件结束），写入时数据被立即丢弃。

总结
/dev/zero 是一个伪设备文件，提供无限的零字节流。它常用于初始化文件或内存、分区和文件系统的清除、内存管理等场景。与 `/dev/null` 不同，`/dev/zero` 主要用于生成数据（零字节流），而不是简单地丢弃数据。

## Device name summary
有时很难找到设备的名称（例如，对磁盘进行分区时）。以下是一些找到它的方法：
1. Query udevd using udevadm
2. Look for the device in the /sys directory.
3. Guess the name from the output of the dmesg command 
4. For a disk device that is already visible to the system, you can check the output of the mount command.
5. Run cat /proc/devices to see the block and character devices for which your system currently has drivers.

这些命令中只有第一个是可靠的，但是它需要有 `udev` 才行。如果没有的话，可以尝试使用其他办法，但是得记住，内核可能没有适用于某些硬件的设备文件。

以下部分列出了最常见的 Linux 设备及其命名约定。

### Hard Disk /dev/sd*
当前Linux系统挂接的大多数硬盘都对应带有sd前缀的设备名称，例如 `/dev/sda`、`/dev/sdb` 等，这些代表了整个磁盘，内核为磁盘上的分区创建单独的设备文件，例如 `/dev/sda1` 和 `/dev/sda2`。名称的 sd 部分代表 `SCSI disk`。

小型计算机系统接口 `Small Computer System Interface` (SCSI) 最初是作为磁盘和其他外围设备等设备之间通信的硬件和协议标准而开发的。尽管大多数现代机器中并未使用传统的 SCSI 硬件，但 SCSI 协议由于其适应性而无处不在。例如，USB 存储设备就使用它来进行通信。 SATA 磁盘的情况稍微复杂一些，但 Linux 内核在与它们通信时在某个时刻仍然使用 SCSI 命令。

要列出系统上的 SCSI 设备，请使用一个实用程序来遍历 sysfs 提供的设备路径。最简洁的工具之一是 `lsscsi`。

```shell
 lsscsi
[0:0:0:0] diskv ATA WDC WD3200AAJS-2 01.0 /dev/sdaw 
[1:0:0:0] cd/dvd Slimtype DVD A DS8A5SH XA15 /dev/sr0 
[2:0:0:0] disk FLASH Drive UT_USB20 0.00 /dev/sdb
```

第一列是设备在系统上的地址，第二列描述了具体的设备是什么，最后一列是设备文件地址，其他是供应商信息。

Linux 按照其驱动程序遇到设备的顺序将设备分配给设备文件。Linux assigns devices to device files in the order in which its drivers encounter devices。所以上面的例子表明内核先发现了磁盘。然后是光盘，最后是 flash 磁盘。

这种方式是有问题的，比如，有三个磁盘 `/dev/sda`、`/dev/sdb`、`/dev/sdc`，如果 `/dev/sdb`坏了，把这个磁盘拿走后，`/dev/sdc` 就变成了 `/dev/sdb`，如果在 `fstab` 文件中用这种方式引用磁盘，机会有问题，现代的系统使用 Universally Unique Identifier (UUID) 来解决这个问题。

### CD and DVD Drivers /dev/sr*
Linux 将大多数光存储驱动器识别为 SCSI 设备 `/dev/sr0`、`/dev/sr1` 等。但是，如果驱动器使用较旧的接口，它可能会显示为 PATA 设备，如下所述。 `/dev/sr*` 设备是只读的，它们仅用于从光盘读取。对于光学设备的写入和重写功能，将使用通用 SCSI 设备，例如 `/dev/sg0`。

### PATA Hard Disks /dev/hd*
Linux 块设备 `/dev/hda`、`/dev/hdb`、`/dev/hdc` 和 `/dev/hdd` 在旧版本的 Linux 内核和旧硬件上很常见。这些是基于接口 0 和 1 上的主设备和从设备的固定分配。有时，可能会发现 SATA 驱动器被识别为这些磁盘之一。这意味着 SATA 驱动器正在兼容模式下运行，这会影响性能。检查 BIOS 设置，看是否可以将 SATA 控制器切换到其本机模式。

### Terminals: /dev/tty*, /dev/pts/*, and /dev/tty
终端是用于在用户进程和 I/O 设备之间移动字符的设备，通常用于将文本输出到终端屏幕。终端设备接口可以追溯到很久以前，当时终端是基于打字机的设备。

伪终端设备是模拟终端，它了解真实终端的 I/O 功能。但内核不是与真实的硬件对话，而是向软件提供 I/O 接口，例如你可能在其中输入大部分命令的 shell 终端窗口。

两个常见的终端设备是/dev/tty1（第一个虚拟控制台）和/dev/pts/0（第一个伪终端设备）。 /dev/pts 目录本身是一个专用文件系统。

`/dev/tty` 设备是当前进程的控制终端。如果程序当前正在读取和写入终端，则该设备是该终端的相同东西。进程不需要附加到终端。

#### Display mode and virtual console
Linux 有两种主要的显示模式：text mode 和 X Window System Server（graphics mode，usually via a display manager）。

尽管 Linux 系统传统上以文本模式启动，但现在大多数发行版都使用内核参数和临时图形显示机制（如 plymouth 等的启动启动画面）来在系统启动时完全隐藏文本模式。在这种情况下，系统会在启动过程接近结束时切换到全图形模式。

Linux 支持虚拟控制台 virtual console 来复用显示器。每个虚拟控制台可以以图形或文本模式运行。在文本模式 text mode 下，可以使用 alt-Function 组合键在控制台之间切换，例如，alt-F1 将转到 `/dev/tty1`，alt-F2 将转到 `/dev/tty2`，依此类推。其中许多可能被运行登录提示的 getty 进程占用。


### Serial Ports /dev/ttyS*
较旧的 RS-232 类型和类似的串行端口是特殊的终端设备。无法在命令行上对串行端口设备执行太多操作，因为有太多设置需要担心，例如波特率和流量控制。

### Parallel Ports: /dev/lp0 and /dev/lp1
单向并行端口设备 `/dev/lp0` 和 `/dev/lp1` 代表已被 USB 很大程度上取代的接口类型。

### Audio Devices: /dev/snd/*, /dev/dsp, /dev/audio, and More
音频设备。

### create device files
在现代 Linux 系统中，不需要创建自己的设备文件；而是创建自己的设备文件。这是通过 `devtmpfs` 和 `udev` 完成的。但是，了解它是如何完成的很有启发性，并且在极少数情况下，可能需要创建命名管道。 `mknod` 命令创建一台设备。必须知道设备名称及其主设备号和次设备号。例如，创建 `/dev/sda1` 需要使用以下命令：`mknod /dev/sda1 b 8 2`。
The b 8 2 specifies a block device with a major number 8 and a minor number 2.

如前所述，`mknod` 命令仅对于创建偶尔使用的命名管道有用。曾经，它有时对于在系统恢复期间以单用户模式创建丢失的设备也很有用。

## udev
`udev` 是 Linux 系统中的一个设备管理系统，用于动态管理设备节点和设备文件的创建和删除。它是 Linux 内核设备管理子系统的一部分，负责处理内核生成的设备事件并在用户空间执行相应的操作。

`udevd` 进程。
### 主要功能
#### 动态设备节点管理
`udev` 负责在 `/dev` 目录中创建和删除设备文件。当你插入一个新的硬件设备（如 USB 驱动器）时，`udev` 会根据设备的属性和规则自动创建相应的设备节点文件（如 `/dev/sda`），并删除设备移除时对应的节点。
#### 设备事件处理
`udev` 监听内核发送的设备事件（如设备添加、移除或属性更改），并根据预定义的规则在用户空间执行相应的动作，例如加载驱动程序、创建符号链接、设置设备权限、或触发用户定义的脚本。
#### 设备属性管理
`udev` 可以根据设备的属性（如设备类型、制造商、序列号等）应用特定的规则，以便在系统中一致地管理设备。例如，`udev` 可以确保特定设备总是分配到同一个设备节点，或者在设备插入时自动挂载文件系统。
#### udev 的工作流程
1. 设备事件生成：当设备连接到系统时，Linux 内核检测到这个事件并生成一个 `uevent`，它包含关于设备的信息，如设备的路径、类型、属性等。
2. 设备事件处理: `udev` 守护进程接收到这个 `uevent`，然后根据系统中定义的 `udev` 规则决定如何处理这个事件。规则文件通常位于 `/etc/udev/rules.d/` 和 `/lib/udev/rules.d/` 目录中。
3. 执行操作：根据匹配的规则，`udev` 可能会执行以下操作之一或多个：在 `/dev/` 目录中创建或删除设备文件。设置设备文件的权限或所有者。创建符号链接。运行指定的脚本或命令。
#### udev 规则
`udev` 规则是一些文本文件，用于定义 udev 如何处理设备事件。每条规则都包含一些匹配条件（如设备属性）和相应的操作。规则的格式如下：`ACTION=="add", KERNEL=="sd*", SUBSYSTEM=="block", RUN+="/path/to/script"`

在上面的规则中，当一个块设备（SUBSYSTEM=="block"）被添加（ACTION=="add"）且设备名匹配 sd*（例如 sda、sdb），`udev` 将运行指定的脚本。

#### udev 的历史
`udev` 是传统 Linux 设备管理方法的现代替代品。在 `udev` 出现之前，Linux 使用 `devfs` 和 `hotplug` 来管理设备节点，但这些方法都存在一些局限性。`udev` 被设计为一种更加灵活和可配置的系统，能够在动态环境中更好地管理设备节点。

#### 总结
`udev` 是 Linux 系统中一个关键的设备管理工具，负责动态创建和删除设备文件、处理设备事件、执行与设备相关的用户空间任务。它通过规则文件来灵活配置不同设备的行为，是现代 Linux 系统中不可或缺的一部分。

udev 中的 u 代表 "user"（用户），表示它是用户空间中的设备管理守护进程。udev 结合了 "user" 和 "device"，表示它是一个在用户空间中运行的设备管理工具。与内核空间不同，用户空间指的是操作系统中应用程序运行的环境。`udev` 的设计初衷就是在用户空间中动态管理设备节点，这与早期的设备管理机制（如 `devfs`）不同，它们主要在内核空间中运行。

#### udevadm
`udevadm` 是 Linux 系统中的一个命令行工具，用于管理和调试 `udev`，这是 Linux 的设备管理系统。`udev` 负责动态管理 /dev 目录中的设备文件，并在设备插入或移除时触发相应的用户空间动作。`udevadm` 工具提供了一系列子命令，可以用于监控、控制、测试和管理设备的 `udev` 规则及事件。

##### 主要功能和常见用法
1. 监控设备事件
`udevadm monitor` 子命令可以实时监控系统中设备的添加、移除、或更改事件。

`udevadm monitor` 这会在终端中显示正在发生的 `udev` 事件，适合用于调试设备连接问题或验证 `udev` 规则。

2. 查看设备属性
`udevadm info` 子命令可以用来获取设备的详细信息和属性。

`udevadm info /dev/sda` 或 `udevadm info --query=all --name=/dev/sda` 这个命令会显示与指定设备相关的所有信息，包括设备的 `udev` 属性。

3. 测试 udev 规则
`udevadm test` 可以用于测试一个设备的 `udev` 规则，而不会实际改变系统状态。

`udevadm test /sys/class/block/sda` 这个命令会模拟设备添加事件并显示 `udev` 规则的处理结果，适合用于调试 `udev` 规则。

4. 触发 `udev` 事件
`udevadm trigger` 可以手动触发 `udev` 事件，通常用于在修改 `udev` 规则后重新应用规则。

`udevadm trigger` 这个命令会重新扫描设备并触发相应的 udev 事件。

5. 控制 udev 守护进程
`udevadm control` 命令用于控制 `udev` 守护进程的行为，例如重新加载规则文件或重新启动守护进程。

`udevadm control --reload` 这个命令会重新加载所有 udev 规则，而不会影响正在运行的守护进程。

##### 总结
`udevadm` 是一个强大的工具，用于管理和调试 Linux 系统中的设备及其 `udev` 规则。通过 `udevadm`，管理员可以监控设备事件、查看设备属性、测试和触发 `udev` 规则，以及控制 `udev` 守护进程的行为。

## 怎么查看磁盘
`lsblk` `fdisk -l`

## 设备号
在 Linux 系统中，每个设备都有一个与之关联的设备号（device number），这个设备号由两个部分组成：major 设备号和 minor 设备号。这些设备号在系统中唯一标识一个设备。

1. Major Device Number（主设备号）
major 设备号用来标识设备驱动程序。每个类型的设备驱动程序都有一个唯一的 major 设备号。内核使用 major 设备号来决定应该调用哪个设备驱动程序来处理请求。例如，所有的硬盘设备可能共享同一个 major 设备号，但由不同的 minor 设备号区分具体的硬盘或分区。

2. Minor Device Number（次设备号）
minor 设备号用来标识设备驱动程序所管理的具体设备或设备实例。对于一个特定的 major 设备号，minor 设备号可以区分多个设备或设备的不同部分。例如，同一个硬盘设备的不同分区可能共享一个 major 设备号，但 minor 设备号会不同。

假设你有一个硬盘 `/dev/sda` 和它的第一个分区 `/dev/sda1`，使用 `ls -l /dev/sda*` 可以查看它们的设备号：

```shell
brw-rw---- 1 root disk 8, 0 Aug 14 10:00 /dev/sda
brw-rw---- 1 root disk 8, 1 Aug 14 10:00 /dev/sda
```

`/dev/sda` 的设备号是 8:0，其中 8 是 major 设备号，0 是 minor 设备号。

`/dev/sda1` 的设备号是 8:1，其中 8 是 major 设备号，1 是 minor 设备号。

### 工作原理
当系统访问 `/dev/sda` 时，内核会通过 major 设备号 8 找到对应的硬盘驱动程序。

该驱动程序通过 minor 设备号 0 确定它正在处理的具体设备（在这个例子中是整个硬盘）。

类似地，访问 `/dev/sda1` 时，内核同样通过 major 设备号找到硬盘驱动程序，但驱动程序通过 minor 设备号 1 确定它正在处理的是 `/dev/sda` 上的第一个分区。

### Major 和 Minor 设备号的分配
major 和 minor 设备号通常是由操作系统分配和管理的。major 设备号可能在不同的系统之间有所不同，具体分配可以参考系统文档或在 `/proc/devices` 中查看。

### 总结
- Major 设备号：用于标识设备类型或驱动程序。
- Minor 设备号：用于标识同一类型设备中的不同设备实例或分区。

这些设备号帮助内核正确地将设备请求路由到适当的设备驱动程序和具体设备实例。了解 major 和 minor 设备号对于理解 Linux 系统中的设备管理非常重要，特别是在调试设备问题时。

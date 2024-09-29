---
title: how linux work 06
date: 2024-09-22 16:07:13
updated: 2024-09-22 16:07:13
categories: 操作系统
tags: [linux]
description: how linux work 06 - How user space starts 
---

内核启动 `init`（其第一个用户空间进程）的那一刻意义重大——不仅仅是因为内存和 CPU 终于准备好进行正常的系统操作，还因为在那里你可以看到系统其余部分是如何整体构建的。

用户空间可以粗略的认为从以下顺序启动：
1. init
2. 必要的底层服务，比如 udevd 和 syslogd
3. 网络配置
4. 中层和高层服务（cron、printing 等）
5. 登陆提示符、GUI 和高层的应用，比如 web 服务

## init 简介
init 就像 Linux 系统上其他程序一样是个用户空间的程序，可以在 `/sbin` 中找到它和其他程序的二进制可执行程序包。它的主要目的是在系统上启动和停止必要的服务。

在当前绝大数 Linux 发行版中 init 都被 systemd 替代了。还有两种 init 的变种，老的 System V （sys-five）上的 sysv-init 和 Ubuntu 上的 upstart。

还有其他版本的 init，尤其是在嵌入式平台，比如 Android 有他自己的 init 系统，BSD 也有。

已经开发的不同的 init 实现来解决 System V init 中的几个缺点。要理解这些问题，就要考虑传统 init 的内部工作原理。它基本上是一系列脚本，init 按顺序逐个运行。每个脚本通常启动一个服务或配置系统的单个部分。在大多数情况下，解决依赖关系相对容易，而且通过修改脚本可以灵活地满足不寻常的启动要求。

但是，这种方案存在一些重大的局限性。这些问题可以分为两类，一个是 performance problems 和 system management hassles。最重要的几个问题如下：
1. 由于启动序列的两个部分通常无法同时运行，因此性能会受到影响。
2. 管理正在运行的系统可能很困难。启动脚本需要启动服务守护进程。要查找服务守护进程的 PID，需要使用 ps、特定于该服务的其他机制或记录 PID 的半标准化系统，例如 `/var/run/myservice.pid`。
3. 启动脚本趋向于包含需要标准化的样板代码，有时会让人很难阅读和理解脚本到底要做什么。
4. 按需服务和配置的概念很少。大多数服务在启动时启动，浪费资源。即不能按需启动。

当代的 init 系统通过改变服务的启动方式、服务监督方式和依赖项的配置方式来解决这些问题。

## 确定 init 服务实现方式
确定系统运行的 init 版本通常并不困难，只要查看 init 手册就能确定，如果还不确定可以使用如下方式：
1. 如果系统有 `/usr/lib/systemd` 和 `/etc/systemd` 目录，那么系统运行的是 systemd。
2. 如果系统有 `/etc/init` 目录并且包含若干 .conf 文件，系统可能运行的是 upsert。
3. 如果上面两个都没有，但是有 `/etc/initab` 文件，大概运行的是 System V init。

## systemd
systemd init 是 Linux 上最新的 init 实现之一。除了处理常规启动过程之外，systemd 还旨在整合许多标准 Unix 服务的功能，例如 cron 和 inetd。它是从 Apple 的 launchd 中汲取了一些灵感。

systemd 真正与其前辈相比脱颖而出的地方在于其先进的服务管理功能。与传统的 init 不同，systemd 可以在启动后跟踪单个服务守护进程，并将与服务相关的多个进程组合在一起，让用户更有能力和更深入地了解系统上运行的具体内容。

systemd 是面向目标的，在顶层，可以考虑为某些系统任务定义一个目标 goal ，称为单元 unit。单元可以包含常见启动任务的指令，例如启动守护进程，并且它还具有依赖项，即其他单元。启动（或激活）单元时，systemd 会尝试激活其依赖项，然后转到单元的详细信息。

启动服务时，systemd 不会遵循严格的顺序；相反，它会在单元准备就绪时激活它们。启动后，systemd 可以通过激活其他单元来对系统事件做出反应。

### Units and Unit Types
systemd 比以前版本的 init 更雄心勃勃的一点是，它不只是操作进程和服务；它还可以管理文件系统挂载、监视网络连接请求、运行计时器 timers 等。每项功能称为 unit type，每项特定功能（例如服务）称为单元 unit。打开单元时，即激活了它。每个单元都有自己的配置文件。

有许多类型的 units，可以通过 `man systemd` 或者 `man init` 查看。

比较重要的：
1. Service units Control the service daemons found on a Unix system
2. Target units Control other units, usually by grouping them
3. Socket units Represent incoming network connection request locations
4. Mount units Represent the attachment of filesystems to the system

其中，服务和目标单元最为常见，也最容易理解。让我们来看看它们在启动系统时是如何组合在一起的。

### Booting and Unit Dependency Graphs
当启动系统时，会激活一个默认单元，通常是一个名为 `default.target` 的目标单元，它将许多服务和 mount 单元组合在一起作为依赖项。`/usr/lib/systemd/system/default.target`。

有时可能理解单元依赖关系形成一棵树——一个单元在顶部，分支成下面的几个单元，用于引导过程的后期阶段——但它们实际上形成了一个图。引导过程后期出现的单元可以依赖于几个先前的单元，从而使依赖关系树的早期分支重新连接在一起。

在大多数系统上，default.target 是一个链接文件，链接目标是高层次的 target unit，比如一个 GUI 的用户接口。

依赖图
![default](default.png) 

### Systemd Configuration
systemd 配置文件分布在系统的许多目录中 system unit dirctory：`/lib/systemd/system` or `/usr/lib/systemd/system` 和 system configuration 目录 `/etc/systemd/system`，分别存放系统、第三方包和自定义的 unit 文件。

为避免混淆，遵守以下规则：避免更改系统单元目录，因为这是 Linux 发行版维护的。对系统配置目录进行本地更改。此一般规则也适用于整个系统。当可以选择修改 /usr 和 /etc 中的某些内容时，请始终更改 /etc。

可以通过下面的方式查看目录：
```shell
# system unit dir
pkg-config systemd --variable=systemdsystemunitdir
# system conf dir
pkg-config systemd --variable=systemdsystemconfdir
```
#### Unit files
unit files 的文件格式是从 XDG 继承过来的，很像 ini 文件格式，通过 [] 把配置分成 section。

#### Variables
有些 unit files 包含一些变量，比如：
```ini
[Service]
EnvironmentFile=/etc/sysconfig/sshd
ExecStartPre=/usr/sbin/sshd-keygen
ExecStart=/usr/sbin/sshd -D $OPTIONS $CRYPTO_POLICY
ExecReload=/bin/kill -HUP $MAINPID
```

以美元符号 ($) 开头的所有内容都是变量。尽管这些变量具有相同的语法，但它们的来源不同。$OPTIONS $CRYPTO_POLICY 可以从 EnvironmentFile 指定的文件中获取，相比之下，$MAINPID 包含服务跟踪进程的 ID。单元激活后，systemd 会记录并存储此 PID，以便以后可以使用它来操作特定于服务的进程。sshd.service 单元文件使用 $MAINPID 在想要重新加载配置时向 sshd 发送挂断 (HUP) 信号（这是处理重新加载和重新启动 Unix 守护程序的一种非常常见的技术）。

#### Specifiers
Specifier 是一种类似变量的特征，通常在单元文件中可以找到。specifier 以百分号 (%) 开头。例如，%n specifier 是当前 unit 名称，%H specifier 是当前主机名。

还可以使用说明符从单个单元文件创建单元的多个副本。一个例子是控制虚拟控制台（如 tty1 和 tty2）上的登录提示的 getty 进程集。要使用此功能，在单元名称末尾（单元文件名中的点之前）添加一个 @ 符号。例如，在大多数发行版中，getty 单元文件名为 `getty@.service`，允许动态创建单元，例如 `getty@tty1` 和 `getty@tty2`。@ 后面的任何内容都称为实例 instance。查看这些单元文件时，可能还会看到 %I 或 %i 说明符。从带有实例的单元文件激活服务时，systemd 会用实例替换 %I 或 %i 说明符以创建新的服务名称。

### systemd Operation
主要通过 systemctl 命令与 systemd 交互，该命令允许您激活和停用服务、列出状态、重新加载配置等等。

最基本的命令可帮助获取单元信息。例如，要查看系统上的 active 单元列表，使用 list-units 命令。（这是 systemctl 的默认命令，因此从技术上讲不需要 list-units 参数。）`systemctl list-units`. `--all` 查看所有 units。

`systemctl status sshd.service`，可以看到很多信息，包括一些最新的日志。

输出中一个有趣的部分是控制组 (cgroup) 名称。在前面的示例中，控制组为 `/system.slice/sshd.service`，cgroup 中的进程显示在其下方。但是，如果某个单元（例如，挂载单元）的进程已终止，还可能会看到以 systemd:/system 开头的控制组名称。可以使用 `systemd-cgls` 命令查看与 `systemd` 相关的 `cgroup`，而无需查看其余单元的状态。status 只会显示部分日志，可以使用 `journalctl --unit=unit_name` 查看全部日志。


#### 作业与启动、停止和重新加载单元的关系

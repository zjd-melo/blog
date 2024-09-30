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

### Systemd Operation
主要通过 systemctl 命令与 systemd 交互，该命令允许您激活和停用服务、列出状态、重新加载配置等等。

最基本的命令可帮助获取单元信息。例如，要查看系统上的 active 单元列表，使用 list-units 命令。（这是 systemctl 的默认命令，因此从技术上讲不需要 list-units 参数。）`systemctl list-units`. `--all` 查看所有 units。

`systemctl status sshd.service`，可以看到很多信息，包括一些最新的日志。

输出中一个有趣的部分是控制组 (cgroup) 名称。在前面的示例中，控制组为 `/system.slice/sshd.service`，cgroup 中的进程显示在其下方。但是，如果某个单元（例如，挂载单元）的进程已终止，还可能会看到以 systemd:/system 开头的控制组名称。可以使用 `systemd-cgls` 命令查看与 `systemd` 相关的 `cgroup`，而无需查看其余单元的状态。status 只会显示部分日志，可以使用 `journalctl --unit=unit_name` 查看全部日志。


#### 作业与启动、停止和重新加载单元的关系
要 activate、deactivate 和 restart units，可以使用 systemctl start/stop/restart 命令，如果修改了 units 文件，则要重新加载 units 文件，systemctl reload unit_name 重新加载某个 unit 文件，systemctl daemon-reload 重新加载所有 unit 文件。

在 systemd 中 activate、deactivate 和 restart units 通常叫做作业（jobs），这些作业表示了 unit 的状态变更，可以在系统上查看当前运行的作业，`systemctl list-jobs`，如果系统已经启动很久了，可能不会看到任何正在运行的作业，因为这些作业在启动系统时就应当完成了，不完全没有，取决于实际情况。然而，在启动时，有时可以以足够快的速度登录以查看启动非常慢的单元的作业。

这里的作业和 shell 中的 job 是不同的概念。

#### Adding Units to systemd
向 systemd 添加单元主要涉及创建、激活并可能启用单元文件。通常应将自己的单元文件放在系统配置目录 (/etc/systemd/system) 中，这样就不会将它们与发行版附带的任何单元内容混淆，并且发行版不会在升级时覆盖自己的单元文件。

如果 unit 文件中有 `Install` section，那么需要 enable 和 disable。

### Systemd Process Tracking and Synchronization
systemd 需要对其启动的每个进程拥有合理数量的信息和控制权。从历史上看，这一直很困难。A service can start in different ways; it could fork new instances of itself or even daemonize and detach itself from the original process. There’s also no telling how many subprocesses the server can spawn.

为了简单的管理已激活的单元，systemd 使用 Linux 内核提供的 cgroups 特性来更好的跟踪进程层次关系。使用 cgroups 还有助于最大限度地减少软件包开发人员或管理员为创建工作单元文件而需要做的工作。在 systemd 中，不必担心考虑所有可能的启动行为；只需要知道服务启动过程是否需要 forks。在单元文件中使用 type 选项来指定启动的行为，有两种类型的启动风格：
- Type=simple The service process doesn't fork and terminate; it remains the main service process.
- Type=forking The service forks, and systemd expects the original service process to terminate. Upon this termination, systemd assumes the service is ready.

Type=simple 选项没有考虑到服务可能需要一些时间才能启动的事实，因此 systemd 不知道何时启动任何绝对需要此类服务准备就绪的依赖单元。处理此问题的一种方法是使用延迟启动。但是，某些 Type 启动风格可以指示服务本身将在准备就绪时通知 systemd：
- Type=notify When ready, the service sends a notification specific to systemd whit a special function call
- Type=dbus When ready, the service registers itself on the D-Bus (Desktop Bus).

另一种服务启动方式是使用 Type=oneshot 指定；在此方式中，服务进程在启动后完全终止，没有子进程。它类似于 Type=simple，不同之处在于 systemd 在服务进程终止之前不考虑启动服务。任何严格依赖项在终止之前都不会启动。使用 Type=oneshot 的服务还会获得默认的 RemainAfterExit=yes 指令，因此即使服务进程终止，systemd 也会将服务视为活动状态。

最后一个选项是 Type=idle。它的工作方式与 simple 类似，但它指示 systemd 在所有活动作业完成之前不要启动服务。这里的想法只是将服务启动延迟到其他服务启动，以防止服务相互干扰输出。请记住，一旦服务启动，启动它的 systemd 作业就会终止，因此等待所有其他作业完成可确保没有其他作业启动。

### Systemd Dependencies
灵活的启动时间和操作依赖关系系统需要一定程度的复杂性，因为过于严格的规则会导致系统性能不佳和不稳定。例如，假设想在启动数据库服务器后显示登录提示，那么可以定义从登录提示到数据库服务器的严格依赖关系。这意味着如果数据库服务器发生故障，登录提示也会失败，你甚至无法登录到计算机来修复该问题！

Unix boot-time tasks 具有相当强的容错能力，并且经常会失败而不会对标准服务造成严重问题。例如，如果删除了系统的数据磁盘但保留了其 /etc/fstab 条目（或 systemd 中的挂载单元），则启动时文件系统挂载将失败。虽然此故障可能会影响应用程序服务器（例如 Web 服务器），但通常不会影响标准系统操作 standard system operation。

为了满足灵活性和容错性的需求，systemd 提供了多种依赖类型和风格。

- Requires Strict dependencies. 严格依赖，如果严格依赖的 unit 失败了，那么 systemd 会 deactivates 包含 Require 项的 unit 文件。
- Wants 仅用于激活的依赖项。激活单元后，systemd 会激活该单元的 Wants 依赖项，但它并不关心这些依赖项是否失败。
- Requisite 必须已处于活动状态的单元。在激活具有 Requisite 依赖项的单元之前，systemd 首先检查依赖项的状态。如果依赖项尚未激活，则 systemd 无法激活具有依赖项的单元。
- Conflicts Negative dependencies. 当激活具有冲突依赖项的单元时，如果对立的依赖项处于活动状态，则 systemd 会自动停用它。 同时激活冲突的单元会失败。

Wants 依赖类型尤其重要，因为它不会将故障传播到其他单元。systemd.service(5) 手册页指出，如果可能，应该使用 wants 指定依赖项，原因也很容易理解。此行为会产生更强大的系统，享受传统 init 的好处，其中较早启动组件的故障不一定会阻止后续组件启动。

#### Ordering
到目前为止所看到的依赖语法尚未明确指定顺序。例如，激活具有 Requires 或 Wants 依赖关系的大多数服务单元会导致这些单元同时启动。这是最佳选择，因为都希望尽快启动尽可能多的服务以减少启动时间。但是，有些情况下一个单元必须在另一个单元之后启动。

要按特定顺序激活单元，请使用以下依赖项修饰符：
- Before The current unit will activate before the listed unit(s).
- After The current unit activates after the listed unit(s).

使用排序时，systemd 会等到某个单元处于活动状态后，才会激活其依赖的单元。

#### Default and Implicit Dependencies
当探索依赖项（尤其是使用 systemd-analyze）时，可能会开始注意到某些单元获取了单元文件或其他可见机制中未明确说明的依赖项。最有可能在具有 Wants 依赖项的目标单元中遇到这种情况 - 会发现 systemd 会在作为 Wants 依赖项列出的任何单元旁边添加 After 修饰符。这些额外的依赖项是 systemd 内部的，在启动时计算，而不是存储在配置文件中。

添加的 After 修饰符称为默认依赖项，是单元配置的自动添加项，旨在避免常见错误并使单元文件保持较小。这些依赖项因单元类型而异。例如，systemd 不会为目标单元添加与服务单元相同的默认依赖项。这些差异列在单元配置手册页的默认依赖项部分中，例如 systemd.service(5) 和 systemd.target(5)。

#### Conditional Dependencies
可以使用条件依赖参数测试各种系统状态而不是 systemd 的 units。

比如：
- ConditionPathExists=p True if the file/path p exists in the system.
- ConditionPathIsDirectory=p True if p is a directory.
- ConditionFileNotEmpty=p True if p is a file and it is not zero-length.

如果当 systemd 尝试激活单元时，单元中的条件依赖项为 false，则该单元不会激活，尽管这仅适用于该单元所在的单元。也就是说，如果激活具有条件依赖项和一些单元依赖项的单元，则 systemd 会尝试激活这些单元依赖项，无论条件是 true 还是 false。不管条件依赖的结果是什么，都会激活当前 unit 依赖的 unit，即条件依赖是有作用域的。

#### The [install] Section and Enableing Units
到目前为止，我们一直在研究如何在依赖单元的配置文件中定义依赖项。也可以反向执行此操作，即在依赖项的单元文件中指定依赖单元。可以通过在 [Install] 部分中添加 WantedBy 或RequiredBy 参数来实现这一点。此机制允许更改单元的启动时间，而无需修改其他配置文件（例如，当不想编辑系统单元文件时）。

Enabling a unit does not activate it. enable unit 会创建符号链接 `.wants`。

Enabling a unit survives reboots.

### Systemd On-Demand and Resource Parallelized Startup
按需启动：One of systemd’s features is the ability to delay a unit startup until it is absolutely needed. The setup typically works like this:
1. 为你想要提供的系统服务创建一个 systemd unit（call it Unit A）。
2. 确定一个系统资源，比如端口号或者socket、file、device 等，Unit A使用这些资源。
3. 创建另外一个 systemd unit，Unit R，来呈现所需要的资源，这些 units 根据 types 可以分为 socket units、path units 和 device units。
4. 定义 Unit A 和 Unit R 的关系，通常通过名称隐式的指明，也可以显式的。

一旦定义了上面的文件，systemd 的处理过程如下：
1. 一旦激活了 Unit R，systemd 会监控这些资源。
2. 当任何东西试图访问资源时，systemd 会阻止该资源，并且对该资源的输入会被缓冲 buffered。
3. systemd 激活 Unit A。
4. 准备就绪后，Unit A 的服务将控制资源，读取缓冲的输入并正常运行。

需要确定以下几点：
1. 必须确保 resource unit 覆盖服务所需每一个资源，这通常不是问题，因为大多数服务只有一个入口。
2. 必须确保 resource unit 绑定到服务 unit 上，这可以是隐式的或显式的，在某些情况下，许多选项代表 systemd 执行向服务单元切换的不同方式。
3. 并非所有服务器都知道如何与 systemd 提供的资源单元交互。

例子略。

### 使用辅助 units 进行引导优化
systemd 的总体目标是简化依赖顺序并加快启动时间。资源单元（例如套接字单元）提供了一种类似于按需启动的方法。我们仍然会有一个服务单元和一个辅助单元，代表服务单元提供的资源，但在这种情况下，systemd 会在激活辅助单元后立即启动服务单元，而不是等待请求。

采用这种方案的原因是，诸如 systemd-journald.service 之类的基本启动时服务单元需要一些时间才能启动，并且许多其他单元都依赖于它们。但是，systemd 可以非常快速地提供单元（例如套接字单元）的基本资源，然后它不仅可以立即激活基本单元，还可以激活任何依赖于它的单元。一旦基本单元准备就绪，它就会控制资源。

略

### Systemd 辅助 Components
udevd journald resolved 等。

## System V Runlevels
在 Linux 系统上的任何给定时间，都会运行一组基本进程（例如 crond 和 udevd）。在 System V init 中，机器的这种状态称为其运行级别，用 0 到 6 之间的数字表示。系统大部分时间都处于单一运行级别，但当关闭机器时，init 会切换到不同的运行级别，以便有序地终止系统服务并告诉内核停止。

`who -r` 查看运行级别。

运行级别有多种用途，但最常见的用途是区分系统启动、关闭、单用户模式和控制台模式。例如，大多数系统传统上使用运行级别 2 到 4 作为文本控制台；运行级别 5 表示系统启动 GUI 登录。

runlevels 已经属于过去了。尽管 systemd 支持他们，它将运行级别视为系统的最终状态，而更倾向于使用目标单元。对于 systemd 来说，运行级别主要用于启动仅支持 System V init 脚本的服务。

## System V init
System V init 实现是 Linux 上使用最久的实现之一；其核心思想是通过精心构建的启动顺序支持有序启动到不同的运行级别。System V init 现在在大多数服务器和桌面安装中并不常见，但可能会在 7.0 版之前的 RHEL 版本以及嵌入式 Linux 环境（如路由器和电话）中遇到它。此外，一些较旧的软件包可能仅提供为 System V init 设计的启动脚本；systemd 可以使用兼容模式处理这些脚本。我们将在这里介绍基础知识。

典型的 System V init 安装包含两个组件：一个中央配置文件和一组由符号链接场扩充的大型启动脚本。配置文件 `/etc/inittab` 是一切的起点。如果有 System V init，请在 inittab 文件中查找如下行：`id:5:initdefault:`。

这暗示了默认的运行级别是 5。

nittab 中的所有行都采用以下格式，其中四个字段按以下顺序用冒号分隔：
1. 唯一标识符（短字符串，如上例中的 id）
2. 适用的运行级别号。
3. init 应该采取的行动。
4. 要执行的命令。（可选项）

要了解 inittab 文件中的命令如何工作，请考虑以下行：`l5:5:wait:/etc/rc.d/rc 5`.

这一行非常重要，因为它会触发大部分系统配置和服务。在这里，wait 操作决定了 System V init 何时以及如何运行命令：在进入运行级别 5 时运行一次 /etc/rc.d/rc 5，然后等待此命令完成后再执行其他任何操作。rc 5 命令执行 /etc/rc5.d 中以数字开头的任何内容（按数字顺序）。

除了 wait 和 initdefault 之外还有其他的 actions：
- respawn respawn 操作告诉 init 运行后面的命令，如果命令执行完毕，则再次运行它。你很可能会在 inittab 文件中看到类似这样的内容：`1:2345:respawn:/sbin/mingetty tty1`
- ctrlaltdel ctrlaltdel 操作控制在虚拟控制台上按下 `CTRL-ALT-DEL` 时系统执行的操作。在大多数系统中，这是使用关机命令的某种重新启动命令
- sysinit 在进入任何运行级别之前，init 启动时应该运行的第一件事就是 sysinit 操作。

更多 actions 查看 inittab 手册。

### System V init: Startup Command Sequence
现在看看 System V init 如何在你登录之前启动系统服务。回想一下之前的这个 inittab 行：`l5:5:wait:/etc/rc.d/rc 5`。

这一行简短的命令会触发许多其他程序。事实上，rc 代表运行命令，许多人将其称为脚本、程序或服务。但这些命令在哪里呢？

此行中的 5 告诉我们，我们正在讨论运行级别 5。这些命令可能位于 `/etc/rc.d/rc5.d` 或 `/etc/rc5.d` 中。（运行级别 1 使用 rc1.d，运行级别 2 使用 rc2.d，依此类推。）例如，可能会在 rc5.d 目录中找到以下项目

![rc](rc.png)

![rcs](rcs.png)

`rc 5` 命令会按照这些文件的顺序依次执行，请注意每个命令中的启动参数。命令名称中的大写字母 S 表示该命令应在启动模式下运行，数字（00 到 99）决定 rc 在序列中启动命令的位置。rc*.d 命令通常是启动 /sbin 或 /usr/sbin 中程序的 shell 脚本。

某些 rc*.d 目录包含以 K（代表 kill 或停止模式）开头的命令。在这种情况下，rc 使用 stop 参数而不是 start 来运行命令。你很可能会在关闭系统的运行级别中遇到 K 命令。

### The System V init Link Farm
rc*.d 目录的内容实际上是指向另一个目录 init.d 中的文件的符号链接。像这样横跨多个子目录的大量符号链接被称为链接农场。

### Starting and Stopping Services
要手动启动和停止服务，请使用 init.d 目录中的脚本。例如，手动启动 httpd Web 服务器程序的一种方法是运行 init.d/httpd start。同样，要终止正在运行的服务，您可以使用 stop 参数（例如 httpd stop）。

### Modifying the Boot Sequence
更改 System V init 中的启动顺序通常是通过修改链接场来完成的。最常见的更改是阻止 init.d 目录中的某个命令在特定运行级别运行。但是，必须小心执行此操作。例如，可以考虑删除相应 rc*.d 目录中的符号链接。但如果需要重新放回链接，可能很难记住它的确切名称。最好的方法之一是在链接名称的开头添加下划线 (_)，如下所示：`mv S99httpd _S99httpd`。此更改导致 rc 忽略 _S99httpd，因为文件名不再以 S 或 K 开头，但原始名称仍表明其用途。

要添加服务，请创建一个类似于 init.d 目录中的脚本，然后在正确的 rc*.d 目录中创建一个符号链接。

添加服务时，请在启动顺序中选择合适的位置来启动它。如果服务启动得太早，可能会因为依赖其他服务而无法工作。对于非必要服务，大多数系统管理员更喜欢使用 90 后的数字，这样可以将这些服务放在系统自带的大多数服务之后。

### run-parts
System V init 用于运行 init.d 脚本的机制已应用于许多 Linux 系统，无论它们是否使用 System V init。这是一个名为 run-parts 的实用程序，它所做的唯一一件事就是以某种可预测的顺序运行给定目录中的一堆可执行程序。可以将 run-parts 想象成一个人在某个目录中输入 ls 命令，然后运行输出中列出的所有程序。默认行为是运行目录中的所有程序，但通常可以选择某些程序而忽略其他程序。在某些发行版中，不需要对运行的程序进行太多控制。例如，Fedora 附带了一个非常简单的 run-parts 实用程序。实际并不需要了解太多。

### System V init Control
有时，需要给 init 一点提示，告诉它切换运行级别、重新读取其配置或关闭系统。要控制 system V init，请使用 telinit。例如，要切换到运行级别 3，输入 telinit 3，切换运行级别时，init 会尝试终止新运行级别的 inittab 文件中没有的所有进程，因此更改运行级别时要小心。

### systemd 和 system V 的兼容性
systemd 与其他新一代 init 系统不同的一个特点是，它试图更全面地跟踪由 System V 兼容 init 脚本启动的服务。它的工作原理如下：
1. 首先 systemd activates `runlevel<N>.target`，N 代表运行级别。
2. 对于在 `/etc/rc<N>.d` 中的每一个符号连接，systemd 在 /etc/init.d 中找到对于的脚本。
3. systemd 关联脚本名称到一个 service unit 比如 /etc/init.d/foo == foo.service。
4. systemd 激活服务单元并根据 rc<N>.d 中的名称，使用 start 或 stop 参数运行脚本。
5. systemd 尝试将脚本中的任何进程与服务单元 service unit 关联起来。

由于 systemd 与服务单元名称相关联，因此可以使用 systemctl 重新启动服务或查看其状态。但不要指望 System V 兼容模式能带来任何奇迹；例如，它仍然必须按顺序运行 init 脚本。

## Shutting Down Your System
init 控制系统如何关闭和重启。无论运行哪个版本的 init，关闭系统的命令都是相同的。关闭 Linux 机器的正确方法是使用关机命令。在 systemd 中意味着执行 shutdown units，在 system V 中是把运行级别设置为 0。关机顺序如下：

1. init asks every process to shut down cleanly.
2. If a process doesn’t respond after a while, init kills it, first trying a TERM signal.
3. If the TERM signal doesn’t work, init uses the KILL signal on any stragglers.
4. The system locks system files into place and makes other preparations for shutdown.
5. The system unmounts all filesystems other than the root. 
6. The system remounts the root filesystem read-only.
7. The system writes all buffered data out to the filesystem with the sync program.
8. The final step is to tell the kernel to reboot or stop with the reboot(2) system call. This can be done by init or an auxiliary program, such as reboot, halt, or poweroff.

略。
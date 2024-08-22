---
title: how linux work 02
date: 2024-08-18 12:31:40
updated: 2024-08-18 12:31:40
categories: 操作系统
tags: [linux]
description: how linux work 02 - shell commands and dirs
---

> shell and directory hierarchy

It is important to remember that the shell performs expansions before running commands。先变量替换在文件名替换。

环境变量和 shell 变量，环境变量对所有命令可见，即通过 `export` 导出的变量。

## 命令行快捷键
|Key|Action|
|:---|:---|
|CTRL-B|Move the cursor left|
|CTRL-F|Move the cursor right|
|CTRL-P|View the previous command (or move the cursor up)|
|CTRL-N|View the next command (or move the cursor down)|
|CTRL-A|Move the cursor to the beginning of the line vi 模式使用 0|
|CTRL-E|Move the cursor to the end of the line vi 模式使用 $|
|CTRL-W|Erase the preceding word 删除光标之前的内容直到遇到空格|
|CTRL-U|Erase from cursor to beginning of line|
|CTRL-K|Erase from cursor to end of line|
|CTRL-Y|Paste erased text (for example, from ctrl-U)|

## man
手册，`man man` 查看有哪些相关手册。

## File mode and permissions
`-rw-r--r--`，Type、user permissions、group permissions、other permissions。

other permissions 又叫 world permissions。

```shell
chmod u+x f1
chmod go+x f2
chmod u-x  f1
chmod 644 f1
```

r=4，w=2，x=1。

## linux directory hierarchy 
### /bin
包含可立即运行的程序（也称为可执行文件），包括大多数基本 Unix 命令，例如 ls 和 cp。 /bin 中的大多数程序都是二进制格式，由 C 编译器创建，但有些是现代系统中的 shell 脚本。

### /dev
包含设备文件。

### /etc
系统核心配置文件。

### /home
保存普通用户的个人目录。

### /lib
可执行程序依赖的库，两种类型的库：static and shared。

### /proc
通过可浏览的目录和文件界面提供系统统计信息。 Linux 上的许多 /proc 子目录结构都是独特的，但许多其他 Unix 变体也具有类似的功能。 /proc 目录包含有关当前正在运行的进程的信息以及一些内核参数。

### /sys
该目录与/proc 类似，都提供设备和系统接口。

### /sbin
系统可执行文件的位置。 /sbin 目录中的程序与系统管理相关，因此普通用户的命令路径中通常没有 /sbin 组件。如果不以 root 身份运行，这里找到的许多实用程序将无法工作。

### /tmp
临时目录。

### /usr
/usr 目录在 Unix 和 Linux 系统中是一个重要的目录，通常用于存储用户级的应用程序和文件，而非系统的核心部分。尽管 /usr 目录名称中的 usr 似乎与用户（user）有关，但实际上它更接近于 UNIX System Resources（UNIX 系统资源）的缩写。

#### /usr 目录的主要用途
##### 用户应用程序
- `/usr/bin`：存放用户可以执行的二进制文件（可执行程序），包括大多数标准的系统命令和工具。例如，ls、cp 等命令就位于 /usr/bin 目录中。
- `/usr/sbin`：存放系统管理员使用的二进制文件（可执行程序），这些文件通常需要超级用户权限才能执行。例如，fdisk、ifconfig 等命令就位于 /usr/sbin 目录中。
##### 库文件
- `/usr/lib`：存放与用户级程序相关的共享库文件，以及不可执行的应用程序数据，如 systemd 的服务文件。这些文件为 `/usr/bin` 和 `/usr/sbin` 中的程序提供运行时的支持。
##### 共享资源
- `/usr/share`：存放架构无关的共享数据，例如，手册页（man pages）、文档、默认配置文件、图标和其他应用程序数据文件。这些文件可以被多个应用程序或用户共享。
##### 源代码
`/usr/src`：存放内核源代码或其他软件包的源代码，系统管理员或开发人员可以在此目录下编译内核或其他软件。
##### 本地特定的用户程序
- `/usr/local`：存放本地编译和安装的程序及其相关文件，与 /usr 目录下的其他路径相比，/usr/local 是系统管理员自己手动安装的应用程序和库的默认路径，以避免与系统软件包管理器管理的文件发生冲突。它包含类似的子目录结构，如 `/usr/local/bin`、`/usr/local/lib` 等。
##### /usr 目录与其他目录的关系
- `/bin` 和 `/sbin`：这些目录包含系统引导和维护所需的基本命令和工具，而 /usr/bin 和 /usr/sbin 则包含更广泛的用户级命令和管理工具。
- `/lib`：包含核心共享库和内核模块，而 `/usr/lib` 则包含用户级应用程序所需的库文件。
- `/opt`：用于安装可选的软件包，特别是第三方应用程序，通常不通过系统包管理器安装。与 `/usr/local` 类似，/opt 也用于避免与系统管理的软件发生冲突。
##### 历史背景
最初，Unix 系统上的 /usr 目录是存放用户的 home 目录的，但随着系统的发展，用户的 home 目录被转移到 /home，而 /usr 目录则逐渐成为存放系统和用户应用程序的地方。

### /var
/var 目录在 Linux 和 Unix 系统中是一个重要的目录，专门用于存储经常发生变化的数据文件。这些文件可能包括日志文件、缓存、锁文件、临时文件、邮件队列和其他动态数据。

### /boot
Contains kernel boot loader files。

### /media
许多发行版中都可以找到可移动介质（例如闪存驱动器）的基本连接点。

### /opt
This may contain additional third-party software。

## /usr
大多数用户空间的程序和数据都在这里。如 `/usr/bin`、`/usr/lib`。
- /include C 编译器使用的头文件
- /info Contains GNU info manuals
- /local 管理员可以在其中安装自己的软件。它的结构应该类似于 / 和 /usr。
- /man 手册
- /share 

## kernel location
在 Linux 系统上，内核通常位于 `/vmlinuz` 或 `/boot/vmlinuz`。引导加载程序将此文件加载到内存中，并在系统引导时将其设置。一旦引导加载程序运行并启动内核，正在运行的系统就不再使用主内核文件。但是，会发现内核可以在正常系统操作过程中按需加载和卸载许多模块。称为可加载内核模块 `loadable kernel modules`，它们位于 `/lib/modules` 下。

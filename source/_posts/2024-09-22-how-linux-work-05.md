---
title: how linux work 05
date: 2024-09-22 11:27:59
updated: 2024-09-22 11:27:59
categories: 操作系统
tags: [linux]
description: how linux work 05 - How the linux kernel boots 
---

现在已经知道了 Linux 系统的物理和逻辑结构了，但是内核是什么，它怎么和进程一起工作？这一章介绍内核是怎么启动的，它怎么从硬盘加载到内存并启动用户进程的。

简化的启动流程视图如下：
1. 机器的 BIOS 或启动固件加载并运行启动加载程序 boot loader。
2. boot loader 从磁盘上找到内核镜像，把它加载到磁盘并启动它。
3. 内核初始化设备和相关驱动。
4. 内核挂载 root filesystem。
5. 内核启动一个叫做 `init` 的程序并设置它的进程 ID 为 1，从这里开始用户空间启动了。
6. `init` 设置系统的其他进程。
7. 在某些时候，`init` 会启动一个允许登录的进程，通常是在启动流程的末尾或接近末尾时。

## Startup Messages
传统的 Unix 系统在启动时会产生许多诊断消息，告诉你有关启动过程的信息。这些消息首先来自内核，然后来自 `init` 启动的进程和初始化过程。然而，这些信息并不漂亮或一致，在某些情况下，它们甚至没有提供太多信息。此外，硬件的改进使得内核的启动速度比以前快得多；信息闪现得如此之快，以至于很难看清正在发生的事情。正因如此，现在大多数的 Linux 发行版都用一个画面隐藏日志信息了。

查看内核启动和运行时诊断消息的最佳方法是使用 `journalctl` 命令检索内核日志。运行 `journalctl -k` 会显示当前启动的消息，但也可以使用 `-b` 选项查看以前的启动信息。

如果没有 `systemd`，可以检查日志文件，例如 `/var/log/kern.log` 或运行 `dmesg` 命令来查看内核环形缓冲区 ring buffer 中的消息。

内核启动后，用户空间启动过程通常会生成消息。这些消息可能更难以查看和审查，因为在大多数系统上，不会在单个日志文件中找到它们。启动脚本旨在将消息发送到控制台，这些消息在启动过程完成后会被删除。然而，这在 Linux 系统上不是问题，因为 `systemd` 从启动和运行时捕获通常会发送到控制台的诊断消息。

## Kernel Initialization and Boot Options
启动时，Linux 内核按照以下一般顺序进行初始化：
1. CPU inspection
2. Memory inspection
3. Device bus discovery
4. Device discovery
5. Auxiliary kernel subsystem setup (networking)
6. Root filesystem mount
7. User space start 

前两个步骤并不太引人注目，但是当内核到达设备时，就会出现依赖性问题。例如，磁盘设备驱动程序可能依赖于总线支持和 SCSI 子系统支持。然后，在稍后的初始化过程中，内核必须在启动 `init` 之前安装根文件系统。

一般来说，不必担心依赖关系，除了一些必要的组件可能是可加载的内核模块而不是主内核的一部分。有些机器可能需要在安装真正的根文件系统之前加载这些内核模块。

### 什么是 root filesystem
Root filesystem（根文件系统）是一个操作系统在启动时挂载的第一个文件系统，它包含操作系统的核心文件、配置文件、以及用于启动系统的必要文件。

特点：根目录：Root filesystem 包含了操作系统的根目录 /，这个目录是整个文件系统的顶层目录。

启动必要的文件：根文件系统必须包含启动系统所需的最基本的文件和目录，包括内核模块、初始化脚本、配置文件、基本命令（如 bash、ls、mkdir 等）。这些文件通常位于 /bin、/sbin、/etc、/lib、/dev 等目录中。

加载其他文件系统：一旦根文件系统挂载成功，系统可以通过 `/etc/fstab` 或自动挂载的方式加载其他文件系统（如 /home、/var、/usr 等）。这些文件系统可以位于不同的分区或存储设备上。

内核的依赖：当系统启动时，内核会加载根文件系统以启动操作系统的其他组件。启动过程通常由引导程序（如 GRUB）来完成，引导程序加载内核并指示它加载根文件系统。

挂载机制：**根文件系统通常在启动时通过引导加载器（boot loader）挂载**，内核启动后首先挂载根文件系统，然后才能进行后续的启动操作。

Root filesystem 的内容结构：根文件系统至少应包含以下目录结构（根据文件系统层次标准 FHS）：

- /bin：用户可执行的二进制程序，例如常见的命令 ls、cp 等。
- /sbin：系统管理员的二进制程序，例如 fsck、reboot 等。
- /etc：配置文件，存储系统和应用程序的配置，例如 /etc/fstab、/etc/passwd。
- /lib：包含共享库（动态库）和内核模块，必要时支持二进制程序的运行。
- /dev：设备文件，用于访问硬件设备。
- /tmp：临时文件存储目录，系统启动时创建。

root filesystem vs root directory:
- Root filesystem：是整个文件系统的基础，操作系统引导时最先挂载的文件系统。
- Root directory：即 /，是文件系统中的顶层目录，它也是 root filesystem 的挂载点。

使用场景：
- Embedded Systems：在嵌入式系统中，根文件系统可能被放置在只读的存储器中，例如只读内存（ROM）或闪存。
- Live CDs/USBs：类似的，Linux 的 live CD/USB 系统也包含了一个完整的 root filesystem，它包含启动系统和运行用户程序的所有必要文件。
- Root filesystem 是系统启动和运行的核心，确保系统的启动、配置和操作所需的基本环境。

## Kernel Parameters
当 Linux 内核启动时，它会收到一组基于文本的内核参数，其中包含一些额外的系统详细信息。这些参数指定了许多不同类型的行为，例如内核应生成的诊断输出量以及设备驱动程序特定的选项。

```shell
$ cat /proc/cmdline
BOOT_IMAGE=(hd0,msdos1)/boot/vmlinuz-4.18.0-305.3.1.el8.x86_64 root=UUID=659e6f89-71fa-463d-842e-ccdf2c06e0fe ro crashkernel=auto console=ttyS0,115200 console=tty0 panic=5 net.ifnames=0 biosdevname=0 intel_idle.max_cstate=1 intel_pstate=disable
```
这些参数要么是单个单词标志参数，比如 `ro` 和 `quiet`，或者是 key=value 对。许多参数都不是很重要，比如 `splash` 标志让内核启动时显示一个画面，其中一个参数是非常重要的，`root`，这个参数指定了 root filesystem 的位置。如果没有这个参数内核不能正确的启动用户空间。

`ro` 参数指示内核在用户空间启动时以只读模式挂载根文件系统。这是正常现象；只读模式确保 fsck 在尝试执行任何重要操作之前可以安全地检查根文件系统。检查后，启动过程会以读写模式重新挂载根文件系统。

当遇到一个它不理解的参数时，Linux 内核会保存该参数。内核随后在执行用户空间启动时将参数传递给 `init`。可以从这里给 `init` 参数。

## Boot Loaders
在引导过程开始时，在内核和 `init` 启动之前，引导加载程序会启动内核。引导加载程序的工作听起来很简单：它将内核从磁盘上的某个位置加载到内存中，然后使用一组内核参数启动内核。然而，这项工作比看起来更复杂。要理解原因，请考虑引导加载程序必须回答的问题：
1. 内核在哪？
2. 启动时应该向内核传递哪些内核参数？

答案（通常）是内核及其参数通常位于根文件系统上的某个位置。听起来内核参数应该很容易找到，但请记住内核本身尚未运行，通常是内核遍历文件系统来查找必要的文件，更糟糕的是，通常用于访问磁盘的内核设备驱动程序也不可用。就像先有鸡还是先有蛋的问题一样。

A boot loader 确实需要一个驱动来读取磁盘，但它和内核使用的驱动不是完全相同的。在 PCs 上，boot loaders 使用传统的 Basic Input/Output System（BIOS）或者更新的 Unified Extensible Firmware Interface（UEFI）来 access 磁盘。

当代磁盘硬件包括允许 BIOS 或 UEFI 通过逻辑块寻址 (LBA Logical Block Addressing) 访问连接的存储硬件的固件。LBA 是一种通用、简单的方法，可以从任何磁盘访问数据，但其性能较差。不过，这不是问题，因为引导加载程序通常是唯一必须使用此模式进行磁盘访问的程序。启动后，内核可以访问自己的高性能驱动程序。

一旦解决了对磁盘原始数据的访问问题，引导加载程序必须完成在文件系统上定位所需数据的工作。最常见的引导加载程序可以读取分区表，并内置对文件系统只读访问的支持。因此，他们可以找到并读取将内核放入内存所需的文件。此功能使动态配置和增强引导加载程序变得更加容易。 Linux 引导加载程序并不总是具有此功能；没有它，配置引导加载程序会更加困难。

### Boot Loader Tasks
1. Select from multiple kernels
2. Switch between sets of kernel parameters
3. Allow the user to manually override and edit kernel image names and parameters
4. Provide support for booting other operating systems

自 Linux 内核问世以来，引导加载程序已经变得相当先进，具有命令行历史记录和菜单系统等功能，但基本需求始终是内核映像和参数选择的灵活性。

### Boot Loader Overview
- GRUB Linux 系统上近乎通用的标准，具有 BIOS/MBR 和 UEFI 版本。
- LILO 第一个 Linux 引导加载程序之一。 ELILO 是 UEFI 版本。
- SYSLINUX 可以配置为从许多不同类型的文件系统运行。
- LOADLIN Boots a kernel from MS_DOS。
- systemd-boot
- coreboot

### BIOS 和 boot loader 的区别
BIOS 和 Boot Loader 是计算机启动过程中两个重要但不同的组件。它们各自承担不同的职责，以下是它们的主要区别：

1. BIOS（Basic Input/Output System）
- 位置：BIOS 是嵌入在主板上的固件，存储在 ROM 或闪存中。
- 功能：
  - 硬件初始化：当计算机通电时，BIOS 首先运行，它对计算机的所有硬件（如 CPU、内存、硬盘等）进行检测、初始化和配置。
  - POST（自检）：BIOS 会执行一个自检过程（Power-On Self Test, POST），确保各硬件设备正常工作。
  - 启动引导顺序：BIOS 负责找到适合的启动设备（如硬盘、光驱、USB 等），并将控制权移交给 Boot Loader。
  - 主要职责：初始化硬件并找到引导设备，将控制权交给 Boot Loader。
2. Boot Loader
- 位置：Boot Loader 通常存储在硬盘的主引导记录（MBR）或 GUID 分区表（GPT）的引导区中。
- 功能：
  - 加载操作系统：Boot Loader 负责从存储设备上加载操作系统内核，并将控制权移交给操作系统，使其开始运行。
  - 引导选择：某些 Boot Loader（如 GRUB 或 LILO）可以提供菜单，允许用户选择要启动的操作系统或引导选项。
  - 阶段划分：Boot Loader 通常分为两阶段：第一阶段是在 MBR 或 GPT 引导分区中，第二阶段是加载实际的操作系统内核。
  - 主要职责：引导操作系统，负责加载内核并启动操作系统。

启动过程中的关系：
BIOS 先运行，初始化硬件，并查找可引导的设备（如硬盘、光驱、USB）。一旦找到可引导设备，BIOS 将控制权移交给该设备的 Boot Loader。

Boot Loader 负责从该设备加载操作系统并启动它。
  
进一步比较：
- BIOS 是计算机硬件级的固件，属于最底层的程序；
- Boot Loader 是硬盘上的一个小程序，属于软件级，它具体负责选择和启动操作系统。
- BIOS 和 Boot Loader 是互补的：一个初始化硬件，另一个启动操作系统。

## GRUB Introduction
GRUB = Grand Unified Boot Loader。 

GRUB 最重要的功能之一是文件系统浏览，可以轻松选择内核映像和配置。查看其实际情况并总体了解 GRUB 的最佳方法之一是查看其菜单。该界面很容易导航，但你很可能从未见过它。

要看到 GRUB 的界面需要在 BIOS 启动时按某些键，不同的系统有不同的键。

可以在 GRUB 中设置或运行一些 GRUB 的命令，有些命令和 linux 的命令相同，但功能完全不同。

略。

## UEFI Secure Boot Problems
安全引导，某些系统需要对 boot loaders 进行数字签名，固件才会去加载 boot loader。

略

## How GRUB works
1. The PC BIOS or firmware initializes the hardware and searches its boot-order storage devices for boot code.
2. Upon finding the boot code, the BIOS/firmware loads and executes it. This is where GRUB begins.
3. The GRUB core loads.
4. The core initializes. At this point, GRUB can now access disks and filesystems.
5. GRUB identifies its boot partition and loads a configuration there.
6. GRUB gives the user a chance to change the configuration.
7. After a timeout or user action, GRUB executes the configuration.
8. In the course of executing the configuration, GRUB may load additional code (modules) in the boot partition. Some of these modules may be preloaded.
9. GRUB executes a boot command to load and execute the kernel as specified by the configuration’s linux command.

略。


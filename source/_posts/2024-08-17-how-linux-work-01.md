---
title: how linux work 01
date: 2024-08-17 12:02:14
updated: 2024-08-17 12:02:14
categories: 操作系统
tags: [linux]
description: how linux work 01 - summary
---

> 现代操作系统非常复杂，最有效的方式来理解一个操作系统是如何工作的方式就是 `abstract`，通过忽略一些细节来了解全局。

软件开发人员在构建操作系统及其应用程序时使用抽象作为工具。计算机软件中的抽象细分有很多术语，包括子系统 subsystem、模块 module 和包 packages，这里使用术语组件 component，因为它很简单。在构建软件组件时，开发人员通常不会过多考虑其他组件的内部结构，但他们确实关心可以使用哪些其他组件以及如何使用它们。

## levels and layers of abstraction in a linux system
使用抽象把计算机系统分成若干组件更易于理解，但是如果没有组织结构的话，这些组件也不会正常工作。

![layers](layers.png)

Linux 有三层抽象：

### hardware
主要包括内存、CPU、硬盘和网卡等。

### kernel
操作系统的核心，内核是驻留在内存的软件，告诉 CPU 做什么事情。内核管理硬件并主要充当硬件和任何正在运行的程序之间的接口。

### process
进程（内核管理的正在运行的程序）共同构成了系统的上层，称为用户空间 user space。通常叫做用户进程 user process。

内核进程和用户进程的运行方式之间有一个关键的区别：内核运行在内核态 kernel mode，用户进程运行在用户态 user mode。运行在内核态的代码在访问 CPU 和 内存时没有限制，这是一个强大但危险的特权，它允许内核进程轻松地使整个系统崩溃。只有内核可以访问的区域称为内核空间 kernel space。

相比之下，user mode 限制对内存子集（通常很小）的访问和安全的 CPU 操作。用户空间是指用户进程可以访问的主内存部分。如果一个用户进程发生错误崩溃，对整个系统的影响很小，内核会负责清理。即一个用户进程崩溃一般不会影响别的进程。

理论上，用户进程失控 haywire 不会对系统的其余部分造成严重损害。

## 硬件 内存
在计算机系统的所有硬件中，主存储器可能是最重要的，在其最原始的形式中，主存只是一个存储一堆 0 和 1 的大存储区域。每个 0 或 1 称为一个位。这是正在运行的内核和进程所在的位置——它们只是位的大集合。外围设备的所有输入和输出都以一堆位的形式流经主存储器。 CPU 只是内存上的一个操作符；它从内存中读取指令和数据，并将数据写回内存。经常会听到术语“状态”来指代内存、进程、内核和计算机系统的其他部分。严格来说，状态是位的特定排列。例如，如果内存中有四位，0110、0001 和 1011 代表三种不同的状态。当考虑到单个进程可以轻松地由内存中的数百万位组成时，在谈论状态时使用抽象术语通常更容易。不是使用位来描述状态，而是描述某件事此时已完成或正在执行的操作。例如，可能会说"进程正在等待输入"或"进程正在执行启动的第 2 阶段"。

## Kernel
我们为什么要谈论主内存和状态 state？内核所做的几乎所有事情都围绕着主内存。内核的任务之一是将内存分成许多分区，并且它必须始终维护有关这些分区的某些状态信息。每个进程都有自己的内存份额，内核必须确保每个进程保留其份额。

内核负责管理系统中四种不同的系统区域：
1. Processes: 内核负责调度进程，决定哪些进程能够使用 CPU。
2. Memory: 内核需要一直维护所以内存，哪些分配给了哪个进程，哪些是被进程共享的，哪些是没有被使用的。
3. Device drivers：内核充当硬件和进程之间的接口，一般操作硬件是内核的工作。
4. System calls and support: 进程使用系统调用和内核交互。

### Process Management
进程管理描述进程的启动、暂停、恢复和终止。启动和终止进程背后的概念相当简单，但描述进程在正常操作过程中如何使用 CPU 则有点复杂。

在现代的操作系统中，许多进程"同时"运行，然而事情并不是这样的，这些进程并不是同时在 CPU 上执行的。

比如只有一个 CPU 的系统，在给定的任意时间，只有一个进程在使用 CPU，实际上，每个进程都会使用 CPU 一小部分时间，然后暂停；然后另一个进程又使用了 CPU 一小部分时间；然后另一个进程轮流进行，依此类推。一个进程将 CPU 控制权交给另一进程的行为称为上下文切换 context switch。

每个短小的时间叫做时间片 time slice，时间片很短，人类感觉不出来，系统就像是在同时运行这些进程。

内核负责上下文切换。为了理解它是如何工作的，让我们考虑一个进程在用户模式下运行但其时间片已到的情况：
1. CPU interrupts the current process based on an internal timer，switches into kernel mode，and hands control back to the kernel
2. Kernel records the current state of the CPU and memory，which will be resuming the process that was just interrupted
3. The kernel perfors any tasks that might have come up during the preceding time slice（such as collecting data from input and output，or I/O operation）
4. The kernel now is ready to let another process run, the kernel anayzes the list of processes that are ready to run and chooses one
5. The kernel preares the memory for this new process, and then prepare the CPU 
6. The kernel tells the CPU how long the time slice for the new process will last
7. The kernel switches the CPU into user mode and hands control of the CPU to the process

**The context switch answers the important question of when the kernel runs. The answer is that it runs between process time slices during a context switch.**

在多 CPU 系统的情况下，事情会变得稍微复杂一些，因为内核不需要放弃对其当前 CPU 的控制来允许进程在不同的 CPU 上运行。然而，为了最大限度地利用所有可用的 CPU，内核通常会这样做（并且可能使用某些技巧来为自己争取更多的 CPU 时间）。

### Memory Management
在上下文切换时，内核必须管理内存，有很多复杂的工作要做：
1. 内核必须有自己私有的内存区域，用户进程无法访问。
2. 每个用户进程都需要自己的内存。
3. 一个用户进程无法访问其他进程的私有内存区域。
4. 用户进程可以共享内存。
5. 用户进程中某些内存单元只读。
6. 通过使用磁盘空间作为辅助，系统可以使用比物理内存更多的内存。

幸运的是，对于内核来说，有帮助。现代 CPU 包含内存管理单元 (MMU)，它支持虚拟内存的内存访问方案。使用虚拟内存时，进程不会通过内存在硬件中的物理位置直接访问内存。相反，内核将每个进程设置为就好像它自己拥有一整台机器一样。当进程访问其某些内存时，MMU 会拦截该访问并使用内存地址映射将进程中的内存位置转换为机器上的实际物理内存位置。内核仍然必须初始化并持续维护和更改此内存地址映射。例如，在上下文切换期间，内核必须将映射从调出进程更改为调入进程。

**The implementation of a memory address map is called a page table.**

### Device Drivers and Management
内核在设备中的作用非常简单。设备通常只能在内核模式下访问，因为不正确的访问（例如用户进程要求关闭电源）可能会使机器崩溃。另一个问题是，不同的设备很少有相同的编程接口，即使设备做相同的事情，例如两个不同的网卡。因此，设备驱动程序传统上是内核的一部分，它们努力为用户进程提供统一的接口，以简化软件开发人员的工作。

### System Calls and Support
还有几种其他类型的内核功能可供用户进程使用。例如，系统调用（或 syscalls）执行用户进程单独无法完成或根本无法完成的特定任务。例如，打开、读取和写入文件的行为都涉及系统调用。

`fork()` 和 `exec()` 这两个系统调用对于理解进程如何启动非常重要：
- fork() When a process calls fork(), the kernel creates a nearly identical copy of the process.
- exec() When a process calls exec(program), the kernel starts program, replacing the current process.

除了 init 之外，Linux 系统上的所有用户进程都是通过 fork() 启动的，大多数时候，还运行 exec() 来启动一个新程序，而不是运行现有程序的副本。

## User Space
Linux 系统上的大部分实际操作都发生在用户空间中。尽管从内核的角度来看，所有进程本质上都是平等的，但它们为用户执行不同的任务。

## Users
Users exist primarily to support permissions and boundaries. Every user-space process has a user owner, and processes are said to run as the owner. 


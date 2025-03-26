---
title: how linux work 08
date: 2024-11-02 11:13:13
updated: 2024-11-02 11:13:13
categories: 操作系统
tags: [linux]
description: how linux work 08 - 进程和资源监控
---

三种基本的硬件资源：CPU、memory、I/O。进程竞争这些资源，内核负责公平的分配这些资源。同样内核本身也是一种资源，是一种软件资源，比如创建进程进程间通信。

## Tracking Processes
`ps` 只是显示系统当前进程的状态，不能动态显示，此时要使用 `top`。

## lsof
列出进程打开的文件，不仅仅是普通文件，还包括网络资源 socket、动态库、pipes 等。

### lsof output
- COMMAND The command name for process that holds the file descriptor
- PID 
- USER
- FD 该字段可以包含两种元素。FD 列显示文件的用途。FD 字段还可以列出打开文件的文件描述符，进程与系统库和内核一起使用来识别和操作文件的数字
- TYPE file type regular file、dir、socket
- DEVICE The major and minor number of the device that holds the file
- SIZE/OFF file size
- NODE inode number
- Name filename

## Tracing Program Execution and System Calls
- strace system call trace
- ltrace library trace
### strace
sys call is a privileged opration that a user-space process asks the kernel to perform. 比如打开并读取文件。strace 工具打印一个进程的所有的系统调用。

`strace cat /dev/null`，默认情况下，strace 输出到标准错误，可以使用 -o 选项把数据写入文件中，也可以使用 `2>` 重定向。

当一个进程想用启动其他进程时，使用 `fork()` 系统调用来复制自身，然后复制出来的进程使用 `exec()` strace 是在 `fork()` 后介入的，因此可以从输出开头部分中看到 `execve()` 调用，然后时内存初始化 `brk`。然后是处理一些共享库。

## Threads
### Single-Threaded and Multithreaded Processes
多线程能利用多核加速。
### 查看线程
`ps m` or `ps m -o pid,tid,command`

## 资源监控介绍
现在讨论资源监控中的一些主题，包括处理器 (CPU) 时间、内存和磁盘 I/O。我们将在系统范围内以及每个进程的基础上检查利用率。

许多人为了提高性能而接触 Linux 内核的内部工作原理。但是，大多数 Linux 系统在发行版的默认设置下表现良好，你可能要花几天时间尝试调整机器的性能，却得不到有意义的结果，尤其是当你不知道要寻找什么时。因此，在使用本章中的工具进行实验时，不要考虑性能，而要考虑观察内核在进程之间分配资源时的实际运行情况。

### 测量 CPU Time
```shell
$ time ls
real 0m0.442s
user 0m0.052s
sys  0m0.091s
```

User time 表示 CPU 执行程序代码的时间，sys time 表示为该进程花费的系统调用时间（读文件等），real time 表示程序开始到结束的时间，包括切换到其他进程运行的时间，可以用 real time 减去 user time 和 sys time 来计算进程等待资源的时间。

### Adjusting Process Priorities
可以改变内核调度进程的方式，以便为该进程分配比其他进程更多或更少的 CPU 时间。内核根据每个进程的调度优先级来运行它们，优先级是 -20 到 20 之间的一个数字，其中 -20 是最高优先级。

### Load Averages
整体 CPU 性能是较容易衡量的指标之一。平均负载是当前准备运行的进程的平均数量。也就是说，它是对在任意给定时间内能够使用 CPU 的进程数量的估计。这包括正在运行的进程和等待使用 CPU 机会的进程。考虑平均负载时，请记住，系统上的大多数进程通常都在等待输入（例如来自键盘、鼠标或网络），这意味着它们尚未准备好运行，不应为平均负载做出任何贡献。只有实际正在执行某些操作的进程才会影响平均负载。

The load average is the average number of processes currently ready to run. That is, it is an estimate of the number of processes that are capable of using the CPU at any given time—this includes processes that are running and those that are waiting for a chance to use the CPU. When thinking about a load average, keep in mind that most processes on your system are usually waiting for input (from the keyboard, mouse, or network, for example), meaning they’re not ready to run and shouldn’t contribute anything to the load average. Only processes that are actually doing something affect the load average.

三个数字分别是过去 1 分钟、5 分钟和 15 分钟的平均负载。如你所见，此系统并不繁忙：过去 15 分钟内，所有处理器上平均只有 0.01 个进程在运行。换句话说，如果只有一个处理器，则它在过去 15 分钟内运行用户空间应用程序的时间只有 1%。

If a load average goes up to around 1, a single process is probably using the CPU nearly all of the time. To identify that process, use the top command; the process will usually rise to the top of the display.

Most modern systems have more than one processor core or CPU, so multiple processes can easily run simultaneously. If you have two cores, a load average of 1 means that only one of the cores is likely active at any given time, and a load average of 2 means that both cores have just enough to do all of the time.

### Managing High Loads
高平均负载并不一定意味着系统有问题。具有足够内存和 I/O 资源的系统可以轻松处理许多正在运行的进程。如果平均负载很高而系统仍然响应良好，请不要惊慌；系统只是有很多进程共享 CPU。这些进程必须相互竞争处理器时间，因此，它们执行计算的时间会比它们被允许一直使用 CPU 时更长。另一种高平均负载可能是正常的情况是使用 Web 或计算服务器，其中进程可以如此快速地启动和终止，以至于平均负载测量机制无法有效运行。

However, if the load average is very high and you sense that the system is slowing down, you might be running into memory performance problems. When the system is low on memory, the kernel can start to thrash, or rapidly swap memory to and from the disk. When this happens, many processes will become ready to run, but their memory might not be available, so they’ll remain in the ready-to-run state (contributing to the load average) for much longer than they normally would.

### Monitoring Memory Status
检查整个系统内存状态的最简单方法之一是运行 `free` 命令或查看 `/proc/meminfo`，以查看缓存和缓冲区使用了多少实际内存。正如刚才提到的，内存短缺可能会导致性能问题。如果 caches 和 buffers 没有占用很多内存，那么很有可能内存资源不足了。

#### How Memory Works

之前提到过，CPU 有一个内存管理单元 memory management unit MMU 来增加访问内存的灵活性，内核通过将进程使用的内存分解为称为页面 pages 的较小块来协助 MMU。内核维护页表，page table，将进程的虚拟页面地址映射到内存中的真实页面地址。当进程访问内存时，MMU 会根据内核的页表将进程使用的虚拟地址转换为实际地址。用户进程实际上并不需要所有内存页面都立即可用才能运行。内核通常会在进程需要时加载和分配页面；该系统称为按需分页 on-demand paging 或简称为请求分页 demand paging。要了解其工作原理，请考虑程序如何启动并作为新进程运行：
1. 内核将程序指令代码的开头加载到内存页面中。
2. 内核可能会为新进程分配一些工作内存页面。
3. 在进程运行时，它可能会到达这样一个点：其代码中的下一条指令不在内核最初加载的任何页面中。此时，内核接管，将必要的页面加载到内存中，然后让程序恢复执行。
4. 类似地，如果程序需要的工作内存比最初分配的要多，内核就会通过查找空闲页面（或腾出空间）并将其分配给进程来处理这个问题。

#### Page Faults

如果进程想要使用内存页面时该页面尚未准备好，则该进程会触发页面错误。发生页面错误时，内核会从进程手中接管 CPU 以准备页面。页面错误有两种：minor and major.
##### Minor page faults
当所需页面实际上位于主内存中，但 MMU 不知道它在哪里时，就会发生 minor 页面错误。当进程请求更多内存或 MMU 没有足够的空间来存储进程的所有页面位置时（MMU 的内部映射表通常很小），就会发生这种情况。在这种情况下，内核会将页面信息告知 MMU，并允许进程继续。minor 页面错误无需担心，许多页面错误都是在进程运行时发生的。当进程访问的内存页尚未在其工作集的物理内存中，但可以通过从操作系统的页面缓存中快速获得时，就会发生 minor page fault。
#### Major page faults
当所需的内存页面根本不在主内存中时，就会发生 major 页面错误，这意味着内核必须从磁盘或其他一些慢速存储机制加载它。许多 major 页面错误会使系统陷入困境，因为内核必须做大量的工作来提供页面，从而剥夺了正常进程的运行机会。一些 major 页面错误是不可避免的，比如第一次运行程序时从磁盘加载代码时发生的页面错误。最大的问题发生在内存不足的时候，这会迫使内核开始将工作内存页面交换到磁盘，以便为新页面腾出空间，并可能导致系统崩溃。

```shell
$ /usr/bin/time cal > /dev/null
0.00user 0.00system 0:00.04elapsed 2%CPU (0avgtext+0avgdata 2708maxresident)k
824inputs+0outputs (2major+174minor)pagefaults 0swaps
```

当这个程序运行时，出现了 2 个 major 页面错误和 174 个 minor 页面错误。major 页面错误发生在内核第一次需要从磁盘加载程序时。如果再次运行此命令，可能不会出现任何 major 页面错误，因为内核会缓存磁盘中的页面。

如何查看正在运行的程序的 page faults？top 和 ps 都支持，top 需要选择字段。

`ps -o pid,min_flt,maj_flt 20365`

### vmstat
在众多可用于监控系统性能的工具中，vmstat 命令是最古老的命令之一，开销最小。它非常方便，可以大致了解内核交换页面的频率、CPU 的繁忙程度以及 I/O 资源的利用情况。

`vmstat -2`

输出分为几类：procs 表示进程，memory 表示内存使用情况，swap 表示从交换区 swap in 和 swap out 的页面，io 表示磁盘使用情况，system 表示内核切换到内核代码的次数，cpu 表示系统不同部分使用的时间。

在最右边的 CPU 标题下，您可以看到 us、sy、id 和 wa 列中 CPU 时间的分布。这些分别列出了 CPU 在用户任务、系统（内核）任务、空闲时间和等待 I/O 上花费的时间百分比。

### iostat iotop pidstat

iotop 是为数不多的显示线程 ID 的命令。

## cgroup
略

## 平均负载
The Load Average which is also known as system load is a metric that indicates the average number of processes that were **runnable** or **waiting to be run** on system CPU over the last couple of minutes. This information helps you represent the overall system workload and identify potential performance. The load average is an important metric in Linux that reflects the average system load over specific time intervals.

### Load avg 是什么

The load average represents the average number of processes in two states:
1. Runnable
These processes are ready to run on the CPU but might be waiting for their turn due to other processes already utilizing it.
2. Uninterruptible
These processes are currently running on the CPU and cannot be interrupted except for critical system events.
Uninterruptible（不可中断）进程是一种进程状态，表示进程正在等待某些系统资源或操作完成，此时该进程无法被中断或杀死。这种状态在进程状态列表中通常用 D 表示（可以通过 ps aux 查看）。不可中断的状态主要用于内核中执行的某些重要操作，例如等待磁盘 I/O 操作完成。

因此，要理解单核系统上 Linux 的平均负载为 1，表示平均有一个进程需要 CPU 资源，要么正在运行，要么正在等待执行。

### 解释负载平均值
There’s no single definitive interpretation of the ideal load average as it depends on several factors:

1. A higher number of cores allows handling more processes simultaneously so a higher load average might be acceptable on a multi-core system compared to a single-core system.
2. The nature of running applications and user activity also impacts the load average.
3. Resource-intensive tasks like video editing will naturally contribute to a higher load average compared to basic tasks like browsing the web.


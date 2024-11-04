---
title: how linux work 07
date: 2024-10-26 12:12:24
updated: 2024-10-26 12:12:24
categories: 操作系统
tags: [linux]
description: how linux work 07 - 系统配置：日志、时间、批作业、用户 
---

## System Logging
大多数系统程序将其诊断输出作为消息写入 syslog 服务。传统的 syslogd 守护程序通过等待消息并在收到消息后将其发送到适当的通道（例如文件或数据库）来执行此服务。在大多数现代系统中，journald（随 systemd 提供）会完成大部分工作。

略 journal

### Logfile Rotation
syslog 是协议 rsyslogd 是实现。linux 上一般有 syslog 和 journal两种日志处理框架。syslog 是一个 client-server 架构，它不仅可以监听 /dev/log 还能监听 socket。

当使用 `syslog` 守护程序时，系统记录的任何日志消息都会进入某个日志文件，这意味着需要偶尔删除旧消息，以免它们最终耗尽所有存储空间。不同的发行版以不同的方式执行此操作，但大多数使用 `logrotate` 实用程序。

该机制称为日志轮换 log rotation。由于传统的文本日志文件在开头包含最旧的消息，在结尾包含最新的消息，因此很难从文件中删除较旧的消息以释放一些空间。相反，由 logrotate 维护的日志被分为许多块 chunks。

假设 /var/log 中有一个名为 auth.log 的日志文件，其中包含最新的日志消息。然后是 auth.log.1、auth.log.2 和 auth.log.3，每个都包含逐渐旧的数据。当 logrotate 决定是时候删除一些旧数据时，它会像这样旋转文件：

1. 删除最旧 auth.log.3 的文件
2. 把 auth.log.2 重命名为 auth.log.3
3. 把 auth.log.1 重命名为 auth.log.2
4. 把 auth.log 重命名为 auth.log.1

关于文件名，在不同的发行版中，logrotate 的处理方式不相同，有些压缩，有些则是加上日期。

You might be wondering what happens if logrotate performs a rotation around the same time that another utility (such as rsyslogd) wants to add to the logfile. For example, say the logging program opens the logfile for writing but doesn’t close it before logrotate performs the rename. In this somewhat unusual scenario, the log message would be written successfully, because in Linux, once a file is open, the I/O system has no way to know it was renamed. But note that the file the message appears in will be the file with the new name, such as auth.log.1.

### Syslog 和 journald 之间的关系
journald 在某些系统上完全取代了 syslog，你可能会问为什么 syslog 在其他系统上仍然存在。主要有两个原因：
- Syslog 具有在多台机器上聚合日志的明确方法。当日志只在一台机器上时，监控日志要容易得多。
- 诸如 rsyslogd 之类的 syslog 版本是模块化的，能够输出为多种不同格式和数据库（包括日志格式）。这使得它们更容易连接到分析和监控工具。

相比之下，journald 强调收集和组织单台机器的日志输出，形成单一格式。

## /etc 的结构
Linux 系统上的大多数系统配置文件都位于 /etc 中。从历史上看，每个程序或系统服务都有一个或多个配置文件，由于 Unix 系统上的组件数量众多，/etc 会很快积累文件。

这种方法有两个问题：很难在正在运行的系统上找到特定的配置文件，而且很难维护以这种方式配置的系统。例如，如果想更改 sudo 配置，则必须编辑 /etc/sudoers。但在更改之后，升级发行版可能会抹去自定义设置，因为它会覆盖 /etc 中的所有内容。

多年来，将系统配置文件放入 /etc 下的子目录中一直是趋势，正如已经看到的 systemd，它使用 /etc/systemd。/etc 中仍有一些单独的配置文件，但如果运行 ls -F /etc，会看到那里的大多数项目现在都是子目录。

### User Management Files
Unix 系统是多用户的，在内核级别上，用户其实就是一些数字（user IDs），但是使用用户名更方便记忆，在管理 Linux 时，一般都用用户名，用户名只存在于用户空间，因此，任何使用用户名的程序在与内核对话时都需要找到其对应的用户 ID。

#### The /etc/passwd File
每一行代表了一个用户，用冒号分隔了七个字段。第一个是用户名，第二个是密码，X 表示密码存在 shadow 中，* 表示用户不能登陆系统。如果这个字段是空的，也就是 `::`，说明该用户不需要密码就能登陆。其他五个字段如下：
- user ID UID，用户在内核的表示形式，可以让两个用户使用相同的 ID，但是这样会让人产生疑惑，同样 software 也会产生疑惑，所以需要确保 user ID 唯一
- group ID GID，which should be one of the numbered entries in the /etc/group file，决定了文件权限和其他事项，这个组也叫做 user's primary group
- user's real name 
- home dir
- user's shell，用户使用终端时使用的 shell

![passwd](passwd.png)

/etc/passwd 文件语法非常严格，不允许注释和空行。

A user in /etc/passwd and a corresponding home directory are collectively known as an account. 然而，这只是用户空间的做法。有没有 home dir 不重要。An entry in the passwd file is usually enough to qualify; the home directory doesn’t have to exist in order for most programs to recognize an account. Furthermore, there are ways to add users on a system without explicitly including them in the passwd file; for example, adding users from a network server using something like NIS (Network Information Service) or LDAP (Lightweight Directory Access Protocol) was once common.

#### Special Users
超级用户 UID 和 GID 都是 0；比如像 daemon 用户没有 login 权限；The nobody user is an underprivileged user；有些进程以 nobody 用户运行。

不能 login 的用户叫做伪用户，尽管他们不能登陆，系统能用他们的 user ID 启动进程。像 nobody 这些伪用户是为了安全考虑而创建的。

同样这些都是用户空间的习俗，这些用户对内核来说并没有任何特殊含义。对内核来说，唯一具有特殊意义的用户 ID 是超级用户，0。可以授予 nobody 用户对系统上所有内容的访问权限，就像授予其他任何用户一样。

#### 操作用户和密码
普通用户使用 passwd 命令和一些其他工具和 /etc/passwd 交互，同样可以使用 chfn 和 chsh 来改变 real name 和 shell，这些工具都是 suid-root executables，因为只有 root 用户才能改变 /etc/passwd 文件。

##### 使用超级用户修改 /etc/passwd 文件
因为 /etc/passwd 只是一个普通的纯文本文件，所以从技术上讲，超级用户可以使用任何文本编辑器进行更改。要添加一个用户，只需要简单的在文件中新增一行并且为新用户创建一个家目录就可以了，要删除用户只需相反的操作就行。

然而像这样直接编辑 /etc/passwd 文件是个很不好的做法，不仅仅是非常容易犯错，并且会遇到并发修改问题。使用终端或 GUI 提供的单独命令更改用户要容易得多（也更安全）。例如，要设置用户密码，请以超级用户身份运行 passwd user。使用 adduser 和 userdel 分别添加和删除用户。

但是，如果确实必须直接编辑文件（例如，如果文件以某种方式损坏），使用 `vipw` 程序，该程序会在编辑 `/etc/passwd` 时备份并锁定它，以作为额外的预防措施。要编辑 `/etc/shadow` 而不是 `/etc/passwd`，使用 `vipw -s`。

### Working with Groups
Unix 中的群组提供了一种在某些用户之间共享文件的方法。这个想法是，可以为特定组设置读取或写入权限位，排除其他所有人。此功能曾经很重要，因为许多用户共享一台机器或网络，但近年来，随着工作站共享频率的降低，它变得不那么重要了。

就像 /etc/passwd 文件一样，/etc/group 中的每一行都是用冒号分隔的字段：
- The group name，比如使用 ls -l 时看到的
- The group password Unix 组密码很少使用，也不应该使用它们（在大多数情况下，sudo 是一个很好的替代方案）。使用 * 或任何其他默认值。这里的 x 表示 /etc/gshadow 中有相应的条目，而且这几乎总是一个禁用的密码，用 * 或 ! 表示。
- The group id GID 在文件中必须唯一
- An optional list of users that belong to the group 除了这里列出的用户之外，在 passwd 文件条目中具有相应组 ID 的用户也属于该组。

![group](group.png)

Linux distributions often create a new group for each new user added, with the same name as the user.

## User Access Topics
### User IDs and User Switching
我们讨论了 sudo 和 su 等 setuid 程序如何允许你临时更改用户，并介绍了登录等控制用户访问的系统组件。也许你想知道这些部分是如何工作的，以及内核在用户切换中扮演什么角色。

当你临时切换到另一个用户时，实际上所做的只是更改你的用户 ID。有两种方法可以做到这一点，内核会处理这两种方法。第一种方法是使用 setuid 可执行文件。第二种方法是通过 `setuid()` 系列系统调用。该系统调用有几种不同的版本，以适应与进程相关的各种用户 ID。

内核对进程能做什么、不能做什么有基本规定，但这里有三个涵盖 setuid 可执行文件和 setuid() 的基本规则：
- 只要具有足够的文件权限，进程就可以运行 setuid 可执行文件。
- 以 root（用户 ID 0）身份运行的进程可以使用 setuid() 成为任何其他用户。
- 不以 root 身份运行的进程在使用 setuid() 时受到严格限制；大多数情况下，它不能使用 setuid()。

由于这些规则，如果你想将用户 ID 从普通用户切换到另一个用户，你通常需要结合使用多种方法。例如，sudo 可执行文件是 setuid root，一旦运行，它就可以调用 setuid() 成为另一个用户。

从本质上讲，用户切换与 password 或 usernames 无关。这些是用户空间概念。

### Process Ownership, Effective UID, Real UID, Saved UID
到目前为止讨论的 user IDS 是很简单的，实际上每个进程不只一个 user IDs。
- Effective user id （Effective UID or EUID）定义了进程的访问权限（最重要的，文件权限）
- Real user ID （real UID，or ruid）indicates who initiated a process

通常情况下这些 IDs 是相同的，但是当你运行一个 setuid 程序时，Linux sets the euid to the program’s owner during execution，但是会把原来的 user ID 存在 ruid 中。

将 euid 视为参与者，将 ruid 视为所有者。ruid 定义可以与正在运行的进程交互的用户——最重要的是，哪个用户可以终止并向进程发送信号。例如，如果用户 A 启动一个以用户 B 身份运行的新进程（基于 setuid 权限），则用户 A 仍然拥有该进程并可以终止它。

我们已经看到大多数进程具有相同的 euid 和 ruid。因此，ps 和其他系统诊断程序的默认输出仅显示 euid。

`ps -eo pid,euser,ruser,comm`，可以复制一个 sleep 程序，并设置 setuid 来观察。

除了 EUID 和 RUID 之外还有 saved user ID，一个进程可以在运行时把 EUID 切换为 RUID 和 saved user ID，Linux 还有 fsuid，file system user ID 很少使用。

### Typical Steuid Program Behavior
ruid 的想法可能与你之前的经验相矛盾。为什么你不必经常处理其他用户 ID？例如，在使用 sudo 启动进程后，如果你想要终止它，你仍然使用 sudo；你不能以自己的普通用户身份终止它。在这种情况下，你的普通用户难道不应该是 ruid，为你提供正确的权限吗

导致此行为的原因是 sudo 和许多其他 setuid 程序使用 setuid() 系统调用之一明确更改 euid 和 ruid。这些程序这样做是因为当所有用户 ID 不匹配时，经常会出现意想不到的副作用和访问问题。

小心使用 setuid 程序。

略
---
title: Python Packaging概述
date: 2024-09-21 19:17:22
updated: 2024-09-21 19:17:22
categories: Python
tags: [Python]
description: 关于 Python 打包相关概念 
---

## Python Packaging Overview
作为一种通用编程语言，Python 被设计用于多种用途。你可以构建网站、工业机器人或游戏供你的朋友玩等等，所有这些都使用相同的核心技术。

Python 的灵活性是为什么每个 Python 项目的第一步都必须考虑项目的受众以及项目运行的相应环境。在编写代码之前考虑打包似乎很奇怪，但这个过程确实可以避免将来的麻烦。

## 想想部署
Packages 的存在就是为了被安装或者被部署，所以在打包任何东西之前，需要了解以下和部署相关问题的一些答案：
- 谁是你的软件的使用者？你的软件是否会由其他进行软件开发的开发人员、数据中心的操作人员或不太精通软件的团队安装？
- 你的软件是要在服务器、台式机、移动客户端（手机、平板电脑等）上运行，还是嵌入专用设备中？
- 你的软件是单独安装还是大批量部署？

打包最重要的是目标环境和部署体验。上述问题有很多答案，每种情况都有自己的解决方案。有了这些信息，以下概述将指导你找到最适合你的项目的封装技术。

## 打包 Python 的库和工具
你可能听说过 `PyPI`、`setup.py` 和 `wheel` 文件，这些只是 Python 生态系统提供的一些工具，用于向开发人员分发 Python 代码。 

下面三个打包方法适应于技术开发人员在开发环境中使用的库和工具。

### Python modules
Python 文件只要只依赖于标准库，就可以重新分发和重用。还需要确保它是为正确的 Python 版本编写的，并且仅依赖于标准库。

这非常适用在具有兼容 Python 版本的人之间共享简单的脚本和代码片段（比如通过电子邮件、StackOverflow或 GitHub gists）。某些完整的 Python 库使用这种方式打包比如 `bottle.py` 和 `boltons`。

但是，这种模式无法扩展到包含多个文件、需要额外库或需要特定版本的 Python 的项目。

### Python source distributions
如果代码包含了多个 Python 文件，通常把他们组织在一个目录结构中，任何包含 Python 文件的目录都会成为一个导入包（import package）。

由于包由多个文件组成，因此它们更难分发。大多数协议支持一次仅传输一个文件。更容易出现不完整的传输，并且更难保证目的地的代码完整性。

只要你的代码只包含纯 Python 文件而不包含其他东西，并且知道部署环境和你的开发环境 Python 版本兼容，那么就可以使用 Python 原生的打包工具来创建一个源码分发包（source Distribution Package）简称 sdist。

Python 的 sdists 是包含一个或多个包或模块的压缩归档文件（.tar.gz）。[源码分发格式](https://packaging.python.org/en/latest/specifications/source-distribution-format/#source-distribution-format)。

### Python binary distributions
Python 的大部分实用功能来自于它与软件生态系统集成的能力，特别是用 C、C++、Fortran、Rust 和其他语言编写的库。

并非所有开发人员都拥有合适的工具或经验来构建用这些编译语言编写的组件，因此 Python 创建了 `Wheel`，这是一种包格式，旨在发布带有编译工件（产物）的库。事实上，Python 的包安装程序 `pip` 总是更喜欢 `Wheel`，因为安装它总是更快，所以即使是纯 Python 包也能更好地使用 `Wheel`。

当二进制发行版与源发行版相匹配时（同时发布二进制包和源码包），它们是最好的。即使不为每个操作系统上传 `wheels`，通过上传 `sdist`，仍然可以让其他平台的用户为自己从源码构建。默认情况下需要同时发布 `wheel` 和 `sdist`。Python 和 PyPI 可以轻松地将wheel 和sdists 一起上传。

![py_pkg_tools_and_libs](py_pkg_tools_and_libs.png)

## 打包 python 应用
到目前为止，只讨论了 Python 的原生分发工具。根据介绍，可以正确地推断这些内置方法仅针对具有 Python 的环境以及知道如何安装 Python 包的受众。

由于操作系统、配置和人员多种多样，这种假设只有在针对开发人员受众时才是安全的。

Python 的原生打包 native packaging 主要是为了在开发人员之间分发可重用代码（称为库）而构建的，可以使用 `setuptools entry_points` 等技术，在 Python 的库打包之上搭载工具或供开发人员使用的基本应用程序（CLI）。

Libraries are building blocks, not complete applications. 对于分发应用程序来说，存在着一个全新的技术世界。

下面的几个方式是根据这些应用程序打包选项对目标环境的依赖关系来组织的，可以为项目选择正确的方式。

### Depending on a framework
某些类型的 Python 应用程序（例如网站后端和其他网络服务）非常常见，因此它们具有支持其开发和打包的框架。其他类型的应用程序，例如动态 Web 前端和移动客户端，都足够复杂，以至于框架不仅仅是为了方便。

所有这些情况下，从框架的打包和部署故事开始向后工作是有意义的，依赖于框架，需要遵循框架的打包指南以获得最简单、最可靠的生产体验。

#### Service platforms
如果在开发平台即服务（PaaS），则需要遵循它们的打包要求，按它们的方式来。

#### Web browsers and mobile applications
Python 的稳步进步正在引领它进入新的领域。如今，可以使用 Python 编写移动应用程序或 Web 应用程序前端。虽然 Python 语言很熟悉，但打包和部署实践是全新的。

比如需要了解：`Kivy`、`Beeware`、`Brython`、`Flexx`。

### Depending on a pre-installed Python
选择任意一台计算机，依赖这个计算机的上下文环境，很有可能已经安装了 Python。多年来，Python 已默认包含在大多数 Linux 和 Mac 操作系统中，可以合理地依赖数据中心或开发人员和数据科学家的个人计算机上预先存在的 Python。

Technologies which support this model:
- PEX Python Executable
- zipapp 
- shiv

这些方法中，依赖于目标系统预安装的 Python 环境，这会让打包后的产物很小，如果较少对目标系统的依赖会增加包的大小。

### Depending on a separate software distribution ecosystem
依赖于单独的软件分发生态系统。长期以来，包括 Mac 和 Windows 在内的许多操作系统都缺乏内置的包管理。直到最近，这些操作系统才获得了所谓的"app stores"，但即使是那些专注于消费者应用程序的操作系统，也为开发人员提供的服务很少。

开发人员长期以来一直在寻求补救措施，并在这场斗争中出现了自己的包管理解决方案，例如 Homebrew。对于 Python 开发人员来说，最相关的替代方案是名为 Anaconda 的包生态系统。

Anaconda 围绕 Python 构建，在学术、分析和其他面向数据的环境中越来越常见，甚至进入了面向服务器的环境。

Anaconda 生态系统构建和发布说明：
- Building libraries and applications with conda
- Transitioning a native Python package to Anaconda

### Bringing your own Python executable
包中自带 python 解释器。
我们知道，计算是由执行程序的能力来定义的。每个操作系统本身都支持它们可以本机执行的一种或多种格式的程序。

有许多技巧和技术可以将 Python 程序转换为其中一种格式，其中大多数涉及将 Python 解释器和任何其他依赖项嵌入到单个可执行文件中。

这种称为冻结的方法提供了广泛的兼容性和无缝的用户体验，但通常需要多种技术和大量的工作。

A selection of Python freezers:
- pyinstaller Cross-platform
- cx_Freeze Cross-platform
- constructor For command-line installers
- py2exe windows only
- osnap windows and mac
- pynsist windows only

### Bringing your own userspace
越来越多的操作系统（包括 Linux、Mac OS 和 Windows）可以设置为运行打包为轻量级映像的应用程序，使用通常称为操作系统级虚拟化或容器化的相对现代的技术。

这些技术大多与 Python 无关，因为它们打包整个操作系统文件系统，而不仅仅是 Python 或 Python 包。

Linux 服务器的采用最为广泛，这是该技术的发源地，并且以下技术最有效：
- AppImage
- Docker
- Flatpak
- Snapcraft

### Bringing your own kernel
大多数操作系统支持某种形式的经典虚拟化，运行打包为包含其自己的完整操作系统的映像的应用程序。运行这些虚拟机或 VM 是一种成熟的方法，在数据中心环境中广泛使用。 这些技术主要保留用于数据中心的大规模部署，尽管某些复杂的应用程序可以从这种封装中受益。这些技术与 Python 无关，包括：
- Vagrant
- VHD，AMI
- OpenStack A cloud management system in Python, with extensive VM support

### Bringing your own hardware
交付软件的最全面的方法是将其交付已安装在某些硬件上的软件。这样，软件的用户只需要有电就行。

虽然上述虚拟机主要是为精通技术的人保留的，但可以发现从最先进的数据中心到最小的孩子的每个人都在使用硬件设备。

将代码嵌入到 Adafruit、MicroPython 或更强大的运行 Python 的硬件上，然后将其发送到数据中心或用户的家中。它们即插即用。

![py_pkg_applications](py_pkg_applications.png)

## 其他打包方式
### Operating system packages
许多操作系统有自己包管理器，比如 deb（for Debian, Ubuntu, etc） or RPM，可以把代码用这种方式打包，并使用这些方式安装。
### virtualenv
Virtualenvs 一直是多代 Python 开发人员不可或缺的工具，但随着它们被更高级别的工具包裹，它正在慢慢淡出人们的视野。特别是在打包方面，virtualenvs 在 dh-virtualenv 工具和 osnap 中用作原语，这两者都以独立的方式包装 virtualenvs。
### Security
梯度越低，更新包的组件就越困难。一切都更加紧密地结合在一起。比如内核出现严重的安全漏洞，我们怎么保证自己软件的安全更新，如果自己的镜像有问题呢。这就回到了使用动态链接库和静态库的争论上了。

## 总结
Python 打包因坎坷不平而闻名。这种印象主要是 Python 多功能性的副产品。一旦了解了每个打包解决方案之间的自然界限，就会开始意识到，Python 程序员为使用最平衡、最灵活的语言之一而付出的代价很小。

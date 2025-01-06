---
title: python warnings
date: 2025-01-04 12:47:22
updated: 2025-01-04 12:47:22
tags: Python
categories: Python
description: warnings
---

Warning messages are typically issued in situations where it is useful to alert the user of some condition in a program, where that condition (normally) doesn’t warrant raising an exception and terminating the program. For example, one might want to issue a warning when a program uses an obsolete module.

警告信息通常输出到 `sys.stderr`，但处置这些消息可以灵活更改，从忽略所有警告到将其转变为异常。处理警告信息可以基于警告的类别、警告消息体和警告消息的来源位置来进行。同一个位置的警告信息通常会被抑制。

警告控制分为两个部分，1. 是否要发，2. 如何格式化：

- each time a warning is issued, a determination is made whether a message should be issued or not
- if a message is to be issued, it is formatted and printed using a user-settable hook

是否发出警告消息的决定由警告过滤器 warning filter 控制，警告过滤器是一系列匹配的规则和操作。

可以通过调用 `filterwarnings()` 将规则添加到过滤器，并通过调用 `resetwarnings()` 将规则重置为其默认状态。

警告消息的打印是通过调用 `showwarning()` 来完成的，该函数可以会被覆盖；此函数的默认实现通过调用 `formatwarning()` 来格式化消息，该函数也可供自定义实现使用。

## Warning Categories

警告是异常的子类。有许多内置异常代表警告类别。此分类有助于过滤掉警告组。
用户代码可以通过对标准警告类别之一进行子类化来定义其他警告类别。警告类别必须始终是警告类 `Warning` 的子类。

|Class|Description|
|:---|:---|
|Warning|This is the base class of all warning category classes. It is a subclass of Exception.|
|UserWarning|The default category for warn().|
|DeprecationWarning|Base category for warnings about deprecated features when those warnings are intended for other Python developers (ignored by default, unless triggered by code in `__main__`).|
|SyntaxWarning|Base category for warnings about dubious syntactic features.|
|RuntimeWarning|Base category for warnings about dubious runtime features.|
|FutureWarning|Base category for warnings about deprecated features when those warnings are intended for end users of applications that are written in Python.|
|PendingDeprecationWarning|Base category for warnings about features that will be deprecated in the future (ignored by default).|
|ImportWarning|Base category for warnings triggered during the process of importing a module (ignored by default).|
|UnicodeWarning|Base category for warnings related to Unicode.|
|BytesWarning|Base category for warnings related to bytes and bytearray.|
|ResourceWarning|Base category for warnings related to resource usage (ignored by default).|

## Warnings filter

The warnings filter controls whether warnings are ignored, displayed, or turned into errors (raising an exception).

从概念上讲，警告过滤器维护一个有序的过滤器规范列表；任何特定警告都会依次与列表中的每个过滤器规范进行匹配，直到找到匹配项；过滤器决定匹配项的处置。每个条目都是形式为 `(action、message、category、module、lineno)` 的元组。

### action

|Value|Disposition|
|:---|:---|
|"default"|print the first occurrence of matching warnings for each location (module + line number) where the warning is issued|
|"error"|turn matching warnings into exceptions|
|"ignore"|never print matching warnings|
|"always"|always print matching warnings|
|"module"|print the first occurrence of matching warnings for each module where the warning is issued (regardless of line number)|
|"once"|print only the first occurrence of matching warnings, regardless of location|

### message

message 是一个包含正则表达式的字符串，警告消息的开头必须匹配该正则表达式（不区分大小写）。在 `-W` 和 `PYTHONWARNINGS` 中，message 是一个文字字符串，警告消息的开头必须包含该文字字符串（不区分大小写），忽略消息开头或结尾的任何空格。

### category

category is a class (a subclass of Warning) of which the warning category must be a subclass in order to match.

### module

module 是一个包含正则表达式的字符串，该正则表达式必须与完全限定模块名称的开头匹配（区分大小写）。在 `-W` 和 `PYTHONWARNINGS` 中，module 是一个文字字符串，该正则表达式必须与完全限定模块名称相等（区分大小写），忽略 module 开头或结尾的任何空格。

### lineno

lineno is an integer that the line number where the warning occurred must match, or 0 to match all line numbers.

Since the Warning class is derived from the built-in Exception class, to turn a warning into an error we simply raise category(message).

If a warning is reported and doesn’t match any registered filter then the “default” action is applied (hence its name).

## 重复警告抑制标准

Repeated Warning Suppression Criteria:

The filters that suppress repeated warnings apply the following criteria to determine if a warning is considered a repeat:

- "default": A warning is considered a repeat only if the (message, category, module, lineno) are all the same.
- "module": A warning is considered a repeat if the (message, category, module) are the same, ignoring the line number.
- "once": A warning is considered a repeat if the (message, category) are the same, ignoring the module and line number.

## 描述警告过滤器

警告过滤器由传递给 Python 解释器命令行的 `-W` 选项 和 `PYTHONWARNINGS` 环境变量初始化。解释器将所有提供的条目的参数（未解释）保存在 `sys.warnoptions` 中；警告模块在首次导入时解析这些参数（在将消息打印到 sys.stderr 后，无效选项将被忽略）。

各个警告过滤器被指定为以冒号分隔的一系列字段：`action:message:category:module:line`。

当在一行中列出多个过滤器时（例如 `PYTHONWARNINGS`），各个过滤器用逗号分隔，后面列出的过滤器优先于前面列出的过滤器（因为它们是从左到右应用的，最近应用的过滤器优先于前面的过滤器）。

常用的警告过滤器适用于所有警告、特定类别的警告或特定模块或软件包引发的警告。以下是一些示例：

```text
default                      # Show all warnings (even those ignored by default)
ignore                       # Ignore all warnings
error                        # Convert all warnings to errors
error::ResourceWarning       # Treat ResourceWarning messages as errors
default::DeprecationWarning  # Show DeprecationWarning messages
ignore,default:::mymodule    # Only report warnings triggered by "mymodule"
error:::mymodule             # Convert warnings to errors in "mymodule"
```

## Default warning filter

```text
default::DeprecationWarning:__main__
ignore::DeprecationWarning
ignore::PendingDeprecationWarning
ignore::ImportWarning
ignore::ResourceWarning
```

## Overriding the default filter

使用 Python 编写的应用程序的开发人员可能希望默认向用户隐藏所有 Python 级别警告，并且仅在运行测试或以其他方式处理应用程序时显示它们。用于将过滤器配置传递给解释器的 `sys.warnoptions` 属性可用作标记，以指示是否应禁用警告：

```python
import sys

if not sys.warnoptions:
    import warnings
    warnings.simplefilter("ignore")
```

建议 Python 代码测试运行器的开发人员确保默认显示被测代码的所有警告，使用如下代码：

```python
import sys

if not sys.warnoptions:
    import os, warnings
    warnings.simplefilter("default") # Change the filter in this process
    os.environ["PYTHONWARNINGS"] = "default" # Also affect subprocesses
```

最后，建议在 `__main__` 以外的命名空间中运行用户代码的交互式 shell 的开发人员确保 DeprecationWarning 消息默认可见，使用如下代码（其中 user_ns 是用于执行交互式输入的代码的模块）：

```python
import warnings
warnings.filterwarnings("default", category=DeprecationWarning,
                                   module=user_ns.get("__name__"))
```

## Temporarily Suppressing Warnings

如果正在使用知道会引发警告的代码（例如已弃用的函数），但不想看到该警告（即使已通过命令行明确配置了警告），那么可以使用 `catch_warnings` 上下文管理器来抑制警告：

```python
import warnings

def fxn():
    warnings.warn("deprecated", DeprecationWarning)

with warnings.catch_warnings():
    warnings.simplefilter("ignore")
    fxn()
```

在上下文管理器中，所有警告都将被忽略。这样，就可以使用已知的弃用代码，而不必看到警告，同时不会抑制其他可能不知道其使用弃用代码的代码的警告。注意：这只能在单线程应用程序中保证。如果两个或多个线程同时使用 `catch_warnings` 上下文管理器，则行为是未定义的。

## Testing Warning

要测试代码引发的警告，使用 `catch_warnings` 上下文管理器。使用它，可以暂时改变警告过滤器以方便测试。例如，执行以下操作以捕获所有引发的警告以进行检查：

```python
import warnings

def fxn():
    warnings.warn("deprecated", DeprecationWarning)

with warnings.catch_warnings(record=True) as w:
    # Cause all warnings to always be triggered.
    warnings.simplefilter("always")
    # Trigger a warning.
    fxn()
    # Verify some things
    assert len(w) == 1
    assert issubclass(w[-1].category, DeprecationWarning)
    assert "deprecated" in str(w[-1].message)
```

还可以通过使用 `error` 而不是 `always` 将所有警告都设为异常。需要注意的一点是，如果由于 `once/default` 规则已经引发了警告，那么无论设置了什么过滤器，除非清除了与警告相关的警告注册表，否则不会再次看到该警告。

一旦上下文管理器退出，警告过滤器就会恢复到进入上下文时的状态。这可以防止测试在测试之间以意外的方式更改警告过滤器并导致不确定的测试结果。模块中的 `showwarning()` 函数也会恢复到其原始值。注意：这只能在单线程应用程序中保证。如果两个或多个线程同时使用 `catch_warnings` 上下文管理器，则行为是未定义的。

当测试引发同一种警告的多个操作时，重要的是以确认每个操作都会引发新警告的方式进行测试（例如，将警告设置为异常并检查操作是否引发异常，检查警告列表的长度在每次操作后是否继续增加，或者在每次新操作之前从警告列表中删除先前的条目）。

略。

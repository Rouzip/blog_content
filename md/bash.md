---
title: "Bash常用技巧"
date: 2019-08-03T21:53:57+08:00
lastmod: 2019-08-4T01:37:56+08:00
draft: false
tags: ["Linux", "bash"]
categories: ["code"]
author: "Rouzip"

contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'
---

> 基于Linux的终端操作，必要的bash操作可以更加提升工作效率，此文记录如何使用bash进行复杂操作。

# 顺序执行

```bash
$ command1; command2; command3; ...
```

简单地顺序执行代码

# 逻辑执行

## 或

当前一条命令执行不正确，执行下一条命令

```bash
$ command1 || command2
```

## 与

当前一条命令执行正确，执行下一条命令

```bash
$ command1 && command2
```

# 通道

将前一条命令产生的结果传送到下一条命令之中

```bash
$ command1 | command2
```

# 重定向

一般将标准输出流重定向到某文件或者某设备

```bash
# 标准输入流
$ command1 > file    # 如果文件不存在则创建新文件，否则清空原文件中内容
$ command1 >> file    # 如果文件不存在则创建新文件，否则添加到原文件的末尾

# 标准输入流
$ cat file    # 将文件内容与文件名一起输出
$ cat < file    # 只将文件内容输出

# 标准错误流
$ command1 2> file    # 将错误信息重定向到文件中
$ command1 > file 2>&1    # 将错误信息重定向到文件中，同时将标准错误流重定向到同一文件
```


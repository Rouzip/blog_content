---
title: "Tricks"
date: 2019-10-01T16:38:19+08:00
draft: false
lastmod: 2020-07-06T11:41:53+08:00
tags: ["tricks", "solution"]
categories: ["code"]
author: "Rouzip"

contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'
---

> 这里会记录一些平时遇到的问题和解决办法，方便自己以后的查询

<!--more-->

## VSCode

### Golang

在安装完推荐的所有插件之后，就会默认开启保存格式化插件，未使用的包会自动删除代码。这时候，将 Preference 中的 Go: Format Tool 从默认的 goreturns 修改为 gofmt 就可以了。

在声明 slcie 之后，如果带了 len，就会初始化为默认值，所以如果只是想作为初始值声明，那么需要将 len 放在 cap 的位置，或者直接声明 0， 或者 var

## mysql

安装完 mysql 需要更改编码为 utf-8

```sql
 ALTER DATABASE sp CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci;
```

## Linux

### 时间同步

通过向 Google 服务器请求当前时间并裁剪出需要的形式，并根据结果进行时间的设定。

```bash
    date -s "$(wget -qSO- --max-redirect=0 google.com 2>&1 | grep Date: | cut -d' ' -f5-8)Z"
```

### 查看进程相关

#### 查看占用某端口的进程

```bash
lsof -i:<port>
```

#### 查看某进程打开的句柄

```bash
lsof -np <pid>
```

## git

### 格式化输出

git 较为直观地输出 log

```bash
    git log --graph --pretty=oneline --abbrev-commit
```

### 代理

```bash
    git config --global http.proxy socks5://127.0.0.1:1080  # 这里需要根据自己本地开放端口的不同进行设置
    git config --global --unset http.proxy
```

### 删除分支

极其**危险**！！！

```bash
    git branch -d xxx
```

### 回滚单个文件

```bash
    git checkout xxxhash xxxFile
```

## vim

跳转到最后一行
shift+g
跳转到最后一个字符
shift+4
vim 直接 copy 是不会拷贝`\t`这样的特殊字符的，都会被转换成空格，所以对于数据有精确要求需要用别的方法拷贝，例如 cat 或 less

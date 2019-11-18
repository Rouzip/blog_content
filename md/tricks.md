---
title: "Tricks"
date: 2019-10-01T16:38:19+08:00
draft: false
lastmod: 2019-11-14T16:23:12+08:00
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
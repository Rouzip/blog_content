---
title: "Tricks"
date: 2019-10-01T16:38:19+08:00
draft: false
lastmod: 2021-11-11T17:29:34+08:00
tags: ["tricks", "solution"]
categories: ["code"]
author: "Rouzip"

contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'
---

> 这里会记录一些平时遇到的问题和解决办法，方便自己以后的查询

<!--more-->

## VSCode

### Golang

#### vscode golang 插件

在安装完推荐的所有插件之后，就会默认开启保存格式化插件，未使用的包会自动删除代码。这时候，将 Preference 中的 Go: Format Tool 从默认的 goreturns 修改为 gofmt 就可以了。

## Go 踩坑记录

### slice 陷阱

在声明 slcie 之后，如果带了 len，就会初始化为默认值，所以如果只是想作为初始值声明，那么需要将 len 放在 cap 的位置，或者直接声明 0， 或者 var

### 打印优美格式的 json

```go
json.MarshalIndent(data, "", "    ")
```

### json 编码问题

golang 默认是使用 utf-8 进行编码的，所以如果使用 GBK 编码，json.Marshal 与 UnMarshal 是无法保证双向可转换的，需要自己实现 Marshal 对应的接口，做一层 Wrapper 封装。

### 检查代码单测覆盖率

在写单测时候，可以使用可视化方式来检查新的逻辑是否覆盖

```bash
# 生成测试覆盖率文件
go test -coverprofile=coverage.out
# 命令行展示覆盖率
go tool cover -func=coverage.out
# 使用html 来进行展示单元测试覆盖率
go tool cover -html=coverage.out
```

### rand 库使用

rand 中有两个 Source，直接创建的 `rand.NewSourc` 是线程不安全的，如果想使用线程安全的 rand，可以直接使用全局的 rand，因为默认会提供带 mutex 的 Source(`var globalRand = New(&lockedSource{src: NewSource(1).(*rngSource)})`)，这样可以做到线程安全的随机。

### 值拷贝

go 中的一切赋值本质上都是值拷贝

```go
type Conf struct {
    A string
}

type Service struct {
    Conf
}

func change(c *Conf) {
    c.A = "b"
}

func main() {
    conf := Conf{"a"}
    service := Service{
        Conf: conf,
    }
    change(&conf)
    fmt.Println(service.Conf.A)  // a
}
```

在嵌套的 struct 中的赋值，并不是直接将这个值赋给 Service，而是创建了一个 struct，将值拷贝给它

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
netstat -tunlp | grep <port>
```

#### 查看某进程打开的句柄

```bash
lsof -np <pid>
```

### 多行 bash

使用`control+x+e` 可以进入 vim 模式，进行多行编辑

### GNOME 分屏插件

gTile

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

## HTTP

### 返回合法 header

如果允许用户设置自定义的 header，需要检查 header 的长度大小，因为对于 nginx，在提前为 header 分配空间的时候，由配置项控制，不能允许过大的 header；如果用户设置中包含了`\x00`这样的特殊控制字符，需要提前将其进行转义或者替换，否则 nginx 返回的时候，会直接被截断，返回 502。

### 重试操作

对于服务端状态码进行区分的重试操作，502 和 504 可以在失败后进行重试，500 这样服务端出错的，可以不考虑重试。

## macOS

### 平台不兼容

macOS 原版 `cp` 的时候会把软链接直接复制成真实文件夹，在拷贝的时候会踩坑

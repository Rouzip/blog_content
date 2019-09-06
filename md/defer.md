---
title: "Defer"
date: 2019-09-04T16:38:19+08:00
draft: false
lastmod: 2019-09-06T22:13:54+08:00
tags: ["defer", "concept"]
categories: ["code"]
author: "Rouzip"

contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'

---

> `defer`是Go中不常见于其它语言的一个关键字。这篇文章将会描述它的作用和其具体的使用场景。

<!--more-->

## 概念

`defer` 是将函数推入到函数栈中，其在所在的函数作用域执行返回（return）时运行。

## 使用规则

### 1. 参数规则

`defer` 函数在运行到求值语句时其参数值被确定，下面用一个例子来解释。

```go
package main

import "fmt"

func main() {
    a := 1
    defer fmt.Printf("%v\n", a)
    a++
    return
}

// 1
```

不是说`defer` 关键字所指的函数会在当前函数作用域返回时才会执行吗，结果为什么是1？

原因是这样的：`defer` 函数在执行到该语句时，所传入的参数不是引用而是传值。这样相当于，在执行到该语句时，所有的参数被当成值，压入栈中，在返回时在执行输出。

### 2. 执行顺序

`多个defer` 函数的执行顺序为先进后出。

```go
package main

import "fmt"

func main() {
    for i := 1; i < 4; i++ {
        defer fmt.Printf("%v", i)
    }
}

//3 2 1
```

### 3. 具名函数返回值

当Go使用具名参数返回时，`defer` 将会在函数返回时执行。

```go
package main

import "fmt"

func c() (i int) {
    defer func() { i++ }()
    return 1
}

func main() {
    fmt.Println(c())
}

// 2
```

虽然函数中明确返回的是1，但是在Go中，已经明确返回的结果为变量`i` 。从结果上说，可以将函数c这样理解：

```go
func c() (i int) {
    i := 1
    i++
    return i
}
```

## 使用场景

其常被用来简化函数，用来执行各种清除操作。比如打开某个文件后，在读取过程中出现错误，`defer` 函数就可以被用作类似`finally` 的操作。

```go
func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }

    dst, err := os.Create(dstName)
    if err != nil {
        return
    }

    written, err = io.Copy(dst, src)
    dst.Close()
    src.Close()
    return
}
```

在这个函数之中，如果在`os.Create` 中发生任何问题，就会导致函数不正常返回，那么已经打开的`srcName` 文件指针就没有关闭，这时候就是`defer` 派上用场的时候。

```go
func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }
    defer src.close()

    dst, err := os.Create(dstName)
    if err != nil {
        return
    }
    defer dst.close()

    return io.Copy(dst, src)
}
```

这样就保证了即使因为出现错误提前退出函数，也不会出现文件资源未关闭这样的情况。

## 参考资料

1. <https://blog.golang.org/defer-panic-and-recover>
2. <https://tiancaiamao.gitbooks.io/go-internals/content/zh/03.4.html>

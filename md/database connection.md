---
title: "database connection pool"
date: 2019-11-14T21:53:57+08:00
lastmod: 2019-11-15T11:38:41+08:00
draft: false
tags: ["database", "concept"]
categories: ["code"]
author: "Rouzip"

contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'
---

> 在进行数据库操作的时候，我们会使用到 driver 和数据库进行交互，这时候的连接并不是直接交互的， 而是有一个 pool 的概念存在。

<!--more-->

## 问题的引出

最近在写爬虫，并将清洗出来的数据存储到关系数据库之中。在这个过程之中我遇到了问题 。在向数据库写入数据的时候，在写入一万条左右数据的时候就会遇到报错:

```txt
Error 1461: Can't create more than max_prepared_stmt_count statements
```

对于这个错误，我一时毫无头绪，针对这个问题我补习了一些基础的概念，和针对数据库的操作。

## database driver

### 概念

在和数据库进行交互的时候，我们需要一层 driver，这层 driver 向下和数据库进行交互，向上给我们提供给我们提供符合语言规范的 API，像 Java 之中，就是 JDBC 这套接口规范，在其他的语言中则是 ODBC 这套接口和规范，但是根据编程语言的不同会有细微的差别。

### 实例

拿我这次使用的 mysql 为例，如果我希望与 mysql 进行通信，需要使用 go-sql-driver 这套接口来使其可以与 driver 进行通信，还需要安装 mysql 的特定 driver——`github.com/go-sql-driver/mysql`与之匹配，driver 就是适配器一样的存在，让我们可以使用统一的接口，去连接各种不同的数据库。

## database connection pool

### 概念

数据库在与服务器进行通信的时候，并不是每次打开数据库都是对应一个连接的。在 Go 中，它使用了一个 connection pool 的概念。字如其名，pool 这个概念就如同一个蓄水池，其稳定地与数据库保持着一定数量的连接，在用户请求使用连接的时候直接将其分配给用户，减少了每次重新建立连接的代价。

### 代码实例

```go
import (
    "database/sql"

    _ "github.com/go-sql-driver/mysql"
)

    func main() {
        db, _ := sql.Open("mysql", "userName:password@tcp(adress:port)/databaseName")
        defer db.Close()

        sqlCommand := "..."

        stmt, ok := db.Prepare(sqlCommand)
        defer stmt.Close()

        if ok != nil {
            panic(ok.Error())
        }

        res, err := stmt.Exec()
        if err != nil {
            panic(err.Error())
        } else {
            fmt.Println(res)
        }
    }

```

我这次的数据库和服务器并不在同一台主机，不过同在一个局域网下， 通过 TCP 进行通信。这样的情况下，建立连接是比在本地建立连接的代价要大的，如果在大型的项目中，需要跨网络连接，那么需要的代价会更大，这时候保持连接的优势就凸现出来了。

## prepared statement

### 概念

在数据库层面，一个连接绑定到指定的数据库。这其实就是客户端将一段 sql statement 发送到服务器准备执行，服务器会发送给客户端一条 statement ID，客户端再发送给服务器这个 ID 和其中需要填写的参数。但是我们并不会直接操纵这个层面的代码，这都被 driver 所封装，方便了我们的调用。

### 工作流程

在 Go 中，它的执行过程是这样的：

1. 当你准备一个 statement 的时候，就会向 pool 中请求一个 connection
2. `Stmt`对象将会记录使用的是哪条 connection
3. 当具体执行语句的时候，他就会真正地使用这个 connection。如果当前 connection 已被关闭或者正忙，他就会重新向 pool 中请求 connection，然后重复上述行为。

## 问题分析

在理解了上述的概念之后，我们就可以理解这个问题出现的原因了。这个过程是我的程序想向数据库写入数据，这时候就会向 connection pool 请求一个 connection，`Stmt`对象会记住使用的是哪个连接，由于性能的限制，数据库同一时刻可以连接的数量是有限的，我在爬虫中使用了并发，所以同时向数据库请求的数量很大，同时我的数据库主机使用的是较慢的机械硬盘，处理速度不是很快。所有的原因加到一起造成了这个问题出现。

## 问题解决方案

在 GitHub 上，也有别人遇到了类似的问题，根据别人的经验，我通过设置`db.SetConnMaxLifetime(time.Second * 10)`，解决了这个问题，那么这个参数背后的原理是什么呢，与其长相类似的函数`SetMaxOpenConns`和`SetMaxIdleConns`又分别起到了什么样的作用呢？

## 深入探究

### SetConnMaxLifetime

这个函数设置了每次连接的最大运行时间，这意味着每条 connection 在创建后的指定时间内将会被销毁，如果有任务在运行，其就会在执行后销毁 connection。默认的 0 值代表对于连接时间没有限制，可以一直重复利用连接。

### SetMaxOpenConns

这个函数限制了 pool 与数据库的连接数量，过多的请求不会被处理。这个函数的默认值是不限制程序的请求数量，也代表着，你可以无限制地向数据库建立 connection，但是过多的请求也会导致数据库的回复变慢，加大数据库的负担。

### SetMaxIdleConns

这个函数设置了程序的最大闲置 connection 数量。在预先设置为一个很大的数量的时候，可以减少在需要时再去建立连接的消耗。当其设置为默认值的时候，其是在需要的时候再建立连接的。这个值是越大越好吗？并不是如此，因为过多的闲置 connection 也会占用系统资源，占用系统和数据库的内存。

### 思考

根绝这篇[blog](http://techblog.en.klab-blogs.com/archives/31093990.html)， 我了解到如此设置的原理。因为 mysql 数据库自身`wait_timeout`的特性， 其会自动关闭太长时间不用的 TCP 连接。我们单纯在 driver 层面设置保持长连接是没有意义的，甚至在还会因为无效而导致资源的消耗拖慢数据库的速度，及时地关闭 connection 也是很重要的，所以在使用的时候需要设置合适的关闭时间。如果有足够的时间对自己的系统进行压力测试，还需要设置合适的`SetMaxOpenConns`和`SetMaxIdleConns`，以提升系统效率。

## 参考资料

1. <http://go-database-sql.org/connection-pool.html>
2. <https://www.zhihu.com/question/52904634/answer/132588175>
3. <https://www.jdatalab.com/information_system/2017/02/16/database-driver.html>
4. <https://en.wikipedia.org/wiki/Open_Database_Connectivity#Drivers_and_Managers>
5. <https://github.com/go-sql-driver/mysql/issues/701>
6. <http://go-database-sql.org/prepared.html>
7. <http://techblog.en.klab-blogs.com/archives/31093990.html>
8. <http://techblog.en.klab-blogs.com/archives/31093990.html>

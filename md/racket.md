---
title: "Racket基础数据类型"
date: 2019-08-06T14:31:27+08:00
lastmod: 2019-08-18T00:06:27+08:00
draft: false
tags: ["racket"]
categories: ["code"]
author: "Rouzip"

contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'
---

> racket作为Lisp的一种方言，通过学习它可以为学习函数式编程打下基础，同时作为不同于C族的一系列语言，也可以扩展编程的思路。下面记录一些学习其的基础知识。

<!--more-->

# Booleans

两个独特的常量来代表布尔值：`#t`和`#f`， 同时`#T`和 `#F` 也会被解析为相同的值，但是小写形式更常被使用。

其可以被`if`， `cond`， `and` 和`or` 等操作所使用。同时除了`#f` 以外的所有值都为`true` 。

# Numbers

该数据类型既是准确的也是不准确的。

**准确情况**

- 任意大小的整数
- 有理数
- 有理数组成的实部与虚部

**不准确情况**

- 符合IEEE标准的浮点数（无理数），包含正负无穷
- 实部或虚部包含浮点数

不准确数据打印其小数点或者指数，精确数字打印其整数与分数部分。通过`#e` 与 `#i` 来强制区分准确与不准确。`#b`，  `#o`， `#x`被用来区分二进制，八进制，十六进制。

计算机在计算准确的数字的时候比计算不准确的数字要慢，其原因是小数可以准确地使用计算机的一个字节。

`=` 运算程序比较的是数字部分，加入准确的数字与不准确的数字进行比较，那么会将不准确的数字转换为准确的数字进行比较。但是`eqv?` 同时比较数字的准确部分和小数部分，只有全部相等才是相等。

同时需要注意的是，有理数与IEEE标准的数字`=`的判断下是不同的。

# Characters

每个字符都被作为`Unicode` 值，即一个21位`bit`的无符号整数。

尽管每个字符都被当作整数，但是其数据类型还是`char`，所以还是需要`char->integer` 和`integer->char` 的相互转换为对应的字符与数字。

`#\`和 `#\u` 分别被作为所代表的字符和不能打印的字符的十六进制数字。一些字符有特殊的含义例如`#\space` 和 `#\newline` 等。

# Strings(Unicode)

字符串的本质是定长的字符数组。常用的转义字符例如`\n` 代表换行， `\r` 代表回车，`\` 后面接三个八进制数字 ， `\u` 后面可以接四个数字代表十六进制。

# Bytes and Byte Strings

一个 *byte* 代表在0到255之间，可以通过`byte?` 来判断数字是否在一 *byte* 来表示。

`byte string` 和 `string` 有相似之处，但是它的内容是字节(byte)而不是字符(character)。对于两者之间的转换，可以使用`UTF-8`， `Latin-1` 和`local's encoding` 三种编码方式。

# Symbols

一个 `symbol` 是一个以\` 开头的原子量，同时其是大小写敏感的。可以使用`#ci` 前缀来使得其变为小写。对于输入，除了空格和以下特殊字符外，任何字符都可以直接出现在标识符中

> `(`, `)`, `[`, `]`, `{`, `}`, `"`, `,`, `'`, \`, `;`, `#`, `|`, `\` 

`#` 只是在`symbol` 的开头不允许使用，然后就是在`%` 后面不允许。除此以外`#` 是允许被使用的，同时`.`本身不是一个`symbol`。`\` 和 `|` 被用来打印特殊字符，以防止和数字形式搞混。

# Keywords

一个`keyword` 与一个`symbol` 类似，但是它的打印前缀为`#:` 。

更确切地来说，一个`keyword` 可以模糊地看作一个识别符。同样地，与通过引用标识符来产生一个`symbol`类似，可以通过引用`keyword`来产生一个值。

> 尽管`keyword` 与`symbol`和标识符类似，它们仍旧是不同的概念。`keyword`被用作参数列表和某些句法形式中的特殊标记。

# Pairs 和 Lists

一个`pair` 可以由两个任意的变量组成。`cons` 产生一个`pair` ，`car` 返回`pair `的第一个值，`cdr` 返回`pair` 的第二个值。

一个`list` 是`pair` 的集合，这个概念有点类似于python中的元组。直接打印的时候`list` 有前缀`'` ，并且在每个元素中间用`.` 隔开，可以使用`display` 和 `write` 来省略前缀。

`pair` 是不可变的，这与传统的Lisp相反。但是可以使用`mcons` 搭配`mcar` 和 `mcdr` 来产生可变的`pair` ，使用`write` 和 `display` 会出现`{}` 。

# Vectors

`vector` 是固定长度数字，其中的值是任意的。与`list` 不同的是，其支持动态的读取和更新其中的元素。

`vector` 开始的前缀是`'#` ，并且当一个值不能被`quote`的时候，其都会默认为`vector` 。

# Hash Tables

`hash table` 实现了键值与值的映射。但是根据定义时候的不同，其比较键值的方法也不同，`make-hash`， `make-hasheqv` 和 `make-hasheq` 分别通过`equal?`， `eqv?` 和 `eq?` 来比较键值。可以通过`hash-set!` 来进行赋值，通过`hash-ref` 来读取值。

除了一个个赋值以外，还可以通过使用`#hash` （基于`equal?`）， `#hasheqv` （基于`eqv?`）和`#hasheq` （基于`eq?`）作为前缀，来生成，例子如下：

```scheme
> (define ht #hash(("apple" . red) 
                   ("banana" . yellow)))

>(hash-ref ht "apple")
'red
```

# Boxes

一个`box` 就像是一个单元素的`vector` ，它以`#&` 作为被打印的前缀。但由于其为常量，所以将其作为表达式没有意义。

# Void 和 Undefined

一些过程和表达式不需要结果的值，那么就可以使用`void` ，其被打印为`#<void>` 。

`undefined` 常量被打印为`#<undefined>` ，当值不可以获得时，即为`undefined` 。

# 参考资料

1. https://docs.racket-lang.org/guide/datatypes.html
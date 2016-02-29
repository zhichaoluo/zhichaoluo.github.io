---
layout: post
title: "R中的正则表达式及字符处理函数总结"
author: "yphuang"
date: "2016-02-29"
categories: blog
tags: [R,正则表达式,字符串处理]

---

我们日常生活中接触到的大部分数据都是以文本的形式存在。如何高效地处理文本数据，将看似杂乱无章的数据整理成可以进行统计分析的规则数据，是『数据玩家』必备的一项重要技能。

今天，我们要学习的『正则表达式』和『字符处理函数』将助你成为点石成金的数据魔法师。


# 正则表达式

在进行[爬虫任务](http://yphuang.github.io/blog/2016/01/21/R-Scraing-JD-MobilePhone-Information/)的时候，部分情况下，我们可以使用Xpath来提取我们需要的网页信息。但是，当我们需要的数据以一定的规则隐藏在一段文字中时，就不可避免地要使用到正则表达式。

正则表达式是对字符串类型数据进行匹配判断，提取等操作的一套逻辑公式。

处理字符串类型数据方面，高效的工具有Perl和Python。如果我们只是偶尔接触文本处理任务，则学习Perl无疑成本太高；如果常用Python，则可以利用成熟的正则表达式模块：`re`库；如果常用R，则使用Hadley大神开发的`stringr`包则已经能够游刃有余。

下面，我们先简要介绍重要并通用的正则表达式规则。接着，总结一下`stringr`包中重要的字符处理函数。

_如果有时间，我将后续补充一个综合的使用案例。_



## 元字符

正则表达式中，有12个字符被保留用作特殊用途。他们分别是：

```
[ ] \ ^ $ . | ? * + ( )

```

它们的作用如下：

- `[ ]`：括号内的任意字符将被匹配；

- `\`：具有两个作用：
    + 1.对元字符进行转义
    + 2.一些以`\`开头的特殊序列表达了一些字符串组
 
- `^`：匹配字符串的开始.将`^`置于character class的首位表达的意思是取反义。如`[^5]`表示匹配除了"5"以外的任何字符。

- `$`：匹配字符串的结束。但将它置于character class内则消除了它的特殊含义。如`[akm$]`将匹配'a','k','m'或者'$'.

- `.`：匹配除换行符以外的任意字符。

- `|`：或者

- `?`：前面的字符(组)最多被匹配一次

- `*`：前面的字符(组)将被匹配零次或多次

- `+`：前面的字符(组)将被匹配一次或多次

- `( )`：表示一个字符组，括号内的字符串将作为一个整体被匹配。
    




## 重复

|代码|含义说明|
|:---:|:---:|
|`?`|重复零次或一次|
|`*`|重复零次或多次|
|`+`|重复一次或多次|
|`{n}`|重复n次|
|`{n,}`|重复n次或更多次|
|`{n,m}`|重复n次到m次|


## 转义

如果我们想查找元字符本身，如"?"和"*"，我们需要提前告诉编译系统，取消这些字符的特殊含义。这个时候，就需要用到转义字符`\`，即使用`\?`和`\*`.当然，如果我们要找的是`\`,则使用`\\`进行匹配。


**注：R中的转义字符则是双斜杠：`\\`**


## R中预定义的字符组

|代码|含义说明|
|:---:|:---:|
|`[:digit:]`|数字：0-9|
|`[:lower:]`|小写字母：a-z|
|`[:upper:]`|大写字母：A-Z|
|`[:alpha:]`|字母：a-z及A-Z|
|`[:alnum:]`|所有字母及数字|
|---|---|
|`[:punct:]`|标点符号，如`. , ;`等|
|`[:graph:]`|Graphical characters,即[:alnum:]和[:punct:]|
|`[:blank:]`|空字符，即：Space和Tab|
|`[:space:]`|Space，Tab，newline，及其他space characters|
|`[:print:]`|可打印的字符，即：[:alnum:]，[:punct:]和[:space:]|



## 代表字符组的特殊符号

|代码|含义说明|
|:---:|:---:|
|`\w`|字符串，等价于`[:alnum:]`|
|`\W`|非字符串，等价于`[^[:alnum:]]`|
|`\s`|空格字符，等价于`[:blank:]`|
|`\S`|非空格字符，等价于`[^[:blank:]]`|
|`\d`|数字，等价于`[:digit:]`|
|`\D`|非数字，等价于`[^[:digit:]]`|
|`\b`|Word edge（单词开头或结束的位置）|
|`\B`|No Word edge（非单词开头或结束的位置）|
|`\<`|Word beginning（单词开头的位置）|
|`\>`|Word end（单词结束的位置）|



***

# `stringr`包中的重要函数


|函数|功能说明|R Base中对应函数|
|:---:|:---:|:----------------:|
|使用正则表达式的函数|||
|`str_extract()`|提取首个匹配模式的字符|`regmatches()`|
|`str_extract_all()`|提取所有匹配模式的字符|`regmatches()`|
|`str_locate()`|返回首个匹配模式的字符的位置|`regexpr()`|
|`str_locate_all()`|返回所有匹配模式的字符的位置|`gregexpr()`|
|`str_replace()`|替换首个匹配模式|`sub()`|
|`str_replace_all()`|替换所有匹配模式|`gsub()`|
|`str_split()`|按照模式分割字符串|`strsplit()`|
|`str_split_fixed()`|按照模式将字符串分割成指定个数|-|
|`str_detect()`|检测字符是否存在某些指定模式|`grepl()`|
|`str_count()`|返回指定模式出现的次数|-|
|其他重要函数|||
|`str_sub()`|提取指定位置的字符|`regmatches()`|
|`str_dup()`|丢弃指定位置的字符|-|
|`str_length()`|返回字符的长度|`nchar()`|
|`str_pad()`|填补字符|-|
|`str_trim()`|丢弃填充，如去掉字符前后的空格|-|
|`str_c()`|连接字符|`paste(),paste0()`|


可见，`stringr`包中的字符处理函数更丰富和完整（其实还有更多函数），并且更容易记忆。或许速度也会更快。



***

# 其他相关的重要函数

windows下处理字符串类型数据最头疼的无疑是编码问题了。这里介绍几个编码转换相关的函数。

|函数|功能说明|
|:---:|:---:|
|`iconv()`|转换编码格式|
|`Encoding()`|查看编码格式；或者指定编码格式|
|`tau::is.locale()`|tests if the components of a vector of character are in the encoding of the current locale|
|`tau::is.ascii()`||
|`tau::is.utf8()`| tests if the components of a vector of character are true UTF-8 strings|

参考文献：

- 『[Handling_and_Processing_Strings_in_R](http://gastonsanchez.com/Handling_and_Processing_Strings_in_R.pdf)』

- 『Automated Data Collection with R』

- [正则表达式30分钟入门教程](http://deerchao.net/tutorials/regex/regex.htm)

- [深入浅出之正则表达式](http://dragon.cnblogs.com/archive/2006/05/08/394078.html)






---
layout: post
title: "RSQLite系列1——入门教程"
author: "yphuang"
date: 2016-02-19
categories: blog
tags: [RSQLite,SQL]

---

***

## 1.什么是SQLite

> SQLite 是一个软件库，实现了自给自足的、无服务器的、零配置的、事务性的 SQL 数据库引擎。SQLite 是在世界上最广泛部署的 SQL 数据库引擎。SQLite 源代码不受版权限制。


***

## 2.为何选择SQLite

选择SQLite的原因很简单，因为它:
  
- 开源
- 轻量级
- 安装配置简单
- 不存在繁琐的用户管理
- 兼容标准的SQL语句操作

这几个特性对一个SQL新手来说，是最好不过的。


***

## 3.RSQLite入门

### 基本概念

- Table：观测的集合
- Field：类似于R中data.frame的column names(列名)
- Column：变量
- Row：观测
- data types:
     + NULL
     + INTEGER,带符号的整数
     + REAL,浮点值
     + TEXT,文本字符串
     + BLOB,blob数值（[binary large object](https://en.wikipedia.org/wiki/Binary_large_object)）,主要针对图片视频等数据对象。


### 安装配置


```r
install.packages("RSQLite")
```


### 建立一个连接对象


```r
library(RSQLite)
```

```
## Loading required package: DBI
## Loading required package: methods
```

```r
db<- dbConnect(SQLite(), dbname = 'Test.sqlite')
```

这样，一个连接数据库的对象就建立起来了。不过，严格来说，这个时候，数据库还未建立起来。


### 建立一个表格

首先，使用CREATE语句建立一个表格。


```r
#当已经存在该table,把它删除，否则后面建立table时会报错
dbSendQuery(conn = db,
            "drop table if exists MOBILE_PHONE")
```

```
## <SQLiteResult>
```

```r
dbSendQuery(conn = db,
            "CREATE TABLE MOBILE_PHONE
            (Product_ID INTEGER,
            product_Name TEXT,
            price REAL,
            Brand_name TEXT)")
```

```
## <SQLiteResult>
```

### 导入数据

#### 向表格添加数据——手动添加

向表格添加数据可以使用INSERT语句。


```r
dbSendQuery(conn = db,
            "INSERT INTO MOBILE_PHONE
            VALUES(1,'iPhone 6s',6000,'Apple')")
```

```
## <SQLiteResult>
```

```r
dbSendQuery(conn = db,
            "INSERT INTO MOBILE_PHONE
            VALUES(2,'华为P8',3000,'华为')")
```

```
## <SQLiteResult>
```

```r
dbSendQuery(conn = db,
            "INSERT INTO MOBILE_PHONE
            VALUES(3,'三星 Galaxy S6',5000,'三星')")
```

```
## <SQLiteResult>
```

#### 查询结果


```r
# the tables in the database
dbListTables(db)
```

```
## [1] "MOBILE_PHONE"
```

```r
#the columns in a table
dbListFields(db,"MOBILE_PHONE")
```

```
## [1] "Product_ID"   "product_Name" "price"        "Brand_name"
```

```r
#the data in the table
head(dbReadTable(db,"MOBILE_PHONE"))
```

```
##   Product_ID   product_Name price Brand_name
## 1          1      iPhone 6s  6000      Apple
## 2          2         华为P8  3000       华为
## 3          3 三星 Galaxy S6  5000       三星
```


### 向表格添加数据——导入外部数据(csv,excel,data.frame)

以[ISLR包中的Hitters数据集](http://rpackages.ianhowson.com/cran/ISLR/man/Hitters.html)为例，导入该数据集。该数据集描述了美国1986年和1987年的棒球运动员相关数据。


```r
library(ISLR)
str(Hitters)
```

```
## 'data.frame':	322 obs. of  20 variables:
##  $ AtBat    : int  293 315 479 496 321 594 185 298 323 401 ...
##  $ Hits     : int  66 81 130 141 87 169 37 73 81 92 ...
##  $ HmRun    : int  1 7 18 20 10 4 1 0 6 17 ...
##  $ Runs     : int  30 24 66 65 39 74 23 24 26 49 ...
##  $ RBI      : int  29 38 72 78 42 51 8 24 32 66 ...
##  $ Walks    : int  14 39 76 37 30 35 21 7 8 65 ...
##  $ Years    : int  1 14 3 11 2 11 2 3 2 13 ...
##  $ CAtBat   : int  293 3449 1624 5628 396 4408 214 509 341 5206 ...
##  $ CHits    : int  66 835 457 1575 101 1133 42 108 86 1332 ...
##  $ CHmRun   : int  1 69 63 225 12 19 1 0 6 253 ...
##  $ CRuns    : int  30 321 224 828 48 501 30 41 32 784 ...
##  $ CRBI     : int  29 414 266 838 46 336 9 37 34 890 ...
##  $ CWalks   : int  14 375 263 354 33 194 24 12 8 866 ...
##  $ League   : Factor w/ 2 levels "A","N": 1 2 1 2 2 1 2 1 2 1 ...
##  $ Division : Factor w/ 2 levels "E","W": 1 2 2 1 1 2 1 2 2 1 ...
##  $ PutOuts  : int  446 632 880 200 805 282 76 121 143 0 ...
##  $ Assists  : int  33 43 82 11 40 421 127 283 290 0 ...
##  $ Errors   : int  20 10 14 3 4 25 7 9 19 0 ...
##  $ Salary   : num  NA 475 480 500 91.5 750 70 100 75 1100 ...
##  $ NewLeague: Factor w/ 2 levels "A","N": 1 2 1 2 2 1 1 1 2 1 ...
```

```r
#建立连接
db.hitters<-dbConnect(SQLite(),dbname = "Hitters.sqlite")

#写入数据
dbWriteTable(conn = db.hitters,name = "Hitters",value = Hitters,overwrite = T,row.names = FALSE)
```

```
## [1] TRUE
```

```r
#读取表格
tmp = dbReadTable(db.hitters,"Hitters")
head(tmp)
```

```
##   AtBat Hits HmRun Runs RBI Walks Years CAtBat CHits CHmRun CRuns CRBI
## 1   293   66     1   30  29    14     1    293    66      1    30   29
## 2   315   81     7   24  38    39    14   3449   835     69   321  414
## 3   479  130    18   66  72    76     3   1624   457     63   224  266
## 4   496  141    20   65  78    37    11   5628  1575    225   828  838
## 5   321   87    10   39  42    30     2    396   101     12    48   46
## 6   594  169     4   74  51    35    11   4408  1133     19   501  336
##   CWalks League Division PutOuts Assists Errors Salary NewLeague
## 1     14      A        E     446      33     20     NA         A
## 2    375      N        W     632      43     10  475.0         N
## 3    263      A        W     880      82     14  480.0         A
## 4    354      N        E     200      11      3  500.0         N
## 5     33      N        E     805      40      4   91.5         N
## 6    194      A        W     282     421     25  750.0         A
```


### 建立基本查询

使用SELECT语句建立一个关于行的查询。


```r
dbGetQuery(db.hitters,"select * from Hitters where Salary >= 1000")[1:5,]
```

```
##   AtBat Hits HmRun Runs RBI Walks Years CAtBat CHits CHmRun CRuns CRBI
## 1   401   92    17   49  66    65    13   5206  1332    253   784  890
## 2   591  168    19   80  72    39     9   4478  1307    113   634  563
## 3   627  177    25   98  81    70     6   3210   927    133   529  472
## 4   677  238    31  117 113    53     5   2223   737     93   349  401
## 5   614  163    29   89  83    75    11   5017  1388    266   813  822
##   CWalks League Division PutOuts Assists Errors Salary NewLeague
## 1    866      A        E       0       0      0   1100         A
## 2    319      A        W      67     147      4   1200         A
## 3    313      A        E     240     482     13   1350         A
## 4    171      A        E    1377     100      6   1975         A
## 5    617      N        W     303       6      6   1900         N
```

使用SELECT语句建立一个关于列的查询。


```r
dbGetQuery(db.hitters,"select League,Hits,Salary from Hitters where League = 'A'")[1:5,]
```

```
##   League Hits Salary
## 1      A   66     NA
## 2      A  130    480
## 3      A  169    750
## 4      A   73    100
## 5      A   92   1100
```

### 其他更复杂的查询

可以结合使用SQL逻辑操作符(AND,OR,NOT等)，以及行，列选取等建立其他更复杂的查询操作。


```r
dbGetQuery(db.hitters,"select League,Hits,Salary from Hitters where League = 'A' AND  Salary >= 1000")[1:5,]
```

```
##   League Hits  Salary
## 1      A   92 1100.00
## 2      A  168 1200.00
## 3      A  177 1350.00
## 4      A  238 1975.00
## 5      A  148 1861.46
```

***




## 4.参考文献

- [Creating SQLite databases from R](http://sandymuspratt.blogspot.jp/2012/11/r-and-sqlite-part-1.html)

- [SQLite 教程](http://www.runoob.com/sqlite/sqlite-tutorial.html)

- <https://github.com/ysquared2/RSQLiteTutorial>


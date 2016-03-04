---
layout: post
title: "R文本挖掘之tm包"
author: "yphuang"
date: 2016-03-04
categories: blog
tags: [R,tm,文本挖掘]

---

## 认识tm包

tm包是R文本挖掘方面不可不知也不可不用的一个package。它提供了文本挖掘中的综合处理功能。如：数据载入，语料库处理，数据预处理，元数据管理以及建立“文档-词条”矩阵。

下面，即从tm包提供的各项功能函数的探索出发，一起开始我们的文本挖掘奇幻之旅。

首先，运行下面的几行代码，即可看到介绍tm包的小品文：[Introduction to the tm Package:Text Mining in R](https://cran.r-project.org/web/packages/tm/vignettes/tm.pdf).

```r

install.packages("tm")

library(tm)

vignette("tm")

```


## tm包重要函数初探

### 数据载入及语料库创建


#### 载入数据的格式要求

tm包支持多种格式的数据。用`getreaders()`函数可以获得tm包支持的数据文件格式。


```r
library(tm)
```

```
## Loading required package: NLP
```

```r
getReaders()
```

```
##  [1] "readDOC"                 "readPDF"                
##  [3] "readPlain"               "readRCV1"               
##  [5] "readRCV1asPlain"         "readReut21578XML"       
##  [7] "readReut21578XMLasPlain" "readTabular"            
##  [9] "readTagged"              "readXML"
```


#### 载入数据的方式

tm包中主要管理文件的数据结构称为语料库（Corpus），它表示一系列文档的集合。

语料库又分为动态语料库（Volatile Corpus）和静态语料库（Permanent Corpus）。

动态语料库将作为R对象保存在内存中，可以使用`VCorpus()`或者`Corpus()`生成。

而动态语料库则作为R外部文件保存，可以使用`PCorpus()`函数生成。

先来看一下`VCorpus()`函数的使用。

```
VCorpus(x, readerControl = list(reader = reader(x), language = "en"))

as.VCorpus(x)

```
第一个参数x即文本数据来源。对于`as.VCorpus()`中的x，指定的是一个R对象；对于`VCorpus()`，可以使用以下几种方式载入x。

- `DirSource()`:从本地文件目录夹导入

- `VectorSource()`:输入文本构成的向量

- `DataframeSource()`:输入文本构成的data frame


对于第二个参数`readerControl`,即指定文件类型的对应的读入方式。默认使用tm支持的（即getReaders()中罗列的）一系列函数。`language`即文件的语言类型。似乎不能支持中文。这个问题稍后解释如何解决。

这里，使用tm包自带的一个数据集进行语料库创建的测试。

- `DirSource()`方式：


```r
txt<-system.file("texts","txt",package = 'tm')

(docs<-Corpus(DirSource(txt,encoding = "UTF-8")))
```

```
## <<VCorpus>>
## Metadata:  corpus specific: 0, document level (indexed): 0
## Content:  documents: 5
```

- `VectorSource()`方式：


```r
docs<-c("this is a text","And we create a vector.")

VCorpus(VectorSource(docs))
```

```
## <<VCorpus>>
## Metadata:  corpus specific: 0, document level (indexed): 0
## Content:  documents: 2
```

下面，导入一个数据集『冰与火之歌』全五部（没错，我就是来剧透的～～），作为后面练习的例子。




```r
IceAndSongs<-VCorpus(DirSource(directory = "D:/my_R_workfile/RPROJECT/textming/data/IceAndSongs",encoding = "UTF-8"))
```

### 数据导出

将语料库导出至本地硬盘上，可以使用`writeCorpus()`函数.


```r
writeCorpus(IceAndSongs,path = "D:/my_R_workfile/RPROJECT/textming/data/Corpus")
```



### 语料库的查看及提取

可以使用`print()`和`summary()`查看语料库的部分信息。而完整信息的提取则需要使用`inspect()`函数。


```r
inspect(IceAndSongs[1:2])
```

```
## <<VCorpus>>
## Metadata:  corpus specific: 0, document level (indexed): 0
## Content:  documents: 2
## 
## [[1]]
## <<PlainTextDocument>>
## Metadata:  7
## Content:  chars: 1745859
## 
## [[2]]
## <<PlainTextDocument>>
## Metadata:  7
## Content:  chars: 2018112
```

文件太大，而没有打印出来。我们可以使用`writeLines()`函数进行完全打印查看。


```r
writeLines(as.character(IceAndSongs[[1]]))
```



对于单个文档的提取，可以类型列表取元素子集一样使用`[[`操作。


```r
identical(IceAndSongs[[1]],IceAndSongs[["冰与火之歌1.txt"]])
```

```
## [1] TRUE
```

### 数据转换

创建好语料库之后，一般还需要做进一步的处理，如：消除空格（Whitespace）,大小写转换，去除停止词，词干化等。

所有的这些处理都可以使用`tm_map()`函数，通过map的方式将转化函数应用到每一个文档语料上。



#### 消除空格


```r
IceAndSongs<-tm_map(IceAndSongs,stripWhitespace)
```


#### 去除数字


```r
IceAndSongs<-tm_map(IceAndSongs,removeNumbers)
```

#### 去除标点符号


```r
IceAndSongs<-tm_map(IceAndSongs,removePunctuation)
```

#### 大小写转换


```r
IceAndSongs<-tm_map(IceAndSongs,tolower)
```

#### 消除停止词

tm包中自带了停止词集。


```r
IceAndSongs<-tm_map(IceAndSongs,removeWords,stopwords("english"))
```



当然，也可以指定你自己设定的停止词集,将`stopwords("english")`替换成你自己的停止词集对象即可。



#### 词干化

词干化，即[词干提取](https://zh.wikipedia.org/wiki/%E8%AF%8D%E5%B9%B2%E6%8F%90%E5%8F%96)。指的是去除词缀得到词根的过程─—得到单词最一般的写法。

如以单复数等多种形式存在的词，或多种时态形式存在的同一个词，它们代表的其实是同一个意思。因此需要通过词干化将它们的形式进行统一。



```r
tm_map(IceAndSongs,stemDocument)
```

```
## <<VCorpus>>
## Metadata:  corpus specific: 0, document level (indexed): 0
## Content:  documents: 5
```


#### 去除特殊字符


```r
for(i in seq(IceAndSongs)){
  IceAndSongs[[i]]<-gsub("/"," ",IceAndSongs[[i]])
  IceAndSongs[[i]]<-gsub("@"," ",IceAndSongs[[i]])
  IceAndSongs[[i]]<-gsub("-"," ",IceAndSongs[[i]])
}
```



### 过滤

过滤功能能够选择出符合我们需要的文档。



```r
idx<-meta(IceAndSongs,"id") == "冰与火之歌1.txt"

IceAndSongs[idx]
```

也可以进行全文搜索匹配。如含有"winter is coming"的文档。


```r
tm_filter(IceAndSongs,FUN = function(x){ any(grep("winter is coming",content(x)))})
```

### 元数据管理

元数据指的是对文档进行标签化的附加信息。可以通过meta()函数进行元数据管理。

`DublinCore()`函数提供了一套介于Simple Dublin Core元数据和tm元数据之间的映射机制，用于获得或设置文档的元数据信息。



```r
DublinCore(IceAndSongs[[1]],tag = "creator") <- "R.R.Martin"

DublinCore(IceAndSongs[[1]])

meta(IceAndSongs[[1]])
```



以上操作示例主要是针对文档级别的元数据管理。而元数据标签其实对应了两个级别：

- 整个语料库级别：文档的集合

- 单个文档级别

而文档级别的标签，可以用于文档分类(classification)。

下面演示一下语料库级别的元数据管理。



```r
meta(IceAndSongs,tag = "test",type = "corpus")<-"test meta"

meta(IceAndSongs,type = "corpus")
```



### 创建词条-文档矩阵


词条-文档矩阵是一个非常重要的对象，它是后续建立文本分类，文本聚类等模型的基础。

词条-文档矩阵指的是词条作为行，文档标签作为列的稀疏矩阵。当然，也可以建立“文档-词条矩阵”。对应的两个操作函数为：`TermDocumentMatrix()`和`DocumentTermMatrix()`.


```r
dtm<-DocumentTermMatrix(IceAndSongs)

inspect(dtm[1:5,100:105])
```

默认情况下，矩阵的元素是词的频率。而我们还有一个重要参数可以设置。可以将矩阵的元素转化为TF-IDF值。



```r
dtm_2<-DocumentTermMatrix(IceAndSongs,
                        control = list(removePunctuation = TRUE,stopwords = FALSE,weighting = 
              function(x)weightTfIdf(x,normalize = TRUE)))

inspect(dtm[1:5,10:15])
```


### 对文档词条矩阵操作

tm包提供的文档-词条矩阵操作有：词频过滤；词语之间的相关性计算；去除稀疏词等。


```r
findFreqTerms(dtm,10)

findAssocs(dtm,"winter",0.5)

inspect(removeSparseTerms(dtm,0.4))
```

### 字典

字典是一个字符集。它可以作为一个控制参数传入`DocumentTermMatrix()`,从而选择我们需要的词条建立文档-词条矩阵。


```r
inspect(DocumentTermMatrix(IceAndSongs,
                           list(dictionary = c("winter","power","ice"))))
```



## 参考文献

- 『R 语言环境下的文本挖掘-刘思喆』

- 『[Introduction to the tm Package:Text Mining in R](https://cran.r-project.org/web/packages/tm/vignettes/tm.pdf)』

- 『[数据科学18：文本挖掘1](http://jackycode.github.io/blog/2014/06/18/text-mining1/)』

---
layout: post
title: "R字符串处理应用之邮件考勤自动化"
author: "yphuang"
date: "2016-03-09"
categories: blog
tags: [R,字符串处理,正则表达式]

---

## 引言

最近发现，担任助教真不是一件轻松的事情啊。虽然老师一直在想方设法减轻我的工作负担，可是核对名单真的是一件考验眼力和耐力的事情。

最近有一件非常艰巨的任务：检查上周的『考勤邮件』。这个『考勤邮件』，容我耐心的解释一番。上周，老师为了不浪费大家的时间，通过在某几分钟内发送一封邮件到公共邮箱的方法来签到。

而我今天才拿到选课学生的名单。我们知道，邮件过了一段时间，标题显示的接收时间就会改变。这个时候，为了确定邮件的发送时间，我必须要每封邮件都打开来看一下，再找到相应的名单，然后打上一个满意的勾。然而，这可是五十多封邮件啊！！！

立志成为数（一）据（名）科（懒）学（人）家的我，怎能甘心做如此机械的活呢？于是，想起最近总结的一篇字符处理相关的博客，正好可以用上。

说干就干！下面，我们就来探索一番，如何用R实现邮件考勤全自动化。


## 载入数据

首先，从公共邮箱批量下载数据。并载入R。


```r
library(stringr)
library(openxlsx)
#load Name list

NameInformation<-read.xlsx("data/名单_20160308.xlsx",sheet = 1,colNames = TRUE)
str(NameInformation)
NameList<-NameInformation$姓名
NameList<-str_trim(NameList)
#read E-mail name
EmailName<-dir("data/第一次考勤/信件打包")
```

## 查看缺勤人员名单

载入数据的第一步，当然是先看看是否全勤啦～

如果没人缺勤，后面的日期提取等脏活累活就可以不用干啦！（再次暴露了懒人的本性= =!）



```r
#match name list,remove E-mails which's subject NOT contain names ON the namelist

# detact weather the subject contains the name
ExistStatus<-lapply(NameList, function(x){
  Exist<-str_detect(EmailName,x)
  return(sum(Exist))
})

ExistStatus<-unlist(ExistStatus)

# find not checked names
print(paste0("缺勤的同学：",NameList[!ExistStatus]))

#str_detect(EmailName,"张三")
```

果不其然，有些同学还是不够团结啊！有几个没发邮件的。当然，谨慎的黄老师还是用`str_detect()`函数重新核对了一下，误伤了同学可不好办呐。

## 提取邮件接收时间

打开文本编辑器，仔细看了一下几封邮件，发现日期格式大概是这样的：

> Date: Wed, 2 Mar 2016 08:06:28 +0800

先将邮件内容读入一个list。接着，用正则表达式，把含有`Date: Wed, 2 Mar 2016`字样的这一行提取出来。然后，只提取我们需要的时间。最后，使用`striptime()`函数将字符串转换成时间格式。然而，在Windows下一直得到的返回值一直是NA，在Linux下可以正确转换。万恶的微软！



```r
###########################
## check in email received time   ###
# get email content
  
EmailContent<-lapply(EmailName,function(x){
  readLines(con<-file(paste0("data/第一次考勤/信件打包/",x),encoding = 'UTF-8'))
})

# get date and time
EmailDate<-lapply(EmailContent,function(x){
  date_vec<-str_subset(x,"Date: Wed, 2 Mar 2016")
  date<-str_sub(date_vec,start = 7, end = 30)
  return(date)
})


# format conveting
## windows 下有问题，linux下没问题
EmailDate<-lapply(EmailDate,function(x){
  strptime(x,"%a, %d %b %Y %I:%M:%S")
})

EmailDate<-unlist(EmailDate)
```


## 提取名字

为了做到有凭有据，还是要从主题提取一下名字。这个时候，跟已有的选课名单进行一一匹配即可。

然而，我们的课实在太火爆！有些没有选到课的同学，为了刷刷自己的存在感，也发来了『贺电』。这可不好办！！！如果是一个个核对，找了半天，发现没在选课名单上，岂不气煞人也！然而，有了R，我只需要一个IF语句就搞定啦。

还有一些不知是手抖还是为了刷存在感，的同学，连发了几封E-mail。当然，我并没有生气，我只需要一行代码就可以轻松化解难题。



```r
########################################33
## exteact name on the subject     ##

NameOnSubject<-lapply(EmailName,function(x){
  ExtractName<-str_extract(x,NameList)
  ## 有的同学没有选课也发了邮件，或者不小心下载了垃圾邮件
  if(sum(is.na(ExtractName))==51){
    return(NA)
  }
  else{
    Name<- ExtractName[!is.na(ExtractName)]
    return(Name)
  }
})

### combine Date and name 
NameOnSubject<-unlist(NameOnSubject)

EmailData<-data.frame(CheckTime= EmailDate,NameOnSubject = NameOnSubject,stringsAsFactors = FALSE)

#先去掉名字为NA的邮件
EmailData<-EmailData[!is.na(EmailData$NameOnSubject),]

# 有的同学手抖，发了几封邮件,需要去重
EmailData<-EmailData[!duplicated(EmailData[,"NameOnSubject"]),]

View(EmailData)
str(EmailData)
```


## 合并数据集

最后，跟选课名单进行合并即大功告成啦！


```r
CheckInNameList<-data.frame(Name = NameInformation$姓名,CheckInStatus = ExistStatus)
str(CheckInNameList)
# merge namelist and the EmailData
CheckInNameData<-merge(x = CheckInNameList, y = EmailData, by.x = "Name",
      by.y = "NameOnSubject",all = TRUE,all.x = TRUE )

library(DT)
datatable(CheckInNameData)
```


其实仔细想想，邮箱考勤这个机制存在很大的bug。

我们可以一起发挥智慧，思考一下如何加入防作弊机制。如给每个在场的人发送一个唯一的随机码随邮件发送？（不想上课的同学会不会打我？！逃～～～～）

## 参考文献：

- [R中的正则表达式及字符处理函数总结](http://yphuang.github.io/blog/2016/02/29/R-Regular-Expressions-And-String-Functions/)


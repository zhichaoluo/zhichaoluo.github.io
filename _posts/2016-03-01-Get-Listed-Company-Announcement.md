
---
layout: post
title: "R爬虫之上市公司公告批量下载"
author: "yphuang"
date: "2016-03-01"
categories: blog
tags: [R,Selenium,爬虫]

---

## selenium的安装及使用介绍

Selenium是一个用于测试网页应用的开源软件。它提供了浏览器中的点击，滚动，滑动，及文字输入等驱动程序。这样，利用Selenium即可以通过脚本程序来替代人工进行测试一个开发软件的各种功能。

在处理爬虫任务中，经常遇到需要输入文字，进行下拉菜单选择，以及鼠标点击等情景。这个时候，selenium就派上大用场了。

下面，我们先介绍一下Selenium的使用环境配置，接着介绍如何通过R的拓展包`Rwebdriver`来使用Selenium，最后，展示一个爬虫案例应用。

### 安装配置

- 安装jre:
    + 下载地址：<http://www.java.com/en/download/manual.jsp#win>

- 配置jre环境变量
    + <http://jingyan.baidu.com/article/09ea3ede2b5f86c0aede39b9.html>
    + <http://blog.sciencenet.cn/blog-830496-778851.html>

- 下载selenium，并放至指定位置
    + 下载地址：<http://www.seleniumhq.org/download/>

### 启动selenium


1. 打开命令提示符

2. 进入selenium所在路径

3. 启动selenium


```
cd "C:\Program Files (x86)\Rwebdriver"
java -jar selenium-server-standalone-2.49.0.jar

```
### selenium接口函数介绍


『Automated Data Collection with R』一书的作者开发了R包`Rwebdriver`,用于连接启用selenium。

该R包重要的函数如下：

|函数|作用说明|
|:---:|:---:|
|`start_session()`|建立一个会话|
|`start_session()`|关闭会话|
|`post.url()`|在浏览器中访问一个url|
|`get.url()`|从当前网页获得URL|
|`element_find()`|通过method和value定位元素|
|`element_xpath_find()`|通过xpath定位元素|
|`element_click()`|点击元素ID|
|`key()`|发送文字至当前元素位置|
|`element_clear()`|清楚当前元素位置输入值|
|`page_source()`|获得当前网页HTML源码|



更多细节请参考：

-『Automated Data Collection with R』第9章P253-P259

- [Selenium with Python](http://selenium-python.readthedocs.org/)

## 网页开发工具的使用介绍

（手动演示）

<center>
    <p><img src="https://raw.githubusercontent.com/yphuang/yphuang.github.io/master/img/Check_for_Element.png" align="center"></p>
</center>

### xpath的提取

（手动演示）

<center>
    <p><img src="https://raw.githubusercontent.com/yphuang/yphuang.github.io/master/img/get_xpath.png" align="center"></p>
</center>

### XML提取器相关函数的使用

- `xpathSApply(doc,path,fun = NULL)`

可传入的`fun`如下：

|函数|返回值|
|:---:|:---:|
|`xmlName`|节点名称|
|`xmlValue`|节点数值|
|`xmlGetAttrs`|节点属性|
|`xmlChildren`|节点的子节点|
|`xmlSize`|节点大小|



## 案例演示——爬取上海证券交易所上市公司公告信息

```r
#### packages we need ####
## ----------------------------------------------------------------------- ##
require(stringr)
require(XML)
require(RCurl)
library(Rwebdriver)

# set path
setwd("ListedCompanyAnnouncement")
# base url
BaseUrl<-"http://www.sse.com.cn/disclosure/listedinfo/announcement/"


#start a session
quit_session()
start_session(root = "http://localhost:4444/wd/hub/",browser = "firefox")


# post Base Url
post.url(url = BaseUrl)

# get xpath
StockCodeField<-element_xpath_find(value = '//*[@id="inputCode"]')
ClassificationField<-element_xpath_find(value = '/html/body/div[7]/div[2]/div[2]/div[2]/div/div/div/div/div[2]/div[1]/div[3]/div/button')
StartDateField<-element_xpath_find(value = '//*[@id="start_date"]')
EndDateField<-element_xpath_find(value = '//*[@id="end_date"]')
SearchField<-element_xpath_find(value = '//*[@id="btnQuery"]')


# fill stock code
StockCode<-"600000"

element_click(StockCodeField)
keys(StockCode)

Sys.sleep(2)

#fill classification field 
element_click(ClassificationField)

# get announcement xpath
RegularAnnouncement<-element_xpath_find(value = '/html/body/div[7]/div[2]/div[2]/div[2]/div/div/div/div/div[2]/div[1]/div[3]/div/div/ul/li[2]')
Sys.sleep(2)
element_click(RegularAnnouncement)

# #fill start and end date 
# element_click(StartDateField)

# today's xpath
EndToday<-element_xpath_find(value = '/html/body/div[13]/div[3]/table/tfoot/tr/th')
Sys.sleep(2)
element_click(EndDateField)
Sys.sleep(2)
element_click(EndToday)

#click search
element_click(SearchField)

###################################
####获得所有文件的link           ##
all_links<-character()


#首页链接
pageSource<-page_source()
parsedSourcePage<-htmlParse(pageSource, encoding = 'utf-8')

pdf_links<-'//*[@id="panel-1"]/div[1]/dl/dd/em/a'

all_links<-c(all_links,xpathSApply(doc = parsedSourcePage,path = pdf_links,
                                   xmlGetAttr,"href"))



#############################
##遍历所有link，下载文件
for(i in 1:length(all_links)){
  Sys.sleep(1)
  if(!file.exists(paste0("file/",basename(all_links[i])))){
    download.file(url = all_links[i],destfile = paste0("file/",basename(all_links[i])),mode = 'wb')
    
  }
}


```

## 参考文献

- 『Automated Data Collection with R』

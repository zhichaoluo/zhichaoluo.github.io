---
layout: post
title: "R爬虫之京东商城手机信息批量获取"
author: "yphuang"
date: 2016-01-21
categories: blog
tags: [爬虫,R]

---

在人手一部智能手机的移动互联网时代，智能手机对很多人来说，它就像我们身上生长出来的一个器官那样重要。如果你不能对各大品牌的『卖点』和『受众』侃上一阵，很可能会被怀疑不是地球人。

今天我们来探索一下，如何从『[京东商城](http://search.jd.com/Search?keyword=%E6%89%8B%E6%9C%BA&enc=utf-8&wq=%E6%89%8B%E6%9C%BA&pvid=3hqqwrji.4umxwy)』爬取各大品牌的手机信息。


## 1.预备知识

R爬虫需要掌握的技能包括：

* 基本的网页知识，如html，XML文件的解析

* 分析XPath

* 使用网页开发工具

* 异常捕捉的处理

* 字符串的处理

* 正则表达式的使用

* 数据库的基本操作

不过不要担心，目前只需要掌握前三项技能，即可开始练习。

前三项技能的掌握可以参考 _Automated Data Collection with R_ 一书。正常情况下，一天之内大致即可掌握。

## 2.页面分析

(待完善)


## 3.提取各大品牌的链接



```r
#### packages we need ####
## ----------------------------------------------------------------------- ##
require(stringr)
require(XML)
require(RCurl)
library(Rwebdriver)

setwd("JDDownload")

BaseUrl<-"http://search.jd.com"

quit_session()
start_session(root = "http://localhost:4444/wd/hub/",browser = "firefox")

# post Base Url
post.url(url = BaseUrl)

SearchField<-element_xpath_find(value = '//*[@id="keyword"]')
SearchButton<-element_xpath_find(value = '//*[@id="gwd_360buy"]/body/div[2]/form/input[3]')
#keyword for search
keywords<-'手机'

element_click(SearchField)
keys(keywords)
element_click(SearchButton)
Sys.sleep(1)
#test
get.url()

pageSource<-page_source()
parsedSourcePage<-htmlParse(pageSource, encoding = 'UTF-8')
## Download Search Results
fname <- paste0(keywords, " SearchPage 1.html")
writeLines(pageSource, fname)

#get all the brand url
Brand<-'//*[@id="J_selector"]/div[1]/div/div[2]/div[3]/ul/li/a/@href'
BrandLinks<-xpathSApply(doc = parsedSourcePage, path = Brand)

View(data.frame(BrandLinks))

BrandLinks<-sapply(BrandLinks,function(x){
  paste0(BaseUrl,"/",x)
  })

save(BrandLinks,file = 'BrandLinks.rda')
```


## 4.访问每个品牌的页面，抓取每个品牌下的商品链接



```r
##############Function 1 #################################3##

### 对各品牌的手机页面进行抓取       ########3#


getBrandPage<-function(BrandUrl,foreDownload = T){
  #获取某品牌搜索页面
  post.url(BrandUrl)
  Brand_pageSource<-page_source()
  #parse 
  parsedSourcePage<-htmlParse(Brand_pageSource, encoding = 'UTF-8')
  
  #get brand name
  BrandNamePath<-'//*[@id="J_crumbsBar"]/div[2]/div/a/em'
  BrandName<-xpathSApply(doc = parsedSourcePage, path = BrandNamePath, fun = xmlValue)
  
  #Save the page
  BrandPageName<-paste0(BrandName,'_PageSource.html')
  #Create a file
  if(!file.exists(BrandName)) dir.create(BrandName)
  # save
  writeLines(text = Brand_pageSource, con = paste0(BrandName,'/',BrandPageName))
  
  # get the product page url
    #path
    Brand_AllProductPath<-'//*[@id="J_goodsList"]/ul/li/div/div[4]/a/@href'
   #url
    Brand_AllProductLinks<-xpathSApply(doc = parsedSourcePage, path = Brand_AllProductPath)
  
#     #remove some false url
#     FalseLink<-grep(x = Brand_AllProductLinks,pattern = 'https',fixed = TRUE)
#     Brand_AllProductLinks<-Brand_AllProductLinks[-FalseLink]
    
    # add a head
    Brand_AllProductLinks<-str_c('http:',Brand_AllProductLinks)
  #save and return the url
    save(Brand_AllProductLinks,file = paste0(BrandName,'_AllProductLinks.rda'))
    return(Brand_AllProductLinks)
}

# test
BrandUrl<-BrandLinks[1]

getBrandPage(BrandUrl)

#get all the links
Brand_ProductLink<-list()
for(i in 1:length(BrandLinks)){
  Sys.sleep(10)
  Brand_ProductLink[[i]]<-getBrandPage(BrandUrl = BrandLinks[i])
}

#clean the links
All_ProductLink<-lapply(Brand_ProductLink,function(x){
   TrueLink<-grep(x = x,pattern = 'http://item.jd.com/',fixed = TRUE,value = FALSE)
   return(x[TrueLink])
})
# save the links
save(All_ProductLink,file = 'All_ProductLink.rda')
```


## 5.访问每个商品页面，提取有用信息 

我们初步提取如下指标：标题(Title),卖点(KeyCount),价格(Price),评论数(commentCount),尺寸(Size),后置摄像头像素(BackBit),后置摄像头像素(ForwardBit),核数(Core),分辨率(Resolution),品牌(Brand),上架时间(onSaleTime).


```r
#################################################
######## Function2 :访问每个商品页面，提取有用信息  ########

Product<-function(ProductLink){
  post.url(ProductLink)
  Sys.sleep(4)
  
  # get the page
  Product_pageSource<-page_source()
  
  #parse 
  Parsed_product_Page<-htmlParse(Product_pageSource, encoding = 'UTF-8')
  
  # get title,,key count,price,CommentCount and so on
  
  #PATH
  TitlePath<-'//*[@id="name"]/h1'
  KeyCountPath<-'//*[@id="p-ad"]'
  PricePath<-'//*[@id="jd-price"]'
  commentCountPath<-'//*[@id="comment-count"]/a'
  SizePath<-'//*[@id="parameter1"]/li[1]/div/p[1]'
  BackBitPath<-'//*[@id="parameter1"]/li[2]/div/p[1]'
  ForwardBitPath<-'//*[@id="parameter1"]/li[2]/div/p[2]'
  CorePath<-'//*[@id="parameter1"]/li[3]/div/p[1]'
  NamePath<-'//*[@id="parameter2"]/li[1]'
  CodePath<-'//*[@id="parameter2"]/li[2]'
  BrandPath<-'//*[@id="parameter2"]/li[3]'
  onSaleTimePath<-'//*[@id="parameter2"]/li[4]'
  ResolutionPath<-'//*[@id="parameter1"]/li[1]/div/p[2]'
  
  Title<-xpathSApply(doc = Parsed_product_Page,path = TitlePath,xmlValue)
  KeyCount<-xpathSApply(doc = Parsed_product_Page,path = KeyCountPath,xmlValue)
  Price<-xpathSApply(doc = Parsed_product_Page,path = PricePath,xmlValue)
  commentCount<-xpathSApply(doc = Parsed_product_Page,path = commentCountPath,xmlValue)
  Size<-xpathSApply(doc = Parsed_product_Page,path = SizePath,xmlValue)
  BackBit<-xpathSApply(doc = Parsed_product_Page,path = BackBitPath,xmlValue)
  ForwardBit<-xpathSApply(doc = Parsed_product_Page,path = ForwardBitPath,xmlValue)
  Core<-xpathSApply(doc = Parsed_product_Page,path = CorePath,xmlValue)
  Name<-xpathSApply(doc = Parsed_product_Page,path = NamePath,xmlValue)
  Code<-xpathSApply(doc = Parsed_product_Page,path = CodePath,xmlValue)
  Resolution<-xpathSApply(doc = Parsed_product_Page,path = ResolutionPath,xmlValue)
  Brand<-xpathSApply(doc = Parsed_product_Page,path = BrandPath,xmlValue)
  onSaleTime<-xpathSApply(doc = Parsed_product_Page,path = onSaleTimePath,xmlValue)
  
  # 整理成data frame
  mydata<-data.frame(Title = Title,KeyCount = KeyCount, Price = Price,
                     commentCount = commentCount, Size = Size, BackBit = BackBit,
                     ForwardBit = ForwardBit, Core = Core, Name = Name,Code = Code,
                     Resolution = Resolution,
                     Brand = Brand, onSaleTime = onSaleTime)
  
  
  #save the page  
  FileName<-paste0('Product/',Brand,Code,'_pageSource.html')
  writeLines(text = Product_pageSource,con = FileName)
 #return the data
  return(mydata)
  
}





# test
quit_session()
start_session(root = "http://localhost:4444/wd/hub/",browser = "firefox")

load(file = 'All_ProductLink.rda')

ProductLink1<-All_ProductLink[[40]][1]

testData<-Product(ProductLink = ProductLink1)



#定义tryCatch

mySpider<-function(ProductLink){
  out<-tryCatch(
    {
      message('This is the try part:')
     Product(ProductLink = ProductLink)
    },
    error=function(e){
      message(e)
      return(NA)
    },
    finally = {
      message("The end!")
    }
  )
  return(out)
}

## loop

# get all data 
ProductInformation<-list()
k <-0

for(i in 1:length(All_ProductLink)){
  for(j in 1:length(All_ProductLink[[i]])){
    k<-k+1
    ProductInformation[[k]]<-mySpider(ProductLink = All_ProductLink[[i]][j])
  }
}

# save my data 
MobilePhoneInformation<-do.call(rbind,ProductInformation)
View(MobilePhoneInformation)
save(MobilePhoneInformation,file = 'MobilePhoneInformation.rda')

nrow(na.omit(MobilePhoneInformation))
View(MobilePhoneInformation)
```

最终，获得800多行的信息，除去缺失值，剩下600多行数据，还不赖。
最后的数据可以在[这里](https://github.com/wise-r/R-Topics/blob/master/Homework2_for_R_Topics/data/MobilePhoneInformation.rda)获得。

不过，数据还需要进一步清洗方能进行分析。

## 6.参考文献

- _Automated Data Collection with R_

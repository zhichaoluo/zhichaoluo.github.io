---
layout: post
title: "R文件系统管理"
author: "yphuang"
date: "2016-03-07"
categories: blog
tags: [R,文件系统管理,备忘录]

---


## 文件系统交互的重要性

文件系统管理是存储和组织我们的数据的方法。在数据科学项目中频繁地接触到文件夹和文件管理。如在爬虫项目中，涉及工作路径的设置，文件夹的创建，文件的批量命名，文件的批量导入等操作。因此，高效、科学的文件管理方式将能够大大地提高我们的工作效率。

下面，让我们一起来全面地梳理R中文件系统管理的相关函数。


## 相关函数介绍

R中与文件系统管理相关的函数非常多，名字也不太好记忆，但从处理的对象上，基本可以划分为：工作路径管理；文件夹管理；文件管理；扩展包管理等类别。而从功能划分上又逃不过“增、删、查、改”这四个类别。

下面，为了方便我们以后查阅，将从处理对象上进行逐一介绍各功能函数。

### 工作路径管理

- `getwd()`：获得当前工作路径

- `setwd()`：设置工作路径

### 文件夹管理

- `dir.create()`：创建一个新的目录


- `unlink()`：删除文件和目录

- `dir()`：查看当前目录下的所有文件夹和文件名

- `list.files()`：查看当前目录的子目录和文件，同`dir()`

- `list.dirs()`：查看当前目录的子目录

- `path.expand()`：扩展路径名

- `normalizePath()`：转换Windows或Linux的路径分割符

- `shortPathName()`：缩短路径的显示长度（Windows中使用）

### 文件管理

- `file.path()`：拼接目录字符串，创建路径名

- `file.info()`：查看文件完整信息

- `file.exists()`：查看文件是否存在

- `file.access()`：查看文件权限

- `Sys.chmod()`：修改文件权限

- `file.rename()`：修改文件名

- `file.remove()`：删除文件

- `file.append()`：文件内容拼接

- `file.copy()`：复制文件

- `basename()`：获得最低等级的路径名（即文件名）

- `dirname()`：获得除文件名外的路径名


### 压缩/解压文件

- `zip()`：创建一个压缩文件

- `unzip()`：从压缩文件中获得某些文件


### 扩展包管理

- `R.home()`：查看R软件的相关目录

- `.Library`：查看R核心包的目录

- `.Library.site`：打印核心包的目录和root用户安装包目录（Linux下）

- `.libPaths()`：打印所有包的存放目录

- `system.file()`：查看指定包所在的目录



## 综合应用案例

### 情景1

以爬虫为例，在批量下载网页时可能需要做这么几件事：

- 创建一个新目录

- 更改当前工作路径

- 文件批量命名并写入本地


```r

if(!file.exists("JDDownload")) dir.create("JDDownload")
setwd("JDDownload")

```

进行文件批量命名并写入本地时，可以自定义一个功能函数，传入一些标记文件唯一性的参数，并在自定义函数内结合`paste()`函数使用。


```r

  FileName<-paste0('Product/',Brand,Code,'_pageSource.html')
  writeLines(text = Product_pageSource,con = FileName)

```

### 情景2

需要获得某一路径下的所有文件名，以及对文件名进行批量的重新命名。可以使用`dir()`函数，`file.rename()`函数，并且和`stringr`包结合使用。

参考之前写的一篇博客：[深入理解SAS之批量数据导入](http://yphuang.github.io/blog/2016/03/03/Uderstanding-SAS-Import-Data-In-Batch/)




## 参考文献

- 『Automated Data Collection with R』

- 『R的极客理想——高级开发篇』

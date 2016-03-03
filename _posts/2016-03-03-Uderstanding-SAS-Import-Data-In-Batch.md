---
layout: post
title: "深入理解SAS之批量数据导入"
author: "yphuang"
date: 2016-03-03
categories: blog
tags: [SAS,macro]

---

当我们在处理『大量的数据』（我偏不说大数据）的时候，如果一个文件一个文件的读入未免太不优雅。（很多同学看到这可能很不屑了：“这不就只是一个循环就能搞定吗？还值得写一篇博客？楼主这博客未免也太LOW了吧”。）

然而，如果我们操作的工具仅限于『上古神器』SAS，那无疑是『戴着镣铐跳舞』。SAS的循环那可不是像R那挥之即来的_for-loop_那么简单。

下面，我们就来演示一番如何优雅地舞动SAS这件『上古神器』，实现批量数据载入。

***

## 1.准备一份文件名数据集

读取某个文件夹下的文件名，这在R中再简单不过了。直接一个`dir()`函数搞定。

当然，我们要做的还要稍微复杂一些。我们需要读取原文件名`in_name`，并提供相应的SAS数据集文件名`out_name`。然而，我们傲娇的SAS在文件名命名的时候，不像R那么随心所欲。它有着这样那样的规矩。如：文件名不能超过32个字符；文件名内不能含小数点`.`。很不幸的是，博主要处理的数据恰好含有小数点`.`。所以，输出文件名不能直接使用原文件名。于是，[前面整理的正则表达式知识](http://yphuang.github.io/blog/2016/02/29/R-Regular-Expressions-And-String-Functions/)终于可以派上用场了。

废话不多说，直接上代码。

```r

library(stringr)

## .dat文件名保存
filename<-dir("F:/Morgage/raw_data/lars__final_year_dat")

Name<-sub("(\\.dat)","",filename,ignore.case = TRUE)
OutName<-str_replace_all(pattern = "\\.",replacement = "_",string = Name)

FileNameList<-data.frame(cbind(Inname = filename, OutName = OutName))

write.table(x = FileNameList,file = "F:/Morgage/raw_data/DAT_FileNameList.csv",
          row.names = FALSE,col.names = FALSE,sep = ' ')

```

当然，如果你说一定要用SAS全程搞定，那也不是不可能。这里提供一行SAS暗黑代码。

```SAS
X "dir F:\Morgage\raw_data\lars__final_year_dat\*.dat/b >F:\Morgage\raw_data\lars__final_year_dat\filename" ;

```
通过X命令，执行命令提示符语句。首先，通过dir命令切换到`F:\Morgage\raw_data\lars__final_year_dat\*.dat`；接着，通过管道操作符`>`把同类型的`.dat`格式文件的文件名输出到文件夹`F:\Morgage\raw_data\lars__final_year_dat\`下的`filename`文件中。

有没有一种Linux命令行的既视感？



## 2.写一个读入数据的宏

```SAS

%macro readfile_dat(v_dir,in_name,out_name);
	/*?????????????v_dir,???????in_name,????SAS???????out_name*/
	%let dir = &v_dir;
	%let gsm= &in_name;
	%let filename = "&dir&gsm";
	libname Mor_dat 'F:\Morgage\sas_data\dat';

	data Mor_dat.&out_name;
		
		infile &filename;

		input variable ; /*此处省略一万个变量的输入格式*/

	run;
	
%mend readfile_dat;
```


## 3.利用数据步实现循环读取数据集

这一步是一个技巧活。这里，读入第一步中创建的数据集，并将每一行得到的变量值作为参数传给第二步中读取我们所需要处理的数据集的宏。

```SAS

data _null_;
	length in_name $ 19 out_name $ 15;
	infile "F:\Morgage\raw_data\DAT_FileNameList.csv"  dsd delimiter=' ';
	input in_name $  out_name $ ;
	call execute(compress('%readfile_dat(F:\Morgage\raw_data\lars__final_year_dat\,'||in_name||','||out_name||')'));

run;


```

最后，可以用`proc print`过程检查一下读取是否正确。

## 参考文献

- 『SAS开发经典案例解析』

- 『SAS编程与数据挖掘商业案例』

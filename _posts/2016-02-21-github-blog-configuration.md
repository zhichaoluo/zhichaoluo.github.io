---
layout: post
title: "github博客配置备忘录"
author: "yphuang"
date: 2016-02-21
categories: blog
tags: [github,jekyll,Rmarkdown]

---

## 配置Rstudio+git+github环境

可以参考罗老师的个人博客：[Rstudio+GIT+Github配置](http://rokia.org/?p=315#more-315)


## Fork 模板

可以参考github上的项目：[beautiful-jekyll](https://github.com/daattali/beautiful-jekyll)

## 添加评论功能

* 注册[多说](http://duoshuo.com/)

* 修改相关源文件
    + `_config.yml`
    + `_includes\JB\comments`

* 创建duoshuo文件
    + `创建_includes\JB\comments-providers\duoshuo`

参考文献：

- [在Github Pages中集成多说评论插件](http://code4pub.github.io/tech/2014/05/04/integrate-with-duoshuo-comment/>)

## 添加`.md`文件的$$\LaTeX$$支持

在`_layouts/post.html`文件下，添加一行代码：


```
<!-- load MathJax -->
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>

```

参考文献：

- <http://stackoverflow.com/questions/10987992/using-mathjax-with-jekyll>

- [ a small tutorial about using MathJax with Jekyll](http://cwoebker.com/posts/latex-math-magic)



## 将`.Rmd`转成`.md`——配置`knitr-jekyll`环境

* fork github上的项目：[knitr-jekyll](https://github.com/yihui/knitr-jekyll)

* 将该项目与你自己的博客对应的github文件比对，将未涵盖的文件夹复制到你的博客对应的github文件目录下。

* 配置Jekyll环境

* 运行`servr::jekyll()`

你存放在`_source/`目录下的`.rmd`文件将被转换成`.md`文件，并保存在`_post/`目录下。

参考文献：

- 谢老大的博客：<http://yihui.name/knitr-jekyll/2014/09/jekyll-with-knitr.html>

- <http://jekyllrb.com/docs/windows/>

- [Run Jekyll on Windows](http://jekyll-windows.juthilo.com/)

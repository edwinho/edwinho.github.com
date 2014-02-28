---
layout: post
category : lessons
title: '使用jekyll上github上搭建blog'
tags : [jekyll, git]
---
{% include JB/setup %}

## 换Blog

之前一直用WordPress搭建的Blog，简单方便，功能齐全，用了一年半的时间。

直到租用的服务器到期，于是就想试下新的Blog——[Jekyll](https://github.com/mojombo/jekyll)，用Jekyll在[Github](https://github.com)上搭建Blog，放在Git托管网站上的建站方式，我使用Git已经有一段时间了，对我来说没有什么大问题，只不过写文章的方式不够KISS。

这种Pages服务大概是Github先弄出来的，但Github的速度很不稳定，有时要loading一分钟才出半个页面。
之前还由于刷票机事件，Github惨遭毒手。
不过国内还有个[GitCafe](https://gitcafe.com)提供了类似的服务。



### 好处

* 完全自定义网站所有代码.
* 免费.
* 版本管理.
* [Markdown](http://daringfireball.net/projects/markdown/syntax)，简洁精确的排版方式，且方便粘贴源码



### 大体原理

* Jekyll 可以根据Jekyll 源码（“模板”与“内容”）编译出静态网站。

  * “模板”是指一种文本文件，
    它里面可以使用[Liquid](http://www.liquidmarkup.org/)代码来控制填充数据。

    * 本质上模板不限于生成HTML，它可以生成任何格式的文本。
    * 通常不会经常更新。

  * “内容”也是指一种文本文件，它是使用Markdown语法来编写的文章。

    * 它里面包括文章内容与排版信息，但重点在于文章内容。
    * 通常会经常发布新的“内容”（如果作者不懒的话……）

  * 生成的静态网站可以托管到任意web服务器上。
    但如果使用GitCafe的Page服务的话，就只需要更新Jekyll源码的Git版本就行了，
    上传速度比较快。

* 用Git版本管理系统保存Jekyll源码。

  * “模板”与“内容”都会完整的记录修改历史.
  * 通常不需要记录静态网站生成结果。

* GitCafe提供Page服务，能在它的服务器上编译你的Git仓库中的Jekyll源码并托管静态网站。

  * 只要更新你的Git仓库，它就会自动生成。

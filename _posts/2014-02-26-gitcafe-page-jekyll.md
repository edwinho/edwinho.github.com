---
layout: post
title: "使用jekyll上github上搭建blog"
category : lessons
tags : [jekyll, git]
---

### 换Blog

之前一直用WordPress搭建的Blog，简单方便，功能齐全，用了一年半的时间。

直到租用的服务器到期，于是就想试下新的Blog——[Jekyll](https://github.com/mojombo/jekyll)，用Jekyll在[Github](https://github.com)上搭建Blog，放在Git托管网站上的建站方式，我使用Git已经有一段时间了，对我来说没有什么大问题，只不过写文章的方式不够KISS。

<!-- more -->

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

  * 生成的静态网站可以托管到任意web服务器上。
    但如果使用GitCafe的Page服务的话，就只需要更新Jekyll源码的Git版本就行了，
    上传速度比较快。

* 用Git版本管理系统保存Jekyll源码。

  * “模板”与“内容”都会完整的记录修改历史.
  * 通常不需要记录静态网站生成结果。

* Github和GitCafe都提供Page服务，能在它的服务器上编译你的Git仓库中的Jekyll源码并托管静态网站。

  * 只要更新你的Git仓库，它就会自动生成。



### 预备能力要求

* 使用[Git](http://git-scm.com)的能力
* 开发Web页面的能力
* 使用文本编辑器的能力
* 搜索与阅读教程的能力


### 建站步骤

1. 安装好git与[gem](http://rubygems.org)。
2. [安装Jekyll](http://wiki.github.com/mojombo/jekyll/install)。
3. 使用[JekyllBootstrap](http://jekyllbootstrap.com)的模板和主题。
4. 编写你的“模板”以及“内容”。
5. 使用`jekyll --server`来调试与预览你的站点。
6. 将你的Jekyll源码目录初始化成本地Git仓库。
  * 注意使用`.gitignore`来忽略掉`_site`目录
  * 选用Gitcafe的要注意切换到`gitcafe-pages`分支再提交第一个版本。而Github的可以直接用master.
7. Push到远程仓库，搞定。


- **github**<br/>
现在你打开 username.github.com 就可以看到刚才新建的页面了，就是这么简单。当然也可以为你的Blog仓库绑定独立域名，具体做法就是：

在你的仓库中新建内容为 www.youdomain.com 的 CNAME 文件；
在你的域名管理页或者是DNS解析的地方，增加一个记录，记录类别为CNAME(Alias)类型.

注意：如果你在CNAME中填写的是顶级域名，就得设置DNS的记录类别为A(Host)型，并设置主机为207.97.227.245(用`dig username.github.io`查看你的IP地址)。详细介绍请移步Github的Pages页面。

- **gitcafe**<br/>
进入你的Page Repo的项目管理界面：
![gitcafe-01](http://edwinho.github.io/images/lessons/gitcafe-01.png)

你将会看到左侧的导航栏中有自定义域名设置，点击进入，输入你想要设置的域名：
![gitcafe-02](http://edwinho.github.io/images/lessons/gitcafe-02.png)

需要注意的是，请正确填写你期望自定义的域名，填写时请不要填写http://这样的协议，当然如果你填写了，gitcafe会智能的擦除这些字符。

最后在你的域名管理界面添加一个A记录，将它指向GitCafe服务器的IP：
![banding-domain](http://edwinho.github.io/images/lessons/banding-domain.png)

大功告成，现在你就可以通过自己的域名来访问GitCafe为你渲染好的Pages了！<br/>

接下来我们只需要按照自己的喜好设计页面。首先认识下Jekyll的文件及目录配置:

    .
    |-- _config.yml
    |-- _includes
    |-- _layouts
    |   |-- default.html
    |   |-- post.html
    |-- _posts
    |   |-- 2011-10-25-open-source-is-good.markdown
    |   |-- 2011-04-26-hello-world.markdown
    |-- _site
    |-- index.html
    |-- assets
        |-- css
            |-- style.css
        |-- javascripts



### 评论功能

老外比较喜欢用[Disqus](https://disqus.com)。
国内也有类似的东西，比如[友言](http://www.uyan.cc/)。


### 各种语法参考

* [YAML](https://github.com/mojombo/jekyll/wiki/yaml-front-matter)
* [Markdown语法中译](http://markdown.tw)与[献给写作者的 Markdown 新手指南](http://jianshu.io/p/q81RER)
* Liquid与[Jekyll扩展过的Liquid](http://wiki.github.com/mojombo/jekyll/liquid-extensions)


### 参考及相关资料

- [理想的写作环境：Git+Github+Markdown+Jekyll](http://www.yangzhiping.com/tech/writing-space.html)
- [使用Jekyll在Github上搭建博客](http://hzmook.github.io/2012/07/01/use-jekyll-build-blog-on-github.html)


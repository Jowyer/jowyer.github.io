---
layout: post
title:  通过GitHub Pages和jekyll在GitHub上建立免费博客
date:   2014-10-24 14:12:10
categories: personal
---
终于设置好可以发博客了！

第一篇就来说说怎么通过[GitHub Pages][pagesUrl]和[jekyll][jekyllrb]([jekyll的中文网页][jekyllcn])，在GitHub上发文章吧。

> GitHub Pages are public webpages freely hosted and easily published through our site.

> Jekyll is a simple, blog-aware, static site generator perfect for personal, project, or organization sites. 

## 搭建GitHub博客项目 

首先根据[GitHub Pages][pagesUrl]的指引，建立好项目：


> https://github.com/**username**/**username**.github.io

其中 `username` 必须是自己在GitHub上的用户名。再把项目clone到本地目录：

> /Users/jowyer/Code/jowyer.github.io/

## 重用jekyll默认工程
为了快速开始，可以使用jekyll默认的模板来进行修改。

{% highlight bash %}
$ cd /Users/jowyer/Code/
$ jekyll new blog
$ cd blog
$ cp -R ./. ../jowyer.github.io/
{% endhighlight %}

将jekyll模板工程中的所有文件，包括`.gitignore`都拷贝至GitHub博客项目的本地目录中去。

最后将本地的博客项目 **Commit & Sync** 到GitHub去就搞定了！

快去你的 *http://**username**.github.io/* 看看吧！

***

## 了解目录结构
至此，构成博客网页的所有内容都组织在你的GitHub工程中了，网页内容，结构，样式都可以随意调整。

进一步关于jekyll的文件结构及其用法请移步 [jekyll官方教程][jekyllStructure] 。

## jekyll调试
jekyll提供了很方便的本地调试方法，只需要在工程目录下输入：

{% highlight bash %}
$ jekyll serve
{% endhighlight %}

就会在本地启动一个开发服务器，这时在浏览器中打开`http://localhost:4000/`就可以进行本地调试了。关闭本地服务器：Ctrl+c。

## jekyll安装
如果你和我之前一样，电脑上没有安装jekyll，那我们还有一段路要走。

> Requirements: 
> 
> * Ruby (including development headers)
> * RubyGems
> * Linux, Unix, or Mac OS X
> * NodeJS, or another JavaScript runtime (for CoffeeScript support).

我在mac上的**安装顺序**是：

1. 安装[Homebrew](http://brew.sh/)
* {% highlight bash %}
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
{% endhighlight %}

2. 安装[RVM](https://rvm.io/)
* {% highlight bash %}
$ \curl -sSL https://get.rvm.io | bash -s stable
{% endhighlight %}

3. 用RVM安装Ruby
* {% highlight bash %}
$ rvm install ruby-2.1.3
{% endhighlight %}

4. 用RubyGems安装jekyll
* {% highlight bash %}
$ gem install jekyll
{% endhighlight %}

当然，你也可以遵循[jekyll官方安装指南](http://jekyllrb.com/docs/installation/)。

## Reference

[kramdown Syntax][kramdown syntax url]

[pagesUrl]:  https://pages.github.com/
[jekyllrb]: 	http://jekyllrb.com/
[jekyllcn]: http://jekyllcn.com/
[jekyllStructure]: http://jekyllrb.com/docs/structure/
[kramdown syntax url]: http://kramdown.gettalong.org/syntax.html
---
title: 2.初识HEXO-页面配置
comments: true
date: 2017-06-17 21:04:09
tags: HEXO
categories:
- WEB
- HEXO
keywords: hexo
---

上一篇简单写了一下hexo发表博客的一些小技巧。这一篇就来说一下hexo的基本配置吧。

<!-- more -->

## about页面的配置

自己安装完主题后发现，about页面无论如何都打不开。搜了又搜，最终发现还得另行配置。在这里简要说一下吧。
首先hexo先新建一个about页面。`hexo new page about`
和新建文章差不多，但路径不太一样，它在根目录的source文件夹下新建了一个名为about的文件夹。我们只需修改里面的index.md即可。在里面添加上我们的个人简介。

## tags页面的配置

这个和上面步骤差不多。
首先`hexo new page tags`
然后找到文件打开，此时我们只需在配置栏加一条`type: "tags"`即可

## tags页面的修改

此时，我们已经配置完成了。但是打开浏览器查看，发现效果不是太如人意，此时该怎么做呢？

![](https://img.5am3.com/5am3/img/20191213230022.png)

身为程序猿的我们，当然不能就此妥协，要改！
在这里说一下我改的方法吧，打开主题的layout文件夹，进入_partial文件夹内，修改其中的page.ejs。

![](https://img.5am3.com/5am3/img/20191213230032.png)

不要问我怎么知道的，没工夫告诉你。
可累了，挨个打开看。
一把辛酸泪。看了三遍才找到。

至于怎么改，我这里就不详细说了，毕竟我也是瞎改。

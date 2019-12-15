---
title: 1.初识HEXO-文档排版
comments: true
date: 2017-06-17 20:04:09
tags: 
- HEXO
categories:
- WEB
- HEXO
---
最近水群，闲逛看到好多大神使用这个框架搭个人博客，效果还蛮棒。于是自己也开始尝试。
尝试途中，也算是经历了一些困难。下面写一下吧。

<!--more-->

## 感受
HEXO是一个很强大的静态博客框架，而且可以直接基于GitHub搭建博客。
真心蛮棒的。而且学习起来也不是太困难，稍微有点前端知识就能上手使用。
博客主要是用markdown书写，还是比较方便的。
下面给大家介绍一些小技巧，自己感觉蛮有意思的。

## 有意思的东西

下面写一下在写hexo博客时的一些小技巧

### 如何实现点击查看更多

在摘要写完后的下一行，写下这段代码即可。

`<!--more-->`

### 如何插入代码
插入代码有两种方式。([]内的为参数，书写时请删掉[]。)
- **markdown语法**

`` `代码` `` 
*效果是这样*

````HTML
```html
{% codeblock 丶诺熙 http://blog.5am3.com/2017/05/23/初识HEXO/ 初始HEXO lang:html %}
<p>代码块</p>
<div class="a">一个div</div>
{% endcodeblock %}
```
````
*效果是这样,html为高亮语法、可以自己随意填*

- **swig语法**

*在这里就不做详细解释了。这个例子大家可以看一下。*
{% codeblock 丶诺熙 http://blog.5am3.com/2017/05/23/初识HEXO/ 初识HEXO lang:html %}
{% raw %}{% codeblock 丶诺熙 http://blog.5am3.com/2017/05/23/初识HEXO/ 初识HEXO lang:html %}
<p>代码块</p>
<div class="a">一个div</div>
{% endcodeblock %}{% endraw %}{% endcodeblock %}

### 如何插入引用
- markdown语法

> \>这是一条引用
> \>这是第二条引用

- swig语法

{% blockquote @ http://blog.5am3.com/2017/05/23/初识HEXO/ 丶诺熙 %}
{% raw %}
{% blockquote @ http:/ /blog.5am3.com/2017/05/23/初识HEXO/ 丶诺熙 %}
Every interaction is both precious and an opportunity to delight.
{% endblockquote %}
{% endraw %}
{% endblockquote %}

自我感觉就是md比较简单粗暴，而swig则功能比较强大，比如引用可以标注引用来源。还可以加超链接。满棒的。

- 以上均参考了[Adam的博客](https://segmentfault.com/a/1190000009478129)，如果有想详细了解的小伙伴可以点进去。 

## 自己遇到的一些小坑
### 如何摆平代码
比如刚刚的那几个代码例子，研究好久才写出来。
因为如果直接写会被hexo编译成代码，而不是原来的样子。
但是写成以下就可以绕过转义。
- md语法

![](https://img.5am3.com/5am3/img/20191213225947.png)

也就是说，如果用md语法。写代码时，你的反引号比他多一个，你就赢了。这就是转义了。

但是写引用的时候要注意了。引用的转义是反斜杠。

![](https://img.5am3.com/5am3/img/20191213230008.png)

- swig语法

![](https://img.5am3.com/5am3/img/20191213225955.png)

如果是swig语法，可以加一个框，把他框起来，表明是源代码就好了。
但是自己用的时候感觉bug不少。但没办法了。



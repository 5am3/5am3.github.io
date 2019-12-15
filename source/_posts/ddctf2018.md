---
title: 2018DDCTF  writeup
comments: true
date: 2018-04-24 15:38:40
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


第二次参加滴滴的比赛，第一次是刚接触CTF不久。身为一个web狗，做出两道逆向题。

这是第二次，总的来说，题目对新人还是蛮友好的，而且还能学到很多东西。

时间方面，也还不错。体验到了肛题的快感。这次...蛮可惜的。差一点就ak了web题。

还好时间不够了。否则后面那道java一定会折磨我许久。


最后，赞一下各位出题师傅，题目很喜欢！

还有安姐姐也辛苦了。这一周，看到安姐姐基本上天天通宵。

<!--more -->

比赛复现链接： http://ddctf.didichuxing.com

## WEB

### web1 数据库的秘密 (100pt)

> **[注意] 本次DDCTF所有WEB题无需使用也禁止使用扫描器**
>
> <http://116.85.43.88:8080/JYDJAYLYIPHCJMOQ/dfe3ia/index.php>



打开后会发现返回如下。

```
非法链接，只允许来自 123.232.23.245 的访问
```

此时可以通过修改HTTP请求头中的X-Forwarded-For即可。即添加以下字段

```
X-Forwarded-For:123.232.23.245
```

在这里，我用的是火狐的一个插件**Modify Header Value (HTTP Headers)**。

![mark](https://img.5am3.com/img/180414/CLmbAc4f7j.png)

发现该网页是一个简单的查询列表。再加上题目中给的hint。可以判断为SQL注入题目。

经过测试，发现以上三个点均不是注入点。此时分析数据包，可以发现存在第四个注入点。

![mark](https://img.5am3.com/img/180414/dGfL1C7DDJ.png)

然后查看源码，发现一个隐藏字段。经过测试发现，该字段可以注入。

```
admin' && '1'='1'#
admin' && '1'='2'#
```

尝试注入 author，可以发现以下内容信息

```
1. and （可以用&&代替）
2. union select （很迷，这两个不能同时出现，然而自己又找不到其他方式）
3. 仅允许#号注释
```



然后注入渣的自己就比较无奈了。。不会啊。只好祭出盲注大法了。经过尝试，最终构造以下payload可用。

```sql
admin' && binary substr((select group_concat(SCHEMA_NAME) from information_schema.SCHEMATA),1,1) <'z' # 
```


然后开始写脚本，此时遇到了一个问题。发现他有一个验证。为了check你中途是否修改数据，而加入的一个hash比对。

首先将你的准备传送的内容进行某种hash后变为sig字段，然后再将sig通过get请求一起发送过去。此时服务器端会将sig与你发送的内容的hash比对一下。此时可以减少抓包中途修改内容的可能性。

所以，为了省事，我选择直接将这个代码调用一下。

用python的execjs库，可以直接执行js代码。

最终跑起脚本，获取到flag `DDCTF{IKIDLHNZMKFUDEQE}`

![mark](https://img.5am3.com/img/180415/AbdF2ADGA3.png)

### web2 专属链接 (200pt)

> 现在，你拿到了滴滴平台为你同学生成的专属登录链接，但是你能进一步拿到专属他的秘密flag么
>
> 提示1：虽然原网站跟本次CTF没有关系，原网站是www.xiaojukeji.com
>
> 注：题目采用springmvc+mybatis编写，**链接至其他域名的链接与本次CTF无关，请不要攻击**
>
> <http://116.85.48.102:5050/welcom/3fca5965sd7b7s4a71s88c7se658165a791e>



首先打开网站，发现是滴滴的官网。。

此时发现所有连接几乎全部重定向到了滴滴官网。

无奈下查看元素。发现hint

```html
<!--/flag/testflag/yourflag-->
```

尝试访问，`http://116.85.48.102:5050/flag/testflag/yourflag`发现报错500，好像是数组越界？

此时尝试将yourflag替换为DDCTF{1321}，返回failed!!!。

猜测爆破flag么？完全没戏啊。看样子应该有其他地方可以入手。

然而又发现了主页js的一句神奇的话。一个ajax语句。

![mark](https://img.5am3.com/img/180415/aj9dA0CmKl.png)

然并卵，404。。。。。

此时只好继续分析题目，发现了令人眼前一亮的东西。对，就是下面这个icon。

![mark](https://img.5am3.com/img/180415/m1i9IHi9H4.png)

```
http://116.85.48.102:5050/image/banner/ZmF2aWNvbi5pY28=
```

访问后，发现下载了favicon.ico

此时发现图标好像图片很奇怪。后来果然验证了这是个hint。

![mark](https://img.5am3.com/img/180415/5akEdiAE93.png)

此时可以愉快地玩耍了，这样一来，题目源码有了，还愁拿不下来么。

美滋滋。此时也知道了题目中hint的用意。`题目采用springmvc+mybatis编写`

百度搜索springmvc+mybatis文件结构，美滋滋读文件。

首先，大概知道了资源文件都是在WEB-INF文件夹下，所以猜测这个icon也在这里，此时我们要先确定文件夹。

WEB-INF下有一个web.xml，此时尝试读取，最终确定目录`../../WEB-INF/web.xml`。

然后拖文件。这里说几点注意事项。

> 1. 通过../../WEB-INF/web.xml确认位置。
> 2. 继续根据web.xml中的内容进行文件读取。classpath是`WEB-INF/classes`
> 3. 读class文件时根据包名判断文件目录`com.didichuxing.ctf.listener.InitListener` 即为`WEB-INF/com/didichuxing/ctf/listener/InitListener.class`
> 4. 制造网站报错，进一步找到更多的文件



差不多，注意一上四点，就可以拿到尽量多的源码了。

拖到源码后，就不美滋滋了。。。还好去年在DDCTF学过2017第二题的安卓逆向，会逆向了。

（此时坑点：jd-jui仅可逆jar，需要将class打成压缩包改为jar再逆向）

此时开始苦逼的分析源码。

**分析后发现，存在接口，用当前用户的邮箱去生成一个flag。**

但是flag是加密的。此时加密流程代码里都有，是一个RSA加密。密钥在服务器中的

![mark](https://img.5am3.com/img/180415/5FK9H98cBm.png)

此时又一次明白了，为什么读文件允许ks文件。



来吧，首先先拿邮箱申请一个flag

然而此时申请flag，邮箱也得先加密。自己提取出来的加密脚本如下。

```java
public static String byte2hex(byte[] b)
  {
    StringBuilder hs = new StringBuilder();
    for (int n = 0; (b != null) && (n < b.length); n++)
    {
      String stmp = Integer.toHexString(b[n] & 0xFF);
      if (stmp.length() == 1) {
        hs.append('0');
      }
      hs.append(stmp);
    }
    return hs.toString().toUpperCase();
  }
  public static void getEmail() throws NoSuchAlgorithmException, InvalidKeyException{
	  SecretKeySpec signingKey = new SecretKeySpec("sdl welcome you !".getBytes(), "HmacSHA256");
	  Mac mac = Mac.getInstance("HmacSHA256");
      mac.init(signingKey);
	  String email="3113936212117314317@didichuxing.com";
	  byte[] e = mac.doFinal(String.valueOf(email.trim()).getBytes());
	  System.out.println(byte2hex(e));
  }

//0DFEE0968F44107479B6CF5784641060DB42952C197C7E8560C2B5F58925FAF4
```

坑：但是此时后端仅允许post方式。且参数是以get传递的。

成功获取到flag

```
Encrypted flag : 506920534F89FA62C1125AABE3462F49073AB9F5C2254895534600A9242B8F18D4E420419534118D8CF9C20D07825C4797AF1A169CA83F934EF508F617C300B04242BEEA14AA4BB0F4887494703F6F50E1873708A0FE4C87AC99153DD02EEF7F9906DE120F5895DA7AD134745E032F15D253F1E4DDD6E4BC67CD0CD2314BA32660AB873B3FF067D1F3FF219C21A8B5A67246D9AE5E9437DBDD4E7FAACBA748F58FC059F662D2554AB6377D581F03E4C85BBD8D67AC6626065E2C950B9E7FBE2AEA3071DC0904455375C66A2A3F8FF4691D0C4D76347083A1E596265080FEB30816C522C6BFEA41262240A71CDBA4C02DB4AFD46C7380E2A19B08231397D099FE
```

然后，解密吧。。

只能百度了，java又不熟，RSA更不熟，尤其还是这种hex的。逆源码都失败了。一个劲报错。（查百度，好像是因为啥空格之类的。打不过打不过）



最终发现一个好玩的，可以从keystore提取RSA私钥。这样一来，又继续美滋滋。

https://blog.csdn.net/zbuger/article/details/51690900



然后照猫画虎，提出私钥。此时祭出自己的一个无敌大件。之前从某次CTF安卓题提出的RSA解密脚本。（当时题目简单，加解密都给了，改个函数名就ok了。）

(╯°□°）╯︵ ┻━┻

要不是在线的解不了。才不会想起这个大招（已放到附件，记得将 密文to ascii 再 to base64。）。。。。。

通过在线工具，提取出公私钥，然后跑脚本。最终拿到flag。

DDCTF{1797193649441981961}



### web3 注入的奥妙 (250pt)

> 本题flag不需要包含`DDCTF{}`，为`[0-9a-f]+`
>
> <http://116.85.48.105:5033/4eaee5db-2304-4d6d-aa9c-962051d99a41/well/getmessage/1>



按照题目要求，这题应该是个注入题，毫无疑问。

查看源码，发现给了big5的编码表，此时猜测可以通过宽字节进行注入。

```
1餐' and 1=1%23
```

orderby，发现有三个字段，尝试构造联合查询语句，发现union会被直接删除。此时**双写**绕过即可。

此时查询数据库：

```
1餐' uniunionon select SCHEMA_NAME,2,3 from information_schema.SCHEMATA %23
```

![mark](https://img.5am3.com/img/180416/F3F2KEjgKb.png)

然后继续查询表名：

```
1餐' uniunionon select TABLE_NAME,2,3 from information_schema.tables where table_schema=sqli %23
```

此时发生了一件尴尬的事情。我们无法继续构造单双引号，这样数据库会报以下错误。

![mark](https://img.5am3.com/img/180416/f345fDj20l.png)

此时祭出hex大法。数据库会直接将0x开头的进行转码解析。

```
1餐' uniunionon select TABLE_NAME,2,3 from information_schema.tables where table_schema=0x73716c69 %23
```

此时成功的爆出来了三个表

```
message,route_rules,users
```

然后就没啥好说的了。挨个查着玩就可以了，基本同上。然后查字段啥的。

*查路由的时候，有点小坑，不知道后端怎么解析的，会将一列数据解析到多列，此时用mysql的to_base64()函数即可。*



通过路由信息，我们可以发现存在`static/bootstrap/css/backup.css `源码泄露。

通过以下三行脚本即可保存该文件。

```
import requests
f=open('a.zip','wb')
f.write(requests.get('http://116.85.48.105:5033/static/bootstrap/css/backup.css').content)
```



接下来就是对PHP代码的审计。

首先，分析路由。我们从数据表内知道了有以下几条规则

> get*/:u/well/getmessage/:s       Well#getmessage   
> get*/:u/justtry/self/:s       JustTry#self     
> post*/:u/justtry/try          JustTry#try       



首先第一条，就是咱刚刚实现注入的那一个。不用多看，逻辑差不多清楚。

第二，三条，调用的都是justtry类下的某个方法。所以可以跟进去，重点分析下这个函数。

![mark](https://img.5am3.com/img/180416/jmeKeiBC89.png)

此时看见了 unserialize ，倍感亲切，这不就是反序列化么。

此时就需要考虑反序列化了。他后面限制了几个类，此时我们可以一一打开分析。

test类，顾名思义，就是一个测试用的。

![mark](https://img.5am3.com/img/180416/aeBdKbceba.png)

此时我们发现他的析构函数中，有一条特殊的句子。跟进去之后发现，他会将falg打印出来。

仔细分析源码后发现，这个test类通过调用Flag类来获取flag，然而Flag类又需要调用SQL类来进行数据库查询。



所以，这个反序列化是个相当大的工程。自己手写是无望了。

首先尝试了一下，自己写三个类的调用。。。然而失败了。

最后复现源码，并在try方法打印序列化对象后。（uuid是你的url那串，uuid类下正则可以看出来。）

![mark](https://img.5am3.com/img/180416/H3dE1bEeC8.png)

发现，他是有一个**命名空间的要求**。序列化后语句如下

```
O:17:"Index\Helper\Test":2:{s:9:"user_uuid";s:36:"4eaee5db-2304-4d6d-aa9c-962051d99a41";s:2:"fl";O:17:"Index\Helper\Flag":1:{s:3:"sql";O:16:"Index\Helper\SQL":2:{s:3:"dbc";N;s:3:"pdo";N;}}}
```



最终的Payload如下：

> url:http://116.85.48.105:5033/4eaee5db-2304-4d6d-aa9c-962051d99a41/justtry/try/
>
> postdata:
>
> serialize=%4f%3a%31%37%3a%22%49%6e%64%65%78%5c%48%65%6c%70%65%72%5c%54%65%73%74%22%3a%32%3a%7b%73%3a%39%3a%22%75%73%65%72%5f%75%75%69%64%22%3b%73%3a%33%36%3a%22%34%65%61%65%65%35%64%62%2d%32%33%30%34%2d%34%64%36%64%2d%61%61%39%63%2d%39%36%32%30%35%31%64%39%39%61%34%31%22%3b%73%3a%32%3a%22%66%6c%22%3b%4f%3a%31%37%3a%22%49%6e%64%65%78%5c%48%65%6c%70%65%72%5c%46%6c%61%67%22%3a%31%3a%7b%73%3a%33%3a%22%73%71%6c%22%3b%4f%3a%31%36%3a%22%49%6e%64%65%78%5c%48%65%6c%70%65%72%5c%53%51%4c%22%3a%32%3a%7b%73%3a%33%3a%22%64%62%63%22%3b%4e%3b%73%3a%33%3a%22%70%64%6f%22%3b%4e%3b%7d%7d%7d



### web4 mini blockchain  (280pt)

> 某银行利用区块链技术，发明了DiDiCoins记账系统。某宝石商店采用了这一方式来完成钻石的销售与清算过程。不幸的是，该银行被黑客入侵，私钥被窃取，维持区块链正常运转的矿机也全部宕机。现在，你能追回所有DDCoins，并且从商店购买2颗钻石么？
>
> `注意事项：区块链是存在cookie里的，可能会因为区块链太长，浏览器不接受服务器返回的set-cookie字段而导致区块链无法更新，因此强烈推荐写脚本发请求`
>
> 题目入口：
> <http://116.85.48.107:5000/b942f830cf97e>



拿到题目，内心是拒绝的。因为虽然说区块链这么火，但是自己还是没怎么了解过。

第一反应是。药丸，没戏了。但是，搞信息安全的孩子怎么可以轻言放弃呢！

时间辣么长，还不信看不明白个区块链。最后肛了两天多，才大概明白了题目

首先，题目给了源码，这个很棒棒。



建议大家分析题目时将代码也多读几遍，然后再结合参考资料进行理解。

在这里不做太多的理解源码的讲解。



最初我是将重心代码的一些逻辑上，以及加密是否可逆。（发现自己太年轻，看不懂）

然后慢慢的开始了解区块链，最后发现这种手段。



这道题目中，利用了区块链一个很神奇的东西。

因为区块链是一个链表，而且还是一个谁都可以增加的，此时，人们达成了一种默认，以最长的那条链为主链（正版），其他的分支都是盗版。



如下图，就是此时该题目的区块链。

![mark](https://img.5am3.com/img/180420/fbLcGAFJ4a.png)

那么我们可以再构造一条链，只要比主链长，那这条链就是我们说了算。

![mark](https://img.5am3.com/img/180420/EL389D98Lb.png)

此时虽然说区块链1是正规的链，但是区块链2要比1长，此时区块链2即为正规链。



但是，说的轻巧，我们该如何构造呢？

首先，我们分析路由可以发现，题目预留了一个创建交易的接口。此时可以生成新块。

![mark](https://img.5am3.com/img/180420/CkLdA2d7gD.png)

只要我们可以挖到一个DDcoin，就可以创建一次新块，然后会判断商店的余额。最终给予砖石奖励。

然而DDcoin是什么呢。

在这道题里，其实就是这个东西，这就是一个区块。对他进行分析一下。

![mark](https://img.5am3.com/img/180420/bAGKCC8KEk.png)

```
nonce:自定义字符串
prev：上一个区块的地址
hash：这个区块的hash
height：当前处于第几个节点
transactions：交易信息
```

再分析transactions

```
input与signature好像是一个凭证，验证这个区块主人身份。
output，收款人信息
amount，收款数额
addr，收款地址
```

hash这里的话，不是太明白。

但是看代码。发现都有现成的可以生成。只要利用这三个函数，即可创建一个新的区块。

```
create_output_utxo(addr_to, amount) // 新建一个output信息
create_tx(input_utxo_ids, output_utxo, privkey_from=None) // 新建一个transactions信息
create_block(prev_block_hash, nonce_str, transactions) // 新建一个区块
```

首先新建output，此时参数很简单，收货人地址（商店），数量（全款）

然后创建tx，此时output_utxo就是刚刚咱创建好的那个。然而问题来了，私钥和id咱是没有的。此时分析代码可以发现，这一步做的主要就是创建一个sig签名。还有就是生成一个hash

![mark](https://img.5am3.com/img/180420/c1c4eeFhhi.png)



此时，邪恶的想到，既然是要创建第二条链，那么可不可以借用一下第一条链的第一块的信息。

也就是直接忽略掉sig的生成，伪造tx，直接重写一下create_tx

![mark](https://img.5am3.com/img/180420/3iC0KHeBd4.png)

然后此时tx也有了，进行下一步create_block

此时他的三个参数也好写，上一个区块的hash，自定义字符串，刚刚做好的tx



此时，我们要通过爆破nonce的方式，来使create_block生成的块的hash为00000开头，

这样，我们才能添加。

然后向那个添加块的地址post由create_block即可成功添加第一个块。

记得改请求头中的content-type为json。还有就是cookie自己手动更新



第二个块的时候，问题又来了。

这条链中，我们之前的tx已经使用过一次，无法使用了。怎么办？

此时可以注意到题目中init中给的hint。

![mark](https://img.5am3.com/img/180420/E8I6mh7ie9.png)

凭啥他可以不写tx就生成块！不开心，你都能那样，我也要！

于是。。。。。通过这个方式，在后面添加几个空区块就好。

成功伪造主链！获取一颗砖石。

再次重复以上做法，完成第三条链即可获取到flag



切记，手动更新cookie......



### web5 我的博客 (320pt) 

> 提示：`www.tar.gz`
>
> <http://116.85.39.110:5032/a8e794800ac5c088a73b6b9b38b38c8d>

题目又给了源码，美滋滋。

然而下载到源码后就不美滋滋了。

一共给了三个页面，主页很明显，有一个SQL注入漏洞。这个题之前安恒杯三月见过。利用率printf函数的一个小漏洞，%1$'可以造成单引号逃逸。

然而，你是进不去主页的。因为。。

![mark](https://img.5am3.com/img/180420/e5eCF99g1b.png)

还没进去，就被die了。

然后只好分析如何能成为admin了。此时看到了。

![mark](https://img.5am3.com/img/180420/4eiEJejmeJ.png)

当你是通过邀请码注册的，你便可以成为admin。

然而，邀请码是完全随机的。

此时，想起LCTF的一道题，感觉完全一样有木有！

https://github.com/LCTF/LCTF2017/tree/master/src/web/%E8%90%8C%E8%90%8C%E5%93%92%E7%9A%84%E6%8A%A5%E5%90%8D%E7%B3%BB%E7%BB%9F

然而当时有两个解，一个非预期条件竞争，另一个正则的漏洞。

此时这题完全没用啊！当时要疯了，猜测，难道是要预测随机数？

![mark](https://img.5am3.com/img/180420/HlF2LhKjaE.png)

然而，当我看到大佬这句话的时候，萌生了放弃的想法，猜测肯定还有其他解法。

奈何，看啊看，看啊看，我瞪电脑，电脑瞪我。

最后还是决定看一下随机数这里。很开心，找到了这篇文章。

http://drops.xmd5.com/static/drops/web-11861.html

然而，每个卵用，他只告诉了我：对！毛病就在随机数，但是你会么？

满满的都是嘲讽....



来吧，一起看，首先这篇文章讲了一种后门的隐藏方式，话说我读了好几遍才理解。

然后不得不感叹，作者....你还是人么。这都能想出来。服！真的服！



首先，大家需要先知道rand()是不安全的随机数。（然而我不知道）

然后str_shuffle()是调用rand()实现的随机。所以此时重点是。如何预测rand？

然而作者没告诉，给的链接都是数学，看不懂.....

此时PHITHON大佬的这篇文章真的是解救了自己。

https://www.leavesongs.com/penetration/safeboxs-secret.html

![mark](https://img.5am3.com/img/180420/4EiBcAcJ4F.png)

所以，此时我们知道了一件事情。当我们可以获取到连续的33个随机数后，我们就可以预测后面连续的所有随机数。

如何连续？大佬文章中说了，通过http请求头中的Connection:Keep-Alive。

此时，我们先获取他100个随机数。

```python
s = requests.Session()
url='http://116.85.39.110:5032/a8e794800ac5c088a73b6b9b38b38c8d/register.php'
headers={'Connection': 'Keep-Alive'}
state=[]
for i in range(50):
		r=s.get(url,headers=headers)
		state.append(int(re.search(r'id="csrf" value="(.+?)" required>', r.text, re.M|re.I).group(1)))
```

然后测试一下

```
yuce_list=[]
for i in range(10):
	yuceTemp=yuce(len(state))
	state.append(yuceTemp)
	yuce_list.append(yuceTemp)
```

此时发现和实际是有一些冲突的。分析后发现，应该将生成的随机数取余2147483647才是真正的数。

但此时又有了一个问题。

![mark](https://img.5am3.com/img/180420/IFf3H6L8I2.png)

之前大佬是说过会有一定的误差，但是误差率太高了。虽然误差不大，但是....

此时，没办法，只能祈求后面会处理误差。此时我们完成了随机数的预测。

接下来需要写如何打乱字符串。

可以发现，一个很简单的流程，生成随机数，然后交换位置。

![mark](https://img.5am3.com/img/180420/JmcdCf9Lk8.png)

唯一不知道的地方就是其中这个地方的一个函数。

此时直接去GitHub翻一下源码。

https://github.com/jinjiajin/php-5.6.9/blob/35e92f1f88b176d64f1d8fc983e466df383ee34e/ext/standard/php_rand.h

![mark](https://img.5am3.com/img/180420/d3Dd8A1eg8.png)

然后就是愉快的重写代码。

```
def rand_range(rand,minN,maxN,tmax=2147483647):
	temp1=tmax+1.0
	temp2=rand/temp1
	temp3=maxN-minN
	temp4=temp3+1.0
	temp5=temp4*temp2
	rand=minN+(int)(temp5)
	return rand
admin_old=['0','1','2','3','4','5','6','7','8','9','a','b','c','d','e','f','g','h','i','j','k','l','m','n','o','p','q','r','s','t','u','v','w','x','y','z','A','B','C','D','E','F','G','H','I','J','K','L','M','N','O','P','Q','R','S','T','U','V','W','X','Y','Z']

for i in range(len(admin_old))[::-1]:
	a=rand_range(int(yuce_list[len(admin_old)-i-1]),0,i)
	admin_old[i],admin_old[a]=admin_old[a],admin_old[i]
key=''
for i in admin_old:
	key+=i
print(key)
```

此时就可以愉快的生成随机数了。然后在进行一下注册。此时csrf记得提前在获取state时保存一下最后一位。

```
def getAdmin(username,passwd,code):
	data={
		"csrf":csrf,
		"username":username,
		"password":passwd,
		"code":code
	}
	r=s.post(url,headers=headers,data=data)
	print(r.text)
```



切记！code是：admin###开头，后面截取32位！

最后用拿到的账号进行登录即可。

后面就是sql注入了。很简单，只要单引号逃逸后，就可以显注了。没有其他过滤

```
/a8e794800ac5c088a73b6b9b38b38c8d/index.php?id=1&title=-1%1$'+union+select+1,f14g,3+from+a8e79480.key+where+1+%23 
```



![mark](https://img.5am3.com/img/180420/4kH7ml65cG.png)



## MISC

### misc1 签到题  (1pt)

> 请点击按钮下载附件

 

出题人是真的皮。下载后会发现一个神奇的东西。flag.txt里面的内容是这个

> 请查看赛题上方“公告”页

然后打开公告页，发现了他。。

> DDCTF{echo"W3Lc0me_2_DiD1${PAAMAYIM_NEKUDOTAYIM}C7f!"}

好歹咱也是个web手。so .....

本来还以为要解开里面的PHP代码。自己误以为是这个。

```
DDCTF{W3Lc0me_2_DiD1::C7f!"}
```

最后发现，原来是真·签到题。



### misc2 (╯°□°）╯︵ ┻━┻  (50pt) 

> (╯°□°）╯︵ ┻━┻
>
> d4e8e1f4a0f7e1f3a0e6e1f3f4a1a0d4e8e5a0e6ece1e7a0e9f3baa0c4c4c3d4c6fbb9b2b2e1e2b9b9b7b4e1b4b7e3e4b3b2b2e3e6b4b3e2b5b0b6b1b0e6e1e5e1b5fd



这道题蛮坑的。。想了无数种密码后都没思路。最后只能老老实实研究，或许是一些简单的编码？

一共134个字符。尝试2位一组，转化为十进制后，发现数值在一定范围内浮动。

![mark](https://img.5am3.com/img/180414/DJJgj7G8m9.png)

然后考虑到ascii码可见区域，于是尝试对其进行取余128的操作。

最后发现余数均在ascii码的可见区域。之后hex2ascii 即可获取到flag。

```
a=[212,232,225,244,160,247,225,243,160,230,225,243,244,161,160,212,232,229,160,230,236,225,231,160,233,243,186,160,196,196,195,212,198,251,185,178,178,225,226,185,185,183,180,225,180,183,227,228,179,178,178,227,230,180,179,226,181,176,182,177,176,230,225,229,225,181,253]
b=''
for i in a:
	b+=chr(i%128)
print(b)
```

![mark](https://img.5am3.com/img/180415/8EheaAIHLI.png)

```
DDCTF{922ab9974a47cd322cf43b50610faea5}
```



### misc3 第四扩展FS （100pt）

> D公司正在调查一起内部数据泄露事件，锁定嫌疑人小明，取证人员从小明手机中获取了一张图片引起了怀疑。这是一道送分题，提示已经在题目里，日常违规审计中频次有时候非常重要。



拿到图片，发现大小出奇的大，于是尝试binwalk，提出来一个压缩包。

![mark](https://img.5am3.com/img/180415/51k5a6LgEL.png)

尝试打开，发现是有密码的。（这里有个技巧，个人比较喜欢用windows的**好压**解压缩软件，这个软件存在一定的压缩包修复。）

然后回到题目，仔细分析。尝试无果后，最终将密码锁定在了`提示已经在题目里`，所以尝试查看文件属性，发现了一些奇怪的字符串。

![mark](https://img.5am3.com/img/180415/e7idiEm4kD.png)

一般来说，图片信息中不会出现备注的。所以尝试将其作为密码解压，解压成功。

然后发现了一串稀奇古怪的。。。。字符。

![mark](https://img.5am3.com/img/180415/IejD01mAdG.png)

此时想到了题目中给的hint：`日常违规审计中频次有时候非常重要`

尝试词频统计。得到flag

![mark](https://img.5am3.com/img/180415/c5Bbi3bc9i.png)

此时有一点小小坑。。D是两个。。

flag ：DDCTF{x1n9shaNgbIci}



### misc4 流量分析 (200pt)

> 提示一：若感觉在中间某个容易出错的步骤，若有需要检验是否正确时，可以比较MD5: 90c490781f9c320cd1ba671fcb112d1c
> 提示二：注意补齐私钥格式
> -----BEGIN RSA PRIVATE KEY-----
> XXXXXXX
> -----END RSA PRIVATE KEY-----



怎么说呢，做完这题，我才知道坑人能有多坑！

流量分析的题，首先可以发现他的大小很小。不像是那种大流量的分析。

尝试了一下学长之前推荐的一款工具《科来网络分析系统》

![mark](https://img.5am3.com/img/180420/a5gJ6kKHdE.png)

可以发现ftp传输了两个包。此时，fl-g极有可能是flag。

于是拿wireshark千辛万苦，提取出来压缩包。然而....没有密码。

只好继续分析了。因为毕竟misc4了，不可能是密码爆破啥的吧。

继续看， 发现一个邮件（不知道科来怎么提文件，查看数据。哭唧唧）

wireshark导出IMF对象。可以发现导出了几个邮件。然后逐个分析。

然而并没卵用，唯一有点用的，感觉奇怪的，就只有一个邮件。

![mark](https://img.5am3.com/img/180420/b5AFL2bKKH.png)

此时这个不是一点的奇怪！而是很奇怪！那么，这串密钥。。是干什么的呢。

经过老司机多年开车经验，呸。做题经验。

猜测！肯定有https流量。当然，科来也说有了。

![mark](https://img.5am3.com/img/180420/K6H9B5H4d8.png)

于是。。这种之前曾听说过的题目，现在到了手里还是有些小激动的。

尤其是那个图片！图片！图片！！！！

ocr也不行，手写也不行。那么多字。心塞ing。

好吧，最后还是百度找了个ocr识别了一下，然后改了几个字符。。

然后就是解密https流量。具体可以看这个链接。

https://blog.csdn.net/kelsel/article/details/52758192

直接导入私钥就可以。这里需要按照hint格式来，在前后加上标志位。

然后就可以解密https流量了。

然后搜索ssl，追踪http流量，最后取得flag



![mark](https://img.5am3.com/img/180420/8dmC106gg8.png)



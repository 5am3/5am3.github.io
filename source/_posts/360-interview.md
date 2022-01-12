---
title: 360面试经历及心得
comments: true
date: 2017-11-9 18:34:27
tags:
 - 面试
categories:
 - 其他
 - 面试
keywords:
 - 面试
---

## 面试心得

**首先感谢手抓饼学长的推荐，让自己有了这次机会。**

感觉经过这一次面试，学到了很多。更能准确的定位自己，知道以后要努力的方向。
也找到了一些自己的不足。发觉自己差的真的好多。
而且CTF这一块，虽然可以很好得入门，但也需要尽量和实战密切链接起来。否则还是不行的。
即便熟知原理，没有经过许多次的实践，甚至还不如脚本小子。
自己也该多写点工具，或者多学习一些大佬们的工具使用。

**2018的目标。写出一套自动化漏洞检测脚本**

<!-- more -->

---
## 面试过程
360那边小哥哥下午的时候给我发的短信，约面试时间。自己因为晚上有hctf，所以约在了6点半。快到6点半的时候，蛮紧张的。自己搬着小马扎去了阳台。本想着会安静点，没想到学校恰好在放广播。无奈之下只能在那了。话说小哥哥那边很准时，6点半准时打来了电话。

- **首先先了解一下我。问：看你简历写的是web渗透方向？**

> 恩，因为自己目前是主攻这一块的。二进制方向不太了解。如果按ctf水平来讲。二进制方向仅仅能做出来签到题。

-  **然后上来先问了一下实战**

关于实战，其实自己没有太多的经历。所以也就直接说了自己的看法。

> 一方面是胆小，技术不够，怕惹出事。
> 再有一方面就是，实战也不会找太大的目标。小目标的话，感觉渗透较简单。没必要去尝试。
> 只是挖过几个手机验证码绕过，xss什么的小洞。


- **xss通常是用来获取cookie。关于cookie，你了解多少。他有多少属性。**

> cookie就是一个用于和服务端校验身份的值。存在本地。
cookie的属性。有失效时间，还有一个记忆犹新的就是http-only。如果这个属性为真时，js无法获取到cookie的值，可以有效减少xss的危害。
但是自己之前看过一篇文章，换了种思路来破解他。通过构造钓鱼页面，来获取管理员的账号密码。
其他的属性，自己暂时想不起来。没太关注这些。。

- **cookie和session有什么区别？**
> 他俩一个是本地的一个是服务器端的。session在服务器端会存储好多信息。然后返回一个id作为cookie。做身份验证。


- **假设有这样一个情景，你通过xss可以拿到管理员cookie，你如何知道后台在哪里**
> 再加一个参数就好了。window.location.href获取到后台网址。
>  - **如果后台在内网里，你无法登录，那么你该怎么办。如何更大效率的利用这个xss**
>  呃，我暂时是没办法了。
>  不过可以尝试拿一下他网站的源码，然后尝试构造一些csrf。


- **又问了一下工具之类的，在爆破验证码时，用了什么工具。**

> 当时用的是burp，但现在用Python也可以写出利用脚本。


- **然后问了一下对渗透流程。例如如何收集信息。**

>谈到扫目录时，自己说用python手写扫描器。
通过requests库及threading库。然后判断响应码
小哥哥又问，有没有做什么优化之类的。
自己默默回答了个异常处理。他说这个不算。
然后自己就想不出来了。向小哥哥询问建议。

- **他说一会聊，然后又问怎么判断存在？如果遇到自定义404怎么办？**
> 自己没考虑这些，仅仅打ctf用。感觉还好。如果通过判断响应码不可以，可以先访问一下网站，然后自己加入规则。

- **小哥哥又问：如果让你写一个通用的怎么办？**
> 回答：这个我真没办法了，不过我有个馊主意，可以用一下，但效果可能不太好。直接通过正则匹配页面中的404，按404出现次数推断。毕竟自定义404里面肯定有这些信息。而且不止一个。


- **然后小哥哥开始下一个问题。问我对响应码了解多少。**
> 回答：302重定向，404未找到，500服务器错误。
> 
> - **然后小哥哥好像又问了个403吧。**
>
> 自己说，忘了，记得最清楚的就这仨。

现在想想，好傻哦。竟然连403都忘！
大家记住了哈。403 Forbidden

- **再往后，问了一下请求类型**
>  回答：get，post，option。这三个挺常见，都记得。还有几个不太常见。记不清了。
> - **然后又问：put知道么。还有head**
> 
> 回答：知道一点。
> - **问：head是干什么用的。**
> 
> 回答：好像是获取一个响应，但不收文件。
> - **问：恩，获取响应头。这不就回答你刚刚的问题了。**
> 
> 我：哇，好神奇哎。之前竟然没想到。（其实是没想过优化。毕竟写得玩，字典也不多。）

- **然后又问了问请求头都有什么**

> 自己回答：先是请求类型 get 还是post ，然后是路径 再往后是协议
> 然后下面是一堆头，例如accept-language，cookie，还有浏览器信息。
> - **问：host知道吧。**
>
> 回答：恩，刚刚没想起来。这里是域名，或者ip
> - **问：如果我改host，能不能发到别的网站上。**
> 
> 回答：我这里不太清楚。但感觉应该是不行的。因为之前用burp的repeater时候，想要发到其他网站，必须从右上角那个小按钮改一下地址和端口。

- **问我，你最擅长那一块**
自己回答没有，那一块都了解一点。说完这句话就后悔了。但是想想。好像自己还真的没什么擅长的。
样样会一点，样样都不精。
- **好吧，那我问问你文件包含吧。现在，有一个网站，存在文件包含漏洞。但是没有上传点。如果是你会怎么利用**
> 尝试一下是否能找到网站日志文件，尝试包含日志。

- **又问网络原理，和系统原理这边你怎么样**
> 我说，网络原理还将就吧，但系统原理真心不怎么样。没有学过

- **然后问tcp三次握手。**
> 回答：我对这块不太熟悉，当初记住，也是因为那个故事，然后自己把打仗那个故事说了一下
> **又问：每一次的标志位是什么。**
> 回答：这个真的不知道了。


- **最后问了一下，还有什么要问我的吗？**

说实话，自己还真有，刚刚聊天时，发现自己好多不足.
想再挖一下，懒得回去百度了。
然后。。。。紧张的忘了。。
只好说没有了。
挂了电话后心直接凉了，一首凉凉送给自己。


## 结果

最终也还是拿到offer了。
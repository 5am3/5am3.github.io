---
title: GourdScan V2 配置安全及使用
comments: true
date: 2018-07-04 15:06:07
tags:
 - 漏洞挖掘
 - 被动扫描
categories:
 - 信息安全
 - 漏洞扫描
keywords:
 - 漏洞挖掘
 - 被动扫描
---

最近i春秋举办挖洞活动，于是自己也去凑个热闹。之前听大佬们讲过被动扫描，于是自己搜到了ysrc的这个扫描器。

https://github.com/ysrc/GourdScanV2

<!-- more -->

## 安装配置

关于安装配置，readme已经介绍的很清楚了。但奈何自己对Redis还有python一些库都不太熟，也懒得去踩那些坑。于是决定直接使用docker进行部署。

> 环境：
>
>   腾讯云学生机  
>
>   Ubuntu Server 16.04.1 LTS 64位
>
>   1 核 1 GB 1 Mbps
>
> 扫描的时候，将带宽临时升级为了5M（一天不到5块钱吧）



1. 首先安装docker

```bash
sudo apt install docker.io
```

2. 然后从GitHub下载该项目

```bash
git clone https://github.com/ysrc/GourdScanV2.git
```

下载完成后，进入该目录

```bash
cd GourdScanV2
```

3. 构造docker镜像

但是自己在使用途中，遇到了一个问题。

在安装tornado包的时候会报错。自己怀疑是python版本的问题，也有可能是其他原因。

百度无果后，自己尝试将基础镜像改为16.04即可。

```bash
sed -i "s/ubuntu:14.04/ubuntu:16.04/g" Dockerfile
```

然后运行

```bash
sudo docker build -t gourdscan:2.1 .
```

4. 此时镜像已经创建好了。接下来创建容器即可

```bash
sudo docker run -d -p 10000:22 -p 8000:8000 -p 10086:10086 -p 10806:10806 gourdscan:2.1 /usr/sbin/sshd -D
```

5. 进入docker，配置服务

然后登陆进入docker，登陆有两个方法。

- ssh登陆  `ssh root@localhost -p 10000`

*用户名: root，密码: Y3rc_admin*

- 直接进入docker `docker exec -it 容器ID bash`

此时运行服务即可

```
redis-server ~/gourdscan/conf/redis.conf
gourdscan
```

### 小技巧

有时候会懒得在进入docker启动两个服务，有没有办法让他直接启动呢？有的。

此时自行创建一个start.sh即可，然后在Dockerfile倒数第二行写入以下代码即可

start.sh内容

```bash
sleep 1
redis-server ~/gourdscan/conf/redis.conf
python ~/gourdscan/gourdscan.py
```



Dockerfile增加的内容

```
#添加start.sh，并准备开机执行
COPY ./start.sh /root/start.sh
RUN chmod +x /root/start.sh
ENTRYPOINT cd /root; ./start.sh
```



## 使用被动扫描

安装成功后，即可开始happy的玩耍了。

直接访问8000端口即可，默认平台用户名密码为：admin:Y3rc_admin

![mark](https://img.5am3.com/img/180704/6KC744Ihi3.png)

登录后会发现四个标签

分别是:

> 扫描记录，代理配置，扫描规则设置，全局设置

我们只需在代理配置处，随便选择一个代理，然后点击start即可。此时记录好端口号。在这里是10086端口。（如果用docker进行配置的话，请勿随意修改端口号，因为docker已经为默认端口做了映射，其他端口会无法访问的。）

![mark](https://img.5am3.com/img/180704/hdBaB96ICH.png)

后面就不用说了，本地挂上该代理，然后流量就会走扫描器了。

此时可以自己挂上代理尝试访问一些网站，然后在scan status处查看是否有日志，如果有的话，就没毛病了。



然后点开Scan Config标签，直接点击 Start Gourdscan Scanner 即可开始扫描。

![mark](https://img.5am3.com/img/180704/kGEgjLdeB8.png)

然后在Scan Status标签即可看到扫描日志。如果扫出漏洞，就会在Vulnerable处显示，点进去查看即可。

由于我扫描时没有截图，所以凑合看吧。

建议大家还是看一下官方readme，写的很清楚了。



## 一些感悟

自己扫了半天，扫出来两个大厂的反射型XSS，感觉还算不错的。

越发感觉扫描器的重要性，有必要研究一下扫描器，争取自己写一个玩。



在利用该扫描器扫描期间，暴露出了一些弊端，或许可以为自己以后避免踩坑。比如

> 1. 代理响应速度过慢，经常打不开网站。
> 2. 对大厂的一些waf应对不是很好。（经常伪报SQL注入漏洞）
> 3. 扫描时有时候会假死。
> 4. 对于反射型xss，仅验证的返回内容是否被转义，没有验证响应头是否可解析。（响应头为json时，浏览器是不会解析的）



关于第一点，可能是一个通病，因为是vps做代理，可能或多或少都会很慢，自己打算抽时间深入看一下，到底为什么。因为这个问题不止这一次困扰自己了。

再有就是第二点，这个其实还是大部分因为大厂的waf，貌似他们有风控，会对恶意请求随机拦截。也就是说同一个恶意请求，你发10次，他会拦截7次。还有3次是可以通过的。所以导致了扫描器的误报。（针对这个问题，也需要思考思考）



最后，貌似最近EnsecTeam在推一些漏洞扫描的建设文章，有兴趣可以关注一下他们，微信直接搜就可以。
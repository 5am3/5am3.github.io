---
title: 通过docker部署ctf题目
comments: true
date: 2017-12-08 11:32:13
tags:
	- docker
	- ctf
categories:
	- WEB
	- docker
keywords:
	- docker
	- ctf
---


最近社团要开始新一轮纳新。
出题的责任便落到了自己身上。
经过和各位大佬们深入交流，最终了解到要用docker部署。
自己的理解是，docker有点像一个轻量级的虚拟机。
这样就方便了主机不被黑。以及可以快速回滚，并且不用占用太多资源。简直是神器。

<!-- more -->


## 0x01 安装docker
> **初始环境**
> 阿里云9.9元学生机
> Ubuntu 16.04 64位

首先使用apt安装docker

`apt install docker.io`

然后执行`docker`，查看一下是否安装成功。
![](https://img.5am3.com/5am3/img/20191214145925.png)

接下来就可以部署镜像了。
## 0x02 部署镜像

找一个适合自己的镜像。 
[阿里镜像中心](https://dev.aliyun.com/search.html)

由于自己要部署web题，所以自己选择了一个apache-php5
![](https://img.5am3.com/5am3/img/20191214145931.png)

`docker pull registry.cn-hangzhou.aliyuncs.com/lxepoo/apache-php5`

然后运行镜像，并绑定一下端口。

`docker run -d -p 2027:80 registry.cn-hangzhou.aliyuncs.com/lxepoo/apache-php5`

![](https://img.5am3.com/5am3/img/20191214145939.png)

此时会返回一个值，表示该运行docker的id。以后如果想访问这个容器，需要通过该id。

然后将本地题目文件拷贝到docker，使用docker的cp命令即可。
在id前几位没有重复的情况下，可以取前几位。
`docker cp ./test e664955e:/var/www/`


因为该镜像已经将环境集成好，所以此时就不用管他了。

直接 可以进行`curl 127.0.0.1:2017 `测试一下是否部署成功

## 0x03 访问docker容器

最后，写一下如何进入docker容器内部.

`docker exec -it e664955e bash`

> -d :分离模式: 在后台运行
> -i :即使没有附加也保持STDIN 打开
> -t :分配一个伪终端



## 参考资料
1. [菜鸟教程-Docker 命令大全](http://www.runoob.com/docker/docker-command-manual.html)
2. [【docker】使用docker快速搭建nginx+php开发环境](http://blog.csdn.net/qq_28602957/article/details/53727865)
3. [IMOOC-Docker入门](https://www.imooc.com/learn/867)
4. [第一个docker化的java应用](https://www.imooc.com/learn/824)
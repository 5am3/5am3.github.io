---
title: 【诺熙】XMAN训练营手记-2-密码
comments: true
date: 2017-08-02 22:44:06
tags: xman
categories: 
	- 信息安全
	- xman
keywords: xman
---
今天是第二天，来自清华的庄泽浩大佬给讲了一下密码学的常见形式。从古至今的密码，典型事例一一讲解。
说实话，由于自己的数学水平。今天被虐了一天。/委屈状
但除了数学相关的一些知识，其他的还是蛮好懂得！

<!--more-->

## 密码学概论

### 密码分析的分类

**密码分析者攻击密码的方法**

1. 穷举攻击
2. 统计分析攻击
3. 数学分析攻击

**根据密码分析者利用的数据来分类**

*以下攻击强度依次增大*

1. 唯密文攻击
2. 已知明文攻击
3. 选择明文攻击
4. 选择密文攻击

### 密码的安全性

**常用准则有以下**

- 计算安全性
- 可证明安全性
- 无条件安全性

## 古典密码

### 单表代换密码

- **移位密码**

当移位为三时，叫做凯撒密码。（之前一直认为移位密码就是凯撒/尴尬）

```
不安全，秘钥空间小，可穷举
```

- **仿射密码**

大佬上课讲的太数学化，有点懵，课下查了查资料。
了解了些许。下面是个栗子，大家尝尝。

> 设密钥K= (7, 3), 用仿射密码加密明文hot。
>
> 三个字母对应的数值是7、14和19。分别加密如下：
>
> (7×7 + 3) mod 26 = 52 mod 26 =0
>
> (7×14 + 3) mod 26 = 101 mod 26 =23
>
> (7×19 + 3) mod 26 =136 mod 26 =6
>
> 三个密文数值为0、23和6，对应的密文是AXG。

- **埃特巴什码**

很简单的一个单表替换。唉，自己出生晚，要不就叫诺熙密码了。

```
明文：A B C D E F G H I J K L M N O P Q R S T U V W X Y Z
密文：Z Y X W V U T S R Q P O N M L K J I H G F E D C B A
```

- **简单替换密码**

杂乱无章，瞎替换。保证一对一且唯一就好。

- **解决单表替换**

1. 文本量大时，可用统计方法解。

![](https://img.5am3.com/5am3/img/20191214150133.png)

2. 移位密码可以直接暴力破解。

### 多表代换加密

将多个字母一起加密，例如单词替换。

- **Playfair**

蛮有意思的一个密码，加密生动有趣/笑哭。

详细见[百度百科-playfair密码](https://baike.baidu.com/item/playfair%E5%AF%86%E7%A0%81/8999814?fr=aladdin)，由于太过占用地方，在此不做过多解释。

- **Polybius**

一种棋盘密码，先制出密码表，然后按坐标进行加密。

- **Nihilist**

类似Polybius，但相对于他，多了一个关键字。将关键字组成棋牌，然后进行解密。

## 其他一些有趣的古典密码

- **Hill**

传说中的希尔密码，我的线代就是因它而挂！！！
不多说了，我默默学线代去了。

- **维吉尼亚密码**

哈哈哈哈哈哈。

![](https://img.5am3.com/5am3/img/20191214150128.jpg)

- **莫斯电码**

声音中的战斗机。
一张图搞定。

![](https://img.5am3.com/5am3/img/20191214150121.jpg)

- **培根密码**

类似莫斯电码。

![](https://img.5am3.com/5am3/img/20191214150110.png)

- **栅栏密码**

传说中的藏头诗哦！
但英语通常是藏好了，然后组成一句话。
一点也没有中国语言的那种博大精深。

- **曲路密码**

先做表格，然后按路径在表格上走就好了。嘻嘻！

- **猪圈密码**

正如大佬说说，能看懂的就能看懂，看不懂的怎么看也看不懂！

![](https://img.5am3.com/5am3/img/20191214150142.png)

- **键盘密码**

正如其名，有手机键盘和电脑键盘之分。

手机直接数字就好。拿到数字，手机一通瞎按，就解出来了！

电脑呢，可以围成一圈，中间那个字是明文，还可以直接用字母在键盘上的位置画一个字出来。
当然，比较6的键盘密码则是**qwe加密**。不是LOL哈、

## 现代密码学

由于本萌新数学不好，这里就不过多介绍了。/委屈

大概说一下就过哈。

### 分类

- **对称密码体制**

这一堆很6很6的密码，表示自己一个也搞不懂。
坐等下学期学离散。

```
DES，AES，RC5，ECB，CBC，CFB，OFB。
```

- **非对称密码体制**

公钥+私钥，公钥公开

```
RSA
```

## 零知识证明

意思是在不透露任何消息的情况下，让别人确信你的确知道消息。

例如，不告诉他银行卡密码。自己独自登进银行账户，让他看一眼。

## 编码

编码和加密不同。加密为了保证数据安全。而编码则是尽量方便数据传输。

### url编码

又称%号编码，在%号后面加十六进制的ASCII码即可。
常用于web。

默默想起了xman预选赛的那道神坑题，明明已经将xman编码为`%78%6d%61%6e`了,没想到还是不行。最后才发现神坑的题，竟然还要把%号编码为`%25`，所以最后结果为`%2578%256d%2561%256e`。
好在最后心血来潮，脑洞一开，试了一下。

### base64编码

具体细节不太清楚，但是大概了解一些特征。

1. 部分编码会用`=`号填充
2. 共63个字符，`A-Z,a-z,+,/`

一般出现以上两点任意一点明显特征，均可试一下base64解码。

## 哈希

常见的有md5，sha1等，但据说这两种都被破解了。不安全。
但ctf常见的还是md5，比较有学习意义。

## 一些工具

词频分析工具：http://quipqiup.com/

md5解密：http://www.dmd5.com/

密码解密：http://cryptool-online.org/

大素数生成：yafu

万能解码工具：cap

豆丁ctf工具合集：http://www.docin.com/p-1492238521.html

## 尾言
**大概就写这么多吧，说实话，今天由于数学不好，上课听得很（shui）认（zhao）真（le）。**

---
title: 【诺熙】XMAN训练营手记-3-Misc&Android
comments: true
date: 2017-08-03 22:16:01
tags: xman
categories: 
	- 信息安全
	- xman
keywords: xman
---
## misc

- `string`,`binwalk`
- Recon
- Encode
	- 二进制转化
		- ASCII
		- 二维码
		- 图片
- 图片
	- JPG
		- 文件格式
		- 隐写软件
	- PNG
		- 文件格式
		- LSB
		- 软件
	- GIF
		- 文件头
		- 空间
		- 时间轴
- 音频
	- 波形
	- 频谱
	- 软件
- 压缩包
	- 文件格式
	- 攻击手法
		- 爆破
		- crc32
		- 明文攻击
		- 伪加密
- 流量包
	- 文件修复
	- 协议分析
	- 数据提取：tshark

## Android

**底层系统**
1. bootloader

2. recovery

3. baseband

4. android

**调试**
1. ida pro
2. gdb

**hook**
1. xposed

2. frida
### 总结

- 尽量使用已有工具
	- Xposed,Frida,IDA,时间最重要
	- 脱壳工具
- 开脑洞找漏洞
	- 很多题目的解法都跟出题者的预想不同
- 直接强行逆向
	- 通用解法
	- 出题者尽力想达到的目标
		- 脑补一个绞尽脑汁纠结的出题者
		- 保证题目争取、时间复杂度足够抵抗爆破。还要新颖有趣。
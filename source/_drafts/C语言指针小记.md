---
title: C语言指针小记
comments: true
date: 2017-07-01 20:22:25
tags:
categories:
keywords:
---
# C语言指针小记


```
char * names[6]
    = { "Guanyu", "Zhangfei",
    "Zhaoyun", "Machao",
    "Huangzhong", "Liubei" };
printf("%s\n", &(names[2][2]));
return 0;
```
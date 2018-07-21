---
layout: post
title:  "推荐一下自己写的通讯调试软件"
date:   2008-12-23 16:51:00
categories: DesktopDev
tags: Debug UART Serial TCP UDP
---

* content
{:toc}

CommDebug v1.1.3.6

自己开发的一个通讯调试软件，个人觉得还是有点特色的。

特点：
1. 支持串口（基于SPComm，并有所修改）和UDP，TCP服务器，TCP客户端。
2. 数据转发：支持主通道到多个转发通道的数据转发。
3. 数据支持十六进制，字符和混合模式三种方式输入和显示，尤其是混合模式比较有特色，可以方便AT命令等输入的需要。
4. 支持几种校验的生成（目前仅仅实现个人用到的几种校验），支持包的前导字符和结束字符的添加。
5. 命令序列功能。示例如下：
```
-----------------------------------
#第一个字母 S -> 发送报文， R -> 等待接收， T -> 等待一定时间。
#S 后面 M: 表示后续报文为混合模式。H:十六进制，S:字符串
#R 后面 T 表示等待一个超时时间或接收到报文(ms)， M: 表示后续为混合模式。
#T 等待一个时间(ms)。
S:M:{0d}{0e}send{0d}{0a}tools programs commdebug
R:T10000:M:{0d}{0a}recv
T10000
S:S:tools programs commdebug
-----------------------------------
```

欢迎使用，并提出宝贵意见。

[意见反馈](mailto:zk_zhb@tom.com) 以及 [软件下载]({{ site.url }}/downloads/CommDebug.zip)
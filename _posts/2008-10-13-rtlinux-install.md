---
layout: post
title:  "RTLinux安装"
date:   2008-10-13 11:11:00
categories: linux real-time
tags: rt-linux
---

* content
{:toc}

RTLinux和Linux上装软件一样，存在很多的不确定，前些时候，老婆因工作需要开始折腾RTLinux，耗费了很多的时间，记下来作为以后参考。

RTLinux 3.1由于不支持Etx3文件系统，而RTLinux又非常容易导致系统死机，还是用3.2。编译了很多版本，忘得差不多了，现在写一下记得的情况。

RTLinux 3.2配合Linux 2.4.19内核的时候，采用gcc编译器会编译出来RTLinux失败，但是可以采用kgcc，使用的是kgcc-1.1.2。使用RTLinux 3.2配合Linux 2.4.29内核，采用gcc就可以，不用改动Makefile。RTLinux 3.2配合Linux 2.4.26内核无论如何都不行。上面RTLinux编译平台为Redhat Linux 8.0和Redhat Enterprise Linux AS Release 3。

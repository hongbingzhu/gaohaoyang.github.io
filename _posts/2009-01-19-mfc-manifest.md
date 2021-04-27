---
layout: post
title:  "VC2005 MFC程序的清单文件"
date:   2009-01-19 14:01:00
categories: desktop mfc
tags: mfc manifest
---

* content
{:toc}

整理接手的VC2003的MFC程序，升级到2005，调试运行提示清单问题。经过研究，发现VC2005程序Debug模式不能嵌入清单，而Release模式嵌入不嵌入均可。


2009-02-06 后来发现也不一定，如果碰到相同的情况，可以一试吧。
---
layout: post
title:  "雪狐密码箱PwdBox记录导出"
date:   2012-11-05 10:01:44
categories: desktop works
tags: pwdbox delphi spy++
---

* content
{:toc}

n年前，选择了一款密码箱软件，当时选择了PwdBox。进来越来越发现这个密码箱不好用，例如多平台等。要命的是它还没有导出功能。看着里面的好几百条记录，手动拷贝显然不靠谱。得，写个专门程序吧。

导出程序思路上不复杂，给程序发消息进行遍历，得内容呗。用Spy++看了下，Delphi程序，基本构成上有一个TTreeView，一个ListView，还有若干个类Edit。程序也比较简单，得到TTreeView和TListView以及若干Edt的HWND，对TTreeView和TListView进行遍历，然后依次获取Edit的内容。但其中也有一些要注意的：

1. 对TTreeView和TListView的操作要注意跨进程问题，所以调TreeView_xxx类和ListView_xxx类函数，要在目标程序的进程空间内分配内存，再进行进程间数据交换，基本手段为VirtualAllocEx，WriteProcessMemory和ReadProcessMemory。

2. 获取文本使用WM_GETTEXT不需要使用目标进程的内存空间，用本进程的即可，操作系统会处理跨进程问题。

注意以上两个问题，遍历就不是什么难事了。不过，这也花费了我一天的时间。
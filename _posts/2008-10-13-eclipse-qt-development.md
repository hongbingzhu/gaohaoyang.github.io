---
layout: post
title:  "搭建Eclipse+Qt开发平台"
date:   2008-10-13 18:06:00
categories: desktop qt
tags: eclipse qt
---

* content
{:toc}

开始研究Qt开发，决定使用Eclipse+Qt平台，经过实践，得出安装过程如下：
1. 下载安装[Eclipse IDE for C/C++ Developers](http://www.eclipse.org/downloads/download.php?file=/technology/epp/downloads/release/ganymede/SR1/eclipse-cpp-ganymede-SR1-win32.zip)，该安装已经含有CDT。
2. 下载安装MinGW，安装完毕以后将bin目录加到PATH中。到这里就可以开发C++程序了。
3. 下载安装gdb，在下载MinGW处单独下载安装，可以下载gdb-5.2.1安装程序，省得麻烦。到这里就可以调试C++程序了。如果从来没有使用过Eclipse，建议调试一个C++程序，可以参考：[Developing applications using the Eclipse C/C++ Development Toolkit](http://www.ibm.com/developerworks/opensource/library/os-eclipse-stlcdt/)。
4. 下载安装Qt。qt-win-opensource-4.4.3-mingw.exe
5. 下载安装Qt Eeclipse Integration。qt-eclipse-integration-win32-1.4.1.exe

到这里就OK了。然后新建Qt工程，开发。呵呵，测试了整整一天，总算弄明白了关系。
---
layout: post
title:  "xnix下的touch命令"
date:   2011-12-17 10:23:08
categories: linux shell
tags: touch
---

* content
{:toc}

### unxutils, picnix

对touch命令，一直有个印象，但是一直不知道有什么用处，昨天才知道touch命令真是很有用的。

项目中，开发测试中，板子上的FW版本众多，后台软件也没成熟，经常要确认板子上的FW的版本，然而，又很难每次都RebuildAll，如果含有\_\_DATE\_\_宏的源文件没有修改，就会导致FW不能更新版本日期信息。这时候就有touch的用武之地了。

例如在我们的项目中，golbal.c中有\_\_DATE\_\_定义，那在IAR环境中，在Pre\_build command line中，填入“touch global.c”，从此就可以高枕无忧了。

windows下是没有Touch的，可以去 http://sourceforge.net/projects/unxutils/ 下载个吧。

相关介绍来源：http://bbs.et8.net/bbs/showthread.php?t=613562

长时间公用Linux和Windows，特别喜欢Linux的命令行工作方式，特别是在分析文本，查看日志什么的，Linux的grep、head、tail、tar等命令是特别好用的，以前一直安装使用Cygwin，安装比较麻烦。

最近整理一下命令行工具，把对应xNix版的命令行工具打了个包共享给大家。

其主要来源于以下两个网站: http://www.loa.espci.fr/winnt/, http://unxutils.sourceforge.net/

SF还有一个: http://sourceforge.net/projects/picnix/

但是picnix的文件普遍比unxutils的大。 

现在可以上传了。 

每个执行文件都有HELP，加上--help参数就可以看见简单介绍了。

我就简单列一下我用的较多的命令和用处说一下。

<pre>
cat 和dos 的type差不多
\*zip\*.exe 命令行的压缩成gz的
cmp.exe 比较文件的
compress.exe 压缩成*.Z的格式，压缩率没有gzip高
cp.exe 等于dos的copy
df.exe 看磁盘空间的
diff.exe/patch.exe 给文件作补丁用的，合并差异文件，变成最新版本
echo.exe =dos echo
env.exe =windows nt中的set
expr.exe 表达式计算
<font color=orange>find.exe 命令行搜索，用惯这个再也不会用windows中的文件搜索了</font>
<font color=orange>gawk.exe 一种脚本解释器</font>
<font color=orange>grep 支持正则表达式的文本分析提取程序</font>
g\*zip\*.exe gnu 的zip程序
head.exe 用来显示一个文本文件的头部一部分内容
id.exe 显示当前用户名和组名
ls.exe 等于dos dir当然要比dir强了
mkdir =dos md
mv.exe =dos move
<font color=orange>ps.exe 显示当前的进程信息，不用再看task manager了</font>
rm.exe 类似deltree 和rd的功能
<font color=orange>sed.exe 流编辑器</font>
su.exe 用来切换用户的，不知道在windows里干什么
sleep.exe 暂停一段时间
sync.exe 同步程序
<font color=orange>tail.exe 显示文件尾部内容</font>
<font color=orange>tar.exe 打包程序</font>
<font color=orange>touch.exe 修改文件时间位当前时间</font>
un\*.exe 都是解压缩的程序
<font color=orange>wc.exe 字数统计程序</font>
wget.exe 下载程序，类似于flashget
<font color=orange>which.exe 搜索你的path找到你想知道某个命令对应的程序，类似于linux中的type</font>
</pre>

其中红色的命令，对程序开发员很有用。`grep`用来提出日志信息，`sed`用来流编辑，类似于ultraedt 中正则表达式替换功能，`touch`用来更改文件时间，不需要用编辑器大开，什么都不干保存一下，`tail -f filename`用来实时查看文件的内容，`tar`用来打包/解tar包。

### MS SFU

好象装个MS SFU也可以。

SFU提供一致的跨Unix操作系统平台脚本执行的能力：
* Korn Shell 
* C Shell
* 超过350常用的UNIX命令和应用

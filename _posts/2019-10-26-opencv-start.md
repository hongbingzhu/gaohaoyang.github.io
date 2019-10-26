---
layout: post
title:  "OpenCV笔记（I）"
date:   2019-10-26 21:23
categories: OpenCV
tags: OpenCV
---

* content
{:toc}

这里记一下开始入手OpenCV碰到的一些问题以及解决办法。学习参考书是《OpenCV 4 计算机视觉项目实战（原书第2版）》，ISBN：978-7-111-63164-4。

### Ubuntu 16.04下安装

构建安装原书最主要的两个命令和测试命令是：
```bash
$ cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/full/path/to/opencv-4.0.0/build -D INSTALL_C_EXAMPLES=ON -D BUILD_EXAMPLES=ON -D OPEN_EXTRA_MODULES_PATH=/full/path/to/opencv_contrib-4.0.0/modules ..
$ cp /full/path/to/opencv-4.0.0/build/lib/pkgconfig/opencv.pc /usr/local/lib/pkgconfig/opencv4.pc
$ cd /full/path/to/opencv-4.0.0/samples/cpp
$ g++ -ggdb `pkg-config --cflags --libs opencv4` opencv_version.cpp -o /tmp/opencv_version && /tmp/opencv_version
```
这里面会碰到几个问题：

1. 下载`ippicv_2019_lnx_intel64_general_20180723.tgz`会被卡住

这个不算是国内问题，是cmake缺省不支持https的问题。![这里](http://morecoder.com/article/1262570.html)有关于这个问题的说明。我只做了
```bash
$ sudo apt-get install libcurl4-openssl-dev
```
cmake就应该可以成功了

2. 复制opencv.pc的时候，会找不到opencv.pc

似乎opencv认为`pkg-config`包管理器已经落伍，所以缺省是不会生成该文件的。要生成该命令，需要修改cmake命令，打开产生opencv.pc的选项：
```bash
$ cmake ... -D OPENCV_GENERATE_PKGCONFIG=ON ..
```

3. 编译测试程序出错

这里面有几个错误，1) 需要c++11； 2) 连接找不到`cv::CommandLineParser`类的一些函数，解决方法是：
```bash
$ export LD_LIBRARY_PATH=/full/path/to/opencv-4.0.0/build/lib
$ g++ -std=c++11 -ggdb opencv_version.cpp  `pkg-config --cflags --libs opencv4` -o /tmp/opencv_version && /tmp/opencv_version
```

### cmake找不到新编译安装的opencv

在编译第二章的示例程序的时候，会发现cmake会失败，需要把CMakeLists.txt的寻找opencv包命令这句修改为：
```cmake
FIND_PACKAGE( OpenCV 4.1.2 REQUIRED PATHS /home/opencv/4.1.2 )
```
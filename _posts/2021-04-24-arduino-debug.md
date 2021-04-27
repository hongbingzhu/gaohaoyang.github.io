---
layout: post
title:  "在线调试Arduino"
date:   2021-04-24 20:52
categories: arduino embedded debug
tags: arduino debug jtag eclipse vscode
---

* content
{:toc}

Arduino是一个比较好的平台，里面丰富的库，但是可惜没有在线调试功能。如果仅仅做教学和教育使用，Arduino的IDE环境是可以满足要求的。但是如果应用到项目中，没有在线调试功能显然是不可接受的。尝试在一个使用STM32F103的项目使用arduino平台，将如何进行调试进行了一下总结。目前看，还没有找到各方面都达到要求的环境，只能综合各个方案，根据问题选用。

### Arduino本身 （没有调试功能）
即使Arduino没有没有在线调试功能，但是Arduino作为官方的环境，显然编译下载的兼容性是最好的。所以我们需要该环境做一下信息对比，以查找出在其他环境下的问题点。Arduino环境下，编译的时候会输出编译过程详细的命令行调用，这为我们脱离Arduino环境编译提供基础。
我使用的STM32芯片，我推荐使用[stm32duino](https://github.com/stm32duino/BoardManagerFiles)项目，将json文件地址添加到Preferences对话框的“Additional Boards Manager URLs”里面，Arduino会完成该STM32系列芯片的支持。该项目支持的板子和芯片及其丰富，基本上能ST官方的板子，都支持了。

### eclipse + sloeber （STM32平台编译失败）
在以前关于arduino的文章中(2019-02-27-stm32-arduino-f103)，曾经使用过“Eclipse C++ IDE for Arduino”，但是这个项目已经不维护了，推荐使用“The Arduino Eclipse plugin named Sloeber V4”来替代，这个插件不依赖Arduino的IDE本身，其会下载自己的Arduino基本支持工具和库，安装比较费时间，安装完毕之后，还需要在“Arduino=>Preferences”对话框下，“Private hardware path”中添加额外的硬件支持库路径，一般是“C:\Users\<User>\AppData\Local\Arduino15”，如果在Arduino下已经下载完毕，这里不需要重复下载。
依据插件的说明，在项目右键的“Properties”菜单下，在项目属性对话框中，在“Arduino”下，有硬件平台，板卡以及其他选项的设置，信息和Arduino环境很类似，正确选择即可，可惜这个方式编译STM32平台有问题，其调用的编译器是“/bin/arm-none-eabi-g++”，显然是不对的。其对于本身自带的基础板子，例如Arduino-DUE，是可以正确编译的，但可惜不是我们要的。由于我也没有AVR的在线调试器，也就没有测试AVR平台的在线调试。
虽然不能编译，但是代码跳转功能是没问题的，还不错。

### CMake和Makefile （没有代码跳转功能）
既然插件编译有问题，那我们可以先脱离Arduino环境生成目标文件，再在eclipse中使用makefile项目来进行调试（期望可以提供调试和代码跳转），经过一番努力，终于搞定了的[CMake]({{ site.url }}/downloads/cmake-arduino-stm32f103.rar)文件，和[Makefile]({{ site.url }}/downloads/Makefile-arduino-stm32f103.rar)文件，CMake文件有比较大的坑，如果将两大块，分别生成wrapper和cores库，会出现链接问题。CMake的缺省link调用也有问题，需要自己去手动指定link调用。这么一来，其实远远没有Makefile本身来得更方便。
然后用带有“GNU ARM eclipse”插件的eclipse环境，新建“GDB Hardware Debugging”目标来调试，就可以顺利调试了。gdbserver可以用“STM32CubeIDE”带的“ST-Link_gdbserver”。复制其一个tools下的小目录，就够了，总共也就2M多一点，找个项目附近的地方放就可以了。
似乎这还挺好，唯一遗憾是，没有代码跳转功能。

### VSCode + Arduino插件 （环境强大，未能成功调试）
之后，无意中发现，VSCode+Arduino插件（均为微软出品），其实是更好的一个方案，该插件依赖Arduino环境本省，安装省事。代码跳转功能也好用强大，编译也顺利。可惜调试缺省用的是OpenOCD，而不是J-Link或者ST-Link，而我对OpenOCD不太熟悉，也不知道怎么设定去更换（README里面的相关设置项，似乎不管用）
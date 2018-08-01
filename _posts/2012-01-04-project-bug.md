---
layout: post
title:  "项目开发中的“绕”"
date:   2012-01-04 21:33:53
categories: Project
tags: Bug Project
---

* content
{:toc}

一个项目在开发过程中，不可避免会遇到一些问题，出现这些问题的时候，该怎么办。目前我看到的处理方法无外乎两种，一种就是找到问题的根源，解决它。另外就是绕开它。就纯粹技术开发角度上讲，“绕”当然是不可接受的，在这之前我对待问题的基本态度也是找到根源，解决它，不然心里不舒服。但是从项目管理角度上讲，第一种方法就未必是当然的选择了，其中要考虑项目的进度等要求。8月份跳到了一家公司，在一个现有的系统上增加功能，做后续开发，和我以前对待问题的截然相反，这家公司对待问题的方法基本是能绕就绕。这可能和他们人手比较紧，追求开发效率有关系。

在我们的平台上，跑的是uC/OS，中断全部由OS接管，为了简单，程序没有中断嵌套。总得来说，我对这个系统的评价是相当低的。这个系统废代码连篇，不同的应用目标靠完全靠宏定义区分，导致程序流程很混乱。系统模块化程度非常差，耦合度很高，这导致这个系统进行改进会相当的困难。大老板的态度是现有的代码经过了测试，比较可靠，也不同意进行改进和重构，这样一来，这个系统的重构，优化，基本是不可能的事情。

最近我接手的程序出现了EEPROM读取失败的问题，经过定位，基本是读取的时候，会读出不连续的EEPROM数据，但是字节数是对的，经过今天排查，发现是由于在系统有频繁中断的时候，由于AT91SAM9G45芯片的TWI外围在都操作的时候，没有overload保护机制，读取EEPROM是如果发生其他的长时间中断，就会导致EEPROM数据被覆盖。导致操作错误。这个问题其实他们以前发现过，当时他们采取的办法是将操作EEPROM的代码换了个位置，躲避了这个问题。

总结这个事情，个人对于问题的看法是，在项目迫不得已的时候，可以采取绕的办法，但是在事后，一定要抽时间解决这个问题，不然，这个问题肯定会在以后某个时候再出来。类似地，对于系统，构建一个结构良好、强健的系统，在构建的时候可能会费时间，效率低，但这些花出去的时间一定会在以后得到回报。可惜国内负责任的程序员太少，能真正追求产品的的老板也太少，导致项目开发总是强调眼前的功能，到最后，一定是的得不偿失的。
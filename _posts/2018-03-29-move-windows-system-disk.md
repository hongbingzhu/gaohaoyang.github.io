---
layout: post
title:  "将Windows系统移到另一个硬盘"
date:   2018-03-29 15:51:01
categories: desktop windows
tags: migration
---

* content
{:toc}

原先的128GB SSD，给Windows用是够了，最近虚拟机用得多，靠以前的SSD外挂着用，实在有点不爽，就入手一个256GB的，重装系统是个令人头疼的事情，当然不能干。想起来以前另一个机器操作的时候，查过直接复制分区就可以。所以直接启动到另一个临时系统，做整盘ghost。替换以后，发现系统启动不了了。想了想，启动到Linux，dd，然后手动调整分区，再安装好，启动，成功。移动系统就是这么简单，哈。
---
layout: post
title:  "嵌入式项目设计应该注意的事项"
date:   2011-12-17 14:22:29
categories: architecture
tags: note
---

* content
{:toc}

1. 模块化设计，模块之间的耦合度一定要低。这样无论对于扩展，延续性，健壮性都有好处。

2. 保持清晰的Debug系统，尤其对于带有一定平台性质的项目，一定要建立一个清晰的Debug系统。Debug系统在工作的时候，不应该成为系统的瓶颈。例如，使用串口输出Debug信息，不应该使用同步方式，一定要带高速缓冲。
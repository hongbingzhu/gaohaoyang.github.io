---
layout: post
title:  "PowerPC的U-Boot Nand启动SPL技巧"
date:   2013-01-09 12:42:24
categories: 
tags: 
---

* content
{:toc}

PowerPC U-Boot程序的Nand启动spl处理很有点意思，由于Nand只有4k可靠，所以u-boot的NAND启动由3部分构成：SPL1，SPL2和u-boot本体。

在SPL1结束的位置，代码大概如下：

```asm
#ifdef CONFIG_NAND_SPL_S1
 mflr r8
 li r3,0x1000
 add r8,r8,r3  /* Shift address by 0x1000 */
 mtlr r8
 blr
#else
 /*
  * First initialize SDRAM. It has to be available *before* calling
  * nand_boot().
  */
.........
#endif
```

这个代码很有意思，这个代码的导致SPL1读取SPL2代码之后，直接跳到#else后面的代码执行，虽然SPL2和SPL1代码在这之前是一样的，但实际在SPL2中不执行。在明白之前，还一直疑惑，寄存器、TLB之类的重复做一遍，不会有问题吗。呵呵。
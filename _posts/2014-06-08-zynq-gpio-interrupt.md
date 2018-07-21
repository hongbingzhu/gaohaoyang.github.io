---
layout: post
title:  "Linux Zynq GPIO中断"
date:   2014-06-08 18:39:37
categories: Kernel
tags: Kernel Zynq interrupt
---

* content
{:toc}

注册中断：对每个pin进行循环遍历
```c
for (pin_num = 0; pin_num < min_t(int, ZYNQ_GPIO_NR_GPIOS,  (int)chip->ngpio); pin_num++)
    gpio_irq = irq_find_mapping(irq_domain, pin_num); 
```
将GPIO号映射为Linux系统中断号。

在Linux中断系统中，一个irq_domain表示一个中断控制器，其内中断由0开始编号（<font color=orange>尚存在疑问</font>）
```c
unsigned int irq_find_mapping(struct irq_domain *domain, irq_hw_number_t hwirq)
```
将一个中断控制器上的某个硬件中断映射为某个Linux系统中断。

```c
/**
 * struct irq_domain - Hardware interrupt number translation object
 * @link: Element in global irq_domain list.
 * @name: Name of interrupt domain
 * @ops: pointer to irq_domain methods
 * @host_data: private data pointer for use by owner.  Not touched by irq_domain
 *             core code.
 *
 * Optional elements
 * @of_node: Pointer to device tree nodes associated with the irq_domain. Used
 *           when decoding device tree interrupt specifiers.
 * @gc: Pointer to a list of generic chips. There is a helper function for
 *      setting up one or more generic chips for interrupt controllers
 *      drivers using the generic chip library which uses this pointer.
 *
 * Revmap data, used internally by irq_domain
 * @revmap_direct_max_irq: The largest hwirq that can be set for controllers that
 *                         support direct mapping
 * @revmap_size: Size of the linear map table @linear_revmap[]
 * @revmap_tree: Radix map tree for hwirqs that don't fit in the linear map
 * @linear_revmap: Linear table of hwirq->virq reverse mappings
 */
struct irq_domain {
	struct list_head link;
	const char *name;
	const struct irq_domain_ops *ops;
	void *host_data;
 
	/* Optional data */
	struct device_node *of_node;
	struct irq_domain_chip_generic *gc;
 
	/* reverse map data. The linear map gets appended to the irq_domain */
	irq_hw_number_t hwirq_max;
	unsigned int revmap_direct_max_irq;
	unsigned int revmap_size;
	struct radix_tree_root revmap_tree;
	unsigned int linear_revmap[];
};
```

revmap_direct_max_irq: 小于该值的中断，Linux中断号和硬件中断号相同，直接返回。

revmap_size: 线性反向映射（<font color=orange>似乎要求域内IRQ从零开始，有点矛盾</font>），小于该值的hwirq直接利用linear_revmap做查找。否则用radix tree来查找映射。

irq_set_chip_and_handler(gpio_irq, &zynq_gpio_irqchip, handle_simple_irq);

调用irq_get_desc_lock(irq, &flags, 0);，获取irq对应的irq_desc。并设定irq_desc的chip：desc->irq_data.chip = chip;

调用irq_reserve_irq(irq);，将allocated_irqs中断位图中相应的中断标识为已占用。

调用__irq_set_handler，将irq_desc中的handle_irq设定：desc->handle_irq = handle;

irq_set_chip_data(gpio_irq, (void *)gpio);

这个比较简单，将要用私有的变量关联到irq，desc->irq_data.chip_data = data;

set_irq_flags(gpio_irq, IRQF_VALID);



总体调用：

irq_set_handler_data(irq_num, (void *)gpio);

这里的irq_num是通过irq_num = platform_get_irq(pdev, 0);获取的系统配置文件里面的irq配置。这个函数也简单，实质为：desc->irq_data.handler_data = data;

irq_set_chained_handler(irq_num, zynq_gpio_irqhandler);

这个函数实质为：desc->handle_irq = handle;



最终调用关系为：调用zynq_gpio_irqhandler，然后在该函数中通过调用generic_handle_irq来调用最终的handle_simple_irq。 
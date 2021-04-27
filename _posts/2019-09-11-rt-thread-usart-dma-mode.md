---
layout: post
title:  "RT-Thread中的串口DMA分析"
date:   2019-9-11 11:09
categories: embedded rtos
tags: rt-thread stm32 usart
---

* content
{:toc}

这里分析一下RT-Thread中串口DMA方式的实现，以供做新处理器串口支持时的参考。

### 背景

在如今的芯片性能和外设强大功能的情况下，串口不实现DMA/中断方式操作，我认为在实际项目中基本是不可接受的，但遗憾的是，rt-thread现有支持的实现中，基本上没有支持串口的DMA，文档也没有关于串口DMA支持相关的说明，这里以STM32实现为背景，梳理一下串口DMA的实现流程，以供新处理器实现时以作参考。

### DMA接收准备

启用DMA接收，需要在打开设备的时候做一些处理，入口函数为rt\_device\_open()。主体实现是：
```c
rt_err_t rt_device_open(rt_device_t dev, rt_uint16_t oflag)
{
    ......
    result = device_init(dev);
    ......
    result = device_open(dev, oflag);
    ......
}
```

device\_init()就是rt\_serial\_init()函数，其主要是调用configure()函数，
```c
static rt_err_t rt_serial_init(struct rt_device *dev)
{
    ......
    if (serial->ops->configure)
        result = serial->ops->configure(serial, &serial->config);
    ......
}
```
在stm32下，其configure()函数是stm32\_configure()，其根据设备打开参数，配置STM32外设的寄存器。包括波特率、校验等串口工作参数。

device\_open()函数就是rt\_serial\_open()函数，其主要实现是：
```c
static rt_err_t rt_serial_open(struct rt_device *dev, rt_uint16_t oflag)
{
    ......

#ifdef RT_SERIAL_USING_DMA        
    else if (oflag & RT_DEVICE_FLAG_DMA_RX)
    {
        if (serial->config.bufsz == 0) {
            struct rt_serial_rx_dma* rx_dma;

            rx_dma = (struct rt_serial_rx_dma*) rt_malloc (sizeof(struct rt_serial_rx_dma));
            RT_ASSERT(rx_dma != RT_NULL);
            rx_dma->activated = RT_FALSE;

            serial->serial_rx = rx_dma;
        } else {
            struct rt_serial_rx_fifo* rx_fifo;

            rx_fifo = (struct rt_serial_rx_fifo*) rt_malloc (sizeof(struct rt_serial_rx_fifo) +
                serial->config.bufsz);
            RT_ASSERT(rx_fifo != RT_NULL);
            rx_fifo->buffer = (rt_uint8_t*) (rx_fifo + 1);
            rt_memset(rx_fifo->buffer, 0, serial->config.bufsz);
            rx_fifo->put_index = 0;
            rx_fifo->get_index = 0;
            rx_fifo->is_full = RT_FALSE;
            serial->serial_rx = rx_fifo;
            /* configure fifo address and length to low level device */
            serial->ops->control(serial, RT_DEVICE_CTRL_CONFIG, (void *) RT_DEVICE_FLAG_DMA_RX);
        }
        dev->open_flag |= RT_DEVICE_FLAG_DMA_RX;
    }
#endif /* RT_SERIAL_USING_DMA */
	......
#ifdef RT_SERIAL_USING_DMA
    else if (oflag & RT_DEVICE_FLAG_DMA_TX)
    {
        struct rt_serial_tx_dma* tx_dma;

        tx_dma = (struct rt_serial_tx_dma*) rt_malloc (sizeof(struct rt_serial_tx_dma));
        RT_ASSERT(tx_dma != RT_NULL);
        tx_dma->activated = RT_FALSE;

        rt_data_queue_init(&(tx_dma->data_queue), 8, 4, RT_NULL);
        serial->serial_tx = tx_dma;

        dev->open_flag |= RT_DEVICE_FLAG_DMA_TX;
        /* configure low level device */
        serial->ops->control(serial, RT_DEVICE_CTRL_CONFIG, (void *)RT_DEVICE_FLAG_DMA_TX);
    }
#endif /* RT_SERIAL_USING_DMA */
    ......
}
```
可见，其主要工作是为DMA接收准备FIFO缓冲区；为DMA发送准备发送数据缓冲队列，但是好像STM32中断并没有用到发送数据缓冲。

DMA配置数据来源是rt\_hw\_usart\_init()函数，缺省的配置参数由宏RT\_SERIAL\_CONFIG\_DEFAULT决定，
这里决定了缺省的接收缓冲区参数是64字节，通讯缺省参数是：115200,8N1。
```c
#define RT_SERIAL_RB_BUFSZ              64
```

### DMA接收

DMA接收我们从DMA中断开始分析，DMA接收中断服务函数为UARTn\_DMA\_RX\_IRQHandler()，其调用HAL库的DMA处理函数HAL\_DMA\_IRQHandler()，该函数调用回调函数HAL\_UART\_RxCpltCallback()或HAL\_UART\_RxHalfCpltCallback()，这两个函数进入真正的中断服务处理函数dma\_isr(struct rt\_serial\_device *)，主体代码如下：
```c
static void dma_isr(struct rt_serial_device *serial)
{
    ......
    /* 如果是DMA-RX中断 */
    if ((__HAL_DMA_GET_IT_SOURCE(&(uart->dma_rx.handle), DMA_IT_TC) != RESET) ||
            (__HAL_DMA_GET_IT_SOURCE(&(uart->dma_rx.handle), DMA_IT_HT) != RESET))
    {
        level = rt_hw_interrupt_disable();
        /* 得到本次接收到的数据量 */
        recv_total_index = serial->config.bufsz - __HAL_DMA_GET_COUNTER(&(uart->dma_rx.handle));
        if (recv_total_index == 0)
        {
            /* 这一句代码，是什么意思？ */
            recv_len = serial->config.bufsz - uart->dma_rx.last_index;
        }
        else
        {
            /* 减去以前接收到的数据量，得到本次接收到的数据数量 */
            recv_len = recv_total_index - uart->dma_rx.last_index;
        }
        /* 更新接收历史数据量 */
        uart->dma_rx.last_index = recv_total_index;
        rt_hw_interrupt_enable(level);

        if (recv_len)
        {
            /* 如果有新数据，调用serial设备模块的通用处理 */
            rt_hw_serial_isr(serial, RT_SERIAL_EVENT_RX_DMADONE | (recv_len << 8));
        }
    }
}
```

在serial模块的函数rt\_hw\_serial\_isr()中，主体代码是：
```c
void rt_hw_serial_isr(struct rt_serial_device *serial, int event)
{
    ......
    case RT_SERIAL_EVENT_RX_DMADONE:
    {
        int length;
        rt_base_t level;

        /* get DMA rx length */
        length = (event & (~0xff)) >> 8;

        if (serial->config.bufsz == 0)
        {
            /* 这个case的处理逻辑不知道怎么应用，看起来STM32实现并没有处理这个case */
            struct rt_serial_rx_dma* rx_dma;

            rx_dma = (struct rt_serial_rx_dma*) serial->serial_rx;
            RT_ASSERT(rx_dma != RT_NULL);

            RT_ASSERT(serial->parent.rx_indicate != RT_NULL);
            serial->parent.rx_indicate(&(serial->parent), length);
            rx_dma->activated = RT_FALSE;
        }
        else
        {
            /* disable interrupt */
            level = rt_hw_interrupt_disable();
            /* update fifo put index, 将数据放入接收缓冲区 */
            rt_dma_recv_update_put_index(serial, length);
            /* calculate received total length, 更新缓冲区信息 */
            length = rt_dma_calc_recved_len(serial);
            /* enable interrupt */
            rt_hw_interrupt_enable(level);
            /* invoke callback, 通知上层，有新数据到达 */
            if (serial->parent.rx_indicate != RT_NULL)
            {
                serial->parent.rx_indicate(&(serial->parent), length);
            }
        }
        break;
    }
    ......
}
```

上层接到通知后，读取函数最终调用驱动读函数rt\_serial\_read()函数，在DMA的条件下，调用\_serial\_dma\_rx()从缓冲区读取数据。其代码为：
```c
static rt_size_t rt_serial_read(struct rt_device *dev, rt_off_t pos, void *buffer, rt_size_t size)
{
    ......
    else if (dev->open_flag & RT_DEVICE_FLAG_DMA_RX)
    {
        return _serial_dma_rx(serial, (rt_uint8_t *)buffer, size);
    }
    ......
}
```

### DMA发送

DMA发送从驱动写函数rt\_serial\_write()开始，在DMA的条件下，调用\_serial\_dma\_tx()，\_serial\_dma\_tx()再调用操作的DMA发送函数发送数据，代码为：
```c
static rt_size_t rt_serial_write(struct rt_device *dev, rt_off_t pos, const void *buffer, rt_size_t size)
{
    ......
    else if (dev->open_flag & RT_DEVICE_FLAG_DMA_TX)
    {
        return _serial_dma_tx(serial, (const rt_uint8_t *)buffer, size);
    }
    ......
}
```
```c
rt_inline int _serial_dma_tx(struct rt_serial_device *serial, const rt_uint8_t *data, int length)
{
    ......
    /* make a DMA transfer */
    serial->ops->dma_transmit(serial, (rt_uint8_t *)data, length, RT_SERIAL_DMA_TX);
    ......
}
```
STM32的dma\_transmit()实现函数是stm32\_dma\_transmit()，其实现就是简单调用HAL\_UART\_Transmit\_DMA()，代码为：
```c
static rt_size_t stm32_dma_transmit(struct rt_serial_device *serial, rt_uint8_t *buf, rt_size_t size, int direction)
{
    ......
    if (HAL_UART_Transmit_DMA(&uart->handle, buf, size) == HAL_OK)
    ......
}
```
实现非常简单。

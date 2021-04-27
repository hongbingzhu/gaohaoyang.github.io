---
layout: post
title:  "VHDL/Verilog里的软件工程"
date:   2020-09-13 9:17
categories: fpga verilog vhdl
tags: verilog vhdl management
---

* content
{:toc}

今年开始接触更改产品的FPGA代码，感觉公司虽然搞了很多年了，但是FPGA这块缺乏一些“软件工程”上的概念导入。

如果对于Altera/Xilinx公司，如果做IP库，可能需要考虑各种编译器的兼容性，不能引入太多的“高级”语法，但是，对于一个公司而言，我认为代码的可维护性是放在第一位的，是在编译器兼容性之类之上的要求。

### 1. VHDL

总体而言，VHDL提供了如下一些语法特性，用于简化代码：

#### 1.1 record和type定义
例如对于KM1024i喷头控制，我们可以定义如下：
```VHDL
	-- 喷头控制信号
	type KM_HEAD_CTRL_TYPE is record
		load	: std_logic;
		lat		: std_logic;
		gsclk	: std_logic;
		stb		: std_logic;
		plstm2	: std_logic;
		plstm1	: std_logic;
	end record;
	type KM_HEADs_CTRL_TYPE is array (natural range <>) of KM_HEAD_CTRL_TYPE;
	-- 喷头数据信号
	type KM_HEAD_DATA_TYPE is record
		sa		: std_logic_vector(2 downto 0);	-- sa(n)=sa_n, 
		dclk	: std_logic;
	end record;
	type KM_HEADs_DATA_TYPE is array (natural range <>) of KM_HEAD_DATA_TYPE;
```
有了上面的定义之后，接口可以为如下的方式：
```VHDL
entity KM1024i_8head_v1 is
	port (
		......

		hd_data			: out KM_HEADs_DATA_TYPE(0 to MAX_HEAD_NUM*CHNL_PER_HEAD-1);
		hd_ctrl			: out KM_HEADs_CTRL_TYPE(0 to MAX_HEAD_NUM*CHNL_PER_HEAD-1);

		......
	);
end KM1024i_8head_v1;
```
接口简化了很多，原先的接口，非常长的一堆信号列表。

#### 1.2 generate语句
我们的FPGA主要在喷头控制上，典型特点就是一块板子上，驱动很多的喷头，典型的是4H和8H，但是甚至有16H的，原先的代码人肉重复，但是在引入generate语句之后，可以大幅度减少代码，甚至可以不同喷头数的板子，代码仅仅需要改几行即可。
```VHDL
PRINT_HEAD_GEN : for i in 0 to 7 generate
	HEAD_OPR : opr_setting
	port map (
		sys_clk		=> sys_clk,
		sys_rst		=> sys_rst,
		reg			=> opr_hd(i),
		ctrl		=> opr_op(i),
		cmd_trig	=> opr_trig(i),
		pon_trig	=> pon_trig
	);

	HEAD_CTRL : head_controller
	port map (
		sys_clk		=> sys_clk,
		sys_rst		=> sys_rst,

		head		=> i_hd_ctrl(i),
		f_fire		=> f_fire,
		gs_level	=> gs_level_hd(i),
		fire_dis	=> fire_dis_any,
		fire_skip	=> data_crc_err and fire_dis_on_err,

		grp_select	=> grp_select_hd(i),
		grp_seg_num	=> grp_seg_num_hd(i),
		grp_seg_desc=> grp_seg_desc_hd(i)
	);
end generate;
```

### 2. Verilog

Verilog语言没有类似于于VHDL的record的定义，也没有C语言的struct。但是，SystemVerilog是支持这些的，所以我们把标准切到SystemVerilog，以支持这些语法特性。以下以Max5183芯片驱动为例：

#### 2.1 struct和enum
```Verilog
// DAC MAX5183 的控制信号
typedef struct {
	logic				clk;
	logic				cs_n;
	logic	[9:0]		data;
	logic 				pwr_down;
	logic				ref_en_n;
	logic				da_en;
} max5183_t;

typedef enum logic[1:0] { ST_OUT_D1, ST_LAT_D1, ST_OUT_D2, ST_LAT_D2 } da_state_t;
```
这是控制信号接口和状态机状态定义。

#### 2.2 generate语句
这个是Verilog支持的，和VHDL的generate类似，例如，某个板子里面用到了6个fifo，可以这样定义：
```Verilog
genvar gi;
generate for (gi = 0; gi < 6; gi = gi+1) begin: fifo_inst_lp
data_fifo data_fifo_inst (
		.clock		(sys_clk				),
		.data		(fifo_inst_data[gi]		),
		.rdreq		(fifo_inst_rd[gi]		),
		.aclr		(fifo_aclr[gi[0]]		),
		.wrreq		(fifo_inst_wr[gi]		),
		.empty		(fifo_inst_empty[gi]	),
		.full		(fifo_inst_full[gi]		),
		.q			(fifo_inst_q[gi]		));
end endgenerate
```

### 3. 总结

虽然说，FPGA代码和一般的软件代码，有截然不同的内涵，但是，毕竟“硬件”已经“软件”化了。引入软件工程的一些概念，对于增加FPGA代码的可维护性和开发效率，是非常有必要，非常有意义的。
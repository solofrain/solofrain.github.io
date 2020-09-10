FPGA时序约束理论
==

Original post: <http://www.technomania.cn/tutorials/fpga/timing-analysis/>{:target="_blank"}

# 周期约束理论

  首先来看什么是时序约束，泛泛来说，就是我们告诉软件（Vivado、ISE等）从哪个pin输入信号，输入信号要延迟多长时间，时钟周期是多少，让软件PAR(Place and  Route)后的电路能够满足我们的要求。因此如果我们不加时序约束，软件是无法得知我们的时钟周期是多少，PAR后的结果是不会提示时序警告的。

  周期约束就是告诉软件我们的时钟周期是多少，让它PAR后要保证在这样的时钟周期内时序不违规。大多数的约束都是周期约束，因为时序约束约的最多是时钟。

  在讲具体的时序约束前，我们先介绍两个概念，在下面的讲解中，会多次用到：

- 发起端/发起寄存器/发起时钟/发起沿：指的是产生数据的源端

- 接收端/接收寄存器/捕获时钟/捕获沿：指的是接收数据的目的端

# 1. 建立/保持时间

  讲时序约束，这两个概念要首先介绍，因为我们做时序约束其实就是为了满足建立/保持时间。对于DFF的输入而言，

- 在clk上升沿到来之前，数据提前一个最小时间量“预先准备好”，这个最小时间量就是建立时间；

- 在clk上升沿来之后，数据必须保持一个最小时间量“不能变化”，这个最小时间量就是保持时间。

![](https://upload-images.jianshu.io/upload_images/16278820-1c69e2d8baefaf07?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

建立和保持时间是由器件特性决定了，当我们决定了使用哪个FPGA，就意味着建立和保持时间也就确定了。Xilinx FPGA的setup time基本都在0.04ns的量级，hold time基本在0.2ns的量级，不同器件会有所差异，具体可以查对应器件的DC and AC Switching Characteristics，下图列出K7系列的建立保持时间。

![](https://upload-images.jianshu.io/upload_images/16278820-908de5f43f4f607f?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 2. 时序路径与时序模型

典型的时序路径有4类，如下图所示，这4类路径可分为片间路径（标记①和标记③)和片内路径（标记②和标记④）。

![](https://upload-images.jianshu.io/upload_images/16278820-2e8c926a2f1f9677.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对于所有的时序路径，我们都要明确其起点和终点，这4类时序路径的起点和终点分别如下表。

(https://upload-images.jianshu.io/upload_images/16278820-fbcaa2206528b129.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


这4类路径中，我们最为关心是②的同步时序路径，也就是FPGA内部的时序逻辑。


典型的时序模型如下图所示，一个完整的时序路径包括源时钟路径、数据路径和目的时钟路径，也可以表示为触发器+组合逻辑+触发器的模型。

![](https://upload-images.jianshu.io/upload_images/16278820-af2e5227287daa1f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

该时序模型的要求为(公式1)

Tclk ≥ $T_{co}$ + $T_{logic}$ + $T_{routing}$ + $T_{setup}$ - $T_{skew}$

其中，$T_{co}$为发端寄存器时钟到输出时间；$T_{logic}$为组合逻辑延迟；$T_{routing}$为两级寄存器之间的布线延迟；$T_{setup}$为收端寄存器建立时间；$T_{skew}$为两级寄存器的时钟歪斜，其值等于时钟同边沿到达两个寄存器时钟端口的时间差；Tclk为系统所能达到的最小时钟周期。

这里我们多说一下这个$T_{skew}$，skew分为两种，positive skew和negative skew，其中positive skew见下图，这相当于增加了后一级寄存器的触发时间。

![](https://upload-images.jianshu.io/upload_images/16278820-e5b8a508b2d2d8bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但对于negative skew，则相当于减少了后一级寄存器的触发时间，如下图所示。

![](https://upload-images.jianshu.io/upload_images/16278820-ba9db370ed91e1c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当系统稳定后，都会是positive skew的状态，但即便是positive skew，综合工具在计算时序时，也不会把多出来的$T_{skew}$算进去。

用下面这个图来表示时序关系就更加容易理解了。为什么要减去$T_{skew}$，下面这个图也更加直观。

![](https://upload-images.jianshu.io/upload_images/16278820-1d9a1e07e54b2b70.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



发送端寄存器产生的数据，数据经过$T_{co}$、$T_{logic}$、$T_{routing}$后到达接收端，同时还要给接收端留出$T_{setup}$的时间。而时钟延迟了$T_{skew}$的时间，因此有：（公式2）

$$
T_{data\_path} + T_{setup} < = T_{skew} + T_{clk}
$$

对于同步设计$T_{skew}$可忽略(认为其值为0)，因为FPGA中的时钟树会尽量保证到每个寄存器的延迟相同。

公式中提到了建立时间，那保持时间在什么地方体现呢？

保持时间比较难理解，它的意思是reg1的输出不能太快到达reg2，这是为了防止采到的新数据太快而冲掉了原来的数据。保持时间约束的是同一个时钟边沿，而不是对下一个时钟边沿的约束。

![](https://upload-images.jianshu.io/upload_images/16278820-331504bc38944473.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



reg2在边沿2时刻刚刚捕获reg1在边沿1时刻发出的数据，若reg1在边沿2时刻发出的数据过快到达reg2，则会冲掉前面的数据。因此保持时间约束的是同一个边沿。

![](https://upload-images.jianshu.io/upload_images/16278820-020bd5ac8920564f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



在时钟沿到达之后，数据要保持Thold的时间，因此，要满足：（公式3）

$$
T_{data\_path} = T_{co} + T_{logic} + T_{routing} ≥ T_{skew} + T_{hold}
$$

这两个公式是FPGA的面试和笔试中经常问到的问题，因为这种问题能反映出应聘者对时序的理解。

在公式1中，$T_{co}$跟$T_{su}$一样，也取决于芯片工艺，因此，一旦芯片型号选定就只能通过$T_{logic}$和$T_{routing}$来改善Tclk。其中，$T_{logic}$和代码风格有很大关系，$T_{routing}$和布局布线的策略有很大关系。


# 3. I/O约束

I/O约束是必须要用的约束，又包括管脚约束和延迟约束。

## 3.1 管脚约束

管脚约束就是指管脚分配，我们要指定管脚的`PACKAGE_PIN`和`IOSTANDARD`两个属性的值,前者指定了管脚的位置,后者指定了管脚对应的电平标准。

  在vivado中，使用如下方式在xdc中对管脚进行约束。

```tcl
set_property -dict {PACKAGE_PIN AJ16  IOSTANDARD  LVCMOS18} [get_ports "led[0]"]
```

  在Vivado规定，必须要指定管脚电平，不然在最后一步生成比特流时会出错。

  除了管脚位置和电平，还有一个大家容易忽略但很容易引起错误的就是端接，当我们使用差分电平时比如LVDS，在在V6中我们使用`IBUFDS`来处理输入的差分信号时，可以指定端接为TRUE。

```verilog
   IBUFDS #(
      .DIFF_TERM("TRUE"),       // Differential Termination
      .IOSTANDARD("DEFAULT")     // Specify the input I/O standard
   ) IBUFDS_inst (
      .O(O),  // Buffer output
      .I(I),  // Diff_p buffer input (connect directly to top-level port)
      .IB(IB) // Diff_n buffer input (connect directly to top-level port)
   );

```

但在Ultrascale中的IBUFDS，却把端接这个选项去掉了

```

   IBUFDS #(
      .DQS_BIAS("FALSE")  // (FALSE, TRUE)
   )
   IBUFDS_inst (
      .O(O),   // 1-bit output: Buffer output
      .I(I),   // 1-bit input: Diff_p buffer input (connect directly to top-level port)
      .IB(IB)  // 1-bit input: Diff_n buffer input (connect directly to top-level port)
   );

```

我们必须要在xdc或I/O Pors界面中，手动指定，否则可能会出错。

![](https://upload-images.jianshu.io/upload_images/16278820-49c9ef204b128947.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


笔者之前就采过一个坑，差分端口输入，当连续输入的数据为`11101111`这种时，中间那个0拉不下来，还是1，同样也会发生在`000010000`，这样就导致数据传输错误，后来才发现是忘记加端接。因为端接会影响信号的实际电平，导致FPGA判断错误。

  当综合完成后，我们可以点击DRC，进行设计规则检查，这一步可以报出一些关键问题，比如时钟端口未分配在时钟引脚上等。

![](https://upload-images.jianshu.io/upload_images/16278820-68aef6ee54f26ae2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 3.2 延迟约束

  延迟约束用的是`set_input_delay`和`set_output_delay`，分别用于input端和output端，其时钟源可以是时钟输入管脚，也可以是虚拟时钟。**但需要注意的是，这个两个约束并不是起延迟的作用**，具体原因下面分析。

- set_input_delay

    这个约束跟ISE中的`OFFSET=IN`功能相同，但设置方式不同。下图所示即为input delay的约束说明图。

![](https://upload-images.jianshu.io/upload_images/16278820-aff989e2cee28a8f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


从图中很容易理解，

$$
T_{inputdelay} = T_{co} + T_D
$$

当满足图中的时序时，最大延迟为2ns，最小延迟为1ns。

因此，需要加的时序约束为：

```
create_clock -name sysclk -period 10 [get_ports clkin]
set_input_delay 2 -max -clock sysclk [get_ports Datain]
set_input_delay 1 -min -clock sysclk [get_ports Datain]
```

*   set_output_delay

  set_output_delay的用法跟set_input_delay十分相似，这里就不再展开讲了。我们上面讲set_input_delay的描述中，大家可以看到，这个约束是告诉vivado我们的输入信号和输入时钟之间的延迟关系，跟下面要讲的时钟周期约束是一个原理，让vivado在这个前提下去Place and Route。**并不是调节输入信号的延迟**，因为身边有不少的FPGA工程师在没用过这个约束指令之前，都以为这是调节延迟的约束。

  如果要调整输入信号的延迟，只能使用IDELAY，在V6中，IDELAY模块有32个tap值，每个tap可延迟78ps，这样总共差不多是2.5ns。


 # 4. 时钟周期约束

时钟周期约束，顾名思义，就是我们对时钟的周期进行约束，这个约束是我们用的最多的约束了，也是最重要的约束。

下面我们讲一些Vivado中时钟约束指令。

## 4.1 create_clock

在Vivado中使用`create_clock`来创建时钟周期约束。使用方法为：

```tcl
create_clock -name <name>                        \
             -period <period>                    \
             -waveform {<rise_time> <fall_time>} \
              [get_ports <input_port>]
```

| 参数 | 含义 |
| --- | --- |
| -name | 时钟名称 |
| -period | 时钟周期，单位为ns |
| -waveform | 波形参数，第一个参数为时钟的第一个上升沿时刻，第二个参数为时钟的第一个下降沿时刻 |
| -add | 在同一时刻源上定义多个时钟时使用 |

这里的时钟必须是主时钟`primary clock`，**主时钟**通常有两种情形:一种是时钟由外部时钟源提供，通过时钟引脚进入FPGA，该时钟引脚绑定的时钟为主时钟:另一种是高速收发器(GT)的时钟RXOUTCLK或TXOUTCLK。对于7系列FPGA，需要对GT的这两个时钟手工约束：对于UltraScale FPGA，只需对GT的输入时钟约束即可，Vivado会自动对这两个时钟约束。

如何确定主时钟是时钟周期约束的关键，除了根据主时钟的两种情形判断之外，还可以借助Tcl脚本判断。

在vivado自带的example project里面，打开CPU(HDL)的工程，如下图所示。

![](https://upload-images.jianshu.io/upload_images/16278820-3236217d3aaae6ea?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

把工程的xdc文件中，`create_clock`的几项都注释掉。这里解释下端口（Port）和管脚（Pin）。get_ports获取的是FPGA的IO端口，get_pins获取的是FPGA内部子模块的Pin，具体的我们在第14讲的Tcl命令中会讲到。

![](https://upload-images.jianshu.io/upload_images/16278820-75890983fa7d633b?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

再`Open Synthesized Design`或者`Open Implementation Design`，并通过以下两种方式查看主时钟。

*   方式一

运行tcl指令`report_clock_networks -name mainclock`，显示结果如下：

![](https://upload-images.jianshu.io/upload_images/16278820-3e1a0d3214609b18?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   方式二

运行tcl指令`check_timing -override_defaults no_clock`，显示结果如下：

![](https://upload-images.jianshu.io/upload_images/16278820-72bce99a65bf2278?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

***Vivado中的tcl命令行相当好用，有很多的功能，大家可以开始习惯用起来了。***

对于高速收发器的时钟，我们也以Vivado中的CPU example工程为例，看下Xilinx官方是怎么约束的。

```
# Define the clocks for the GTX blocks
create_clock -name gt0_txusrclk_i -period 12.8 [get_pins mgtEngine/ROCKETIO_WRAPPER_TILE_i/gt0_ROCKETIO_WRAPPER_TILE_i/gtxe2_i/TXOUTCLK]
create_clock -name gt2_txusrclk_i -period 12.8 [get_pins mgtEngine/ROCKETIO_WRAPPER_TILE_i/gt2_ROCKETIO_WRAPPER_TILE_i/gtxe2_i/TXOUTCLK]
create_clock -name gt4_txusrclk_i -period 12.8 [get_pins mgtEngine/ROCKETIO_WRAPPER_TILE_i/gt4_ROCKETIO_WRAPPER_TILE_i/gtxe2_i/TXOUTCLK]
create_clock -name gt6_txusrclk_i -period 12.8 [get_pins mgtEngine/ROCKETIO_WRAPPER_TILE_i/gt6_ROCKETIO_WRAPPER_TILE_i/gtxe2_i/TXOUTCLK]
```

当系统中有多个主时钟，且这几个主时钟之间存在确定的相位关系时，需要用到`-waveform`参数。如果有两个主时钟，如下图所示。

![](https://upload-images.jianshu.io/upload_images/16278820-b5e2ac61155191fa?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

则时钟约束为：

```
create_clock -name clk0 -period 10.0 -waveform {0 5} [get_ports clk0]create_clock -name clk1 -period 8.0 -waveform {2 8} [get_ports clk1]
```

约束中的数字的单位默认是ns，若不写`wavefrom`参数，则默认是占空比为50%且第一个上升沿出现在0时刻。使用`report_clocks`指令可以查看约束是否生效。还是上面的CPU的例子，把约束都还原到最初的状态。执行`report_clocks`后，如下所示，我们只列出其中几项内容。

```
Clock Report

Clock           Period(ns)  Waveform(ns)    Attributes  Sourcessys
Clk             10.000      {0.000 5.000}   P           {sysClk}
gt0_txusrclk_i  12.800      {0.000 6.400}   P           {mgtEngine/ROCKETIO_WRAPPER_TILE_i/gt0_ROCKETIO_WRAPPER_TILE_i/gtxe2_i/TXOUTCLK}
...

====================================================
Generated Clocks
====================================================

Generated Clock   : clkfbout
Master Source     : clkgen/mmcm_adv_inst/CLKIN1
Master Clock      : sysClk
Multiply By       : 1
Generated Sources : {clkgen/mmcm_adv_inst/CLKFBOUT}

Generated Clock   : cpuClk_4
Master Source     : clkgen/mmcm_adv_inst/CLKIN1
Master Clock      : sysClk
Edges             : {1 2 3}
Edge Shifts(ns)   : {0.000 5.000 10.000}
Generated Sources : {clkgen/mmcm_adv_inst/CLKOUT0}...
```

一般来讲，我们的输入时钟都是差分的，此时我们只对P端进行约束即可。如果同时约束了P端和N端，通过`report_clock_interaction`命令可以看到提示unsafe。这样既会增加内存开销，也会延长编译时间。

## 4.2 create_generated_clock

其使用方法为：

```tcl
create_generated_clock -name <generated_clock_name>              \
                       -source <master_clock_source_pin_or_port> \
                       -multiply_by <mult_factor>                \
                       -divide_by <div_factor>                   \
                       -master_clock <master_clk> <pin_or_port>
```

| 参数 | 含义 |
| --- | --- |
| -name | 时钟名称 |
| -source | 产生该时钟的源时钟 |
| -multiply_by | 源时钟的多少倍频 |
| -divide_by | 源时钟的多少分频 |

从名字就能看出来，这个是约束我们在FPGA内部产生的衍生时钟， 所以参数在中有个`-source`，就是指定这个时钟是从哪里来的，这个时钟叫做`master clock`，是指上级时钟，区别于`primary clock`。
它可以是我们上面讲的primary clock，也可以是其他的衍生时钟。该命令不是设定周期或波形，而是描述时钟电路如何对上级时钟进行转换。这种转换可以是下面的关系：

*   简单的频率分频

*   简单的频率倍频

*   频率倍频与分频的组合，获得一个非整数的比例，通常由MMCM或PLL完成

*   相移或波形反相

*   占空比改变

*   上述所有关系的组合

**衍生时钟**又分两种情况：

1.  Vivado自动推导的衍生时钟

2.  用户自定义的衍生时钟

  首先来看第一种，如果使用PLL或者MMCM，则Vivado会自动推导出一个约束。大家可以打开Vivado中有个叫`wavegen`的工程，在这个工程中，输入时钟经过PLL输出了2个时钟，如下图所示。
（补充：关于DCM/DLL/PLL/MMCM的区别，可参考我写的另一篇文章 [DCM/DLL/PLL/MMCM区别](https://mp.weixin.qq.com/s?__biz=MzU4ODY5ODU5Ng==&mid=2247484106&idx=1&sn=82983a8086732717298436e067a64d4d&chksm=fdd98441caae0d57c99c5b22cf72bfaee2372824406014680be9df1f8d85b3071182fe43656c&mpshare=1&scene=21&srcid=0928ySJ3ud0vfaGS85Teu5Xw&sharer_sharetime=1571051171309&sharer_shareid=296cfe717a7da125d89d5a7bcdf65c18&key=6234e09828e71f223a5bbb62942587523cffdc550c50d6713403e50f0f1a03c87c5b1a6fae054a425e6f27eabfd6e48eb8fd421c5841d8d8b3b054113d8e8650ff4a65e51fa211ebe10dc0a436635167&ascene=1&uin=MzkzMzM2Nzc1&devicetype=Windows)。

（在微信公证号Quant_Times中搜索就能看到）

(https://upload-images.jianshu.io/upload_images/16278820-1256f4c94d23f555?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但在xdc文件中，并未对这2个输出时钟进行约束，只对输入的时钟进行了约束，若我们使用`report_clocks`指令，则会看到：

![](https://upload-images.jianshu.io/upload_images/16278820-bfd110c931d8c7ea?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*注：有三个约束是因为PLL会自动输出一个反馈时钟*

自动推导的好处在于当MMCM/PLL/BUFR的配置改变而影响到输出时钟的频率和相位时，用户无需改写约束，Vivado仍然可以自动推导出正确的频率/相位信息。劣势在于，用户并不清楚自动推导出的衍生钟的名字，当设计层次改变时，衍生钟的名字也有可能改变。但由于该衍生时钟的约束并非我们自定义的，因此可能会没有关注到它名字的改变，当我们使用者这些衍生时钟进行别的约束时，就会出现错误。

解决办法是用户自己手动写出自动推导的衍生时钟的名字，也仅仅写出名字即可，其余的不写。如下所示。

```
create_generated_clock -name <generated_clock_name> \
                       -source <master_clock_source_pin_or_port>
```

这一步很容易会被提示critical warning，其实有个很简单的方法，就是name和source都按照vivado中生成的来。具体我们到后面的例子中会讲到。

## 4.3 set_clock_groups

使用方法为：

```
set_clock_groups -asynchronous -group <clock_name_1> -group <clock_name_2>
set_clock_groups -physically_exclusive  -group <clock_name_1> -group <clock_name_2>
```

这个约束常用的方法有三种，第一种用法是当两个主时钟是异步关系时，使用`asynchronous`来指定。这个在我们平时用的还是比较多的，一般稍微大点的工程，都会出现至少两个主时钟，而且这两个时钟之间并没有任何的相位关系，这时就要指定：

```
create_clock -period 10 -name clk1 [get_ports clk1]
create_clock -period 8 -name clk2 [get_ports clk2]
set_clock_groups -asynchronous -group clk1 -group clk2
```

第二种用法是当我们需要验证同一个时钟端口在不同时钟频率下能否获得时序收敛时使用。比如有两个异步主时钟clk1和clk2，需要验证在clk2频率为100MHz，clk1频率分别为50MHz、100MHz和200MHz下的时序收敛情况，我们就可以这样写。

```
create_clock -name clk1A -period 20.0 [get_ports clk1]
create_clock -name clk1B -period 10.0 [get_ports clk1] -add
create_clock -name clk1C -period 5.0  [get_ports clk1] -add 
create_clock -name clk2 -period 10.0 [get_ports clk2]
set_clock_groups -physically_exclusive -group clk1A -group clk1B -group clk1C
set_clock_groups -asynchronous -group "clk1A clk1B clk1C" -group clk2
```

第三种用法就是当我们使用BUFGMUX时，会有两个输入时钟，但只会有一个时钟被使用。比如MMCM输入100MHz时钟，两个输出分别为50MHz和200MHz，这两个时钟进入了BUFGMUX，如下图所示。

(https://upload-images.jianshu.io/upload_images/16278820-6473998007016a4b?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在这种情况下，我们需要设置的时序约束如下：

```
set_clock_groups -logically_exclusive \
                 -group [get_clocks -of [get_pins inst_mmcm/inst/mmcm_adv_inst/CLKOUT0]] \
                 -group [get_clocks -of [get_pins inst_mmcm/inst/mmcm_adv_inst/CLKOUT1]]
```

## 4.4 创建虚拟时钟

虚拟时钟通常用于设定对输入和输出的延迟约束，这个约束其实是属于IO约束中的延迟约束，之所以放到这里来讲，是因为虚拟时钟的创建，用到了本章节讲的一些理论。虚拟时钟和前面讲的延迟约束的使用场景不太相同。顾名思义，虚拟时钟，就是没有与之绑定的物理管脚。
虚拟时钟主要用于以下三个场景：

*   外部IO的参考时钟并不是设计中的时钟

*   FPGA I/O路径参考时钟来源于内部衍生时钟，但与主时钟的频率关系并不是整数倍

*   针对I/O指定不同的jitter和latency

简而言之，之所以要创建虚拟时钟，对于输入来说，是因为输入到FPGA数据的捕获时钟是FPGA内部产生的，与主时钟频率不同；或者PCB上有Clock Buffer导致时钟延迟不同。对于输出来说，下游器件只接收到FPGA发送过去的数据，并没有随路时钟，用自己内部的时钟去捕获数据。

如下图所示，在FPGA的A和B端口分别有两个输入，其中捕获A端口的时钟是主时钟，而捕获B端口的时钟是MMCM输出的衍生时钟，而且该衍生时钟与主时钟的频率不是整数倍关系。

![](https://upload-images.jianshu.io/upload_images/16278820-e27096bd39653077?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这种情况下时序约束如下：

```
create_clock -name sysclk -period 10 [get_ports clkin]
create_clock -name virclk -period 6.4set_input_delay 2 -clock sysclk [get_ports A]
set_input_delay 2 -clock virclk [get_ports B]
```

可以看到，创建虚拟时钟用的也是`create_clock`约束，但后面并没有加`get_ports`参数，因此被称为虚拟时钟。

再举个输出的例子，我们常用的UART和SPI，当FPGA通过串口向下游器件发送数据时，仅仅发过去了uart_tx这个数据，下游器件通过自己内部的时钟去捕获uart_tx上的数据，这就需要通过虚拟时钟来约束；而当FPGA通过SPI向下游器件发送数据时，会发送sclk/sda/csn三个信号，其中sclk就是sda的随路时钟，下游器件通过sclk去捕获sda的数据，而不是用自己内部的时钟，这是就不需要虚拟时钟，直接使用`set_output_delay`即可。

注意，虚拟时钟必须在约束I/O延迟之前被定义。

## 4.5  最大最小延迟约束

顾名思义，就是设置路径的max/min delay，主要应用场景有两个：

*   输入管脚的信号经过组合逻辑后直接输出到管脚

*   异步电路之间的最大最小延迟

![](https://upload-images.jianshu.io/upload_images/16278820-486a65aa06a4c044?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

设置方式为：

```
set_max_delay <delay> [-datapath_only]        \
                      [-from <node_list>]     \
                      [-to <node_list>]       \
                      [-through <node_list>]
                      
set_min_delay <delay> [-from <node_list>]     \
                      [-to <node_list>]       \
                      [-through <node_list>]
```

| 参数 | 含义 |
| --- | --- |
| -from | 有效的起始节点包含:时钟,input(input)端口,或时序单元(寄存器,RAM)的时钟引脚. |
| -to | 有效的终止节点包含:时钟,output(inout)端口或时序单元的数据端口. |
| -through | 有效的节点包含:引脚,端口,线网. |

max/min delay的约束平时用的相对少一些，因为在跨异步时钟域时，我们往往会设置`asynchronous`或者`false_path`。对于异步时钟，我们一般都会通过设计来保证时序能够收敛，而不是通过时序约束来保证。


# 5. 两种时序例外

## 5.1 多周期路径

上面我们讲的是时钟周期约束，默认按照单周期关系来分析数据路径，即数据的发起沿和捕获沿是最邻近的一对时钟沿。如下图所示。

![](https://upload-images.jianshu.io/upload_images/16278820-721c99ad7301222b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  默认情况下,保持时间的检查是以建立时间的检查为前提,即总是在建立时间的前一个时钟周期确定保持时间检查。这个也不难理解，上面的图中，数据在时刻1的边沿被发起，建立时间的检查是在时刻2进行，而保持时间的检查是在时刻1（如果这里不能理解，再回头看我们讲保持时间章节的内容），因此保持时间的检查是在建立时间检查的前一个时钟沿。

  但在实际的工程中，经常会碰到数据被发起后，由于路径过长或者逻辑延迟过长要经过多个时钟周期才能到达捕获寄存器；又或者在数据发起的几个周期后，后续逻辑才能使用。这时如果按照单周期路径进行时序检查，就会报出时序违规。因此就需要我们这一节所讲的多周期路径了。

多周期约束的语句是：

```
set_multicycle_path <num_cycles> [-setup|-hold]                          \
                                 [-start|-end]                           \
                                 [-from <startpoints>] [-to <endpoints>] \
                                 [-through <pins|cells|nets>]

```

| 参数 | 含义 |
| --- | --- |
| num_cycles [-setup -hold] | 建立/保持时间的周期个数 |
| [-start -end] | 参数时钟选取 |
| -from<startpoint></startpoint> | 发起点 |
| -to | 捕获点 |
| -through <pins/cells/nets> | 经过点 |

对于建立时间，num_cycles是指多周期路径所需的时钟周期个数；对于保持时间，num_cycles是指相对于默认的捕获沿，实际捕获沿应回调的周期个数。

发起沿和捕获沿可能是同一个时钟，也可能是两个时钟，参数`start`和`end`就是选择参考时钟是发送端还是接收端。

*   start表示参考时钟为发送端（发端）所用时钟，对于保持时间的分析，若后面没有指定`start`或`end`，则默认为为-start；
*   end表示参考时钟为捕获端（收端）所用时钟,对于建立时间的分析，若后面没有指定`start`或`end`，则默认为为-end；

上面这两句话也不难理解，因为setup-time是在下一个时钟沿进行捕获时的约束，因此默认是对接收端的约束；而hold-up-time是对同一个时钟沿的约束，目的是发送端不能太快，是对发送端的约束。

  对于单周期路径来说，setup的num_cycles为1，hold的num_cycles为0.

  多周期路径要分以下几种情况进行分析：

### 5.1.1 单时钟域

  即发起时钟和捕获时钟是同一个时钟，其多周期路径模型如下图所示。

![](https://upload-images.jianshu.io/upload_images/16278820-e4aa18e997be8a86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



  单时钟域的多周期路径常见于带有使能的电路中，我们以双时钟周期路径为例，其实现电路如下：

![](https://upload-images.jianshu.io/upload_images/16278820-33f15060672ffd3b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



  若我们没有指定任何的约束，默认的建立/保持时间的分析就像我们上面所讲的单周期路径，如下图所示。

![](https://upload-images.jianshu.io/upload_images/16278820-ff54456a182588e3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



  但由于我们的的数据经过了两个时钟周期才被捕获，因此建立时间的分析时需要再延迟一个周期的时间。

采用如下的时序约束：

```
set_multicycle_path 2 -setup -from [get_pins data0_reg/C] -to [get_pins data1_reg/D]
```

在建立时间被修改后，保持时间也会自动调整到捕获时钟沿的前一个时钟沿，如下图所示。

![](https://upload-images.jianshu.io/upload_images/16278820-46f2d34355d63b70.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



很明显，这个保持时间检查是不对的，因为保持时间的检查针对的是同一个时钟沿，因此我们要把保持时间往回调一个周期，需要再增加一句约束：

```
set_multicycle_path 1 -hold -end -from [get_pins data0_reg/C]  -to [get_pins data1_reg/D]
```

这里加上`-end`参数是因为我们要把捕获时钟沿往前移，因此针对的是接收端，但由于我们这边讲的是单时钟域，发送端和接收端的时钟是同一个，因此`-end`可以省略。这样，完整的时序约束如下：

```
set_multicycle_path 2 -setup -from [get_pins data0_reg/C] -to [get_pins data1_reg/D]
set_multicycle_path 1 -hold  -from [get_pins data0_reg/C]  -to [get_pins data1_reg/D]
```

约束完成后，建立保持时间检查如下图所示。

![](https://upload-images.jianshu.io/upload_images/16278820-7bd6a946c67287c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


在单时钟域下，若数据经过N个周期到达，则约束示例如下：

```
set_multicycle_path N -setup -from [get_pins data0_reg/C] -to [get_pins data1_reg/D]
set_multicycle_path N-1 -hold  -from [get_pins data0_reg/C]  -to [get_pins data1_reg/D]
```

### 5.1.2 时钟相移

![](https://upload-images.jianshu.io/upload_images/16278820-6f0013be2500d73a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



  前面我们讨论的是在单时钟域下，发送端和接收端时钟是同频同相的，如果两个时钟同频不同相怎么处理？

![](https://upload-images.jianshu.io/upload_images/16278820-2f466eb4f6160a07.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



  如上图所示，时钟周期为4ns，接收端的时钟沿比发送端晚了0.3ns，若不进行约束，建立时间只有0.3ns，时序基本不可能收敛；而保持时间则为3.7ns，过于丰富。可能有的同学对保持时间会有疑惑，3.7ns是怎么来的？还记得我们上面讲的保持时间的定义么，在0ns时刻，接收端捕获到发送的数据后，要再过3.7ns的时间发送端才会发出下一个数据，因此本次捕获的数据最短可持续3.7ns，即保持时间为3.7ns。

  因此，在这种情况下，我们应把捕获沿向后移一个周期，约束如下：

```
set_multicycle_path 2 -setup -from [get_clocks CLK1] -to [get_clocks CLK2]
```

对setup约束后，hold会自动向后移动一个周期，此时的建立保持时间检查如下：

![](https://upload-images.jianshu.io/upload_images/16278820-108e512989d63d21.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

那如果接收端的时钟比发送端的时钟超前了怎么处理？

![](https://upload-images.jianshu.io/upload_images/16278820-1ca3b03349bf1ddf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


同样的，时钟周期为4ns，但接收端时钟超前了0.3ns，从图中可以看出，此时setup是3.7ns，而保持时间是0.3ns。这两个时间基本已经满足了Xilinx器件的要求，因此无需进行约束。

### 5.1.3 慢时钟到快时钟的多周期

  当发起时钟慢于捕获时钟时，我们应该如何处理？

![](https://upload-images.jianshu.io/upload_images/16278820-28565d3cd9e25613.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



  假设捕获时钟频率是发起时钟频率的3倍，在没有任何约束的情况下，Vivado默认会按照如下图所示的建立保持时间进行分析。

![](https://upload-images.jianshu.io/upload_images/16278820-1882f28f30e8177b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



  但我们可以通过约束让建立时间的要求更容易满足，即

```
set_multicycle_path 3 -setup -from [get_clocks CLK1] -to [get_clocks CLK2]
```

跟上面讲的一样，设置了setup，hold会自动变化，但我们不希望hold变化，因此再增加：

```
set_multicycle_path 2 -hold -end -from [get_clocks CLK1] -to [get_clocks CLK2]
```

这里由于发起和捕获是两个时钟，因此`-end`参数是不可省的。加上时序约束后，Vivado会按照下面的方式进行时序分析。

![](https://upload-images.jianshu.io/upload_images/16278820-d30980946cd3b108.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



### 5.1.4 快时钟到慢时钟的多周期

  当发起时钟快于捕获时钟时，我们应该如何处理？

![](https://upload-images.jianshu.io/upload_images/16278820-910fbb2c401f3193.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



  假设发起时钟频率是捕获时钟频率的3倍，在没有任何约束的情况下，Vivado默认会按照如下图所示的建立保持时间进行分析。

![](https://upload-images.jianshu.io/upload_images/16278820-9b6119026f6a4d60.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



  同理，我们可以通过约束，让时序条件更加宽裕。

```
set_multicycle_path 3 -setup -start -from [get_clocks CLK1] -to [get_clocks CLK2]
set_multicycle_path 2 -hold -from [get_clocks CLK1] -to [get_clocks CLK2]
```

这里的hold约束中没有加`-end`参数，这样的话默认就是`-start`，是因为我们把发起时钟回调2个周期，如下图所示。

![](https://upload-images.jianshu.io/upload_images/16278820-4e70e7b121bd8800.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


针对上面讲的几种多周期路径，总结如下：

![](https://upload-images.jianshu.io/upload_images/16278820-f19539fc3cdd783b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 5.2 伪路径

  什么是伪路径？伪路径指的是该路径存在，但该路径的电路功能不会发生或者无须时序约束。如果路径上的电路不会发生，那Vivado综合后会自动优化掉，因此我们无需考虑这种情况。

  为什么要创建伪路径？创建伪路径可以减少工具运行优化时间,增强实现结果,避免在不需要进行时序约束的地方花较多时间而忽略了真正需要进行优化的地方。

  伪路径一般用于：

- 跨时钟域
- 一上电就被写入数据的寄存器
- 异步复位或测试逻辑
- 异步双端口RAM

  可以看出，伪路径主要就是用在异步时钟的处理上，我们上一节讲的多周期路径中，也存在跨时钟域的情况的，但上面我们讲的是两个同步的时钟域。

伪路径的约束为：

```
set_false_path [-setup] [-hold]                      \
               [-from <node_list>] [-to <node_list>] \
               [-through <node_list>]
```

- `-from`的节点应是有效的起始点。有效的起始点包含时钟对象，时序单元的clock引脚，或者input(or inout)原语；

- `-to`的节点应包含有效的终结点。一个有效的终结点包含时钟对象，output(or inout)原语端口，或者时序功能单元的数据输入端口；

- `-through`的节点应包括引脚、端口、或线网。当单独使用`-through`时，应注意所有路径中包含`-through`节点的路径都将被时序分析工具所忽略。

需要注意的是，`-through`是有先后顺序的，下面的两个约束是不同的约束：

```
set_false_path -through cell1/pin1 -through cell2/pin2
set_false_path -through cell2/pin2 -through cell1/pin1
```

因为它们经过的先后顺序不同，伪路径的约束是单向的，并非双向的，若两个时钟域相互之间都有数据传输，则应采用如下约束：

```
set_false_path -from [get_clocks clk1] -to [get_clocks clk2]
set_false_path -from [get_clocks clk2] -to [get_clocks clk1]
```

也可以直接采用如下的方式，与上述两行约束等效：

```
set_clock_groups -async -group [get_clocks clk1] -to [get_clocks clk2]
```

还有一些其他的约束，比如`case analysis`、`disabling timing`和`bus_skew`等，由于平时用的比较少，这里就不讲了。

# 6. xdc约束优先级

  在xdc文件中，按约束的先后顺序依次被执行，因此，针对同一个时钟的不同约束，只有最后一条约束生效。

  虽然执行顺序是从前到后，但优先级却不同；就像四则运算一样，+-x÷都是按照从左到右的顺序执行，但x÷的优先级比+-要高。

时序例外的优先级从高到低为：

> 1. Clock Groups (**set_clock_groups**)
> 2. False Path (**set_false_path**)
> 3. Maximum Delay Path (**set_max_delay**) and Minimum Delay Path (**set_min_delay**)
> 4. Multicycle Paths (**set_multicycle_path**)

`set_bus_skew`约束并不影响上述优先级且不与上述约束冲突。原因在于`set_bus_skew`并不是某条路径上的约束，而是路径与路径之间的约束。

对于同样的约束，定义的越精细，优先级越高。各对象的约束优先级从高到低为：

> 1. ports --> pins --> cells
> 2. clocks

路径声明的优先级从高到低为：

> 1. -from -through -to
> 2. -from -to
> 3. -from -through
> 4. -from
> 5. -through -to
> 6. -to
> 7. -through

优先考虑对象，再考虑路径。

### Example1：

```
set_max_delay 12 -from [get_clocks clk1] -to [get_clocks clk2]
set_max_delay 15 -from [get_clocks clk1]
```

该约束中，第一条约束会覆盖第二条约束。

### Example2：

```
set_max_delay 12 -from [get_cells inst0] -to [get_cells inst1]
set_max_delay 15 -from [get_clocks clk1] -through [get_pins hier0/p0] -to [get_cells inst1]
```

该约束中，第一条约束会覆盖第二条约束。

### Example3：

```
set_max_delay 4 -through [get_pins inst0/I0]
set_max_delay 5 -through [get_pins inst0/I0] -through [get_pins inst1/I3]
```

这个约束中，两条都会存在，这也使得时序收敛的难度更大，因为这两条语句合并成了：

```
set_max_delay 4 -through [get_pins inst0/I0] -through [get_pins inst1/I3]
```



# 参考
http://www.technomania.cn/tutorials/fpga/timing-analysis/
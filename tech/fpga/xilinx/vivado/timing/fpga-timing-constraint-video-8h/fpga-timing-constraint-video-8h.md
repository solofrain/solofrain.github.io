八小时超长视频教你掌握FPGA时序约束
==

Original post: <https://my.oschina.net/u/4583591/blog/4455472>{:target="_blank"}

> 完整视频链接：
链接: <https://pan.baidu.com/s/1AXvJKXzFjahE5ZLa4pE_8w >{:target="_blank"}  
提取码: y6ud 

# 1. 时序分析基础知识

## 1.1 什么是时序分析？

时序分析本质上就是一种时序检查，目的是检查设计中所有的D触发器是否能够正常工作，也就是检查D触发器的同步端口（数据输入端口）的变化是否满足建立时间要求（Setup）和保持时间要求（Hold）；检查D触发器的异步端口（异步复位端口）的变化是否满足恢复时间要求（Recovery）和移除时间要求（Removal）。

[^_^]:
    ![](https://upload-images.jianshu.io/upload_images/16278820-f261f3a949a50e1a.png)

![](1.png)

## 1.2 时序分析有哪些类型？

时序分析包括静态时序分析（STA）和动态时序分析。

- 动态时序分析：将布局布线生成的布线延迟信息反标注到门级网表中进行仿真，检查是否存在时序违例。此时的仿真包括门延迟和布线延迟信息，能够较好反应芯片的实际工作情况。因为不可能产生完备的测试向量，覆盖门级网表中的每一条路径。因此在动态时序分析中，无法暴露一些路径上可能存在的时序问题。

- 静态时序分析：采用穷尽分析方法来提取出整个电路存在的所有时序路径，计算信号在这些路径上的传播延时，检查信号的建立和保持时间是否满足时序要求，通过对最大路径延时和最小路径延时的分析，找出违背时序约束的错误。它不需要输入向量就能穷尽所有的路径，且运行速度很快、占用内存较少，不仅可以对芯片设计进行全面的时序功能检查，而且还可利用时序分析的结果来优化设计，因此静态时序分析已经越来越多地被用到数字集成电路设计的验证中。

## 1.3 时序分析使用的工具有哪些？

静态时序分析使用的工具：

- Xilinx Vivado Intergreated Design Environment Time Engine
- Altera Quartus II Timequest
- DC等

动态时序分析使用的工具：

- Mentor Modelsim
- Cadence NC-Verilog
- Sysnopsys  VCS

## 1.4 静态时序分析之前的准备工作

撰写基本的时序约束文件，告知时序引擎一些必要的信息（比如时钟，输入输出延时等）。若没有正确的时序约束，那么时序分析的结果是没有意义的。

![](2.png)
![D触发器中Tsetup，Thold，Tco的由来](3.png)

(https://upload-images.jianshu.io/upload_images/16278820-f4db83c260dc29ea.png)

## 1.5 时序分析中的常用术语

- 源时钟（Source Clock/Launch Clock，也称为发起时钟）
- 目的时钟（Destination Clock/Capture Clock，也称为捕获时钟）
- 发起沿（launch edge,源时钟产生数据的有效时钟沿）
- 捕获沿（capture edge,目的时钟捕获数据的有效时钟沿）发起沿通常在0ns，捕获沿通常在下一个发起沿，发起沿和捕获沿通常相差一个时钟周期。

![](4.png)

[^_^]:
    ![](https://upload-images.jianshu.io/upload_images/16278820-0e087e4f622890be.png)

## 1.6 一条普通时序路径的三要素

- 源时钟路径：从源时钟的源节点（通常是FPGA的时钟输入引脚）到源寄存器的时钟端口的路径。当时序路径的起点是FPGA输入端口时，该时序路径是没有源时钟路径的。

- 数据路径：从时序路径的起点到时序路径的终点之间的数据传播路径。时序路径的起点可以是源寄存器的时钟端口或FPGA的输入端口，时序路径的终点可以是目的寄存器的输入端口或FPGA的输出端口。

- 目的时钟路径：从目的时钟的源节点（通常是FPGA的时钟输入引脚）到目的寄存器的时钟端口的路径。当时序路径的终点是FPGA的输出端口时，该时序路径是没有目的时钟路径的。

![](5.png)

[^_^]:
    (https://upload-images.jianshu.io/upload_images/16278820-a2e757ab88ff5e0e.png)

## 1.7 FPGA中常见的四种时序路径

![](6.png)

[^_^]:
    (https://upload-images.jianshu.io/upload_images/16278820-7e62b58cd7ca85e4.png)

**第一种，从FPGA的输入端口到目的寄存器的数据输入端口 。**

![](7.png)

[^_^]:
    (https://upload-images.jianshu.io/upload_images/16278820-dd8c68716e89e702.png)

- 数据由Board clock发起并在FPGA外部产生；
- 数据经过Input Delay的延迟后到达FPGA的输入端口；
- 数据经过FPGA的Internal Delay后到达由目的时钟驱动的目的寄存器。这种路径是没有源时钟路径的，用户需要约束`Input Delay`和`时钟`来告知时序引擎必要信息，时序引擎才能正确的分析这种路径。

**第二种，从源寄存器的时钟端口到目的寄存器的数据输入端口。**

![](8.png)

[^_^]:
    (https://upload-images.jianshu.io/upload_images/16278820-1324828c4e3dc953.png)

- 数据由源时钟发起并在FPGA内部产生；
- 数据经过Data Path Delay后到达由目的时钟驱动的目的寄存器。这种时序路径是最常见的，用户需要约束`源时钟`和`目的时钟`告知时序引擎必要的信息，时序引擎才能正确的分析这种时序路径。

**第三种，从源寄存器的时钟端口到FPGA的输出端口。**

![](9.png)

[^_^]:
    https://upload-images.jianshu.io/upload_images/16278820-0e8cca2a51a72a2f.png)

- 数据由源时钟发起并在FPGA内部产生；
- 数据经过`Internal Delay`后到达输出端口；
- 数据经过`Output Delay`后被`Board Clock`捕获到。这种路径是没有目的时钟路径的，用户需要约束`Output Delay`和`时钟`来告知时序引擎必要信息，时序引擎才能正确的分析这种路径。

**第四种，从FPGA的输入端口到FPGA的输出端口。**

![](10.png)

[^_^]:
    (https://upload-images.jianshu.io/upload_images/16278820-246df11ad75d3597.png)

- 数据横穿FPGA，没有经过任何触发器，这种路径也叫`in-to-out path`，约束输入和输出延时的参考时钟可以使用虚拟时钟。这种路径中只有数据路径，用户需要约束`Input Delay`和`Output Delay`，告知时序引擎必要的信息，时序引擎才能正确的分析这种时序路径。


# 2. Setup检查

## 2.1 时序引擎如何进行Setup检查？

- 确定建立时间要求（确定用于建立时间分析的发起沿和捕获沿）

  
    时序引擎会找出发起时钟和捕获时钟的最小公共周期，然后在最小公共周期内找到所有发起时钟沿和捕获时钟沿的所有可能的情况，并在所有可能的情况中挑选出最小的建立时间需求（大于0），从而确定了Launch edge和Capture edge。

![](11.png)

(https://upload-images.jianshu.io/upload_images/16278820-7e83f629b9f497bc.png)

在上图中，有建立时间要求有两个：`Setup(1)`和`Setup(2)`。取更小的`Setup(2)`，然后分别取对应的`Launch edge`和`Capture edge`对应的时间为`Source Clock Launch Edge Time`和`Destination Clock Capture Edge Time`：

    Source Clock Launch Edge Time: 0ns + 1*T(clk0) = 6ns
    Destination Clock Capture Edge Time: 0ns + 2*T(clk1) = 8ns
    Setup Path Requirement = capture edge time - launch edge time = 2ns

- 计算数据的需求时间

$$
Data\_Required\_Time (setup) = capture\_edge\_time + destination\_clock\_path\_delay - clock\_uncertainty - setup\_time
$$

[^_^]:
    $$
Data\ Required\ Time (setup) &= capture\ edge\ time + destination\ clock\ path\ delay - clock\ uncertainty - setup\ time
$$

[^_^]:
    $$
\begin{equation}
\begin{aligned}
Data\ Required\ Time (setup) &= capture\ edge\ time \\
&+ destination\ clock\ path\ delay \\
&- clock\ uncertainty \\
&- setup\ time
\end{aligned}
\end{equation}
$$

[^_^]:
    ![](https://upload-images.jianshu.io/upload_images/16278820-9c3d54b58de5b396.png)

- 计算数据的到达时间

$$
Data\_Arrival\_Time (setup) = launch\_edge\_time + source\_clock\_path\_delay + datapath\_delay
$$

[^_^]:
    $$
\begin{equation}
\begin{aligned}
Data\ Arrival\ Time (setup) &= launch\ edge\ time \\
&+ source\ clock\ path\ delay \\
&+ datapath\ delay
\end{aligned}
\end{equation}
$$

[^_^]:
    ![](https://upload-images.jianshu.io/upload_images/16278820-b6d4cbafbc49e4dd.png)

- 计算Setup的裕量（Slack）

$$
Slack\ (setup) = Data\_Required\_Time - Data\_Arrival\_Time
$$


> 对于单个时钟而言：$T^{FF1}_{clk\_delay}+T^{FF1}_{co}+T_{setup}<=T^{FF2}_{clk\_delay}+T_{period}$

## 2.2 Setup分析实例

Data Arrival time = lauch_edge + Tclka + Tco + Tdata(Tlogic+Tnet)  

![](https://upload-images.jianshu.io/upload_images/16278820-9c193a90549330ef.png)

Data Require Time = capture edge + Tclkb - Tsu

![](https://upload-images.jianshu.io/upload_images/16278820-899b2da0dcf73489.png)

Setup Slack= Data Require Time  - Data Arrival Time

![](https://upload-images.jianshu.io/upload_images/16278820-0ca4bffb83f26321.png)

## 2.3 根据公式分析哪些因素会导致Setup Slack为负呢？

Setup Slack = （Capture edge – Launch edge）+ （destination clk delay – source clk delay）- Setup time - clk uncertainty – datapath delay 

Setup Slack =  Setup Requirement（一定大于0） + clk skew – Tsu – Tclk uncertainty – Tlogic – Tnet - Tco

**① Setup Requirement 与实际情况不符**

建立时间需求过小，这种情况通常会在同步跨时钟域路径中出现，在同步跨时钟域路径中的源时钟频率与目的时钟频率的相位关系虽然是已知的，但是时序引擎默认选择的捕获沿通常都是错误的，需要用户通过多周期路径约束的方式手动修正建立时间需求。比如下图中，两个同频不同相的同步时钟，时序引擎默认选择的捕获沿是目的时钟第二个上升沿，导致建立时间需求非常小，最终肯定会导致时序违例。

![](https://upload-images.jianshu.io/upload_images/16278820-f31384fbf08a18cf.png)

**② clk skew为负值，且很大**

通常情况下，同一个时钟下的时钟歪斜不应该超过300ps，同步跨时钟域路径的时钟歪斜不应该超过500ps，异步跨时钟域路径的时钟歪斜一般比较大，因为它们的时钟源不同。当出现时钟歪斜大的情况时:

- 检查源时钟路径和目的时钟路径上是否干净，时钟路径上是否引入了组合逻辑，时钟路径是否使用了过多的BUFGCE，时钟路径上是否级联了多个BUFGCE导致时钟延时变大
- 检查源时钟路径和目的时钟路径所经过的模块是否相同，比如源时钟路径上经过全局缓冲，PMMCM，但是目的时钟路径上只经过了全局缓冲。如下图所示，第一条路径的源时钟路径上有BUFGCE/MMCM/BUFGCE，而目的时钟路径上只有BUFGCE，所以源时钟路径和目的时钟路径的延时不同，导致时钟歪斜较大，应该尽量避免此类路径。第二条路径的源时钟和目的时钟都是来源于MMCM的不同的时钟，所以时钟歪斜较小。

![](https://upload-images.jianshu.io/upload_images/16278820-61fa0cf5f4cc00a2.png)

**③ $T_{su}$/$T_{co}$大**

当设计中使用Block（DSP/Block RAM等）时，应该要注意以下问题。对于以这些Block为时序路径的起点或终点的时序路径，这些Block的Tsu/Th/Tco都比普通的寄存器大，而且这些Block的布线延时和时钟歪斜比较大。所以当使用这些Block作为时序路径的终点时，它的起点一定要是触发器，比如说一个Block RAM的写数据信号，输入进Block前最好打一拍。当使用这些Block作为时序路径的起点时，应该使用Block 内部的输出寄存器，比如使用由Block RAM组成的FIFO时，尽量不要使用首字置出的，而使用打一拍后输出的，使用后者可以显著降低Tco。当时序路径为从一个Block到另一个Block时，中间需要进行打拍操作。当使用这些Block的控制端口时，应该保证这些控制信号的低扇出，如使用由Block RAM组成的FIFO时，应该尽量降低读/写能信/地址信号的扇出。

![](https://upload-images.jianshu.io/upload_images/16278820-6c1ef4f7de52515a.png)

**④ $T_{logic}$大**

一般情况下，逻辑延时与时序路径的逻辑层级数息息相关，逻辑层级是指时序路径的起点和终点之间组合逻辑单元（LUT）的个数，而逻辑层级多一级意味着多1个LUT的延时加1条连接LUT的网线延时。通常一级逻辑层级的延时标准是1个LUT加1根网线的总延迟为0.5ns，如果某条路径的逻辑级数大于时钟周期/0.5ns，那么这条路径就被称为长路径。

常用的处理长路径的方案有两种：

- 修改rtl代码，在长路径的逻辑中插入流水线，将长路径打破分为多条短路径；

- retiming优化方式，retiming实际上是寄存器重定向，原理是当某条长路径的相邻路径的建立时间裕量较大，那么它可以调整中间寄存器的位置，来调整布线延迟，通过适当增加相邻路径的布线延迟而减少长路径的布线延迟，使得那些时序违例较小的长路径通过这种微调实现时序收敛。需要强调的是，这种方式优化的力度非常有限，它只适合时序违例较小的长路径，对于一些延时特别大的长路径而言，也是无力回天。

![](https://upload-images.jianshu.io/upload_images/16278820-5df06563e25d1bc6.png)

**⑤Tnet大**

一般情况下，布线延迟与设计整体或局部模块的资源利用率以及拥塞程度息息相关。在正常情况下，一条网线的延时小于1ns，在发生拥塞的区域，网线的延时可能达到若干ns，导致布线延时显著增加。为了解决布线延迟大，需要从降低资源利用率和降低拥塞程度下手，比如某个模块使用了大量的寄存器堆，占用了大量的资源，此时应该考虑使用Block RAM代替这些寄存器堆；某个模块使用了大量的数据选择器，此时应该考虑如何优化这些数据选择器；某个模块的控制信号扇出比较大，与其他模块的互联很重，此时应该考虑如何降低这些信号的扇出；某条时序路径的起点或终点是Block，由于Block的位置比较固定，所以Block的布线延迟会大一些。最后需要强调的是，一定要额外关注高扇出的网线也会对布线延时产生影响。🔺TimeQuest时序分析（Setup）[图片上传失败...(image-23b6a0-1596829289696)]

# 3. Holdup检查

## 3.1 时序引擎如何进行Holdup检查？

**① 确定保持时间要求（确定发起时钟沿和捕获时钟沿）**

保持时间要求是以建立时间要求为基础的，保持时间要求有两种： 

- 当前建立时间的发起沿产生的数据不能被当前建立时间的捕获沿的前一个有效沿捕获

- 当前建立时间发起沿的下一个有效沿产生的数据不能被当前建立时间的捕获沿捕获

![](https://upload-images.jianshu.io/upload_images/16278820-503757e500d8b3bb.png)

根据所有的建立时间需求找到所有的保持时间需求，并从保持时间需求（可正可负）中找到最大的保持时间需求。

![](https://upload-images.jianshu.io/upload_images/16278820-a92e8938a8f3ad4c.png)

**② 计算数据的需求时间**

![](https://upload-images.jianshu.io/upload_images/16278820-ee12cd231c1fd766.png)

**③ 计算数据的到达时间**

![](https://upload-images.jianshu.io/upload_images/16278820-707c865d2a2588e7.png)

**④ 计算Hold up的裕量（slack）**

![](https://upload-images.jianshu.io/upload_images/16278820-a787282d8af61140.png)

## 3.2 Holdup分析实例

 Data Arrival time（new data） = lauch edge + Tclka + Tco + Tdata(Tlogic+Tnet) 

 Data Require time = capture edge + Tclkb + Th 

Hold up slack = Data Arrival time - Data Require time

![](https://upload-images.jianshu.io/upload_images/16278820-bdce9bb9dc3b60d8.png)

## 3.3 根据公式分析哪些因素会导致Holdup Slack为负呢？

 Holdup Slack = （lauch edge - capture edge） + （Tclka – Tclkb） + Tco + Tdata(Tlogic+Tnet) -Th 

 Holdup Slack =  Tco + Tdata(Tlogic+Tnet) -Th - Holdup Requirement - clk skew  

Hold up Slack为负的情况比较少见，当Setup Slack有较大裕量时，通常工具会自动插入延时来增加Hold up Slack。

**① 保持时间需求大于0（通常由时序引擎选择错误的捕获沿导致）**

**② 时钟歪斜大于300ps（通常由时钟路径上的组合逻辑导致） **

**③ Th过大（通常由时序路径终点为Block导致）**

![TimeQuest时序分析（Holdup slack）](https://upload-images.jianshu.io/upload_images/16278820-341337fbcaa1f6b4.png)


# 4. Recovery检查

## 4.1 时序引擎如何进行Recovery检查？

![](https://upload-images.jianshu.io/upload_images/16278820-e626431c36e5d4e3.png)

![TimeQuest时序分析（Recovery） ](https://upload-images.jianshu.io/upload_images/16278820-fb01e42822c90d44.png)


## 4.2 时序引擎如何进行Removal检查？

![](https://upload-images.jianshu.io/upload_images/16278820-061bd35725088d42.png)

![TimeQuest时序分析（Removal）](https://upload-images.jianshu.io/upload_images/16278820-7c5158c7811a978e.png)

# 5. 时序约束

时序引擎能够正确分析4种时序路径的前提是，用户已经进行了正确的时序约束。时序约束本质上就是告知时序引擎一些进行时序分析所必要的信息，这些信息只能由用户主动告知，时序引擎对有些信息可以自动推断，但是推断得到的信息不一定正确。

- 第一种路径需要约束Input_delay；
- 第二种路径需要约束时钟；
- 第三种路径需要约束output_delay；
- 第四种路径需要约束Max_delay/Min_delay；

![](https://upload-images.jianshu.io/upload_images/16278820-37d52bafb1b5a729.png)

# 6. 时钟约束

## 6.1 时钟约束类型

首先用户必须要正确的约束时钟，时序引擎才能根据时钟信息进行各种时序检查。

用户约束时钟时，一般有两种类型的时钟需要约束。 

- 主时钟（Primary Clock）
主时钟（Primary Clock）有两种类型：第一种是从FPGA的全局时钟输入引脚输入的时钟；第二种是从高速收发器输出给用户的恢复时钟

- 生成时钟（Generated Clock）
生成时钟（Generated Clock）有两种类型：第一种是由FPGA的专用时钟管理模块（PLL/MMCM）产生的时钟（这种时钟可以由时序引擎自动推断出来）；第二种是由用户通过LUT或寄存器产生的时钟（这种时钟必须由用户手动约束）。

**① 主时钟（Primary Clock）约束**

使用Create_clock进行时序约束

- 第一种是从FPGA的全局时钟输入引脚输入的时钟：全局时钟输入引脚是sysclk，时钟周期10ns，占空比50%，相移0度。

![](https://upload-images.jianshu.io/upload_images/16278820-4bada57e33a0b303.png)

全局时钟输入引脚是ClkIn，时钟周期10ns，占空比25%，相移90度。

![](https://upload-images.jianshu.io/upload_images/16278820-078a8f43c7d989f6.png)

- 第二种是从高速收发器输出给用户的恢复时钟 全局时钟输入引脚是sysclk，时钟周期3.3ns，占空比50%，相移0度。

![](https://upload-images.jianshu.io/upload_images/16278820-55f1e9fe2a38485f.png)

**② 生成时钟（Generated Clock）约束**

用Create_generated_clock进行时序约束 每个生成时钟都会对应一个时钟源（Master_clk），这个时钟源可以是Primary Clock或者另一个Generated Clock。在约束生成时钟时，用户不需要描述生成时钟的周期和波形，只需要描述由Master_clk经过了怎样的变化而产生的生成时钟即可。比如经过分频（-devide_by），倍频(-multiply_by)，反相（-invert），相移（-edge_shift）等等操作。

![](https://upload-images.jianshu.io/upload_images/16278820-806f3504bf8fe5b4.png)

当生成时钟需要进行相移时，使用-edge_shift选项。-edge_shift不能与-divide_by/-multipl_by/-invert同时使用 。

![](https://upload-images.jianshu.io/upload_images/16278820-d61d0e48be555354.png)

时序引擎默认情况下会分析所有时钟之间的时序路径，用户可以通过时钟分组（ set_clock_group）命令或伪路径（set_false_path）命来关闭一部分路径的时序分析。

- 使用set_clock_group命令，时序引擎会将不同分组的时钟之间的路径分析关闭，相同分组的时钟之间的路径仍然存在。

- 使用set_false_path命令，时序引擎会忽略两个时钟的双向路径，被忽略的路径不一定能够正常工作，需要用户确保该路径使用了两级触发器同步或按照异步数据传输方式传输数据。用户负责保证这些被忽略的路径可以正常工作。

## 6.2 两个时钟的关系

**① 同步时钟（synchronous clock）**

两个时钟之间的相对相位关系是固定的（两个时钟来源于同一个Primary clock），并且这两个时钟的频率的最小公共周期是个整数。比如一个生成时钟（200M）和该生成时钟的Master_clk（100M）之间就属于同步时钟关系，因为这两个时钟的相位关系肯定是确定的，并且可以找到两个时钟的最小公共周期。通常情况下，一个Primary Clock和它产生的生成时钟之间都属于同步时钟关系，除非找不到最小公共周期。 属于同步时钟关系的两个时钟之间的路径是可以进行时序分析的。

**② 异步时钟（ asynchronous clock ）**

两个时钟之间的相对相位关系不确定。比如FPGA上两个晶振分别产生两个Primary clock（相对相位关系不固定），这两个Primary clock分别从FPGA的两个全局时钟引脚输入给两个MMCM，由两个MMCM分别产生的生成时钟之间属于异步时钟。一般情况下，不同的Primary clock之间都属于异步时钟，这些Primary clock分别产生的生成时钟之间也属于异步时钟关系。 **属于异步时钟关系的两个时钟之间的路径无法进行正确的时序分析。

一般情况下，如果用户不通过时钟分组对时钟之间的关系进行约束，时序引擎会默认所有的时钟之间都属于同步时钟关系。

**③ 不可扩宽的时钟（unexpandable clock）**

对于这类时钟，时序引擎无法在1000个时钟周期内找到两个时钟的公共周期，时序引擎就会从这1000个时钟周期中找到建立时间需求最差的情况，并进行时序分析，然而它不一定FPGA实际允许过程中建立时间需求最差的情况，因为在1000个时钟周期外可能还会有建立时间需求更差的情况，这样一来，时序引擎的分析结果就无法保证该路径一定不会出现问题，所以时序引擎的分析结果也就变的无意义。比如说由同一个Primary Clock驱动的两个MMCM的生成时钟分别是clk0（5.125ns）和clk1（6.666ns），虽然它们的相对相位关系是固定的，但是时序引擎无法保证对两个时钟之间路径的分析属于最差情况，这种情况和异步时钟之间的时序分析类似，时序分析的结果都看起来正常，但是这个结果确是不可信的。所以对这种时钟的处理方式与处理异步时钟是相同的，用户都需要进行跨时钟域的操作。

总结：异步时钟和不可扩展的时钟之间的路径都无法进行正确的时序分析，所以在时序分析之前，需要使用set_clock_group对时钟进行分组，从而将这些无法进行正确时序分析的路径忽略掉。

## 6.3 时钟分组实例

- 第一个Primary clock clk0从FPGA的全局时钟引脚进入了一个MMCM，MMCM生成了两个生成时钟userclk，itfclk 

- 第二个Primary clock clk1是一个高速收发器输出的恢复时钟，将该恢复时钟输入进一个MMCM，MMCM生成了两个生成时钟gtxclkrx， gtxclktx

![](https://upload-images.jianshu.io/upload_images/16278820-1f954fc1fb3dde3b.png)

![时钟约束后时序引擎如何进行第二种路径时序分析
](https://upload-images.jianshu.io/upload_images/16278820-9654b0ff7cdbb0d8.png)

![时钟约束后时序引擎如何进行第二种路径时序分析](https://upload-images.jianshu.io/upload_images/16278820-77875c77e74692ae.jpg)

# 7. Input延时约束/Output延时约束

## 7.1 什么是Input delay？如何计算它？如何约束它？

Input delay概念

![](https://upload-images.jianshu.io/upload_images/16278820-b6bc11c1b69243bd.png)

Input delay计算

![](https://upload-images.jianshu.io/upload_images/16278820-aac7ce8fb293a7eb.png)

Max Input Delay = Tco(Max) + Tpcb(Max) - Clk skew(Min)

![](https://upload-images.jianshu.io/upload_images/16278820-cb7b2f909cffd3c4.jpg)

Min Input Delay = Tco(Min) + Tpcb(Min) - Clk skew(Max)

![](https://upload-images.jianshu.io/upload_images/16278820-38ffaf6902b99c46.jpg)

Input delay约束

![](https://upload-images.jianshu.io/upload_images/16278820-cff67ee26f6c71d6.png)

## 7.2 什么是Output delay？如何计算它？如何约束它？

Output delay概念

![](https://upload-images.jianshu.io/upload_images/16278820-34f3cd8ced6a9b49.png)

Output delay计算

![](https://upload-images.jianshu.io/upload_images/16278820-60ba8f3ffb17b921.png)

![](https://upload-images.jianshu.io/upload_images/16278820-4f2c5f5e5c6cb612.png)

Max Output Delay = Tpcb(Max) + Tsu - Clk skew(Min) 

![](https://upload-images.jianshu.io/upload_images/16278820-309aa875840a5aa5.jpg)

Min Output Delay = Tpcb(Min) - Th - Clk skew(Max)

![](https://upload-images.jianshu.io/upload_images/16278820-a2a48dcbff8fd7eb.jpg)

Output delay约束

![](https://upload-images.jianshu.io/upload_images/16278820-483da8ceba92ffb0.png)


# 8. Multicycle约束

## 8.1 为什么要进行Multicycle约束？

时序引擎默认情况下会在建立时间需求/保持时间需求最差的情况下进行时序分析，而时序引擎选择的这种需求不一定是用户真正希望的，而且时序引擎默认选择的这种需求是非常严苛的，甚至是根本无法满足的。此时就需要用户进行Multicycle约束，手动修改建立时间需求/保持时间需求。用户希望放松某些路径的约束力度，就可以通过Multicycle约束调整建立时间需求/保持时间需求。 

使用set_multicycle_path命令进行约束

![](https://upload-images.jianshu.io/upload_images/16278820-3b5a901b2cd876cf.png)

![](https://upload-images.jianshu.io/upload_images/16278820-0cadf310522eef3c.png)

## 8.2 如何进行正确Multicycle约束？

注：使用set_multicycle_path命令

**① 在源时钟和目的时钟相同的情况下进行Multicycle约束**

![](https://upload-images.jianshu.io/upload_images/16278820-7f30bc889a4fb263.png)

![](https://upload-images.jianshu.io/upload_images/16278820-1710765b9614fa27.png)

- 每两个时钟发起一次数据，每两个时钟捕获一次数据

![](https://upload-images.jianshu.io/upload_images/16278820-cfb211c7f6a929e5.png)

![](https://upload-images.jianshu.io/upload_images/16278820-cdb25582831e9ed9.png)

![](https://upload-images.jianshu.io/upload_images/16278820-f553b5f33942bb39.png)

![](https://upload-images.jianshu.io/upload_images/16278820-7f318953b0364067.png)

![](https://upload-images.jianshu.io/upload_images/16278820-007564a571009512.png)

- 每四个时钟发起一次数据，每四个时钟捕获一次数据

![](https://upload-images.jianshu.io/upload_images/16278820-2347a1cbfc4beb7c.png)

- 只调整setup需求，Holdup需求会发生相应的变化，但是变化结果是错误的

![](https://upload-images.jianshu.io/upload_images/16278820-326a043970f869dc.png)

- 进一步调整Holdup需求

![](https://upload-images.jianshu.io/upload_images/16278820-969156b1c302cb97.png)

- Set_multicycle_path 4 – hold –end –from [get_pins data0_reg/C] – to [get_pins data1_reg/C]

![](https://upload-images.jianshu.io/upload_images/16278820-4b6ae099f697e187.png)

**② 在源时钟和目的时钟频率相同且有正向偏移的情况下（正向偏移0.3ns）**

![](https://upload-images.jianshu.io/upload_images/16278820-8db347ea4da11ad8.png)

![](https://upload-images.jianshu.io/upload_images/16278820-26004aa10b7be3b3.png)

先调整setup需求，holdup需求发生相应的变化，变化的结果是正确的

![](https://upload-images.jianshu.io/upload_images/16278820-99e4b0fed00833bc.png)

![](https://upload-images.jianshu.io/upload_images/16278820-d8f20a24f8e7c7f6.png)

**③ 在源时钟和目的时钟频率相同且有负向偏移的情况下（负向偏移0.3ns）发生负向偏移时，通常不需要进行Multicycle的约束，除非负向偏移过大**

![](https://upload-images.jianshu.io/upload_images/16278820-941f7fcbb9cfbe8b.png)

![](https://upload-images.jianshu.io/upload_images/16278820-a073dabdd41f7b4b.png)

**④ 在源时钟和目的时钟频率不同的情况下（源时钟慢，目的时钟快）**

![](https://upload-images.jianshu.io/upload_images/16278820-7e8ea4f615481260.png)

![](https://upload-images.jianshu.io/upload_images/16278820-38907d8d9e820cb8.png)

调整setup需求（只能用-end，不能用-start ），Holdup需求发生相应的变化，变化的结果是错误的。

![](https://upload-images.jianshu.io/upload_images/16278820-ed8f32d35c8c2849.png)

![](https://upload-images.jianshu.io/upload_images/16278820-c1c41d0117c0dcd5.png)

进一步调整Holdup需求。（只能用-end，不能用-start）

![](https://upload-images.jianshu.io/upload_images/16278820-9d19ece5bb435c81.png)

![](https://upload-images.jianshu.io/upload_images/16278820-38f7a2f863bee9b0.png)

**⑤ 在源时钟和目的时钟频率不同的情况下（源时钟快，目的时钟慢）**

![](https://upload-images.jianshu.io/upload_images/16278820-290285ea58fc4aa3.png)

![](https://upload-images.jianshu.io/upload_images/16278820-72b991a08897e233.png)

同时调整setup需求（只能用-start，不能用-end）和Holdup需求（只能用-start，不能用-end）

![](https://upload-images.jianshu.io/upload_images/16278820-74d5e32df61af6ec.png)

![](https://upload-images.jianshu.io/upload_images/16278820-a960f569cb93cdea.png)

# 9. Falsepath约束

## 9.1 什么样的路径属于FalsePath？

注：使用`set_false_path`命令FalsePath路径是指该路径在拓扑结构中的确存在，但是满足以下任一条件的路径：

- 在实际运行过程中，该路径不工作。

- 该路径不需要进行时序分析。

比如：已经做了两级触发器同步的跨时钟域路径 只在上电时工作一次的寄存器异步复位路径或用于测试的路径注意：如果用户的目的只是放松某条路径的时序约束，可以用`set_multicycle_path`命令进行约束，不要使用`set_false_path`。

设置FalsePath的好处：

- 减小综合/实现/时序分析的时间。

-  极大的提升工具对设计的综合/实现/优化的结果。-from/-to的node_list必须是有效的时序路径的起点和终点，或是时钟。-through的node_list可以是ports/pins。

```tcl
set_false_path [-setup] [-hold] [-from <node_list>] [-to <node_list>] [-through <node_list>]
```
![](https://upload-images.jianshu.io/upload_images/16278820-61ff8bc414ac60b7.png)

## 9.2 FalsePath约束举例

注：使用set_false_path命令

**① 将异步复位信号设置为伪路径**

```tcl
set_false_path -from [get_port reset] -to [all_registers]
```

![](https://upload-images.jianshu.io/upload_images/16278820-642c0434a16ec4c0.png)

**② 将CLKA到CLKB的单向路径设置为伪路径 **

```tcl
set_false_path -from [get_clocks CLKA] -to [get_clocks CLKB]
```

![](https://upload-images.jianshu.io/upload_images/16278820-18b6fb3ceb4afdbd.png)

**③ 将CLKA到CLKB以及CLKB到CLKA的双向路径设置为伪路径 **

```tcl
set_false_path -from [get_clocks CLKA] -to [get_clocks CLKB]
set_false_path -from [get_clocks CLKB] -to [get_clocks CLKA]
```

![](https://upload-images.jianshu.io/upload_images/16278820-bcdc9410a6bc9031.png)

注：当有多个异步时钟之间的路径需要设置为伪路径时，推荐使用set_clock_group命令。

![](https://upload-images.jianshu.io/upload_images/16278820-d1025be24cdf1858.png)

```tcl
set_false_path -through [get_pins MUX1/a0] -through [get_pins MUX2/a1]
```

![](https://upload-images.jianshu.io/upload_images/16278820-e570c6b69dcc0bde.png)

凡是经过`MUX1/a0`，到`MUX2/a1`的时序路径都会被忽略，采用`-through`的好处是不需要具体指定时序路径的起点和终点，`-through`的顺序需要注意，上述命令描述的是先经过`MUX1/a0`，然后再经过`MUX2/a1`。

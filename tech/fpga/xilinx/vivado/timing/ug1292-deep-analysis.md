深度解析UG1292
===
[原文](https://codingnote.cc/p/19460)

本质上，时序收敛要解决的是如下图所示的公式。在这个公式中，一旦芯片规格确定，Tco和Tsu是固定的，用户能改变的是其余三个值。针对这三个值，从FPGA设计角度而言，优化Tdata的可能性会更大一些。

![](http://app.eda365.com:8082/upload/weixin/images/167/67efda2af3d0ff1a03b861218ea60db9.jpg)

Tdata由两部分构成Tlogic和Tnet，前者为逻辑延迟，后者为布线延迟。逻辑延迟主要跟逻辑级数相关，显然，过高的逻辑级数会导致逻辑延迟增大。因此，在设计之初就要对逻辑级数有所评估。由此可见，降低逻辑延迟很大程度上需要在HDL代码层面优化，或者借助Vivado提供的综合选项以及综合技术。导致布线延迟过大的因素比较多，例如逻辑级数过大，路径中的某个net的扇出过大，或者工具优先对关键路径布线而使得某些路径的发送端和接收端相距太远等。针对这些问题，ug1292给出了明确的解决方法。

##概述

如何快速、高效地使时序收敛是很多FPGA工程师都要面临的一个问题。这时，大家可能都会想到ug949。这是Xilinx提供的一个很好的文档。作为工程经验的总结，这个文档也包含了时序收敛的方法。在这个文档的基础上，Xilinx最近又发布了一篇新文档ug1292（可直接在Xilinx官网搜索下载）。这个文档把ug949中时序收敛的相关内容单独提取出来，更系统、更直观地介绍了时序收敛的方法。ug1292可以视为时序收敛的一个快速参考手册，而ug949可以当作“字典“，用于查找更为具体的信息。

###ug1292 时序收敛快速参考手册

这个手册只有十页内容，每一页都有流程图或表格，因此具有很强的可操作性。这十页内容如下表格所示。可以看到，该手册几乎涵盖了所有解决时序违例的基本方法。
![](https://mmbiz.qpic.cn/mmbiz_jpg/amLPbwsBTyu5f9Fyq2IqbKQP1AN6DjWPaHr5diajicdwEAkJJ7C1sQPPQAdbkC7xHCzZU9ARicfs3Qg6b3U40IASw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

###如何使用这个手册

这个手册与ug949的理念是一致的即”尽可能地把所有问题放在设计初期解决“。宁可在设计初期花费更多的时间，也不要等到布局布线后才开始发现问题再解决问题。因为，在设计后期，往往会面临牵一发而动全身的被动局面。即使一个小的改动都有可能花费很多的时间和精力甚至造成返工。就时序收敛而言，在定义设计规格时就要有所考虑；写代码时要从代码风格角度考虑对时序的影响；综合之后就要查看时序报告，检查设计潜在问题。这也是ug1292为什么把初始设计检查放在第一页的主要原因。建议最好在开始设计之前通读一下该文档，了解一下对于时序违例路径：

-    逻辑延迟占总延迟多大百分比时需要优化

-    布线延迟占总延迟多大百分比时需要优化

-    时钟歪斜和时钟不确定性超过哪个界限时需要优化

-    WHS在哪个阶段超过哪个界限时需要优化


如果能对这些数值做到心中有数，那么当面临相关问题时就可以有的放矢。更为关键的是充分理解第二页内容：时序收敛基线流程。

 

在设计综合之后，就开始按照手册第一页流程对设计进行初始检查。当检查都过关之后，才可以进行下一页的操作。在设计后期，根据上述数值判定造成时序违例的主要因素，然后回到相应页码查看解决方案。

##1. 初始设计检查
ug1292第一页的主题是初始设计检查。这一步是针对综合后或者opt_design阶段生成的dcp。尽管在Vivado下，从功能仿真到综合、布局布线、直至生成.bit文件是相对自动化的流程，但是解决时序违例仍然是一个复杂且耗时的过程。仅仅靠log信息或者布线后的时序报告往往很难定位，这是因为实现过程中的每一步（opt_design逻辑优化，place_design布局, phys_opt_design物理优化, route_design布线）都会做一些优化，这些优化可能会导致关键路径被掩盖，例如，有时发现设计中逻辑级数（Logic Level）较高的路径时序收敛了，反倒是逻辑级数较低甚至为0的路径出现时序违例。因此，采取按部就班的策略，检查每一步的结果，及时且尽早发现设计中的问题是很有必要的。

初始设计检查流程如下图所示。对象是综合后或opt_design阶段生成的dcp。会依次执行三个命令（图中红色标记），生成三个报告：FailFast报告、时序报告和UFDM（UltraFast Design Methodology）报告。

![](https://mmbiz.qpic.cn/mmbiz_jpg/amLPbwsBTysibX7WKXxMXxGIOfImumpuZ4wApTmoGyiaVKtLLib0ADGyI7pS39AqKlAQQ0NDG1X0BGPHqHPFnNnNg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
，如下图所示，这是Vivado自带例子工程cpu的FailFast报告。可以看到，对于LUT，利用率应控制在70%以内；触发器（FD）应控制在50%以内；BlockRAM和DSP48可以达到80%。在这个报告中尤其要关注Status为Review的条目，这是会给时序收敛带来负面影响的，需要优化的。对于设计中存在Pblock情形，report_failfast提供了-pblock选项，对于SSI器件，report_failfast提供了-slr和-by_slr（需要在place_design阶段生成的dcp下使用）选项。这样，可针对某个pblock或某个SLR进行分析。

![](https://mmbiz.qpic.cn/mmbiz_jpg/amLPbwsBTysibX7WKXxMXxGIOfImumpuZjzfCB5V8iaW1stLTQuStXLV7qhK2lrNqfuQkNN1lYNIOwYu76ibC5chg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

report_timing_summary可以生成时序报告，除了查看时序违例路径之外，该报告还可显示时序约束是否存在潜在问题。如下图所示，Check Timing下包含12个条目，这个阶段需要格外关注是否有未约束的时序路径，是否有Timing loop，同时还要关注时钟约束是否合理。

![](https://mmbiz.qpic.cn/mmbiz_jpg/amLPbwsBTysibX7WKXxMXxGIOfImumpuZEtk1aLwoLibM07wSGd4kJ4wV9DeqfwAlxz2wFGDVgHYdbNxnA27uuMA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

report_methodology可以生成UFDM报告。该命令不仅可以检查RTL代码存在的问题，例如Block RAM没有使用内部Embedded Registers，DSP48用做乘法器时没有使能MREG等，还可以检查时序约束存在的问题。如图所示，要尤其关注其中的Bad Practice。

![](https://mmbiz.qpic.cn/mmbiz_jpg/amLPbwsBTysibX7WKXxMXxGIOfImumpuZF0A5ia8iaDD7PqZxNxhxRoxwrXibC78uJFHptC4ZCCwu4tmHvlbQ6PFTQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

对于这三个报告中存在的问题，要尽可能地在综合阶段或者opt_design阶段加以解决，最终确保这三个报告足够“干净”，即所有隐患都被消除。

 此外，对于大规模的设计，可针对设计中的关键模块使用上述三个命令，因为这些关键模块很有可能成为时序收敛的瓶颈。为了使用这三个命令，可以对关键模块采用OOC（Out-of-Context）的综合方式或单独创建Vivado工程以便生成相应的dcp。

##2. 时序收敛基线流程

ug1292第二页的主题是时序收敛基线流程。该流程如下图所示。可以看到该流程要求在实现（Implementation）过程中的每一个子步骤结束之后都要检查WNS是否大于0，只有当WNS大于0时，才可以进行下一个子步骤。同时，在布局之后，还要检查WHS是否大于-0.5ns。此外，只有当WNS小于0时，执行phys_opt_design才有意义，毕竟phys_opt_design的目的是修复建立时间违例。由此可见，在实现的前期，更多关注的是建立时间违例。


![](https://mmbiz.qpic.cn/mmbiz_png/amLPbwsBTyuibxwktbkD5ttoTvvhuSjyianGs3cbQeLS14WtQCw4XYfAM5TXGQYk8JtJaeloON1iaXIDiaj6ukONow/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)


基线流程是UltraFast设计方法学的重要组成部分，其目的是尽可能早地发现设计存在的问题，并在早期解决这些问题。对于opt_design生成的dcp，如果发现WNS小于0，则要根据设计初期检查结果找出原因（具体操作看这里）；对于place_design生成的dcp，如果WNS小于0，可能的原因包括过高的资源利用率、过重的逻辑互联导致的布局拥塞、逻辑级数不合理、过高的时钟歪斜或时钟不确定性。可尝试执行phys_opt_design（-directive设置为Explore或AgressiveExplore），如果无效，则要将焦点放在改善布局质量上；对于route_design生成的dcp，通过report_route_status可检查是否有布线错误。如果WNS或WHS依然小于0，则需进一步分析（后续会介绍）。如果WNS小于0且大于-0.2ns，可尝试执行phys_opt_design。
 

基线流程的一个重要方面是设计分析。这里列出常用的用于设计分析的Tcl命令：

-    report_timing_summary

-    report_design_analysis

-    report_methodology

-    report_failfast (需要在Xilinx Tcl Store中先安装)


这些命令的具体使用方法可参考ug835或在Vivado Tcl Console中执行<命令名 -help>查看。同时，也可采用图形界面方式。

##3. 建立时间违例分析流程

通常，我们优先解决建立时间违例。Setup slack与逻辑延迟、布线延迟、时钟歪斜和时钟不确定性有关。因此，首先要明确这几个因素中哪个因素对建立时间违例起关键作用。具体的衡量标准可由如下几个数值确定。这也是ug1292第三页的主题。

![](https://mmbiz.qpic.cn/mmbiz_png/amLPbwsBTysyxJ81ozu3v0ovEhLia1K7N9D7YYg3ZiblnGBjzGM9ABr1cZv4dpfkfbeCLWVSQZ3zKwjA9xzFZ1lg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

ug1292第三页也给出了建立时间违例分析流程，如下图所示。当逻辑延迟占比超过50%时，要着重降低逻辑延迟；当布线延迟占比超过50%时，要把焦点放在布线延迟上；同时，也要关注一下时钟歪斜和时钟不确定性。当时钟歪斜小于-0.5ns或时钟不确定性大于0.1ns时，两者将成为时钟违例的主要“贡献者”。

![](https://mmbiz.qpic.cn/mmbiz_png/amLPbwsBTysyxJ81ozu3v0ovEhLia1K7NicghBHKIwaO6fUgPiaSJpe1icfDGZ46IQZicyd8zowyR5EN0mwIJ55oibuQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

上述四个数值，无论是在timing report（通过report_timing_summary生成）还是design analysis report（通过report_design_analysis生成）中都有所体现。以timingreport为例，如下图所示，可清晰地显示上述四个数值。

![](https://mmbiz.qpic.cn/mmbiz_png/amLPbwsBTysyxJ81ozu3v0ovEhLia1K7NibwBuTSjESXVJzNibMxtZQHAF55lhV5pbicdYMuyrzQJiaY0CLC4QrQalw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

注：上述数据只针对UltraScale系列芯片。

##4. 保持时间违例分析流程

在分析place_design生成的dcp时，就要开始关注保持时间违例，尤其是当WHS < -0.5ns时。这是因为过大的保持时间违例往往会导致布线时间增大，同时，在布线阶段，工具未必能修复此类违例。

解决保持时间违例流程如下图所示。按照此流程，要逐步关注以下几个因素：

-    Clock Skew是否大于0.5ns

-    Hold Requirement是否为正（应该为0）

-    WHS是否小于-0.4ns或THS是否小于-1000ns

-    Clock Uncertainty是否大于0.1ns

![](https://mmbiz.qpic.cn/mmbiz_png/amLPbwsBTys54Py2lHaMNpgVsDwnuXgk01ibOdEVWy4KztPmK8nVONt8oDK7ibzT23G5rYhiaG67ltiafLyrdasRXA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这几个指标的具体数值可在时序报告中查到，如下图所示。图中，Path Type用于确定分析的是保持时间。

![](https://mmbiz.qpic.cn/mmbiz_png/amLPbwsBTys54Py2lHaMNpgVsDwnuXgkNTXiaprUmDEBCLH6AWuI8l5MWia3qzichicjxibFvwic4zQXtg4DqibHXl9BA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Hold Requirement为正的可能情形出现在使用多周期路径约束的时序路径中。如下图所示，时钟使能信号EN使得路径的Setup Requirement为3个时钟周期，但Hold Requirement仍应为0。

![](https://mmbiz.qpic.cn/mmbiz_png/amLPbwsBTys54Py2lHaMNpgVsDwnuXgkZuDkIPylQqCBHJzzLU9ex8lHeiaUNApquU9bdoj7TA6gbz69FYP3Mzw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在此情况下，应采用如下图所示的多周期路径约束。其中的第2条约束是对hold的调整，却往往容易被遗漏。对于-hold，它表示相对于缺省捕获沿（图中的Default hold）,实际捕获沿应回调的时钟周期个数。

![](https://mmbiz.qpic.cn/mmbiz_png/amLPbwsBTys54Py2lHaMNpgVsDwnuXgks9ZWU5881Wx8UW4Vic3jl04jZCK3M1jndpicUZKto8E2xNSaAULd7kVw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

对于过大的WHS或THS，应在布线之前做一些优化，尽可能地降低WHS和THS。为此，可在phys_opt_design阶段采取如下图所示的几个措施。

![](https://mmbiz.qpic.cn/mmbiz_png/amLPbwsBTys54Py2lHaMNpgVsDwnuXgkxNibKoDibVEFld9p1PBEnwc1FazKHYmRXvA2HHNzhpGG8hLyKcIITRHg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

其中方法（1）是在两个同步时序元件之间插入与至相反的时钟沿触发的寄存器，将该路径一分为二，该方法的前提是建立时间不会被恶化。方法（2）至方法（4）都是在路径中插入LUT1。方法（2）只在WHS最大的路径中插入LUT1；方法（3）则是在更多的路径中插入LUT1；方法（4）则是在-directive为Explore的基础上进一步修正保持时间违例，等效于-directive Explore +-aggressive_hold_fix。

##5. 降低逻辑延迟

在实现阶段，Vivado会把最关键的路径放在首位，这就是为什么在布局或布线之后可能出现逻辑级数低的路径时序反而未能收敛。因此，在综合或opt_design之后就要确认并优化那些逻辑级数较高的路径。这些路径可有效降低工具在布局布线阶段为达到时序收敛而迭代的次数。同时，这类路径往往逻辑延迟较大。因此，降低这类路径的逻辑延迟对于时序收敛将大有裨益。

 

降低逻辑延迟的流程如下图所示。不难看出，这一工作应在综合或者opte_design阶段完成。

![](https://mmbiz.qpic.cn/mmbiz_jpg/amLPbwsBTysTyqSmkwcmKF7sFia3ZvAjXILP79ms0ZFearYN7LXuxLs2ialh174w4JeVbgibmclSJ7bQgDBvSB1pw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在这个流程中，我们需要关注两类路径。一类路径是由纯粹的CLB中的资源（FF，LUT，Carry，MUXF）构成的路径；另一类则是Block（DSP，BRAM，URAM，GT）之间的路径。

无论是哪种路径，首先要通过命令report_design_analysis进行定位，具体命令格式如下图所示（也可在Vivado菜单Reports -> Report Design Analysis下执行）。

```
report_design_analysis -logic_level_distribution -logic_level_dist_paths 1000 -name logic_level
```

该命令可分析当前设计的逻辑级数分布情况，如下图所示，从而便于找到逻辑级数较高的路径。

![](https://mmbiz.qpic.cn/mmbiz_jpg/amLPbwsBTysTyqSmkwcmKF7sFia3ZvAjXxNSiaibRiajZ8hWtgeGY1icSWWz9pwR2Jc9x2MjyzPuZr4hF40lDEZy83A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

点击逻辑级数分布报告中的数字，例如图中的19，可生成相应的时序报告，从而确定属于哪类路径，并进一步观察路径特征。


- 对于级联的小的LUT

如果路径中包含多个级联的小的LUT，检查一下这些LUT是否是因为设计层次、综合属性（KEEP，KEEP_HIERARCHY，DONT_TOUCH，MARK_DEBUG）等导致无法合并。


- 对于路径中存在单个的Carry

如果路径中有单个的Carry（不是级联的），检查一下这个Carry是否限制了工具对LUT的优化，从而造成布局不是最优的。如果是，可尝试在综合时使用FewerCarryChains策略或者在opt_design阶段对这个Carry设置CARRY_REMAP属性（具体使用方法可查看ug904）。

- 对于终点是SRL的路径

如果路径的终点是SRL，可尝试将SRL变为FF+SRL+FF或SRL+FF。这可在综合时通过使用SRL_STYLE综合属性实现，也可在opt_design阶段通过使用SRL_STAGES_TO_INPUT或SRL_STAGES_TO_OUTPUT实现。

- 对于终点是触发器控制端的路径

如果路径的终点是由LUT输出连接到触发器的同步使能端或同步复位端，可尝试将这类逻辑搬移到触发器的数据端，这可在综合时通过设置EXTRACT_ENABLE或EXTRACT_RESET综合属性实现，或者在opt_design阶段通过设置CONTROL_SET_REMAP属性（具体使用方法可查看ug904）实现。

- 使用Retiming

此外，还可以在综合时对全局使用retiming（选中-retiming选项）或者采用模块化综合方式，对某个模块使用retiming。

 - 对于Block到Block的路径

对于Block到Block的路径，最好将其优化为Block + FF + Block。这里的FF可以是Block内部自带的触发器（如果有的话），也可以是Slice中的触发器。


如果数据由Block RAM输出，可采用如下命令观察使能Block RAM自带的寄存器之后是否对时序有所改善。这里要注意，如下命令用于评估，因为已造成设计功能改变，所有不要在此基础上生成bit文件。

```
set_property –dict {DOA_REG 1 DOB_REG 1} [get_cellsxx/ramb18_inst]
```

该命令等效于

```
set_property DOA_REG 1  [get_cells xx/ramb18_inst]

set_property DOB_REG 1  [get_cells xx/ramb18_inst] 
```

##6. 降低布线延迟（1）

当设计出现布线拥塞时，通常会导致布线延迟增大，从而影响时序收敛。布线拥塞程度可通过如下两种方式获取：

- 布线阶段的log文件中会显示拥塞程度

- 对于place_design或route_design生成的dcp文件，可通过如下命令获取

```
report_design_analysis -congestion -name cong
```

生成的拥塞报告如下图所示。要格外关注拥塞程度（Congestion Level）大于4的区域。

![](https://mmbiz.qpic.cn/mmbiz_png/amLPbwsBTys0PjSqByy2cLg93ZcbHzvrUKS6F11QMh5UL80wxib7O8nWrYGab3TuIFz86Vjq3VldNrcQfKRg28A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

对于拥塞程度大于4的情形，可采用如下流程加以改善并降低布线延迟。在如下的案例中，可以看到布线延迟占到了总延迟的94%，据此，可断定布线延迟是导致时序违例的根本原因。从布线结果看，在拥塞区域出现了绕线。

![](https://mmbiz.qpic.cn/mmbiz_jpg/amLPbwsBTys0PjSqByy2cLg93ZcbHzvrxPFMM27zVzuqs3vkKuJ3K5h9gchTSX51icNdKrNdA47YwicylglF7flg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz_jpg/amLPbwsBTys0PjSqByy2cLg93ZcbHzvrYGJ1ljndO9ibDUhwwiafFBibxOKdLRYZZcp4nn5BltP2jmpmbP4ic6icRZg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)


降低拥塞程度可改善布线质量。Xilinx建议采用如下方法以改善布线拥塞。

（1）当整体资源利用率达到70%～80%时（对于多die芯片，这个数值是指每个SLR的资源利用率），需要砍掉一些模块以降低资源利用率。尤其要避免LUT/BRAM/DSP/URAM利用率同时出现大于80%的情形。如果BRAM/DSP/URAM这些Block利用率无法降低，那么要确保LUT利用率低于60%。

```
xilinx::designutils::report_failfast -by_slr
```

（2）执行place_design时，尝试将-directive设置为AltSpreadLogic*或SSI_Spread*或将Implementation的策略设置为Congestion_*，如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_jpg/amLPbwsBTys0PjSqByy2cLg93ZcbHzvrDQLjddsoI5ibGApqDSOhVLUbx9nyL5IfEUejtKrMFdvXoIicyftcQGoQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

（3）通过如下命令分析设计复杂度，找到设计中出现拥塞的模块（Rent值大于0.65或AverageFanout大于4）。之后，对这些模块实施模块化综合，其中的综合策略设置为ALTERNATE_ROUTABILITY。


生成设计复杂度报告：

![](https://mmbiz.qpic.cn/mmbiz_png/amLPbwsBTys0PjSqByy2cLg93ZcbHzvryULj4iazoCUphcuWcibiapJ3JUO5fx7k9l7oBhFFyZicH4pBb913ZBdctQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

对拥塞模块采用模块化综合技术：

```
set_property BLOCK_SYNTH.STRATEGY {ALTERNATE_ROUTABILITY} [get_cells <congestedHierCellName>]
```

（4）降低拥塞区域MUXF*和LUT-Combining的使用率，具体方法有时可通过report_qor_suggestions获得。但采用模块化综合技术是一个值得一试的方法。

```
set_property BLOCK_SYNTH.MUXF_MAPPING 0 [get_cells <CellName>]
set_property BLOCK_SYNTH.LUT_COMBINING 0 [get_cells <CellName>]
```

（5）在布线区域内非关键的高扇出网线上引入BUFG，可通过如下命令实现。

```
set_property CLOCK_BUFFER_TYPE BUFG [get_nets <highFanoutNetName>]
```

（6）从之前低拥塞的布线或布局结果中继承DSP/BRAM/URAM的布局。这可通过如下脚本实现。

```
set brams [get_cells -hier -filter"PRIMITIVE_SUBGROUP==BRAM || PRIMITIVE_SUBGROUP==FIFO"]
set dsps [get_cells -hier -filter {REF_NAME==DSP48E2}]

set_property IS_LOC_FIXED TRUE $dsps
write_xdc -exclude_timing dsp_loc.xdc -force
set_property IS_LOC_FIXED FALSE $dsps

set_property IS_LOC_FIXED TRUE $brams
write_xdc -exclude_timing bram_loc.xdc -force
set_property IS_LOC_FIXED FALSE $brams
```

- 优化高扇出网线：

（1）在RTL层面，基于设计层次复制寄存器降低扇出，或者在opt_design阶段通过-hier_fanout_limit选项降低扇出。

```
opt_design -merge_equivalent_drivers -hier_fanout_limit 512
```

（2）在phys_opt_design（布局之后）阶段通过-force_replication_on_nets对关键的高扇出网线通过复制寄存器降低扇出。

```
phys_opt_design -force_replication_on_nets <net>
```

##7. 降低布线延迟（2）

布线延迟过大除了拥塞导致之外，还可能是其他因素。下图显示了降低布线延迟的另一流程（因其他因素导致布线延迟过大的处理流程）。

![](https://mmbiz.qpic.cn/mmbiz_png/amLPbwsBTyuyfLx31sAyQKIicibicJpibA5ph3YuJxlrCdQBjEWMl9cV6DoZdhKxYsfduR0TANClX3D8SkscEvY7Hg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

首先，通过report_desigan_analysis分析路径特征。有时还需要结合report_utilization和report_failfast两个命令。

###第1步：分析路径的Hold Fix Detour是否大于0ps？

HoldFix Detour是工具为了修复保持时间违例而产生的绕线（该数值在design analysis报告中显示，如果没有显示，可在报告标题栏内点击右键，选择HoldFix Detour）。如果该数值大于0，就有可能造成建立时间违例。这时其实应关注的是该路径对应的保持时间报告，诊断为什么工具会通过绕线修复保持时间违例。

###第2步：违例路径的各个逻辑单元是否存在位置约束？

通常，设计中不可避免地会有一些物理约束，如管脚分配。除此之外，还可能会有其他位置约束，如通过create_macro或Pblock创建的位置约束。如果设计发生改变，就需要关注这些位置约束是否仍然合理，尤其是那些穿越多个Pblock的路径。

###第3步：违例路径是否穿越SLR？

如果目标芯片为多die芯片，那么在设计初期就要考虑到以下几个因素，以改善设计性能。

-    在设计的关键层次边界上以及跨die路径上插入流水寄存器，尤其是跨die路径，这些寄存器是必需的；

-   检查每个SLR的资源利用率是否合理，这可通过report_failfast –by_slr实现。-by_slr选项只能在place_design或route_design生成的dcp中使用，这也不难理解，毕竟在布局阶段工具才会把设计单元向相应的SLR内放置；

-    每个die的设计可以看作一个顶层，因此，要对每个顶层指定一个die，以确保相应的设计单元被正确放置在目标die内。这可通过属性USER_SLR_ASSIGNMENT实现（Vivado 2018.2开始支持）；

-    如果上述属性未能正确工作，可直接画Pblock进行约束；

-    在布局或布线之后如果仍有时序违例，可尝试使用phys_opt_design -slr_crossing_opt。

###第4步：唯一控制集百分比是否大于7.5%？

唯一控制集个数可通过report_failfast查看。如果控制集百分比超过7.5%，可通过如下方法降低控制集。

-    关注MAX_FANOUT属性：

-    移除时钟使能、置位或复位信号的MAX_FANOUT属性。这是因为该属性会复制寄存器以降低扇出，但同时也增加了控制集；

-    在Synthesis阶段：

    - 提高–control_set_opt_threshold的数值，可使工具将更多同步控制信号搬移到数据路径，从而降低控制集；

    - 也可采用Block Level Synthesis技术，对指定模块设置该数值；

-    在opt_design阶段：

    - -control_set_merge 

    - -merge_equivalent_drivers

    这两个选项可帮助降低控制集。但这两个选项不能与-directive同时使用，所以如果是工程模式下，可将其放置在Hook文件中（Tcl.pre或Tcl.post）。非工程模式下，可在执行完-directive之后，再次执行这两个选项；

-    关注低扇出信号：

    对于低扇出的控制信号（同步使能、同步置位/同步复位），可对其连接的寄存器设置CONTROL_SET_REMAP属性，将控制信号搬移到数据路径上，从而降低控制集。

###第5步：尝试其他实现策略

Vivado提供了多种实现策略。因此，尝试不同实现策略是达到时序收敛的一个手段。

-    尝试多种place_design和phys_opt_design，这可通过设置不同的-directive实现；

-    尝试使用过约束（过约最大0.5ns），这可通过设置Clock Uncertainty实现。需要用到set_clock_uncertainty；

-    对关键时钟域下的路径设置更高的优先级，使工具对其优先布局布线，这可通过命令group_path实现；

-    尝试使用增量布局布线，继承之前好的布局布线结果，并缩短编译时间。

##8. 降低Clock Skew

过大的Clock Skew也可能导致时序违例，尤其是其数值超过0.5ns时。如下三个命令生成的报告中均可显示Clock Skew的具体数值。

- report_design_analysis

- report_timing_summary

- report_timing

降低Clock Skew可采用如下流程操作。

![](https://mmbiz.qpic.cn/mmbiz_png/amLPbwsBTyvfFUiaIeouhvwGYreyoEiaND4m5Cj9aBZacF9NcrvdstDyQatv7T8u3EFgEl0dnpS1VSQ7ic3Y4CQibw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

###异步跨时钟域路径是否被安全合理地约束

时钟关系有两种：同步时钟和异步时钟。如果发送时钟和接收时钟是同源的，例如来自于同一个MMCM，则认为二者是同步时钟，否则就按异步时钟处理。对于异步跨时钟域路径，可采用如下三者之一进行约束：

- set_clock_groups

- set_false_path

- set_max_delay -datapath_only

###发送时钟和接收时钟的时钟树结构是否平衡

时钟树结构其实就是时钟的拓扑结构。从发送时钟和接收时钟的角度看，平衡的时钟树结构是指二者“走过相同或等效的路径”。如下图所示，发送时钟和接收时钟来自级联的BUFG的不同位置上，这就是典型的不平衡时钟树。在设计中要避免这种情形。通过Tcl命令report_methodology可检查出设计中级联的BUFG。

![<不推荐>](https://mmbiz.qpic.cn/mmbiz_png/amLPbwsBTyvfFUiaIeouhvwGYreyoEiaNDztU5rVFpOKG0F5jtgFNO9m4wARzfB79KSKcQho8IBTge3a0GEDiaR3A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



同时，还要利用好BUFG_GT和BUFGCE_DIV，两者均可实现简单地分频。如下图所示，利用BUFG_GT实现二分频，从而节省了MMCM。

![](https://mmbiz.qpic.cn/mmbiz_png/amLPbwsBTyvfFUiaIeouhvwGYreyoEiaNDxvUbTxnqLdeKKoQhNTm6xAIRPlQ8NI0DUPtb6OyMmGWQEmhs73cj1g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

此外，还要保持时钟路径“干净”，即不能在时钟路径上存在组合逻辑。在时序报告中，点击如下图标记的按钮，按下F4，在显示时序路径的同时也会显示该路径的时钟拓扑结构。

![](https://mmbiz.qpic.cn/mmbiz_png/amLPbwsBTyvfFUiaIeouhvwGYreyoEiaNDg4EI6hpodYJ9193XaIZmZROSJ2WViaybe36uhKfsk504DNagKTjklng/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)


###检查Clock Skew是否小于0.5ns

CLOCK_DELAY_GROUP可有效改善同步跨时钟域路径的Clock Skew，因此，Xilinx建议对于关键的同步跨时钟域路径，可通过设置该属性降低Clock Skew，即使发送时钟和接收时钟具有相同的CLOCK_ROOT值。CLOCK_DELAY_GROUP的具体使用方法如下图所示，其中clk1_net和clk2_net是Clock Buffer的输出端连接的net。但是，避免过多使用该属性，否则会适得其反。

```
set_property CLOCK_DELAY_GROUP grp12 [get_nets {clk_net, clk2_net}]
```

###时钟是否同时驱动I/O和Slice中的逻辑资源

如果时钟同时驱动I/O和Slice中的逻辑资源，且负载小于2000时，可通过CLOCK_LOW_FANOUT属性对相应的时钟net进行设置，最终可使工具将该时钟驱动的所有负载放置在同一个时钟域内。通过命令report_clock_utilization生成的报告可查看每个时钟的负载，如下图所示。
CLOCK_LOW_FANOUT的具体使用方法如下图所示。

```
set_property CLOCK_LOW_FANOUT TRUE [get_nets -of [get_pins clk_
```

###检查数据路径是否穿越SLR或I/O Column

如果时钟负载较小且穿越SLR或I/O Column时，可通过Pblock实施位置约束，将负载限定在一定区域内，例如在一个SLR内，以避免穿越一些特殊列，例如I/O Column。相反地，如果数据路径并未穿越SLR或I/O Column，可尝试对相应的MMCM或PLL做位置约束，使其位于这些负载的中央。

##9. 降低Clock Uncertainty

Clock Uncertainty跟图1所示的几个因素有关。当时序违例路径的Clock Uncertainty超过0.1ns时，应引起关注。这一数值可在时序报告中查找到，如图2所示，如果需要降低Clock Uncertainty，可采用如图3所示的流程。

![图1 Clock Uncertainty相关因素](https://mmbiz.qpic.cn/mmbiz_png/amLPbwsBTyspDBrfDFqJ2PXvtWSt7JiaMfzn8cniaMXn8IFicjkibicsBag8cOUHFaribMU8iacTfyAUgUiawmstHQqdfw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图1 Clock Uncertainty相关因素

![](https://mmbiz.qpic.cn/mmbiz_jpg/amLPbwsBTyspDBrfDFqJ2PXvtWSt7JiaMb6BKtXBB9Qnp5j80jTic3xEAm0QhrE6C7j7Lx3zBr2micfALRlF4WoLA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图2 Timing Report中查看Clock Uncertainty

![](https://mmbiz.qpic.cn/mmbiz_jpg/amLPbwsBTyspDBrfDFqJ2PXvtWSt7JiaMjr6ToLgXzCQB9OaSCdVgHNqTaoBcY91c6jl0aR4tAdr0svSsatGrBg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图3 降低Clock Uncertainty的操作流程

###同步时钟是否由两个并行的MMCM或PLL生成

在UltraScale和UltraScale Plus系列芯片中，BUFGCE_DIV可提供分频功能。如图4所示，如果需要通过MMCM生成两个时钟，其频率分别为300MHz和600MHz。此时，可利用BUFGCE_DIV的分频功能，同时可对这两个时钟设置CLOCK_DELAY_GROUP属性，从而降低Clock Uncertainty。

![](https://mmbiz.qpic.cn/mmbiz_png/amLPbwsBTyspDBrfDFqJ2PXvtWSt7JiaMGW0ribwKwKll6ibcwYD4XWl1CaN8VtGh21dGxfHAzOrHNbqxlyGD1Axg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图4 利用BUFGCE_DIV生成分频时钟

###生成时钟其Discrete Jitter>0.05ns?

Discrete Jitter是由MMCM/PLL引入的，其具体数值可通过点击图2中Clock Uncertainty的数值查看，如图5所示。通常，VCO的频率越高，引入的DiscreteJitter会越小。因此，可通过手工调整VCO的频率（在ClockingWizard中修改M和D两个参数）达到降低Discrete Jitter的目的。此外，如果可以的话，用PLL替代MMCM。相比于MMCM，PLL引入的Jitter会小一些。

![](https://mmbiz.qpic.cn/mmbiz_jpg/amLPbwsBTyspDBrfDFqJ2PXvtWSt7JiaMHI6t7nIgHE6gznwttC3ycTFerFRia1HkGGcG7HiaYYr9U92d0a3039qQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图5 查看Discrete Jitter具体数值

###同步跨时钟域路径是否超过1000条

过多的同步跨时钟域路径会对时序收敛带来一定的挑战，尤其是时钟频率比较高时，例如频率为500MHz。此时要检查这些路径。

（1）能否对这些路径设置多周期路径约束

（2）在Latency允许的情况下，通过FIFO或XPM_CDC处理跨时钟域路径

##10. 用好report_failpast

###安装Design Utilities

使用report_failpast之前，要先确保Design Utilities已经安装。安装方法是点击Tools->Xilinx Tcl Store，如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/amLPbwsBTytmZybsTpkr5NNbG7XvNZ7I4QziaaXg91DvUSL8UZh9ojmxDzQH149T1Nf6QaXOA46iaAj6pJEEhcnA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

###report_failpast生成报告

report_failpast生成的报告分为三部分：设计特征、时钟检查和关键路径分析，如下图所示。其中在设计特征部分，该报告会给出资源利用率的建议值，一旦超过这个建议值，Status列内会呈现REVIEW标记。如图中的FD（D触发器），实际利用率为57.66%，超过建议值50%。同时该部分还会给出控制集分析（Control Set），可帮助判断是否需要降低控制集。此外，对于不是由FD驱动的扇出大于10K的net，这部分也会有所显示。

![](https://mmbiz.qpic.cn/mmbiz_jpg/amLPbwsBTytmZybsTpkr5NNbG7XvNZ7IpjPQk65XWPa02JEUG3ew3IMZkiaj69KNSd6eYC8mLUs0geeARiajKdhw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

###report_failpast使用方法

report_failpast除了基本的使用方法（不添加任何选项）之外，还提供了其他的选项。例如，-pblock选项可用于分析Pblock对应的约束是否合理。该选项可在布局之前使用（要求已经提供了Pblock具体位置约束），也可在布局之后使用。显然，在布局之前使用是非常有意义的，因为可以据此判定Pblock的约束是否合理。同时，针对SSI芯片，该命令还提供了-slr和-by_slr选项，这两个选项需要在布局之后使用。此外，对于报告中Status为REVIEW的条目可通过选项-detailed_report生成相应的更为详细的报告，具体使用方法如下所示。例如，DONT_TOUCH为REVIEW状态，则该命令可生成impl.DONT_TOUCH.rpt报告，可显示使用了DONT_TOUCH属性的cell。

```
xilinx::designutils::report_failfast -detailed_report impl -file failfast.rpt
```

report_failfast的另一特征在于既可以应用于整个设计，也可以针对某个指定的模块进行分析。例如，对于设计中的关键模块采用此命令分析，可预先发现潜在的问题，从而加速时序收敛。
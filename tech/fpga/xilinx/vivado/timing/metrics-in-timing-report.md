时序报告要看哪些指标
===

By Lauren

[原文地址](https://mp.weixin.qq.com/s?__biz=MzI5NTQwODcyMQ==&mid=2247486280&idx=1&sn=91d43636ebed6f0175e287b7035d17f7&chksm=ec554e76db22c7605bf41bf98667efb6dd4f6e28f144fda557e299b08981d174d7de2c9013af&xtrack=1&scene=90&subscene=93&sessionid=1598457085&clicktime=1598457161&enterid=1598457161&ascene=56&devicetype=android-28&version=27000f41&nettype=WIFI&abtest_cookie=AAACAA%3D%3D&lang=zh_CN&exportkey=AgrlTTp8JtTPuCjMBwCm%2FYQ%3D&pass_ticket=Bj8%2BIhE7BvrsH52CTKDBl9hY2MiqOXfVEhJeJIlxLfkRQvF1O5gONDJs%2Fcfx8AGQ&wx_header=1)

生成时序报告后，如何阅读时序报告并从时序报告中发现导致时序违例的潜在问题是关键。首先要看`Design Timing Summary`。在这个Summary里，呈现了`Setup`、`Hold`和`Pulse Width`的总体信息，但凡`WNS`、`WHS`或`WPWS`有一个小于0，就说明时序未收敛。

![](https://upload-images.jianshu.io/upload_images/16278820-2c7f642a5f08f49d?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 1. 找到时序最糟糕的路径

如果时序未收敛，并不需要分析所有未收敛的路径，而是先关注时序最糟糕的路径，先优化这些路径，有可能优化这些路径之后，这些路径收敛了，同时其他路径也能够收敛。只需要点击上图中WNS之后的数字，即可显示这些最糟糕的路径，如下图所示。

![](https://upload-images.jianshu.io/upload_images/16278820-357f72d716889391.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2. 分析时序最糟糕的路径

只需要双击上图中的路径，就能显示该路径对应的时序报告的详细信息，如下图所示。

![](https://upload-images.jianshu.io/upload_images/16278820-180f21a865a9cdd6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在这个报告中，首先可以看到`Slack`，其值为负，表明时序未收敛。

接着看`Source`和`Destination`。通常，`Source`为时钟端口，如图中触发器的C端口；`Destination`为数据端口，如图中触发器的D端口。**从`Source`和`Destination`还可以看到起始cell和终止cell的驱动时钟，从而可判定该路径是否为跨时钟域路径。**

这一点也可以从`Requirement`部分给出的信息加以验证。图中`Requirement`显示均为同一时钟，故此处为单一时钟下的时序路径。紧接着`Path Type`为`Setup`，表明该报告为建立时间路径报告，其后的信息`Max at Slow Process Corner`其中的`Slow`意味着`High Temperature Low Voltage`。如果是`Path Type`为`Hold`，表明该报告为保持时间路径报告，其后的信息为`Min at Fast Process Corner`，Fast意味着`Low Temperature High Voltage`。

>对于`Requirement`一栏，一定要看`Requirement`是否合理，例如，如果`Requirement`为1ns，那么显然是不合理的，这说明时序约束本身有问题。

其后的`Data Path Delay`由两部分构成，逻辑延迟（对应图中的`logic`）和线延迟（对应图中的`route`）。这一栏同时显示了每部分延迟占的百分比。

- 对于7系列FPGA，如果逻辑延迟超过了25%，那么说明时序违例的主要原因是逻辑级数太高了；对于UltraScale系列FPGA，这个指标则为50%。**通常认为1个LUT+1根net的延迟为0.5ns，**据此来评估逻辑级数是否过高。例如如果时钟为100MHz，那么逻辑级数在10/0.5=20左右是可以接受的。

- 对于7系列FPGA，如果线延迟超过了75%，那么说明时序违例的主要原因是线延迟太高了；对于UltraScale系列FPGA，这个指标则为50%。


对于`Clock Path Skew`，如果该值超过了0.5ns，就要关注；对于`Clock Uncertainty`，如果该时钟是由`MMCM`或`PLL`生成，且`Discrete Jitter`超过了50ps，就要回到`Clocking Wizard`界面尝试修改参数改善`Discrete Jitter`。


对于跨时钟域路径，如下图所示，从`Requirement`部分信息可以看到源时钟和目的时钟是不同的，即可表明该路径为跨时钟域路径。这里`Requirement`为1ns，显然是不合理的，这说明跨时钟域路径的约束不合理。

![](https://upload-images.jianshu.io/upload_images/16278820-820e026e94c5ffce?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

总体而言，打开时序报告，要看路径`Source`、`Destination`、`Requirement`、`Data Path Delay`、`Logic Levels`、`Clock Path Skew`和`Clock Uncertainty`。还有一点至关重要，**时序分析在综合之后就要开始分析，而不是等到布局布线之后再看。综合之后着重分析的是逻辑级数、资源利用率和控制集。**

 DCM/DLL/PLL/MMCM区别
 ==

 [Original post](https://mp.weixin.qq.com/s?__biz=MzU4ODY5ODU5Ng==&mid=2247484106&idx=1&sn=82983a8086732717298436e067a64d4d&chksm=fdd98441caae0d57c99c5b22cf72bfaee2372824406014680be9df1f8d85b3071182fe43656c&mpshare=1&scene=21&srcid=0928ySJ3ud0vfaGS85Teu5Xw&sharer_sharetime=1571051171309&sharer_shareid=296cfe717a7da125d89d5a7bcdf65c18&key=6234e09828e71f223a5bbb62942587523cffdc550c50d6713403e50f0f1a03c87c5b1a6fae054a425e6f27eabfd6e48eb8fd421c5841d8d8b3b054113d8e8650ff4a65e51fa211ebe10dc0a436635167&ascene=1&uin=MzkzMzM2Nzc1&devicetype=Windows)

 对于FPGA工程师来说，DCM/DLL/MMCM/PLL这些词简直每天都能看到，但很多人并不是很清楚它们之间的差异。

在Xilinx的FPGA中，时钟管理器叫做Clock Management，简称CMT。我们所用到的DCM/PLL/MMCM都包含在CMT中。

`DCM`是比较早的FPGA中使用的，比如Sparten-3和Virtex-4，后面的器件不再使用了。在Virtex-4中，CMT包括一个PLL和两个DCM。DCM的核心是DLL，即Delay Locked Loop，它是一个数字模块，可以产生不同相位的时钟、分频、倍频、相位动态调整等，但精度有限。

`PLL`就是Phase Locked Loop，这个大家应该都熟悉，时钟倍频、分频、调节相位等都是可以用PLL，而且PLL是一个模拟电路，它产生的频率比DCM更加准备，jitter也更好，但PLL不能动态调整相位。

`MMCM`是Mixed Mode Clock Manager，它的官方解释是：This is a PLL with some small part of a DCM tacked on to do fine phase shifting (that's why its mixed mode - the PLL is analog, but the phase shift is digital).也就是说，它是**在PLL的基础上加上了相位动态调整功能**。因为PLL是模块电路，而动态调相是数字电路，所以叫Mixed Mode。MMCM是在Virtex-6中被引入的，而且Virtex-6中也只有MMCM。

到了7系列和Ultrascale中，MMCM和PLL同时存在。7s FPGA中，最高包含了24个CMT，每个CMT包含一个MMCM和一个PLL。Ultrascale中，一个CMT包含一个MMCM和 两个PLL。

MMCM相对PLL的优势就是相位可动态调整，但PLL占用的面积更小。

在Vivado中，在使用Clock Wizard时，我们可以选择使用MMCM或者PLL，而且可以它们的区别也仅仅下图的红框部分。

![](https://mmbiz.qpic.cn/mmbiz_png/bmUD0vQPK9ufTSkxraLUcNibRQu7kjzibfPysl1nONK6CeNmIIWeCRibXAhichbibBClyrbLM8mavVflJgbVaRG9BQg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz_png/bmUD0vQPK9ufTSkxraLUcNibRQu7kjzibfcJyNDpjKR95EI0HibLMTTS2ownBMkJ0A7GVp44MIHjw6uSic3aovJBuw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
FPGA时序约束中常用公式推导
==

Original post: <https://blog.csdn.net/huan09900990/article/details/76079820>


# 1. Data Arrival Time 数据到达目的寄存器REG2时间

![](http://xilinx.eetrend.com/files/2019-09/wen_zhang_/100045372-81417-1.png)

# 2. Clock Arrival Time 时钟到达目的寄存器REG2时间

![](http://xilinx.eetrend.com/files/2019-09/wen_zhang_/100045372-81418-2.png)

# 3. Data Required Time 数据需求时间 - Setup

指数据需要在需求时间前到达目的寄存器，否则不满足建立时间关系，不能被正确采样。

最大延迟是防止数据来的太慢 ，当时钟沿已经到来时，数据还没到，这样就不能在上升沿被寄存器正确采样。

![](http://xilinx.eetrend.com/files/2019-09/wen_zhang_/100045372-81419-3.png)

# 4. Data Required Time 数据需求时间-Hold

指数据在时钟锁存沿到达后，必须保持一段稳定的时间，使数据被正确采样。做最小延迟约束是为了防止数据传输过快，使得寄存器还在锁存上一个数据时，下一个数据就来了，使得上次锁存数据发生错误。

所以 保持时间必须小于 $t_{co}+t_{logic}$，这里 $t_{co}+t_{logic}$ 就是数据从源寄存器到目的寄存器的时间。

![](http://xilinx.eetrend.com/files/2019-09/wen_zhang_/100045372-81420-4.png)

# 5. 时序裕量slack

Setup Slack=Setup Required Time - Data Arrival Time

Hold Slack=Data Arrival Time - Hold Required Time

时序裕量为正 表示时序满足时序约束条件，为负，时序不满足。

# 6. Input Delay 输入最大最小延迟

Input Delay=数据路径延迟-时钟路径延迟+utco(外部器件)


$输入延迟 Input Delay = Data Arrival Time - Clock Arrival Time$
$=Launch Edge + T_{clk1} + T_{co} + T_{data} - Latch Edge - T_{clk2}$

数据相对于时钟到达目的寄存器的时间差值。即数据和时钟从同一时间点（launch）开始，到达目的寄存REG2的时间差。

数据到达REG2走的路径延时是：时钟从launch开始 经过$T_{clk1}$的延迟到达REG1，REG1在时钟沿来之后，经过$T_{co}$的时间把数据送出REG1，然后数据再经过路径延迟 $T_{data}$ 到达REG2的数据管脚。

时钟到达REG2走的路径延时是：时钟也从同一时间点（launch）开始，经过路径延迟Tclk2就到达REG2的时钟管脚。
输入最大延迟是约束为了满足寄存器的建立时间，输入最小延迟是位了满足寄存器的保持时间。

Input Maximum Delay=Data Arrival Time最大值-Clock Arrival Time 最小值

“fpga-centric”Input Maximum Delay<=tclk-tsu(fpga)

![](http://xilinx.eetrend.com/files/2019-09/wen_zhang_/100045372-81421-5.png)

Input Minimum Delay=Data Arrival Time最小值-Clock Arrival Time 最大值

“fpga-centric” Input Minimum Delay>=th(fpga)

![](http://xilinx.eetrend.com/files/2019-09/wen_zhang_/100045372-81422-6.png)

# 7. Output Delay 输出最大最小延迟

$Output Maximum Delay=外部器件t_{su} + 数据路径最大延迟 - 时钟路径最小延迟$

![](http://xilinx.eetrend.com/files/2019-09/wen_zhang_/100045372-81423-7.png)

$output Minimum Delay = 外部器件t_{h} + 数据路径最小延迟 - 时钟路径最大延迟$

![](http://xilinx.eetrend.com/files/2019-09/wen_zhang_/100045372-81424-8.png)

# 8. Fmax

指设计能运行的最高频率，即周期为最小时，频率最大
当Setup Slack=0时，系统刚好满足建立时间，此时周期为最小值。

$period = t_{co} + data_delay + t_{su} - t_{skew}$

![](http://xilinx.eetrend.com/files/2019-09/wen_zhang_/100045372-81425-9.png)
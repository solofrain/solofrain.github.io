Circuit
==

# 1. Theory

## 1.1 Interface

### 1.1.1  General

- [Logic Levels ](https://learn.sparkfun.com/tutorials/logic-levels/ttl-logic-levels){:target="_blank"}

- [What is the best way to convert 1.8 V to 5 V?](https://electronics.stackexchange.com/questions/127619/what-is-the-best-way-to-convert-1-8-v-to-5-v){:target="_blank"}


### 1.1.2 JESD204B

- [JESD204B Overview](https://www.ti.com.cn/cn/lit/ml/slap161/slap161.pdf?ts=1600271439525){:target="_blank"}

- [Introduction to JESD204B](intro-jesd204b.md)

- [JESD204B: Understanding the protocol](https://e2e.ti.com/blogs_/b/analogwire/archive/2014/07/30/jesd204b-understanding-the-protocol){:target="_blank"}

- An Intro to JESD204B Subclasses and Deterministic Latency
    - [Park 1](https://www.electronicdesign.com/technologies/analog/article/21139021/an-intro-to-jesd204b-subclasses-and-deterministic-latency-part-1?utm_source=EG+ED+Today&utm_medium=email&utm_campaign=CPS200828012&o_eid=0048A7615490B5U&rdx.ident%5Bpull%5D=omeda%7C0048A7615490B5U&oly_enc_id=0048A7615490B5U){:target="_blank"}

    - [Part 2](https://www.electronicdesign.com/technologies/analog/article/21140723/an-intro-to-jesd204b-subclasses-and-system-considerations-part-2?utm_source=EG+ED+Today&utm_medium=email&utm_campaign=CPS200911056&o_eid=0048A7615490B5U&rdx.ident%5Bpull%5D=omeda%7C0048A7615490B5U&oly_enc_id=0048A7615490B5U){:target="_blank"}

- [JESD204B: deterministic latency]

    - [What is deterministic latency? Why do I need it? How do I achieve it?](https://e2e.ti.com/blogs_/b/analogwire/archive/2014/12/22/jesd204b-what-is-deterministic-latency-why-do-i-need-it-how-do-i-achieve-it){:target="_blank"}
    - [How to measure and verify your deterministic latency](https://e2e.ti.com/blogs_/b/analogwire/archive/2015/02/27/jesd204b-how-to-measure-and-verify-your-deterministic-latency){:target="_blank"}
    - [How to calculate your deterministic latency](https://e2e.ti.com/blogs_/b/analogwire/archive/2015/01/16/jesd204b-how-to-calculate-your-deterministic-latency){:target="_blank"}

# 2. Function block

## 2.1 Interface

### 2.1.1 General

- [Analog Devices - MT-098 Low Voltage Logic Interfacing](https://www.analog.com/media/en/training-seminars/tutorials/MT-098.pdf){:target="_blank"}

- [SparkFun Logic Level Converter - Bi-Directional](https://www.sparkfun.com/products/12009){:target="_blank"}


- [UltraScale deivces and 3.3V I/O](https://forums.xilinx.com/t5/Versal-and-UltraScale/New-FPGAs-Supporting-3-3V-IO/td-p/911112){:target="_blank"}

    - For Zynq UltraScale+ MPSoC there are three types of banks: HP, HD and MIO. The HD and MIO banks support 3.3V ([DS925](http://www.xilinx.com/support/documentation/data_sheets/ds925-zynq-ultrascale-plus.pdf){:target="_blank"}). HDIO is intended for low speed control and status signals. 

    - Virtex Ultrascale+ doesn't support HD banks. 

- Level shifting ICs

    - [PI4GTL2014 4-bit LVTTL to GTL transceiver](https://www.diodes.com/part/view/PI4GTL2014){:target="_blank"}

     - [PI4GTL200 2-Channel GTL level shifter & transceiver](https://www.diodes.com/part/view/PI4GTL2002){:target="_blank"}

     - [LSF0108-Q1 Automotive 8-Channel Bidirectional Multi-Voltage-Level Translator](https://www.ti.com/product/LSF0108-Q1) / [14-24-LOGIC-EVM](https://www.digikey.com/products/en?mpart=14-24-LOGIC-EVM&v=296){:target="_blank"}


# 3. Project


- [开源示波器 - ScopeFun Open source Oscilloscope](https://blog.csdn.net/qq_38376586/article/details/90735013?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param){:target="_blank"}

    - [Demo](https://www.youtube.com/watch?v=mdjJTs8R46g){:target="_blank"}

    - [开源资料地址](https://gitlab.com/scopefun){:target="_blank"}

    - [上位机](https://www.scopefun.com/download){:target="_blank"}

    - [KiCad](https://kicad-pcb.org/download/){:target="_blank"}


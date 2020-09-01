FPGA时序约束实战
==

我们以Vivado自带的`wave_gen`工程为例，该工程的各个模块功能较为明确，如下图所示。为了引入异步时钟域，我们在此程序上由增加了另一个时钟–`clkin2`，该时钟产生脉冲信号`pulse`，`samp_gen`中在`pulse`为高时才产生信号。

![](https://upload-images.jianshu.io/upload_images/16278820-87ad6a642af969b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


下面我们来一步一步进行时序约束。

# 1. 梳理时钟树

  我们首先要做的就是梳理时钟树，就是工程中用到了哪些时钟，各个时钟之间的关系又是什么样的，如果自己都没有把时钟关系理清楚，不要指望综合工具会把所有问题暴露出来。

  在我们这个工程中，有两个主时钟，四个衍生时钟，如下图所示。

![](https://upload-images.jianshu.io/upload_images/16278820-6113fe08e301c1ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



确定了主时钟和衍生时钟后，再看各个时钟是否有交互，即clka产生的数据是否在clkb的时钟域中被使用。

这个工程比较简单，只有两组时钟之间有交互，即：

- `clk_rx`与`clk_tx`
- `clk_samp`与`clk2`

其中，`clk_rx`和`clk_tx`都是从同一个MMCM输出的，两个频率虽然不同，但他们却是同步的时钟，因此他们都是从同一个时钟分频得到（可以在Clock Wizard的Port Renaming中看到VCO Freq的大小），因此它们之间需要用`set_false_path`来约束；而`clk_samp`和`clk2`是两个异步时钟，需要用`asynchronous`来约束。

![](https://upload-images.jianshu.io/upload_images/16278820-4defd76638d9d2f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


完成以上两步，就可以进行具体的时钟约束操作了。

# 2. 约束主时钟

  在这一节开讲之前，我们先把`wave_gen`工程的`wave_gen_timing.xdc`中的内容都删掉，即先看下在没有任何时序约束的情况下会综合出什么结果？

对工程综合并Implementation后，Open Implemented Design，会看到下图所示内容。

![](https://upload-images.jianshu.io/upload_images/16278820-c4f958409e195515.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到，时序并未收敛。可能到这里有的同学就会有疑问，我们都已经把时序约束的内容都删了，按我们第一讲中提到的“因此如果我们不加时序约束，软件是无法得知我们的时钟周期是多少，PAR后的结果是不会提示时序警告的”，这是因为在该工程中，用了一个MMCM，并在里面设置了输入信号频率，因此这个时钟软件会自动加上约束。

  接下来，我们在tcl命令行中输入`report_clock_networks -name main`，显示如下：

![](https://upload-images.jianshu.io/upload_images/16278820-cd5df18983c439c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看出，Vivado会自动设别出两个主时钟，其中clk_pin_p是200MHz，这个是直接输入到了MMCM中，因此会自动约束；另一个输入时钟clk_in2没有约束，需要我们手动进行约束。

或者可以使用`check_timing -override_defaults no_clock`指令，这个指令我们之前的内容讲过，这里不再重复讲了。

在tcl中输入

```
create_clock -name clk2 -period 25 [get_ports clk_in2]
```

注：在Vivado中，可以直接通过tcl直接运行时序约束脚本，运行后Vivado会自动把这些约束加入到xdc文件中。

再执行`report_clock_networks -name main`，显示如下：

![](https://upload-images.jianshu.io/upload_images/16278820-358892111b831e72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到，主时钟都已被正确约束。

# 3. 约束衍生时钟

  系统中有4个衍生时钟，但其中有两个是MMCM输出的，不需要我们手动约束，因此我们只需要对`clk_samp`和`spi_clk`进行约束即可。约束如下：

```
create_generated_clock -name clk_samp -source [get_pins clk_gen_i0/clk_core_i0/clk_tx] \
                       -divide_by 32 [get_pins clk_gen_i0/BUFHCE_clk_samp_i0/O]

create_generated_clock -name spi_clk -source [get_pins dac_spi_i0/out_ddr_flop_spi_clk_i0/ODDR_inst/C] \
                       -divide_by 1 -invert [get_ports spi_clk_pin]
```

我们再运行`report_clocks`，显示如下：

![](https://upload-images.jianshu.io/upload_images/16278820-bccdd0c4a2cdca7c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们在理论篇的“create_generated_clock”一节中讲到，我们可以重新设置Vivado自动生成的衍生时钟的名字，这样可以更方便我们后续的使用。按照前文所讲，只需设置`name`和`source`参数即可，其中这个`source`可以直接从`report_clocks`中得到，因此我们的约束如下：

```
create_generated_clock -name clk_tx \
                       -source [get_pins clk_gen_i0/clk_core_i0/inst/mmcm_adv_inst/CLKIN1] \ 
                               [get_pins clk_gen_i0/clk_core_i0/inst/mmcm_adv_inst/CLKOUT1]

create_generated_clock -name clk_rx \
                       -source [get_pins clk_gen_i0/clk_core_i0/inst/mmcm_adv_inst/CLKIN1] \
                               [get_pins clk_gen_i0/clk_core_i0/inst/mmcm_adv_inst/CLKOUT0]
```

大家可以对比下`report_clocks`的内容和约束指令，很容易就能看出它们之间的关系。

把上述的约束指令在tcl中运行后，我们再运行一遍`report_clocks`，显示如下：

![](https://upload-images.jianshu.io/upload_images/16278820-fb6759496253873a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在时序树的分析中，我们看到，`clk_samp`和`clk2`两个异步时钟之间存在数据交互，因此要进行约束，如下：

```
set_clock_groups -asynchronous -group [get_clocks clk_samp] -group [get_clocks clk2]
```
# 4. 延迟约束

对于延迟约束，相信很多同学是不怎么用的，主要可能就是不熟悉这个约束，也有的是嫌麻烦，因为有时还要计算PCB上的走线延迟导致的时间差。而且不加延迟约束，Vivado也只是在Timing Report中提示warning，并不会导致时序错误，这也会让很多同学误以为这个约束可有可无。

![](https://upload-images.jianshu.io/upload_images/16278820-451c91129c51a48b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但其实这种想法是不对的，比如在很多ADC的设计中，输出的时钟的边沿刚好是数据的中心位置，而如果我们不加延迟约束，则Vivado会默认时钟和数据是对齐的。

![](https://upload-images.jianshu.io/upload_images/16278820-fba55c59b261d182.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对于输入管脚，首先判断捕获时钟是主时钟还是衍生时钟，如果是主时钟，直接用`set_input_delay`即可，如果是衍生时钟，要先创建虚拟时钟，然后再设置delay。对于输出管脚，判断有没有输出随路时钟，若有，则直接使用`set_output_delay`，若没有，则需要创建虚拟时钟。

在本工程中，输入输出数据管脚的捕获时钟如下表所示：

| 管脚 | 输入输出 | 捕获时钟 | 时钟类型 | 是否有随路时钟 | 是否需要虚拟时钟 |
| --- | --- | --- | --- | --- | --- |
| rxd_pin | 输入 | clk_pin_p | 主时钟 | x | No |
| txd_pin | 输出 | clk_tx | x | 无 | Yes |
| lb_sel_pin | 输入 | clk_tx | 衍生时钟 | x | Yes |
| led_pins[7:0] | 输出 | clk_tx | 衍生时钟 | 无 | Yes |
| spi_mosi_pin | 输出 | spi_clk | 衍生时钟 | 有 | No |
| dac_* | 输出 | spi_clk | 衍生时钟 | 有 | No |

根据上表，我们创建的延迟约束如下，其中的具体数字在实际工程中要根据上下游器件的时序关系（在各个器件手册中可以找到）和PCB走线延迟来决定。未避免有些约束有歧义，我们把前面的所有约束也加进来。

```
# 主时钟约束
create_clock -period 25.000 -name clk2 [get_ports clk_in2]

# 衍生时钟约束
create_generated_clock -name clk_samp -source [get_pins clk_gen_i0/clk_core_i0/clk_tx] \
                       -divide_by 32 [get_pins clk_gen_i0/BUFHCE_clk_samp_i0/O]

create_generated_clock -name spi_clk -source [get_pins dac_spi_i0/out_ddr_flop_spi_clk_i0/ODDR_inst/C] \
                       -divide_by 1 -invert [get_ports spi_clk_pin]

create_generated_clock -name clk_tx \
                       -source [get_pins clk_gen_i0/clk_core_i0/inst/mmcm_adv_inst/CLKIN1] \
                               [get_pins clk_gen_i0/clk_core_i0/inst/mmcm_adv_inst/CLKOUT1]

create_generated_clock -name clk_rx \
                       -source [get_pins clk_gen_i0/clk_core_i0/inst/mmcm_adv_inst/CLKIN1] \
                               [get_pins clk_gen_i0/clk_core_i0/inst/mmcm_adv_inst/CLKOUT0]

# 设置异步时钟
set_clock_groups -asynchronous -group [get_clocks clk_samp] -group [get_clocks clk2]

# 延迟约束
create_clock -period 6.000 -name virtual_clock

set_input_delay -clock [get_clocks -of_objects [get_ports clk_pin_p]] 0.000 [get_ports rxd_pin]

set_input_delay -clock [get_clocks -of_objects [get_ports clk_pin_p]] -min -0.500 [get_ports rxd_pin]

set_input_delay -clock virtual_clock -max 0.000 [get_ports lb_sel_pin]

set_input_delay -clock virtual_clock -min -0.500 [get_ports lb_sel_pin]

set_output_delay -clock virtual_clock -max 0.000 [get_ports {txd_pin {led_pins[*]}}]

set_output_delay -clock virtual_clock -min -0.500 [get_ports {txd_pin {led_pins[*]}}]

set_output_delay -clock spi_clk -max 1.000 [get_ports {spi_mosi_pin dac_cs_n_pin dac_clr_n_pin}]

set_output_delay -clock spi_clk -min -1.000 [get_ports {spi_mosi_pin dac_cs_n_pin dac_clr_n_pin}]
```

# 5. 伪路径约束

  在不加伪路径的时序约束时，Timing Report会提示很多的error，其中就有跨时钟域的error。

![](https://upload-images.jianshu.io/upload_images/16278820-9b8e8a84a178f4a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们可以直接在上面右键，然后设置两个时钟的伪路径。

![image.png](https://upload-images.jianshu.io/upload_images/16278820-eab9f87e734378e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样会在xdc中自动生成如下约束：

```
set_false_path -from [get_clocks -of_objects [get_pins clk_gen_i0/clk_core_i0/inst/mmcm_adv_inst/CLKOUT0]] \
               -to [get_clocks -of_objects [get_pins clk_gen_i0/clk_core_i0/inst/mmcm_adv_inst/CLKOUT1]]
```

其实这两个时钟我们已经在前面通过generated指令创建过了，因此get_pins那一长串就没必要重复写了，所以我们可以手动添加这两个时钟的伪路径如下：

```
set_false_path -from [get_clocks clk_rx] -to [get_clocks clk_tx]
```

伪路径的设置是单向的，如果两个时钟直接存在相互的数据的传输，则还需要添加从`clk_tx`到`clk_rx`的路径，这个工程中只有从rx到tx的数据传输，因此这一条就可以了。

这里再修改一条第7讲中的错误，第7讲中时钟树的图里，只有从`clk_rx`到`clk_tx`的箭头，不应该有从`clk_tx`到`clk_rx`的箭头。

在伪路径一节中，我们讲到过异步复位也需要添加伪路径，`rst_pin`的复位输入在本工程中就是当做异步复位使用，因此还需要添加一句：

```
set_false_path -from [get_ports rst_pin]
```

对于`clk_samp`和`clk2`，它们之间存在数据交换，但我们在前面已经约束过`asynchronous`了，这里就可以不用重复约束了。

这里需要提示一点，添加了上面这些约束后，综合时会提示xdc文件的的warning。

![](https://upload-images.jianshu.io/upload_images/16278820-ecc88fe2e21126ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但这可能是Vivado的综合过程中，读取到该约束文件时，内部电路并未全都建好，就出现了没有发现`clk_gen_i0/clk_core_i0/inst/mmcm_adv_inst/CLKIN1`等端口的情况，有如下几点证明：

- 这个端口在代码中确实是存在的
- 若把该xdc文件，设置为仅在Implementation中使用，则不会提示该warning
- 在Implementation完成后，无论是Timing Report还是通过tcl的`report_clocks`指令，都可以看到这几个时钟已经被正确约束。下图所示即为设置完上面的约束后的Timing Report。

![image.png](https://upload-images.jianshu.io/upload_images/16278820-ecfbf613683cd5fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---

# 6. 多周期路径约束

  多周期路径，我们一般按照以下4个步骤来约束：

## 6.1 带有使能的数据

首先来看带有使能的数据，在本工程中的Tming Report中，也提示了同一个时钟域之间的几个路径建立时间不满足要求

![](https://upload-images.jianshu.io/upload_images/16278820-4eb0c6f8c2d09a20.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


其实这几个路径都是带有使能的路径，使能的周期为2倍的时钟周期，本来就应该在2个时钟周期内去判断时序收敛。因此，我们添加时序约束：

```
set_multicycle_path 2 -setup -from [get_cells {cmd_parse_i0/send_resp_data_reg[*]}  \ 
                             -include_replicated_objects]                           \
                             -to [get_cells {resp_gen_i0/to_bcd_i0/bcd_out_reg[*]}]

set_multicycle_path 1 -hold -from [get_cells {cmd_parse_i0/send_resp_data_reg[*]}   \
                            -include_replicated_objects]                            \
                            -to [get_cells {resp_gen_i0/to_bcd_i0/bcd_out_reg[*]}]
```

也可以写为：

```
set_multicycle_path -from [get_cells {cmd_parse_i0/send_resp_data_reg[*]}    \
                    -include_replicated_objects]                             \
                    -to [get_cells {resp_gen_i0/to_bcd_i0/bcd_out_reg[*]}] 2

set_multicycle_path -hold                                                    \
                    -from [get_cells {cmd_parse_i0/send_resp_data_reg[*]}    \
                    -include_replicated_objects]                             \
                    -to [get_cells {resp_gen_i0/to_bcd_i0/bcd_out_reg[*]}] 1
```

这两种写法是等价的。

我们也可以直接点击右键通过GUI的方式进行约束，效果都是一样的。

在工程的`uart_tx_ctl.v`和`uart_rx_ctl.v`文件中，也存在带有使能的数据，但这些路径在未加多路径约束时并未报出时序错误或者警告。

在接收端，捕获时钟频率是200MHz，串口速率是115200，采用16倍的Oversampling，因此使能信号周期是时钟周期的200e6/115200/16=108.5倍。

在接收端，捕获时钟频率是166667MHz，串口速率是115200，采用16倍的Oversampling，因此使能信号周期是时钟周期的166.667e6/115200/16=90.4倍。

因此，时序约束如下：

```
# 串口接收端
set_multicycle_path  -from [get_cells uart_rx_i0/uart_rx_ctl_i0/* -filter IS_SEQUENTIAL]   \
                     -to [get_cells uart_rx_i0/uart_rx_ctl_i0/* -filter IS_SEQUENTIAL] 108

set_multicycle_path -hold                                                                  \
                    -from [get_cells uart_rx_i0/uart_rx_ctl_i0/* -filter IS_SEQUENTIAL]    \
                    -to [get_cells uart_rx_i0/uart_rx_ctl_i0/* -filter IS_SEQUENTIAL] 107
# 串口发送端
set_multicycle_path -from [get_cells uart_tx_i0/uart_tx_ctl_i0/* -filter IS_SEQUENTIAL] -to [get_cells uart_tx_i0/uart_tx_ctl_i0/* -filter IS_SEQUENTIAL] 90
set_multicycle_path -hold -from [get_cells uart_tx_i0/uart_tx_ctl_i0/* -filter IS_SEQUENTIAL] -to [get_cells uart_tx_i0/uart_tx_ctl_i0/* -filter IS_SEQUENTIAL] 89

```

约束中的`filter`参数也将在下一章节具体讲解。

## 6.2 两个有数据交互的时钟之间存在相位差

在本工程中，没有这种应用场景，因此不需要添加此类约束。

## 6.3 存在快时钟到慢时钟的路径

在本工程中，没有这种应用场景，因此不需要添加此类约束。

## 6.4 存在慢时钟到快时钟的路径

在本工程中，没有这种应用场景，因此不需要添加此类约束。

综上，我们所有的时序约束如下：

```
# 主时钟约束
create_clock -period 25.000 -name clk2 [get_ports clk_in2]

# 衍生时钟约束
create_generated_clock -name clk_samp                                                       \
                       -source [get_pins clk_gen_i0/clk_core_i0/clk_tx]                     \
                       -divide_by 32 [get_pins clk_gen_i0/BUFHCE_clk_samp_i0/O]

create_generated_clock -name spi_clk \
                       -source [get_pins dac_spi_i0/out_ddr_flop_spi_clk_i0/ODDR_inst/C]    \
                       -divide_by 1                                                         \
                       -invert [get_ports spi_clk_pin]

create_generated_clock -name clk_tx                                                         \
                       -source [get_pins clk_gen_i0/clk_core_i0/inst/mmcm_adv_inst/CLKIN1]  \
                               [get_pins clk_gen_i0/clk_core_i0/inst/mmcm_adv_inst/CLKOUT1]

create_generated_clock -name clk_rx                                                         \
                       -source [get_pins clk_gen_i0/clk_core_i0/inst/mmcm_adv_inst/CLKIN1]  \
                               [get_pins clk_gen_i0/clk_core_i0/inst/mmcm_adv_inst/CLKOUT0]

# 设置异步时钟
set_clock_groups -asynchronous -group [get_clocks clk_samp] -group [get_clocks clk2]

# 延迟约束
create_clock -period 6.000 -name virtual_clock
set_input_delay -clock [get_clocks -of_objects [get_ports clk_pin_p]] 0.000 [get_ports rxd_pin]
set_input_delay -clock [get_clocks -of_objects [get_ports clk_pin_p]] -min -0.500 [get_ports rxd_pin]
set_input_delay -clock virtual_clock -max 0.000 [get_ports lb_sel_pin]
set_input_delay -clock virtual_clock -min -0.500 [get_ports lb_sel_pin]
set_output_delay -clock virtual_clock -max 0.000 [get_ports {txd_pin {led_pins[*]}}]
set_output_delay -clock virtual_clock -min -0.500 [get_ports {txd_pin {led_pins[*]}}]
set_output_delay -clock spi_clk -max 1.000 [get_ports {spi_mosi_pin dac_cs_n_pin dac_clr_n_pin}]
set_output_delay -clock spi_clk -min -1.000 [get_ports {spi_mosi_pin dac_cs_n_pin dac_clr_n_pin}]

# 伪路径约束
set_false_path -from [get_clocks clk_rx] -to [get_clocks clk_tx]
set_false_path -from [get_ports rst_pin]

# 多周期约束
set_multicycle_path 2 -setup                                                \
                      -from [get_cells {cmd_parse_i0/send_resp_data_reg[*]} \
                      -include_replicated_objects]                          \
                      -to [get_cells {resp_gen_i0/to_bcd_i0/bcd_out_reg[*]}]

set_multicycle_path 1 -hold \
                      -from [get_cells {cmd_parse_i0/send_resp_data_reg[*]}  \
                      -include_replicated_objects]                           \
                      -to [get_cells {resp_gen_i0/to_bcd_i0/bcd_out_reg[*]}]

# 串口接收端
set_multicycle_path 108 -setup \
                        -from [get_cells uart_rx_i0/uart_rx_ctl_i0/* -filter IS_SEQUENTIAL] \
                        -to [get_cells uart_rx_i0/uart_rx_ctl_i0/* -filter IS_SEQUENTIAL]

set_multicycle_path 107 -hold \
                        -from [get_cells uart_rx_i0/uart_rx_ctl_i0/* -filter IS_SEQUENTIAL] \
                        -to [get_cells uart_rx_i0/uart_rx_ctl_i0/* -filter IS_SEQUENTIAL]

# 串口发送端
set_multicycle_path 90 -setup \
                       -from [get_cells uart_tx_i0/uart_tx_ctl_i0/* -filter IS_SEQUENTIAL] \
                       -to [get_cells uart_tx_i0/uart_tx_ctl_i0/* -filter IS_SEQUENTIAL] 

set_multicycle_path 89 -hold \
                       -from [get_cells uart_tx_i0/uart_tx_ctl_i0/* -filter IS_SEQUENTIAL] \
                       -to [get_cells uart_tx_i0/uart_tx_ctl_i0/* -filter IS_SEQUENTIAL]
```

重新Synthesis并Implementation后，可以看到，已经没有了时序错误

![](https://upload-images.jianshu.io/upload_images/16278820-3c11414817f75860.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

仅有的两个warning也只是说rst没有设置input_delay，spi_clk_pin没有设置output_delay，但我们已经对rst设置了伪路径，而spi_clk_pin是我们约束的输出时钟，无需设置output_delay。

![](https://upload-images.jianshu.io/upload_images/16278820-cde80dff1a1b4b36.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

到这里，教科书版的时序约束教程就基本讲完了。但我们平时的工程中，跟上面这种约束还是有差异的：

- 首先是虚拟时钟，这个约束在平时的工程中基本不会用到，像需要设置虚拟时钟的场景，我们也都是通过设计来保证时序收敛，设置虚拟时钟的意义不大。

- 第二就是`output delay`，在FPGA的最后一级寄存器到输出的路径上，往往都使用了IOB，也就是IO block，因此最后一级寄存器的位置是固定的，从buffer到pad的走线延时是确定的。在这种情况下，是否满足时序要求完全取决于设计，做约束只是验证一下看看时序是否收敛。所以也基本不做。但是`input delay`是需要的，因为这是上一级器件输出的时序关系。

- 第三个就是多周期路径，我们讲了那么多多周期路径的应用场景，但实际我们是根据Timing report来进行约束的，即便那几种场景都存在，但如果Timing report中没有提示任何的时序 warning，我们往往也不会去添加约束。

- 第四个就是在设置了多周期后，如果还是提示`Intra-Clocks Paths`的setup time不过，那就要看下程序，是否写的不规范。比如：

    ```verilog
    always @ (posedge clk)
    begin
        regA <= regB;

        if(regA != regB)
            regC <= 4'hf;
        else 
            regC <= {regC[2:0], 1'b0};

        if((&flag[3:0]) && regA != regB) 
            regD <= regB;
    end
    ```

    这么写的话，如果时钟频率稍微高一点，比如250MHz，就很容易导致从regB到regD的setup time不满足要求。因为begin end里面的代码都是按顺序执行的，要在4ns内完成这些赋值与判断的逻辑，挑战还是挺大的。因此，我们可以改写为：

    ```verilog
    always @ (posedge clk)
    begin
        regA <= regB;
    end 

    always @ (posedge clk)
    begin
        if(regA != regB)
            regC <= 4'hf;
        else 
            regC <= {regC[2:0], 1'b0};
    end 

    always @ (posedge phy_clk)
    begin
        if((&flag[3:0]) && regA != regB) 
            regD <= regB;
    end 
    ```

    把寄存器的赋值分开，功能还是一样的，只是分到了几个always中，这样就不会导致时序问题了。
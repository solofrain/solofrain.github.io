# Reference HDL Design for DAQ3

## 1. Overview

The JESD204B/C standard defines multiple layers, each layer being responsible for a particular function. The Analog Devices JESD204B/C HDL solution follows the standard here and defines 4 layers. Physical layer, link layer, transport layer and application layer. For the first three layers Analog Devices provides standard components that can be linked up to provide a full JESD204B/C protocol processing chain.

Depending on the FPGA and converter combinations that are being interfaced different components can be chosen for the physical and transport layer. The FPGA defines which physical layer component should be used and the interfaced converter defines which transport layer component should be used.

The link layer component is selected based on the direction of the JESD204B/C link.

The application layer is user defined and can be used to implement application specific signal processing. 

![](https://wiki.analog.com/_media/resources/fpga/peripherals/jesd204_chain.png?cache=&w=789&h=527&tok=1c08c9)

## 2. Physical Layer

Physical layer peripherals are responsible for interfacing and configuring the high-speed serial transceivers. Currently we have support for GTXE2, GTHE3, GTHE4, GTYE4 for Xilinx and Arria 10 transceivers for Intel. 

### 2.1 AXI_ADXCVR: JESD204B Gigabit Transceiver Register Configuration Peripheral

The AXI_ADXCVR is a utility core used to control and configure the highspeed transceivers instantiated in UTIL_ADXCVR. There are separate AXI_ADXCVR cores for Intel and Xilinx designs, due to the small differences between the Xilinx's and Intel's transceivers architecture.

**Features**

- Supports Intel and Xilinx devices.
- Software can access the core's registers through an AXI4 Lite Memory Mapped interface.
- Link reset and monitor for Intel and Xilinx.
- Reconfiguration interface control with broadcast capability for Xilinx.
- Access to the Statistical eye scan interface of the PHY (Xilinx).
- Supports up to 16 transceiver lanes per link. (Xilinx)

In Xilinx Devices, the core configures itself to be interfaced with the GT variant supported by the UTIL_ADXCVR core. All the transceiver primitives are configured and programmed identically.

#### 2.1.1 Parameters

Name |	Description |	Default Value
-|-|-
ID |	Instance identification number, if more than one instance is used. |	0
NUM_OF_LANES |	The number of lanes (primitives) used in this link. |	8
XCVR_TYPE |	Define the current GT type, GTXE2(2), GTHE3(5), GTHE4(7). |	0
FPGA_TECHNOLOGY |	Encoded value describing the technology/generation of the FPGA device (7series/ultrascale) |	0
FPGA_FAMILY |	Encoded value describing the family variant of the FPGA device(e.g., zynq, kintex, virtex) |	0
SPEED_GRADE |	Encoded value describing the FPGA's speed-grade |	0
DEV_PACKAGE |	Encoded value describing the device package. The package might affect high-speed interfaces	| 0
FPGA_VOLTAGE |	Contains the value(0-5000 mV) at wich the FPGA device supplied |	0
TX_OR_RX_N |	If set (0x1), configures the link in transmit mode, otherwise receive. |	0
QPLL_ENABLE |	If set (0x1), configures the link to use QPLL on QUAD basis. If multiple links are sharing the same transceiver, only one of them may enable the QPLL. |	1
LPM_OR_DFE_N |	Chosing between LPM or DFE of modes for the RX Equalizer |	1
RATE |	Defines the initial values for Transceiver Control Register (REG_CONTROL 0x0008) |	0
TX_DIFFCTRL |	Driver Swing Control(TX Configurable Driver) |	8
TX_POSTCURSOR |	Transmitter post-cursor TX pre-emphasis control. |	0
TX_PRECURSOR |	Transmitter pre-cursor TX pre-emphasis control. |	0
SYS_CLK_SEL |	Selects the PLL reference clock source to drive the RXOUTCLK Table 1 |	3
OUT_CLK_SEL |	select the transceiver reference clock as the source of TXOUTCLK Table 2 |	4

#### 2.1.2 Interface

Interface |	Pin |	Type |	Description
-|-|-|-
axi_clk |	axi_clk |	Input |	The CPU clock (assumed to be 100MHz), must be same as the DRP clock.
axi_aresetn |	axi_aresetn |	Input |	The CPU reset (internally used asynchronous to the axi_clk).
up_status |	up_status |	Output| 	If set, indicates that the link is up and active. The same information is read on the register bit (see below). This signal may be connected to the JESD204B IP reset done input.
s_axi |	Slave-AXI |	IO |	The programmable interface, must be connected to a CPU master.
m_axi |	Master-AXI |	IO |	The Eye-Scan DMA interface, must be connected to a memory slave. This interface is available only if parameter TX_OR_RX_N is set to 0x0.
up_cm_* |	Common-DRP |	IO |	The common DRP interface, must be connected to the equivalent DRP ports of UTIL_ADXCVR. This is a QUAD interface, shared by four transceiver lanes. This interface is available only if parameter QPLL_ENABLE is set to 0x1.
up_ch_* |	Channel-DRP |	IO |	The channel DRP interface, must be connected to the equivalent DRP ports of UTIL_ADXCVR. This is a channel interface, one per each transceiver lane.
up_es_* |	Eye-Scan-DRP |	IO |	The Eye-Scan DRP interface, must be connected to the equivalent DRP ports of UTIL_ADXCVR. This is a channel interface, one per each transceiver lane. This interface is available only if parameter TX_OR_RX_N is set to 0x0.

#### 2.1.3 Register Map

Address |	Bits |	Name |	Type |	Description
-|-|-|-|-
DWORD /	BYTE|||
0x0000 /	0x0000 ||	REG_VERSION ||	Version Register
||[31:0] |	VERSION[31:0] |	RO |	Version number.
0x0001 /	0x0004 ||	REG_ID ||	Instance Identification Register
||[31:0] |	ID[31:0] |	RO |	Instance identifier number.
0x0002 /	0x0008 ||	REG_SCRATCH ||	Scratch (GP R/W) Register
||[31:0] |	SCRATCH[31:0] |	RW |	Scratch register.
0x0004 /	0x0010 ||	REG_RESETN ||	Reset Control Register
||[0] |	RESETN |	RW |	If clear, link is held in reset, set this bit to 0x1 to activate link. Note that the reference clock and DRP clock must be active before setting this bit.
0x0005 /	0x0014 ||	REG_STATUS ||	Status Reporting Register
||[0] |	STATUS |	RO |	After setting the RESETN bit above, wait for this bit to set. If set, indicates successful link activation.
0x0007 /	0x001c ||	REG_FPGA_INFO ||	FPGA device information [Xilinx encoded values](https://github.com/analogdevicesinc/hdl/blob/master/library/scripts/adi_xilinx_device_info_enc.tcl)
||[31:24] |	FPGA_TECHNOLOGY |	RO |	Encoded value describing the technology/generation of the FPGA device (e.g, 7series, ultrascale)
||[23:16] |	FPGA_FAMILY |	RO |	Encoded value describing the family variant of the FPGA device(e.g., zynq, kintex, virtex)
||[15:8] |	SPEED_GRADE |	RO |	Encoded value describing the FPGA's speed-grade
||[7:0] |	DEV_PACKAGE |	RO |	Encoded value describing the device package. The package might affect high-speed interfaces
0x0008 /	0x0020 ||	REG_CONTROL ||	Transceiver Control Register
||[12] |	LPM_DFE_N |	RW |	Transceiver primitive control, refer Xilinx documentation.
||[10:8] |	RATE[2:0] |	RW |	Transceiver primitive control, refer Xilinx documentation.
||[5:4] |	SYSCLK_SEL[1:0] |	RW |	For GTX drives directly the (RX/TX)SYSCLKSEL pin of the transceiver. Refer to Xilinx documentation. <br>For GTH/GTY drives directly the (RX/TX)PLLCLKSEL pin of the transceiver and indirectly the (RX/TX)SYSCLKSEL pin of the transceiver see Table 1.
||[2:0] |	OUTCLK_SEL[2:0] |	RW |	Transceiver primitive control Table 2, refer Xilinx documentation.
0x0009 /	0x0024 ||	REG_GENERIC_INFO ||	Physical layer info
||[31:21] |	RESERVED |	RO |	0
||[20] |	QPLL_ENABLE |	RO |	Using QPLL.
||[19:16] |	XCVR_TYPE[3:0] |	RO |	[Xilinx encoded values.](https://github.com/analogdevicesinc/hdl/blob/master/library/scripts/adi_xilinx_device_info_enc.tcl)
||[15:9] |	RESERVED |	RO |	0
||[8] |	TX_OR_RX_N |	RO |	Transceiver type (transmit or receive)
||[7:0] |	NUM_OF_LANES |	RO |	Physical layer number of lanes.
0x0010 /	0x0040 ||	REG_CM_SEL ||	Transceiver Access Register
||[7:0] |	CM_SEL[7:0] |	RW |	Transceiver common-DRP sel, set to 0xff for broadcast.
0x0011 /	0x0044 ||	REG_CM_CONTROL ||	Transceiver Access Register
||[28] |	CM_WR |	RW |	Transceiver common-DRP sel, set to 0x1 for write, 0x0 for read.
||[27:16] |	CM_ADDR[11:0] |	RW |	Transceiver common-DRP read/write address.
||[15:0] |	CM_WDATA[15:0] |	RW |	Transceiver common-DRP write data.
0x0012 /	0x0048 ||	REG_CM_STATUS ||	Transceiver Access Register
||[16] |	CM_BUSY |	RO |	Transceiver common-DRP access busy (0x1) or idle (0x0).
||[15:0] |	CM_RDATA[15:0]| 	RW |	Transceiver common-DRP read data.
0x0018 /	0x0060 ||	REG_CH_SEL ||	Transceiver Access Register
||[7:0] |	CH_SEL[7:0] |	RW |	Transceiver channel-DRP sel, set to 0xff for broadcast.
0x0019 /	0x0064 ||	REG_CH_CONTROL ||	Transceiver Access Register
||[28] |	CH_WR |	RW |	Transceiver channel-DRP sel, set to 0x1 for write, 0x0 for read.
||[27:16] |	CH_ADDR[11:0] |	RW |	Transceiver channel-DRP read/write address.
||[15:0] |	CH_WDATA[15:0] |	RW |	Transceiver channel-DRP write data.
0x001a /	0x0068 ||	REG_CH_STATUS ||	Transceiver Access Register
||[16] |	CH_BUSY |	RO |	Transceiver channel-DRP access busy (0x1) or idle (0x0).
||[15:0] |	CH_RDATA[15:0] |	RW |	Transceiver channel-DRP read data.
0x0020 /	0x0080 ||	REG_ES_SEL ||	Transceiver Access Register
||[7:0] |	ES_SEL[7:0] |	RW |	Transceiver eye-scan-DRP sel, set to 0xff for broadcast.
0x0028 /	0x00a0 ||	REG_ES_REQ ||	Transceiver eye-scan Request Register
||[0] |	ES_REQ 	|RW |	Transceiver eye-scan request, set this bit to initiate an eye-scan, this bit auto-clears when scan is complete.
0x0029 /	0x00a4 ||	REG_ES_CONTROL_1 ||	Transceiver eye-scan Control Register
||[4:0] |	ES_PRESCALE[4:0] |	RW |	Transceiver eye-scan control, refer Xilinx documentation.
0x002a /	0x00a8 ||	REG_ES_CONTROL_2 ||	Transceiver eye-scan Control Register
||[25:24] |	ES_VOFFSET_RANGE[1:0] |	RW |	Transceiver eye-scan control, refer Xilinx documentation.
||[23:16] |	ES_VOFFSET_STEP[7:0] |	RW |	Transceiver eye-scan control, refer Xilinx documentation.
||[15:8] |	ES_VOFFSET_MAX[7:0] |	RW |	Transceiver eye-scan control, refer Xilinx documentation.
||[7:0] |	ES_VOFFSET_MIN[7:0] |	RW |	Transceiver eye-scan control, refer Xilinx documentation.
0x002b /	0x00ac ||	REG_ES_CONTROL_3 ||	Transceiver eye-scan Control Register
||[27:16] |	ES_HOFFSET_MAX[11:0] |	RW |	Transceiver eye-scan control, refer Xilinx documentation.
||[11:0] |	ES_HOFFSET_MIN[11:0] |	RW |	Transceiver eye-scan control, refer Xilinx documentation.
0x002c /	0x00b0 ||	REG_ES_CONTROL_4 ||	Transceiver eye-scan Control Register
||[11:0] |	ES_HOFFSET_STEP[11:0] |	RW |	Transceiver eye-scan control, refer Xilinx documentation.
0x002d /	0x00b4 ||	REG_ES_CONTROL_5 ||	Transceiver eye-scan Control Register
||[31:0] |	ES_STARTADDR[31:0] |	RW |	Transceiver eye-scan control, DMA start address (ES data is written to this memory address).
0x002e /	0x00b8 ||	REG_ES_STATUS ||	Transceiver eye-scan Status Register
||[0] |	ES_STATUS[0] |	RO |	If set, indicates an error in ES DMA.
0x0030 /	0x00c0 ||	REG_TX_DIFFCTRL ||	Transceiver primitive control, refer Xilinx documentation.
||[31:0] |	REG_TX_DIFFCTRL[31:0] |	RW 	|TX driver swing control.
0x0031 /	0x00c4 ||	REG_TX_POSTCURSOR ||	Transceiver primitive control, refer Xilinx documentation.
||[31:0] |	REG_TX_POSTCURSOR[31:0] |	RW |	Transmiter post-cursor TX pre-emphasis control.
0x0032 /	0x00c8 ||	REG_TX_PRECURSOR ||	Transceiver primitive control, refer Xilinx documentation.
||[31:0] |	REG_TX_PRECURSOR[31:0] |	RW |	Transmiter pre-cursor TX pre-emphasis control.
0x0050 /	0x0140 ||	REG_FPGA_VOLTAGE ||	FPGA device voltage information
||[15:0] |	FPGA_VOLTAGE |	RO |	The voltage of the FPGA device in mv

### 2.1.4 Software Guidelines

The system must have active DRP and reference clocks before any software access. The software is expected to write necessary control parameters to `LPM_DFE_N`, `RATE`, `SYSCLK_SEL`, `OUTCLK_SEL` register bits and then set `RESETN` bit to `0x1`. After which monitor the `STATUS` bit to be set. There are no other requirements for initialization.

The `DRP` access is identical for common and channel interfaces. The `SEL` bits may be set to a specific transceiver lane or `0xff` to broadcast. A write to the `CONTROL` register (bits `WR`, `ADDR`, `WDATA`) initiates `DRP` access in hardware. A read to this register has no effect. In order to write to the transceiver, set `WR` to `0x1` with the address. In order to read from the transceiver, set `WR` to `0x0` with the address. As soon as this register is written, the `BUSY` signal is set and is cleared only after the access is complete. The broadcast read is a logical OR of all the channels. After an access is started, do NOT interrupt the core for any reason (including setting `RESETN` to `0x0`), allow the access to finish itself. Though the core itself is immune to a software abort, the transceiver may fail on further accesses and may require a system-wide reset.

The eye-scan feature also allows a `SEL` option and a broadcast has the effect of a combined mask. That is, the error counter will be zero ONLY if all the transceiver error counters are zero. To start eye-scan, set `ES_REQ` to `0x1` and wait for the same bit to self-clear. If eye-scan needs to be stopped, set the `ES_REQ` bit to `0x0`.

Table 1
SYSCLK_SEL ​ |	00 |	01 |	10 |	11
-|-|-|-|-
GTXE2 |	CPLL |	RESERVED ​| 	RESERVED ​| 	QPLL
GTHE3 |	CPLL |	RESERVED ​| 	QPLL1 |	QPLL0
GTHE4 |	CPLL |	RESERVED ​| 	QPLL1 |	QPLL0
GTYE4 |	CPLL |	RESERVED |​ 	QPLL1 |	QPLL0

Table 2
OUTCLK_SEL ​ |	001 |	010 |	011 |	100 |	All other combinations ​
-|-|-|-|-|-
GTXE2 |	OUTCLKPCS ​ |	OUTCLKPMA ​ |	REFCLK ​ |	REFCLK/​2 ​ |	RESERVED ​
GTHE3 |	OUTCLKPCS ​ |	OUTCLKPMA ​ |	REFCLK ​ |	REFCLK/​2 ​ |	RESERVED ​
GTHE4 |	OUTCLKPCS ​ |	OUTCLKPMA ​ |	REFCLK ​ |	REFCLK/​2 ​ |	RESERVED ​
GTYE4 |	OUTCLKPCS ​ |	OUTCLKPMA ​ |	REFCLK ​ |	REFCLK/​2 ​ |	RESERVED ​

The `REFCLK` selected by `OUTCLK_SEL` depends on the `SYSCLK_SEL`, it may be `CPLL`, `QPLL0` or `QPLL1` refclk. ​


### 2.2 UTIL_ADXCVR: JESD204B Gigabit Transceiver Interface Peripheral for Xilinx FPGAs

The util_adxcvr IP core instantiate a Gigabit Transceiver (GT) and set's up the required configuration. Basically is a simple wrapper file for a GT* Column, exposing just the necessary ports and attributes.

To understand the below wiki page is important to have a basic understanding about High Speed Serial I/O interfaces and Gigabit Serial Transceivers. To find more information about these technologies please visit the Xilinx's solution center.

Currently this IP supports three different GT type:

- GTXE2 (7 Series devices)
- GTHE3 (Ultrascale and Ultrascale+)
- GTHE4 (Ultrascale and Ultrascale+)
- GTYE4 (Ultrascale and Ultrascale+)

#### 2.2.1 Features

- Supports GTX2, GTH3 and GTH4
- Exposes all the necessary attribute for QPLL/CPLL configuration
- Supports shared transceiver mode
- Support dynamic reconfiguration
- RX Eye Scan

#### 2.2.2 Block Diagram


The following diagram shows a GTXE2 Column, which contains four GT Quad. Each quad contains a GTEX2_COMMON and four GTXE2_CHANNEL primitive.

![GTXE2 Column](https://wiki.analog.com/_media/resources/fpga/docs/hdl/gtx_column.png)



#### 2.2.3 Configuration Parameters
Name |	Description |	Default Value
-|-|-
XCVR_TYPE |	Define the current GT type, GTXE2(0), GTHE3(1), GTHE4(2) |	0
QPLL_REFCLK_DIV |	QPLL reference clock divider M, see User Guide for more info |	1
QPLL_FBDIV_RATIO |	QPLL reference clock divider N ratio, see User Guide for more info |	1
QPLL_CFG |	Configuration settings for QPLL, see User Guide for more info |	27'h0680181
QPLL_FBDIV |	QPLL reference clock divider N, see User Guide for more info |	10'b0000110000
CPLL_FBDIV |	CPLL feedback divider N2 settings, see User Guide for more info |	2
CPLL_FBDIV_45 |	CPLL reference clock divider N1 settings, see User Guide for more info| 	5
TX_NUM_OF_LANES |	Number of transmit lanes. |	8
TX_OUT_DIV |	CPLL/QPLL output clock divider D for the TX datapath, see User Guide for more info |	1
TX_CLK25_DIV |	Divider for internal 25 MHz clock for the TX datapath, see User Guide for more info |	20
TX_LANE_INVERT |	Per lane polarity inversion. Set the n-th bit to invert the polarity of the n-th transmit lane. |	0
RX_NUM_OF_LANES |	Number of transmit lanes |	8
RX_OUT_DIV |	CPLL/QPLL output clock divider D for the RX datapath, see User Guide for more info |	1
RX_CLK25_DIV |	Divider for internal 25 MHz clock for the RX datapath, see User Guide for more info |	20
RX_DFE_LPM_CFG |	Configure the GT use modes, LPM or DFE, see User Guide for more info |	16'h0104
RX_PMA_CFG |	Search for PMA_RSV in User Guide for more info |	32'h001e7080
RX_CDR_CFG |	Configure the RX clock data recovery circuit for GTXE2, see User Guide for more info |	72'h0b000023ff10400020
RX_LANE_INVERT |	Per lane polarity inversion. Set the n-th bit to invert the polarity of the n-th receive lane. |	0

#### 2.2.4 Interface

Interface Pin |	Type |	Description
-|-|-
Microprocessor clock and reset||
up_clk |	input |	System clock, running on 100 MHz
up_rstn |	input |	System reset, the same as AXI memory map slave interface reset
PLL reference clock||
qpll_ref_clk_0 |	input |	Reference clock for the QPLL
cpll_ref_clk_0 |	input |	Reference clock for the CPLL
RX interface||
rx_*_p |	input |	Positive differential serial data input
rx_*_n |	input |	Negative differential serial data input
rx_out_clk_* |	output |	Core logic clock output. Frequency = serial line rate/40
rx_clk_* |	input |	Core logic clock loop-back input
rx_charisk_* |	output[ 3:0] |	RX Char is K to the JESD204B IP
rx_disperr_* |	output[ 3:0] |	RX disparity error to the JESD204B IP
rx_notintable_* |	output[ 3:0] |	RX Not In Table to the JESD204B IP
rx_data_* |	output[31:0] |	RX data to the JESD204B IP
rx_calign_* |	input |	RX enable comma alignment from the JESD204B IP
TX interface||
tx_*_p |	output |	Positive differential serial output
tx_*_n |	output |	Negative differential serial output
tx_out_clk_* |	output |	Core logic clock output. Frequency = lane rate/40
tx_clk_* |	input |	Core logic clock loop-back input
tx_charisk_* 	input[ 3:0] |	TX Char is K from the JESD204B IP
tx_data_* |	input[31:0] |	TX data from the JESD204B IP
Common DRP interface||
up_cm_* |	IO |	The common DRP interface, must be connected to the equivalent DRP ports of AXI_ADXCVR. This is a QUAD interface, shared by four transceiver lanes. This interface is available only if parameter QPLL_ENABLE is set to 0x1.
Channel DRP interface||
up_ch_* |	IO |	The channel DRP interface, must be connected to the equivalent DRP ports of AXI_ADXCVR. This is a channel interface, one per each transceiver lane.
Eye Scan DRP interface||
up_es_* |	IO |	The Eye-Scan DRP interface, must be connected to the equivalent DRP ports of UTIL_ADXCVR. This is a channel interface, one per each transceiver lane. This interface is available only if parameter TX_OR_RX_N is set to 0x0.

#### 2.2.5 Design Guidelines

For porting on different Xilinx transceiver types, there is the following [guide](https://wiki.analog.com/resources/fpga/docs/xgt_wizard).

#### 2.2.6 Software Guidelines

The software can configure this core through the AXI_ADXCVR IP core described in **2.1**.


## 3. Link Layer

Link layer peripherals are responsible for JESD204B/C protocol handling, including scrambling/descrambling, lane alignment, character replacement and alignment monitoring.

### 3.1 JESD204B/C Transmit Peripheral: JESD204B/C Link Layer Transmit Peripheral


### 3.2 JESD204B/C Receive Peripheral: JESD204B/C Link Layer Receive Peripheral



## 4. Transport Layer

Transport layer peripherals are responsible for converter specific data framing and de-framing.

- ADC JESD204B/C Transport Peripheral : JESD204B/C Transport Layer Receive Peripheral
- DAC JESD204B/C Transport Peripheral : JESD204B/C Transport Layer Transmit Peripheral

## 5. Interfaces

Interfaces are a well-defined collection of wires that are used to communicate between components. The following interfaces are used to connect components of the HDL JESD204B/C processing stack.

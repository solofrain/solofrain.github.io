# ZCU106 Overview

## 1. Clock

- Fixed frequency clocks

Clock Name | Net Name | Frequency (MHz) | Clock Source | I/O Standard | Stability | XCZU7EV Pin
-|:-:|:-:|:-:|:-:|:-:|:-:
PS_REF_CLK | PS_REF_CLK | 33.33 | U69 SI5341B | * | <100 fs RMS | R24
CLK_74_25 | CLK_74_25_P<BR>CLK_74_25_N | 74.25 | U69 SI5341B | LVDS_25 | <100 fs RMS | D15<BR>D14
CLK_125 | CLK_125_P<BR>CLK_125_N | 125 | U69 SI5341B | LVDS_25 | <100 fs RMS | H9<BR>G9
GTR_REF_CLK_SATA | GTR_REF_CLK_SATA_P<BR>GTR_REF_CLK_SATA_N | 125 | U69 SI5341B | ** | <100 fs RMS | P27<BR>P28
GTR_REF_CLK_USB3 | GTR_REF_CLK_USB3_P<BR>GTR_REF_CLK_USB3_N | 26 | U69 SI5341B | ** | <100 fs RMS | M27<BR>M28
GTR_REF_CLK_DP | GTR_REF_CLK_DP_P<BR>GTR_REF_CLK_DP_N | 27 | U69 SI5341B | ** | <100 fs RMS | M31<BR>M32

- Programmable frequency clocks

Clock Name | Net Name | Frequency (MHz) | Clock Source | I/O Standard | Stability | XCZU7EV Pin (P/N)
-|:-:|:-:|:-:|:-:|:-:|:-:
USER_SI570 | USER_SI570<BR>USER_SI570 | 10 ~ 810 (300 default) | U42 SI570 | DIFF_SSTL12 | 61.5ppm| AH12<BR>AJ12
USER_MGT_SI570 | USER_MGT_SI570_CLOCK1_P<BR>USER_MGT_SI570_CLOCK1_N<BR>USER_MGT_SI570_CLOCK2_P<BR>USER_MGT_SI570_CLOCK2_N | 10 ~ 810 (156.25 default) | U56 SI570 | ** | 61.5ppm | U10<BR>U9<BR>R10<BR>R9
USER_MGT_SMA | USER_SMA_MGT_CLOCK_P<BR>USER_SMA_MGT_CLOCK_N | User-Provided source | J79<BR>J80 | ** | AA0<BR>AA9
HDMI_SI5324_OUT | HDMI_SI5324_OUT_P<BR>HDMI_SI5324_OUT_N | Variable | U108 SI519C clock recovery | ** | AD8<BR>AD7
SFP_SISI5328_OUT | SFP_SISI5328_OUT_P<BR>SFP_SISI5328_OUT_N | Variable | U20 SI5328B clock recovery | ** | W10<BR>W9

\* XCU7EV Bank 503 suport LVCMOS level inputs
** MGT

## 2. SFP / SFP+

Bank 225 provides Quad SFP+ interface (P1/P2).

SFP+ modules typically provide an I2C based control interface through the I2C multiplexer topology (U135 -> PS I2C1).

- Pins / nets

XCZU7EV Pin | Net Name 
:-:|:-:
Y4 | SFP0_TX_P
Y3 | SFP0_TX_N
AA2 | SFP0_RX_P
AA1 | SFP0_RX_N
AE22 | SFP0_TX_DISABLE_B
W6 | SFP1_TX_P
W5 | SFP1_TX_N
W2 | SFP1_RX_P
W1 | SFP1_RX_N
AF20 | SFP1_TX_DISABLE_B

- Clock recovery

XCZU7EV Bank | Pin | Net Name (Rx) | Net Name (Rx) | XCZU7EV Bank | Pin 
:-:|:-:|:-:|:-:|:-:|:-:
68 | H11 | SFP_REC_CLOCK_C_P | SFP_SI5328_OUT_C_P | 225 | W10
68 | G11 | SFP_REC_CLOCK_C_N | SFP_SI5328_OUT_C_N | 225 | W9

## 3. I2C1 (MIO 16-17)

U135 I2C1 Mux (ADDR 0x75) Port | I2C1 Bus Device | Target Device Address
:-:|:-:|:-:
6 | SFP1 P2 | 0x50
7 | SFP0 P1 | 0x50


## 4. MSP30 System Controller

MSP430 is controlled through a system controller user interface (SCUI) that can be downloaded on [ZCU106 web page](https://www.xilinx.com/products/boards-and-kits/zcu106.html#documentation).

- clocks
- FMC functionality
- power system parameters


## 5. GPIO

- User PMOD GPIO Headers

Header | Voltage | Nets
:-:|:-:|:-:
J55 | 3.3 V | PMOD0_[7:0]
J78 | 3.3 V | PMOD1_[7:0]

- Prototype Header

J3: 10 nets

- User I/O

    - 8 user LEDs: GPIO_LED[7:0]: DS[44:37]
    - 5 user pushbuttons: SW[18:15]
    - 1 CPU reset switch: SW20
    - 8 user DIP switchs: GPIO_DIP_SW[7:0S]

## 6. Switches

- Program_B pushbutton (SW5)
    Clears PL configuration and reprogram by PS.

- System reset pushbutton - PS_POR_B (SW4)

- Power-On-Reset pushbutton - PS_POR_B (SW4)

    Holds PS in reset until all PS power supplies are on.

- System reset pushbutton - PS_SRST_B (SW3)
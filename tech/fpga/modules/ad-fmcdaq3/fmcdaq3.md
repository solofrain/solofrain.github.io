Analog Devices FMCDAQ3 EBZ Rapid Prototyping FMC Module
===
    
- Wiki page (schematic, layout, HDL, software drivers)
       
    [https://wiki.analog.com/resources/eval/user-guides/ad-fmcdaq3-ebz](https://wiki.analog.com/resources/eval/user-guides/ad-fmcdaq3-ebz){:target="_blank"}

    - [Schematic](https://wiki.analog.com/_media/resources/eval/user-guides/ad-fmcdaq3-ebz/fmcdaq3_revc.pdf){:target="_blank"}

    - [FMC pinout](daq3-fmc-pins.xlsx){:target="_blank"}

        | FMC PIN | FMC NET         | DAQ3 NET             |
        |---------|-----------------|----------------------|
        | A2      | DP1\_M2C\_P     | FMC\_SERDOUT3\_P     |
        | A3      | DP1\_M2C\_N     | FMC\_SERDOUT3\_N     |
        | A6      | DP2\_M2C\_P     | FMC\_SERDOUT2\_P     |
        | A7      | DP2\_M2C\_N     | FMC\_SERDOUT2\_N     |
        | A10     | DP3\_M2C\_P     | FMC\_SERDOUT0\_P     |
        | A11     | DP3\_M2C\_N     | FMC\_SERDOUT0\_N     |
        | A14     | DP4\_M2C\_P     | DLOGIC\_0            |
        | A15     | DP4\_M2C\_N     | DLOGIC\_1            |
        | A18     | DP5\_M2C\_P     | DLOGIC\_0            |
        | A19     | DP5\_M2C\_N     | DLOGIC\_1            |
        | A22     | DP1\_C2M\_P     | FMC\_SERDIN2\_P      |
        | A23     | DP1\_C2M\_N     | FMC\_SERDIN2\_N      |
        | A26     | DP2\_C2M\_P     | FMC\_SERDIN1\_P      |
        | A27     | DP2\_C2M\_N     | FMC\_SERDIN1\_N      |
        | A30     | DP3\_C2M\_P     | FMC\_SERDIN0\_P      |
        | A31     | DP3\_C2M\_N     | FMC\_SERDIN0\_N      |
        | B1      | CLK\_DIR        | FMC\_CLK\_DIR        |
        | B4      | DP9\_M2C\_P     | DLOGIC\_0            |
        | B5      | DP9\_M2C\_N     | DLOGIC\_1            |
        | B8      | DP8\_M2C\_P     | DLOGIC\_0            |
        | B9      | DP8\_M2C\_N     | DLOGIC\_1            |
        | B12     | DP7\_M2C\_P     | DLOGIC\_0            |
        | B13     | DP7\_M2C\_N     | DLOGIC\_1            |
        | B16     | DP6\_M2C\_P     | DLOGIC\_0            |
        | B17     | DP6\_M2C\_N     | DLOGIC\_1            |
        | B20     | GBTCLK1\_M2C\_P | ADC\_CLK\_FMC\_P     |
        | B21     | GBTCLK1\_M2C\_N | ADC\_CLK\_FMC\_N     |
        | C2      | DP0\_C2M\_P     | FMC\_SERDIN3\_P      |
        | C3      | DP0\_C2M\_N     | FMC\_SERDIN3\_N      |
        | C6      | DP0\_M2C\_P     | FMC\_SERDOUT1\_P     |
        | C7      | DP0\_M2C\_N     | FMC\_SERDOUT1\_N     |
        | C10     | LA06\_P         | FMC\_ADC\_PD         |
        | C11     | LA06\_N         | FMC\_SPI\_SDIO\_CTRL |
        | C14     | LA10\_P         | FMC\_DAC\_SPI\_CSB   |
        | C30     | SCL             | FMC\_SCL             |
        | C31     | SDA             | FMC\_SDA             |
        | C34     | GA0             | FMC\_GA0             |
        | D1      | PG\_C2M         | FMC\_PG\_C2M         |
        | D4      | GBTCLK0\_M2C\_P | DAC\_CLK\_FMC\_P     |
        | D5      | GBTCLK0\_M2C\_N | DAC\_CLK\_FMC\_N     |
        | D8      | LA01\_P\_CC     | ADC\_SYNC\_P         |
        | D9      | LA01\_N\_CC     | ADC\_SYNC\_N         |
        | D11     | LA05\_P         | FMC\_CLKD\_SPI\_CSB  |
        | D12     | LA05\_N         | FMC\_SPI\_SCLK       |
        | D14     | LA09\_P         | FMC\_SPI\_SDIO       |
        | D15     | LA09\_N         | FMC\_ADC\_SPI\_CSB   |
        | D17     | LA13\_P         | FMC\_SYSREF\_P       |
        | D18     | LA13\_N         | FMC\_SYSREF\_N       |
        | D20     | LA17\_P\_CC     | DAC\_REFCLK\_P       |
        | D21     | LA17\_N\_CC     | DAC\_REFCLK\_N       |
        | D35     | GA1             | FMC\_GA1             |
        | G6      | LA00\_P\_CC     | ADC\_REFCLK\_P       |
        | G7      | LA00\_N\_CC     | ADC\_REFCLK\_N       |
        | G9      | LA03\_P         | CLKD\_SYSREF\_ADC\_P |
        | G10     | LA03\_N         | CLKD\_SYSREF\_ADC\_N |
        | G12     | LA08\_P         | FMC\_CLKD\_STATUS0   |
        | G13     | LA08\_N         | FMC\_CLKD\_STATUS1   |
        | G15     | LA12\_P         | FMC\_DAC\_IRQ        |
        | G16     | LA12\_N         | FMC\_DAC\_TXEN       |
        | H4      | CLK0\_M2C\_P    | DLOGIC\_0            |
        | H5      | CLK0\_M2C\_N    | DLOGIC\_1            |
        | H7      | LA02\_P         | DAC\_SYNC\_P         |
        | H8      | LA02\_N         | DAC\_SYNC\_N         |
        | H10     | LA04\_P         | CLKD\_SYSREF\_DAC\_P |
        | H11     | LA04\_N         | CLKD\_SYSREF\_DAC\_N |
        | H13     | LA07\_P         | EXT\_TRIG\_P         |
        | H14     | LA07\_N         | EXT\_TRIG\_N         |
        | H16     | LA11\_P         | FMC\_ADC\_FDA        |
        | H17     | LA11\_N         | FMC\_ADC\_FDB        |
        | J2      | CLK3\_BIDIR\_P  | DLOGIC\_0            |
        | J3      | CLK3\_BIDIR\_N  | DLOGIC\_1            |
        | K4      | CLK2\_BIDIR\_P  | DLOGIC\_0            |
        | K5      | CLK2\_BIDIR\_N  | DLOGIC\_1            |

    - [AD-FMCDAQ3-EBZ HDL Reference Design](https://wiki.analog.com/resources/eval/user-guides/ad-fmcdaq3-ebz/reference_hdl){:target="_blank"}

    - [AD-FMCDAQ3-EBZ HDL Reference Design (ZCU102)
        - [Document](https://wiki.analog.com/resources/eval/user-guides/ad-fmcdaq3-ebz/reference_hdl)
        
        - [Source code](https://github.com/analogdevicesinc/hdl/tree/master/projects/daq3/zcu102){:target="_blank"}

        - [Building HDL](https://wiki.analog.com/resources/fpga/docs/build){:target="_blank"}

- Analog Devices EngineerZone support community
        
    [https://ez.analog.com/fpga](https://ez.analog.com/fpga){:target="_blank"}
        

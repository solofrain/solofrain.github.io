# Migrating ADI DAQ3 ZCU102 Project to ZCU106

> [Porting ADI's HDL reference designs](https://wiki.analog.com/resources/fpga/docs/hdl/porting_project_quick_start_guide){:target="_blank"}



![](https://wiki.analog.com/_media/resources/eval/user-guides/ad-fmcdaq3-ebz/parts_list.png)


![AD-FMCDAQ3-EBZ Functional Overview](https://wiki.analog.com/_media/resources/eval/user-guides/ad-fmcdaq3-ebz/hardware/daq3_block_diagram.png)


ADI provide DAQ3 project for ZCU102. This article describes how to migrate it to ZCU106.

## 1. Get source code

Clone the Github project to local disk:

<https://github.com/analogdevicesinc/hdl>{:target="_blank"}

The directory of the project is $(adi).

## 2. Modify source code

### 2.1 Prepare ZCU106 board files

 Navigate to `$(adi)/projects/common`. Make a copy of `zcu102/` folder, name this folder and the files inside from `*zcu102*` to `*zcu106*`.

This folder contains the following files:

- **system_project.tcl** - This script is creating the actual Vivado project and runs the synthesis/implementation of the design. The name of the carrier must be updated.

- **system_bd.tcl** - In this file is sourced the base design's Tcl script and the board design's Tcl script. The name of the carrier must be updated.

- **system_constr.xdc** - Constraint files of the board design. Here is defined the FMC IO's and board specific clock signals. All the IO definition must be updated, with the new pin names.

- **system_top.v** - Top wrapper file, in which the system_wrapper.v module is instantiated, and a few I/O macros are defined. The IO port of this Verilog module will be connected to actual IO pads of the FPGA. The simplest way to update the system_top is to let the synthesis fail and the tool will tell which ports are missing or which ports are redundant. The first thing to do after the failure is to verify the instantiation of the system_wrapper.v. This file is a tool generated file and can be found at `<project_name>.srcs/sources_1/bd/system/hdl/system_wrapper.v`. Fixing the instantiation of the wrapper module in most cases eliminates all the errors. If you get errors that you can not fix, ask for support.

- **Makefile** - This is an auto-generated file, but after updating the carrier name, should work with the new project without an issue.


### 2.2 Modify block design file.

`zcu106_system_bd.tcl` describes the base block design. Modify it as following:

```tcl
```

### 2.3 Modify constraint file.

`zcu106_system_constr.xdc` is IO constraint file for the base design. Will contain IO definitions for GPIO, switches, LEDs or other peripherals of the board. Modify it as following:

```tcl
# constraints
# gpio (switches, leds and such)

set_property  -dict {PACKAGE_PIN  A17   IOSTANDARD LVCMOS18} [get_ports gpio_bd_i[0]]           ; ## GPIO_DIP_SW0
set_property  -dict {PACKAGE_PIN  A16   IOSTANDARD LVCMOS18} [get_ports gpio_bd_i[1]]           ; ## GPIO_DIP_SW1
set_property  -dict {PACKAGE_PIN  B16   IOSTANDARD LVCMOS18} [get_ports gpio_bd_i[2]]           ; ## GPIO_DIP_SW2
set_property  -dict {PACKAGE_PIN  B15   IOSTANDARD LVCMOS18} [get_ports gpio_bd_i[3]]           ; ## GPIO_DIP_SW3
set_property  -dict {PACKAGE_PIN  A15   IOSTANDARD LVCMOS18} [get_ports gpio_bd_i[4]]           ; ## GPIO_DIP_SW4
set_property  -dict {PACKAGE_PIN  A14   IOSTANDARD LVCMOS18} [get_ports gpio_bd_i[5]]           ; ## GPIO_DIP_SW5
set_property  -dict {PACKAGE_PIN  B14   IOSTANDARD LVCMOS18} [get_ports gpio_bd_i[6]]           ; ## GPIO_DIP_SW6
set_property  -dict {PACKAGE_PIN  B13   IOSTANDARD LVCMOS18} [get_ports gpio_bd_i[7]]           ; ## GPIO_DIP_SW7

set_property  -dict {PACKAGE_PIN  AC14  IOSTANDARD LVCMOS12} [get_ports gpio_bd_i[8]]           ; ## GPIO_SW_E
set_property  -dict {PACKAGE_PIN  AP20  IOSTANDARD LVCMOS12} [get_ports gpio_bd_i[9]]           ; ## GPIO_SW_S
set_property  -dict {PACKAGE_PIN  AG13  IOSTANDARD LVCMOS12} [get_ports gpio_bd_i[10]]          ; ## GPIO_SW_N
set_property  -dict {PACKAGE_PIN  AK12  IOSTANDARD LVCMOS12} [get_ports gpio_bd_i[11]]          ; ## GPIO_SW_W
set_property  -dict {PACKAGE_PIN  AL10  IOSTANDARD LVCMOS12} [get_ports gpio_bd_i[12]]          ; ## GPIO_SW_C

set_property  -dict {PACKAGE_PIN  AL11  IOSTANDARD LVCMOS12} [get_ports gpio_bd_o[0]]           ; ## GPIO_LED_0
set_property  -dict {PACKAGE_PIN  AL13  IOSTANDARD LVCMOS12} [get_ports gpio_bd_o[1]]           ; ## GPIO_LED_1
set_property  -dict {PACKAGE_PIN  AK13  IOSTANDARD LVCMOS12} [get_ports gpio_bd_o[2]]           ; ## GPIO_LED_2
set_property  -dict {PACKAGE_PIN  AE15  IOSTANDARD LVCMOS12} [get_ports gpio_bd_o[3]]           ; ## GPIO_LED_3
set_property  -dict {PACKAGE_PIN  AM8   IOSTANDARD LVCMOS12} [get_ports gpio_bd_o[4]]           ; ## GPIO_LED_4
set_property  -dict {PACKAGE_PIN  AM9   IOSTANDARD LVCMOS12} [get_ports gpio_bd_o[5]]           ; ## GPIO_LED_5
set_property  -dict {PACKAGE_PIN  AM10  IOSTANDARD LVCMOS12} [get_ports gpio_bd_o[6]]           ; ## GPIO_LED_6
set_property  -dict {PACKAGE_PIN  AM11  IOSTANDARD LVCMOS12} [get_ports gpio_bd_o[7]]           ; ## GPIO_LED_7

# Define SPI clock
create_clock -name spi0_clk      -period 40   [get_pins -hier */EMIOSPI0SCLKO]
create_clock -name spi1_clk      -period 40   [get_pins -hier */EMIOSPI1SCLKO]
```

### 2.4 Define ZCU106  board and its device in the project flow script.

In `${adi}/projects/scripts/adi_project_xilinx.tcl`, add:

```tcl
if [regexp "_zcu106$" $project_name] {
    set p_device "xczu7ev-ffvc1156-2-e"
    set p_board [lindex [lsearch -all -inline [get_board_parts] *zcu106*] end]
    set sys_zynq 2
}
```

> The valid board parts and parts can be retrieved by running commands `get_parts` and `get_board_parts` in Tcl console. Run the commands like join [get_parts] \n, so each part name will be listed on a separate line.


## 3. Build

### 3.1 Environment set up

- Set environment variable to ignore Vivado version check:

  ```sh
  export ADI_IGNORE_VERSION_CHECK=1
  ```

- Enable utilization report when necessary:

  ```sh
  export ADI_GENERATE_UTILIZATION=1
  ```

- Enable OOC synthesis

  ```sh
  export ADI_USE_OOC_SYNTHESIS=1
  ```

- Run `make` to build.


## 4. Hardware 

- **Transmit**
Key component: `AD9152` **Dual**, **16-Bit**, **2500 MSPS**, TxDAC+® Digital-to-Analog JESD204B Converter with offset, phase and gain compensation.
The reference design generates the signals for AD9152 either from an internal DDS or external memory (via VDMA). The internal DDS consists of four independent signal generators with programmable phase offset and frequency. These four signal generators are paired to create two tones that are interleaved and driven to the DAC.

- **Receive**
Key component: `AD9680` **14-Bit**, **1250 MSPS**, **Dual** Analog-to-Digital JESD204B Converter (ADC).
The reference design transfers the received data to DDR via DMA. An optional off-line FFT core may be used to generate a spectrum plot.

- **Clocking**
Key component: `AD9528` Low Jitter Clock Generator (1MHz to 1.25GHz) with 14 Outputs.
The system is clocked through an **on board crystal (100MHz)**. The clock path mainly consists the AD9528 which up converts this signal to ~3.7GHz, and then divides this back down to any integer divider of this ~3.7GHz output.
The default reference design that ADI provides does the following:
    - Crystal generates a fixed clock frequency of 100MHz.
    - This clock is sent to the AD9528.
    - The AD9528 takes this, and creates:
        - **1233 MHz** for the DAC sample rate
        - **1233 MHz** for the ADC sample rate
        - **616 MHz** for the reference clocks to FPGA
    - Use [Eval board software](http://www.analog.com/EVAL-AD9528) to change settings of AD9528.


## 5. HDL

![](daq3.svg)

- Digital Interface

  The digital interface consists of 4 transmit and 4 receive lanes running at 12.33Gbps (default). The transceivers then interface to the cores at 128bits@308.25MHz. The data is sent or received based on the configuration (programmable) from separate transmit and receive chains.

- DAC Interface

  The DAC data may be sourced from an internal data generator (DDS, pattern or PRBS) or from the external DDR via DMA. The internal DDS phase and frequency are programmable.

- ADC Interface

  The ADC data is sent to the DDR via DMA. The core also supports PN monitoring at the sample level. This is different from the JESD204B specific PN sequence (though they both claim to be from the same equation).

- Control and SPI

  The device control and monitor signals are interfaced to a GPIO module. The SPI signals are controlled by a separate AXI based SPI core.

### 5.1 JESD204 Interface Framework

<https://wiki.analog.com/resources/fpga/peripherals/jesd204>



## Appendix 

### A. Dependence

- DAQ3 project-specific (from `$adi/projects/daq3/zcu106/Makefile`):

```makefile
PROJECT_NAME := daq3_zcu106

M_DEPS += ../common/daq3_spi.v
M_DEPS += ../common/daq3_bd.tcl
M_DEPS += ../../common/zcu106/zcu106_system_constr.xdc
M_DEPS += ../../common/zcu106/zcu106_system_bd.tcl
M_DEPS += ../../common/xilinx/dacfifo_bd.tcl
M_DEPS += ../../../library/xilinx/common/ad_iobuf.v
M_DEPS += ../../../library/jesd204/scripts/jesd204.tcl

LIB_DEPS += axi_ad9152
LIB_DEPS += axi_ad9680
LIB_DEPS += axi_dmac
LIB_DEPS += axi_sysid
LIB_DEPS += jesd204/axi_jesd204_rx
LIB_DEPS += jesd204/axi_jesd204_tx
LIB_DEPS += jesd204/jesd204_rx
LIB_DEPS += jesd204/jesd204_tx
LIB_DEPS += sysid_rom
LIB_DEPS += util_dacfifo
LIB_DEPS += util_pack/util_cpack2
LIB_DEPS += util_pack/util_upack2
LIB_DEPS += xilinx/axi_adxcvr
LIB_DEPS += xilinx/util_adxcvr

include ../../scripts/project-xilinx.mk
```

- General for ADI projects (from `$adi/library/scripts/project-xilinx.mk`)

```makefile
# Assumes this file is in prpojects/scripts/project-xilinx.mk
HDL_PROJECT_PATH := $(subst scripts/project-xilinx.mk,,$(lastword $(MAKEFILE_LIST)))
HDL_LIBRARY_PATH := $(HDL_PROJECT_PATH)../library/

include $(HDL_PROJECT_PATH)../quiet.mk

VIVADO := vivado -mode batch -source

CLEAN_TARGET := *.cache
CLEAN_TARGET += *.data
CLEAN_TARGET += *.xpr
CLEAN_TARGET += *.log
CLEAN_TARGET += *.jou
CLEAN_TARGET +=  xgui
CLEAN_TARGET += *.runs
CLEAN_TARGET += *.srcs
CLEAN_TARGET += *.sdk
CLEAN_TARGET += *.hw
CLEAN_TARGET += *.sim
CLEAN_TARGET += .Xil
CLEAN_TARGET += *.ip_user_files
CLEAN_TARGET += *.str
CLEAN_TARGET += mem_init_sys.txt
CLEAN_TARGET += *.csv

# Common dependencies that all projects have
M_DEPS += system_project.tcl
M_DEPS += system_bd.tcl
M_DEPS += $(wildcard system_top*.v)
M_DEPS += $(wildcard system_constr.xdc) # Not all projects have this file
M_DEPS += $(HDL_PROJECT_PATH)scripts/adi_project_xilinx.tcl
M_DEPS += $(HDL_PROJECT_PATH)scripts/adi_env.tcl
M_DEPS += $(HDL_PROJECT_PATH)scripts/adi_board.tcl

M_DEPS += $(foreach dep,$(LIB_DEPS),$(HDL_LIBRARY_PATH)$(dep)/component.xml)

.PHONY: all lib clean clean-all
all: lib $(PROJECT_NAME).sdk/system_top.hdf

clean:
	-rm -f reference.dcp
	$(call clean, \
		$(CLEAN_TARGET), \
		$(HL)$(PROJECT_NAME)$(NC) project)

clean-all: clean
	@for lib in $(LIB_DEPS); do \
		$(MAKE) -C $(HDL_LIBRARY_PATH)$${lib} clean; \
	done

MODE ?= "default"

$(PROJECT_NAME).sdk/system_top.hdf: $(M_DEPS)
	@if [ $(MODE) = incr ]; then \
		if [ -f */impl_1/system_top_routed.dcp ]; then \
			echo Found previous run result at `ls */impl_1/system_top_routed.dcp`; \
			cp -u */impl_1/system_top_routed.dcp ./reference.dcp ; \
		fi; \
		if [ -f ./reference.dcp ]; then \
			echo Using reference checkpoint for incremental compilation; \
		fi; \
	else \
		rm -f reference.dcp; \
	fi;
	-rm -rf $(CLEAN_TARGET)
	$(call build, \
		$(VIVADO) system_project.tcl, \
		$(PROJECT_NAME)_vivado.log, \
		$(HL)$(PROJECT_NAME)$(NC) project)

lib:
	@for lib in $(LIB_DEPS); do \
		$(MAKE) -C $(HDL_LIBRARY_PATH)$${lib} xilinx || exit $$?; \
	done
```

### B. Compilation flow

- **system_project.tcl**

    ```tcl
    # 
    source ../../scripts/adi_env.tcl

    #----------------------------------------------------------
    # Set up Vivado, board, device, project
    # Synthesize, implement
    # Timing analysis, utilization report
    # Write bitstream, hwdef, sysdef
    source $ad_hdl_dir/projects/scripts/adi_project_xilinx.tcl
    ## Creates a Xilinx project.
    #    proc adi_project
    #
    ## Add source files to an exiting project.
    #    proc adi_project_files
    #
    ## Run an existing project (generate bit stream).
    #    proc adi_project_run
    #
    ## Run synthesis on an partial design; use it in Partial Reconfiguration flow.
    #    proc adi_project_synth
    #
    ## Run implementation on an partial design; use it in Partial Reconfiguration
    ## flow.
    #    proc adi_project_impl
    #
    ## Verify an implemented partial reconfiguration design, checks if all the
    ## partial design are compatible with the base design.
    #    proc adi_project_verify
    #----------------------------------------------------------


    # 
    source $ad_hdl_dir/projects/scripts/adi_board.tcl

    adi_project daq3_zcu106
    adi_project_files daq3_zcu106 [list \
    "../common/daq3_spi.v" \
    "system_top.v" \
    "system_constr.xdc"\
    "$ad_hdl_dir/library/xilinx/common/ad_iobuf.v" \
    "$ad_hdl_dir/projects/common/zcu106/zcu106_system_constr.xdc" ]

    adi_project_run daq3_zcu106
    ```

- **system_top.tcl**

  ```tcl

  ## Define the supported tool version
  if {![info exists REQUIRED_VIVADO_VERSION]} {
    set REQUIRED_VIVADO_VERSION "2019.1"
  }

  ## Define the ADI_IGNORE_VERSION_CHECK environment variable to skip version check
  if {[info exists ::env(ADI_IGNORE_VERSION_CHECK)]} {
    set IGNORE_VERSION_CHECK 1
  } elseif {![info exists IGNORE_VERSION_CHECK]} {
    set IGNORE_VERSION_CHECK 0
  }

  ## Define the ADI_USE_OOC_SYNTHESIS environment variable to enable out of context
  #  synthesis
  if {[info exists ::env(ADI_USE_OOC_SYNTHESIS)]} {
    set ADI_USE_OOC_SYNTHESIS 1
  } elseif {![info exists ADI_USE_OOC_SYNTHESIS]} {
    set ADI_USE_OOC_SYNTHESIS 0
  }

  ## Set to enable incremental compilation
  set ADI_USE_INCR_COMP 1

  ## Set to enable power optimization
  set ADI_POWER_OPTIMIZATION 0

  ## Initialize global variables
  set p_board "not-applicable"
  set p_device "none"
  set sys_zynq 1

  set p_prcfg_init ""
  set p_prcfg_list ""
  set p_prcfg_status ""

  ## Creates a Xilinx project.
  #
  # \param[project_name] - name of the project
  # \param[mode] - if set non-project mode will be used, otherwise project mode
  # flow, see UG892 for more information
  # \param[parameter_list] - a list of global parameters (parameters of the
  # system_top module)
  #
  # Supported carrier names are: ac701, kc705, vc707, vcu118, kcu105, zed,
  # microzed, zc702, zc706, mitx405, zcu102.
  #
  proc adi_project {project_name {mode 0} {parameter_list {}} } {

    global ad_hdl_dir
    global ad_ghdl_dir
    global p_board
    global p_device
    global sys_zynq
    global REQUIRED_VIVADO_VERSION
    global IGNORE_VERSION_CHECK
    global ADI_USE_OOC_SYNTHESIS
    global ADI_USE_INCR_COMP

    if [regexp "_ac701$" $project_name] {
      set p_device "xc7a200tfbg676-2"
      set p_board [lindex [lsearch -all -inline [get_board_parts] *ac701*] end]
      set sys_zynq 0
    }
    if [regexp "_kc705$" $project_name] {
      set p_device "xc7k325tffg900-2"
      set p_board [lindex [lsearch -all -inline [get_board_parts] *kc705*] end]
      set sys_zynq 0
    }
    if [regexp "_vc707$" $project_name] {
      set p_device "xc7vx485tffg1761-2"
      set p_board [lindex [lsearch -all -inline [get_board_parts] *vc707*] end]
      set sys_zynq 0
    }
    if [regexp "_vcu118$" $project_name] {
      set p_device "xcvu9p-flga2104-2L-e"
      set p_board [lindex [lsearch -all -inline [get_board_parts] *vcu118*] end]
      set sys_zynq 0
    }
    if [regexp "_kcu105$" $project_name] {
      set p_device "xcku040-ffva1156-2-e"
      set p_board [lindex [lsearch -all -inline [get_board_parts] *kcu105*] end]
      set sys_zynq 0
    }
    if [regexp "_zed$" $project_name] {
      set p_device "xc7z020clg484-1"
      set p_board [lindex [lsearch -all -inline [get_board_parts] *zed*] end]
      set sys_zynq 1
    }
    if [regexp "_coraz7s$" $project_name] {
      set p_device "xc7z007sclg400-1"
      set p_board "not-applicable"
      set sys_zynq 1
    }
    if [regexp "_microzed$" $project_name] {
      set p_device "xc7z010clg400-1"
      set p_board "not-applicable"
      set sys_zynq 1
    }
    if [regexp "_zc702$" $project_name] {
      set p_device "xc7z020clg484-1"
      set p_board [lindex [lsearch -all -inline [get_board_parts] *zc702*] end]
      set sys_zynq 1
    }
    if [regexp "_zc706$" $project_name] {
      set p_device "xc7z045ffg900-2"
      set p_board [lindex [lsearch -all -inline [get_board_parts] *zc706*] end]
      set sys_zynq 1
    }
    if [regexp "_mitx045$" $project_name] {
      set p_device "xc7z045ffg900-2"
      set p_board "not-applicable"
      set sys_zynq 1
    }
    if [regexp "_zcu102$" $project_name] {
      set p_device "xczu9eg-ffvb1156-2-e"
      set p_board [lindex [lsearch -all -inline [get_board_parts] *zcu102*] end]
      set sys_zynq 2
    }
    if [regexp "_zcu106$" $project_name] {
      set p_device "xczu7ev-ffvc1156-2-e"
      set p_board [lindex [lsearch -all -inline [get_board_parts] *zcu106*] end]
      set sys_zynq 2
    }

    set VIVADO_VERSION [version -short]
    if {$IGNORE_VERSION_CHECK} {
      if {[string compare $VIVADO_VERSION $REQUIRED_VIVADO_VERSION] != 0} {
        puts -nonewline "CRITICAL WARNING: vivado version mismatch; "
        puts -nonewline "expected $REQUIRED_VIVADO_VERSION, "
        puts -nonewline "got $VIVADO_VERSION.\n"
      }
    } else {
      if {[string compare $VIVADO_VERSION $REQUIRED_VIVADO_VERSION] != 0} {
        puts -nonewline "ERROR: vivado version mismatch; "
        puts -nonewline "expected $REQUIRED_VIVADO_VERSION, "
        puts -nonewline "got $VIVADO_VERSION.\n"
        puts -nonewline "This ERROR message can be down-graded to CRITICAL WARNING by setting ADI_IGNORE_VERSION_CHECK environment variable to 1. Be aware that ADI will not support you, if you are using a different tool version.\n"
        exit 2
      }
    }

    if {$mode == 0} {
      set project_system_dir "./$project_name.srcs/sources_1/bd/system"
      create_project $project_name . -part $p_device -force
    } else {
      set project_system_dir ".srcs/sources_1/bd/system"
      create_project -in_memory -part $p_device
    }

    if {$mode == 1} {
      file mkdir $project_name.data
    }

    if {$p_board ne "not-applicable"} {
      set_property board_part $p_board [current_project]
    }

    set lib_dirs $ad_hdl_dir/library
    if {$ad_hdl_dir ne $ad_ghdl_dir} {
      lappend lib_dirs $ad_ghdl_dir/library
    }

    # Set a common IP cache for all projects
    if {$ADI_USE_OOC_SYNTHESIS == 1} {
      if {[file exists $ad_hdl_dir/ipcache] == 0} {
        file mkdir $ad_hdl_dir/ipcache
      }
      config_ip_cache -import_from_project -use_cache_location $ad_hdl_dir/ipcache
    }

    set_property ip_repo_paths $lib_dirs [current_fileset]
    update_ip_catalog

    ## Load custom message severity definitions
    source $ad_hdl_dir/projects/scripts/adi_xilinx_msg.tcl

    ## In Vivado there is a limit for the number of warnings and errors which are
    ## displayed by the tool for a particular error or warning; the default value
    ## of this limit is 100.
    ## Overrides the default limit to 2000.
    set_param messaging.defaultLimit 2000

    # Set parameters of the top level file
    # Make the same parameters available to system_bd.tcl
    set proj_params [get_property generic [current_fileset]]
    foreach {param value} $parameter_list {
      lappend proj_params $param=$value
      set ad_project_params($param) $value
    }
    set_property generic $proj_params [current_fileset]

    create_bd_design "system"
    source system_bd.tcl

    save_bd_design
    validate_bd_design

    if {$ADI_USE_OOC_SYNTHESIS == 1} {
      set_property synth_checkpoint_mode Hierarchical [get_files  $project_system_dir/system.bd]
    } else {
      set_property synth_checkpoint_mode None [get_files  $project_system_dir/system.bd]
    }
    generate_target {synthesis implementation} [get_files  $project_system_dir/system.bd]
    if {$ADI_USE_OOC_SYNTHESIS == 1} {
      export_ip_user_files -of_objects [get_files  $project_system_dir/system.bd] -no_script -sync -force -quiet
      create_ip_run [get_files  $project_system_dir/system.bd]
    }
    make_wrapper -files [get_files $project_system_dir/system.bd] -top

    if {$mode == 0} {
      import_files -force -norecurse -fileset sources_1 $project_system_dir/hdl/system_wrapper.v
    } else {
      write_hwdef -file "$project_name.data/$project_name.hwdef"
    }

    if {$ADI_USE_INCR_COMP == 1} {
      if {[file exists ./reference.dcp]} {
        set_property incremental_checkpoint ./reference.dcp [get_runs impl_1]
      }
    }

  }

  ## Add source files to an exiting project.
  #
  # \param[project_name] - name of the project
  # \param[project_files] - list of project files
  #
  proc adi_project_files {project_name project_files} {

    foreach pfile $project_files {
      if {[string range $pfile [expr 1 + [string last . $pfile]] end] == "xdc"} {
        add_files -norecurse -fileset constrs_1 $pfile
      } else {
        add_files -norecurse -fileset sources_1 $pfile
      }
    }

    # NOTE: top file name is always system_top
    set_property top system_top [current_fileset]
  }

  ## Run an existing project (generate bit stream).
  #
  # \param[project_name] - name of the project
  #
  proc adi_project_run {project_name} {

    global ADI_POWER_OPTIMIZATION
    global ADI_USE_OOC_SYNTHESIS

    if {$ADI_USE_OOC_SYNTHESIS == 1} {
      launch_runs -jobs 4 system_*_synth_1 synth_1
    } else {
      launch_runs synth_1
    }
    wait_on_run synth_1
    open_run synth_1
    report_timing_summary -file timing_synth.log

    if {![info exists ::env(ADI_NO_BITSTREAM_COMPRESSION)] && ![info exists ADI_NO_BITSTREAM_COMPRESSION]} {
      set_property BITSTREAM.GENERAL.COMPRESS TRUE [current_design]
    }

    if {$ADI_POWER_OPTIMIZATION == 1} {
    set_property STEPS.POWER_OPT_DESIGN.IS_ENABLED true [get_runs impl_1]
    set_property STEPS.POST_PLACE_POWER_OPT_DESIGN.IS_ENABLED true [get_runs impl_1]
    }

    launch_runs impl_1 -to_step write_bitstream
    wait_on_run impl_1
    open_run impl_1
    report_timing_summary -warn_on_violation -file timing_impl.log

    if {[info exists ::env(ADI_GENERATE_UTILIZATION)]} {
      set csv_file resource_utilization.csv
      if {[ catch {
        xilinx::designutils::report_failfast -csv -file $csv_file -transpose -no_header -ignore_pr -quiet
        set MMCM [llength [get_cells -hierarchical -filter { PRIMITIVE_TYPE =~ *MMCM* }]]
        set PLL [llength [get_cells -hierarchical -filter { PRIMITIVE_TYPE =~ *PLL* }]]
        set worst_slack_setup [get_property SLACK [get_timing_paths -setup]]
        set worst_slack_hold [get_property SLACK [get_timing_paths -hold]]

        set fileRead [open $csv_file r]
        set lines [split [read $fileRead] "\n"]
        set names_line [lindex $lines end-3]
        set values_line [lindex $lines end-2]
        close $fileRead

        set fileWrite [open $csv_file w]
        puts $fileWrite "$names_line,MMCM*,PLL*,Worst_Setup_Slack,Worst_Hold_Slack"
        puts $fileWrite "$values_line,$MMCM,$PLL,$worst_slack_setup,$worst_slack_hold"
        close $fileWrite
        } issue ] != 0 } {
          puts "GENERATE_REPORTS: tclapp::xilinx::designutils not installed"
        }

        # Define a list of IPs for which to generate report utilization
        set IP_list {
          ad_ip_jesd_204_tpl_adc
          ad_ip_jesd_204_tpl_dac
          axi_jesd204_rx
          axi_jesd204_tx
          jesd204_rx
          jesd204_tx
          axi_adxcvr
          util_adxcvr
          axi_dmac
          util_cpack2
          util_upack2
        }

        foreach IP_name $IP_list {
    set output_file ${IP_name}_resource_utilization.log
          file delete $output_file
          foreach IP_instance [ get_cells -quiet -hierarchical -filter " ORIG_REF_NAME =~ $IP_name || REF_NAME =~ $IP_name " ] {
            report_utilization -hierarchical -hierarchical_depth 1 -cells $IP_instance -file $output_file -append -quiet
            report_property $IP_instance -file $output_file -append -quiet
            set report_file [ open $output_file a ]
            puts $report_file "\n\n\n"
            close $report_file
          }
        }
      } else {
      puts "GENERATE_REPORTS: Resource utilization files won't be generated because ADI_GENERATE_UTILIZATION env var is not set"
    }

    if {[info exists ::env(ADI_GENERATE_XPA)]} {
      set csv_file power_analysis.csv
      set Layers "8to11"
      set CapLoad "20"
      set ToggleRate "15.00000"
      set StatProb "0.500000"

      set_load $CapLoad [all_outputs]
      set_operating_conditions -board_layers $Layers
      set_switching_activity -default_toggle_rate $ToggleRate
      set_switching_activity -default_static_probability $StatProb
      set_switching_activity -type lut -toggle_rate $ToggleRate -static_probability $StatProb -all
      set_switching_activity -type register -toggle_rate $ToggleRate -static_probability $StatProb -all
      set_switching_activity -type shift_register -toggle_rate $ToggleRate -static_probability $StatProb -all
      set_switching_activity -type lut_ram -toggle_rate $ToggleRate -static_probability $StatProb -all
      set_switching_activity -type bram -toggle_rate $ToggleRate -static_probability $StatProb -all
      set_switching_activity -type dsp -toggle_rate $ToggleRate -static_probability $StatProb -all
      set_switching_activity -type gt_rxdata -toggle_rate $ToggleRate -static_probability $StatProb -all
      set_switching_activity -type gt_txdata -toggle_rate $ToggleRate -static_probability $StatProb -all
      set_switching_activity -type io_output -toggle_rate $ToggleRate -static_probability $StatProb -all
      set_switching_activity -type bram_enable -toggle_rate $ToggleRate -static_probability $StatProb -all
      set_switching_activity -type bram_wr_enable -toggle_rate $ToggleRate -static_probability $StatProb -all
      set_switching_activity -type io_bidir_enable -toggle_rate $ToggleRate -static_probability $StatProb -all
      report_power -file $csv_file

      set fileRead [open $csv_file r]
      set filecontent [read $fileRead]
      set input_list [split $filecontent "\n"]

      set TextList [lsearch -all -inline $input_list "*Total On-Chip Power (W)*"]
      set on_chip_pwr "[lindex [lindex $TextList 0] 6] W"
      set TextList [lsearch -all -inline $input_list "*Junction Temperature (C)*"]
      set junction_temp "[lindex [lindex $TextList 0] 5] *C"
      close $fileRead

      set fileWrite [open $csv_file w]
      puts $fileWrite "On-chip_power,Junction_temp"
      puts $fileWrite "$on_chip_pwr,$junction_temp"
      close $fileWrite
    } else {
      puts "GENERATE_REPORTS: Power analysis files won't be generated because ADI_GENERATE_XPA env var is not set"
    }

    # Look for undefined clocks which do not show up in the timing summary
    set timing_check [check_timing -override_defaults no_clock -no_header -return_string]
    if {[regexp { (\d+) register} $timing_check -> num_regs]} {

      if {[info exist num_regs]} {
        if {$num_regs > 0} {
          puts "CRITICAL WARNING: There are $num_regs registers with no clocks !!! See no_clock.log for details."
          check_timing -override_defaults no_clock -verbose -file no_clock.log
        }
      }

    } else {
      puts "CRITICAL WARNING: The search for undefined clocks failed !!!"
    }

    file mkdir $project_name.sdk

    set timing_string $[report_timing_summary -return_string]
    if { [string match "*VIOLATED*" $timing_string] == 1 ||
        [string match "*Timing constraints are not met*" $timing_string] == 1} {
      file copy -force $project_name.runs/impl_1/system_top.sysdef $project_name.sdk/system_top_bad_timing.hdf
      return -code error [format "ERROR: Timing Constraints NOT met!"]
    } else {
      file copy -force $project_name.runs/impl_1/system_top.sysdef $project_name.sdk/system_top.hdf
    }
  }

  ## Run synthesis on an partial design; use it in Partial Reconfiguration flow.
  #
  # \param[project_name] - project name
  # \param[prcfg_name] - name of the partial design
  # \param[hdl_files] - hdl source of the partial design
  # \param[xdc_files] - XDC constraint source of the partial design
  #
  proc adi_project_synth {project_name prcfg_name hdl_files {xdc_files ""}} {

    global p_device

    set p_prefix "$project_name.data/$project_name"

    if {$prcfg_name eq ""} {

      read_verilog .srcs/sources_1/bd/system/hdl/system_wrapper.v
      read_verilog $hdl_files
      read_xdc $xdc_files

      synth_design -mode default -top system_top -part $p_device > $p_prefix.synth.rds
      write_checkpoint -force $p_prefix.synth.dcp
      close_project

    } else {

      create_project -in_memory -part $p_device
      read_verilog $hdl_files
      synth_design -mode out_of_context -top "prcfg" -part $p_device > $p_prefix.${prcfg_name}_synth.rds
      write_checkpoint -force $p_prefix.${prcfg_name}_synth.dcp
      close_project
    }
  }

  ## Run implementation on an partial design; use it in Partial Reconfiguration
  #  flow.
  #
  # \param[project_name] - project name
  # \param[prcfg_name] - name of the partial design
  # \param[xdc_files] - XDC constraint source of the partial design
  #
  proc adi_project_impl {project_name prcfg_name {xdc_files ""}} {

    global p_device
    global p_prcfg_init
    global p_prcfg_list
    global p_prcfg_status

    set p_prefix "$project_name.data/$project_name"

    if {$prcfg_name eq "default"} {
      set p_prcfg_status 0
      set p_prcfg_list ""
      set p_prcfg_init "$p_prefix.${prcfg_name}_impl.dcp"
      file mkdir $project_name.sdk
    }

    if {$prcfg_name eq "default"} {

      open_checkpoint $p_prefix.synth.dcp -part $p_device
      read_xdc $xdc_files
      read_checkpoint -cell i_prcfg $p_prefix.${prcfg_name}_synth.dcp
      set_property HD.RECONFIGURABLE 1 [get_cells i_prcfg]
      opt_design > $p_prefix.${prcfg_name}_opt.rds
      write_debug_probes -force $p_prefix.${prcfg_name}_debug_nets.ltx
      place_design > $p_prefix.${prcfg_name}_place.rds
      route_design > $p_prefix.${prcfg_name}_route.rds

    } else {

      open_checkpoint $p_prefix.default_impl_bb.dcp -part $p_device
      lock_design -level routing
      read_checkpoint -cell i_prcfg $p_prefix.${prcfg_name}_synth.dcp
      read_xdc $xdc_files
      opt_design > $p_prefix.${prcfg_name}_opt.rds
      place_design > $p_prefix.${prcfg_name}_place.rds
      route_design > $p_prefix.${prcfg_name}_route.rds
    }

    write_checkpoint -force $p_prefix.${prcfg_name}_impl.dcp
    report_utilization -pblocks pb_prcfg -file $p_prefix.${prcfg_name}_utilization.rpt
    report_timing_summary -file $p_prefix.${prcfg_name}_timing_summary.rpt

    if [expr [get_property SLACK [get_timing_paths]] < 0] {
      set p_prcfg_status 1
      puts "CRITICAL WARNING: Timing Constraints NOT met ($prcfg_name)!"
    }

    write_checkpoint -force -cell i_prcfg $p_prefix.${prcfg_name}_prcfg_impl.dcp
    update_design -cell i_prcfg -black_box
    write_checkpoint -force $p_prefix.${prcfg_name}_impl_bb.dcp
    open_checkpoint $p_prefix.${prcfg_name}_impl.dcp -part $p_device
    write_bitstream -force -bin_file -file $p_prefix.${prcfg_name}.bit
    write_sysdef -hwdef $p_prefix.hwdef -bitfile $p_prefix.${prcfg_name}.bit -file $p_prefix.${prcfg_name}.hdf
    file copy -force $p_prefix.${prcfg_name}.hdf $project_name.sdk/system_top.${prcfg_name}.hdf

    if {$prcfg_name ne "default"} {
      lappend p_prcfg_list "$p_prefix.${prcfg_name}_impl.dcp"
    }

    if {$prcfg_name eq "default"} {
      file copy -force $p_prefix.${prcfg_name}.hdf $project_name.sdk/system_top.hdf
    }
  }

  ## Verify an implemented partial reconfiguration design, checks if all the
  #  partial design are compatible with the base design.
  #
  # \param[project_name] - project name
  #
  proc adi_project_verify {project_name} {

    # checkpoint for the default design
    global p_prcfg_init
    # list of checkpoints with all the PRs integrated into the default design
    global p_prcfg_list
    global p_prcfg_status

    set p_prefix "$project_name.data/$project_name"

    pr_verify -full_check -initial $p_prcfg_init \
      -additional $p_prcfg_list \
      -file $p_prefix.prcfg_verify.log

    if {$p_prcfg_status == 1} {
      return -code error [format "ERROR: Timing Constraints NOT met!"]
    }
  }

  ```


## Reference 

- [Porting ADI's HDL reference designs](https://wiki.analog.com/resources/fpga/docs/hdl/porting_project_quick_start_guide)

- [HDL Architecture](https://wiki.analog.com/resources/fpga/docs/arch)
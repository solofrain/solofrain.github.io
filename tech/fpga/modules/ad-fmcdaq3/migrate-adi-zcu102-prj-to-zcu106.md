# Migrating ADI DAQ3 ZCU102 Project to ZCU106

> [Porting ADI's HDL reference designs](https://wiki.analog.com/resources/fpga/docs/hdl/porting_project_quick_start_guide){:target="_blank"}

ADI provide DAQ3 project for ZCU102. This article describes how to migrate it to ZCU106.

## 1. Get source code

Clone the Github project to local disk:

<https://github.com/analogdevicesinc/hdl>{:target="_blank"}

The directory of the project is $(adi).

## 2. Modify source code

### 2.1 Prepare ZCU106 board files

 Navigate to $(adi)/projects/common. Make a copy of zcu102/ folder and name it as zcu106.

This folder contains the following files:

- **system_project.tcl** - This script is creating the actual Vivado project and runs the synthesis/implementation of the design. The name of the carrier must be updated.

- **system_bd.tcl** - In this file is sourced the base design's Tcl script and the board design's Tcl script. The name of the carrier must be updated.

- **system_constr.xdc** - Constraint files of the board design. Here is defined the FMC IO's and board specific clock signals. All the IO definition must be updated, with the new pin names.

- **system_top.v** - Top wrapper file, in which the system_wrapper.v module is instantiated, and a few I/O macros are defined. The IO port of this Verilog module will be connected to actual IO pads of the FPGA. The simplest way to update the system_top is to let the synthesis fail and the tool will tell which ports are missing or which ports are redundant. The first thing to do after the failure is to verify the instantiation of the system_wrapper.v. This file is a tool generated file and can be found at `<project_name>.srcs/sources_1/bd/system/hdl/system_wrapper.v`. Fixing the instantiation of the wrapper module in most cases eliminates all the errors. If you get errors that you can not fix, ask for support.

- **Makefile** - This is an auto-generated file, but after updating the carrier name, should work with the new project without an issue.


### 2.2 Modify block design file.

zcu102_system_bd.tcl describes the base block design. Rename zcu102_system_bd.tcl as zcu106_system_bd.tcl and modify it as following:

```tcl
```

### 2.3 Modify constraint file.

zcu102_system_constr.xdc is IO constraint file for the base design. Will contain IO definitions for GPIO, switches, LEDs or other peripherals of the board.  Rename zcu102_system_constr.xdc as zcu106_system_constr.xdc and modify it as following:

```tcl
```

### 2.4 Define ZCU106  board and its device in the project flow script.

In ${adi}/projects/scripts/adi_project_xilinx.tcl, add:

```tcl
if [regexp "_zcu106$" $project_name] {
    set p_device "xczu7ev-ffvc1156-2-e"
    set p_board [lindex [lsearch -all -inline [get_board_parts] *zcu106*] end]
    set sys_zynq 2
}
```

> The valid board parts and parts can be retrieved by running the following commands in Tcl console: get_parts and get_board_parts. Run the commands like join [get_parts] \n, so each part name will be listed on a separate line.


## Reference 

- [Porting ADI's HDL reference designs](https://wiki.analog.com/resources/fpga/docs/hdl/porting_project_quick_start_guide)

- [HDL Architecture](https://wiki.analog.com/resources/fpga/docs/arch)
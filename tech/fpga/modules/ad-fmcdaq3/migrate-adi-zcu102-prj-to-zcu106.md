# Migrating ADI DAQ3 ZCU102 Project to ZCU106

> [Porting ADI's HDL reference designs](https://wiki.analog.com/resources/fpga/docs/hdl/porting_project_quick_start_guide){:target="_blank"}

ADI provide DAQ3 project for ZCU102. This article describes how to migrate it to ZCU106.

## 1. Get source code

Clone the Github project to local disk:

<https://github.com/analogdevicesinc/hdl>{:target="_blank"}

The directory of the project is $(adi).

## 2. Modify source code

### 2.1 Prepare ZCU106 board files

- Navigate to $(adi)/projects/common. Make a copy of zcu102/ folder and name it as zcu106.

- Rename zcu102_system_bd.tcl as zcu106_system_bd.tcl and modify it as following:

```tcl
```

- Rename zcu102_system_constr.xdc as zcu106_system_constr.xdc and modify it as following:

```tcl
```

- In ${adi}/projects/scripts/adi_project_xilinx.tcl, add

```tcl
if [regexp "_zcu106$" $project_name] {
    set p_device "xczu7ev-ffvc1156-2-e"
    set p_board [lindex [lsearch -all -inline [get_board_parts] *zcu106*] end]
    set sys_zynq 2
}
```
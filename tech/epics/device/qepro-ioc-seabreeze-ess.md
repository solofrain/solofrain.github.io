Building QEPro IOC Based on Seabreeze Library
==


> Examples have been built on
>  - 130.199.219.190.
          Library: /home/liji/data/epics/lib/seabreeze/3.0.11/SeaBreeze
        IOC:     /home/liji/data/epics/iocs/qepro
>
> - xf28id2-ws4
        Library: /epics/lib/seabreeze/3.0.11
        IOC:     /epics/iocs/qepro

# 1. Build Seabreeze library

API manual

<https://www.oceaninsight.com/globalassets/catalog-blocks-and-images/software-downloads-installers/javadocs-api/seabreeze/html/index.html>{:target="_blank"}

## 1.1 Dependence: 

- libusb-dev
- gcc
- g++

## 1.2 Download SeaBreeze

<https://sourceforge.net/projects/seabreeze/>{:target="_blank"}

<https://github.com/solofrain/seabreeze>{:target="_blank"}

## 1.3 Modify Makefiles

In `seabreeze-3.0.11/SeaBreeze/Makefile`, change line 30

from:

``` Makefile
$(CPP) $(LFLAGS_LIB) -o $@ lib/*.o
```

to:


``` Makefile
$(CPP) -o $@ lib/*.o  $(LFLAGS_LIB)
```

You may also need to append `-lusb`

``` Makefile
$(CPP) -o $@ lib/*.o  $(LFLAGS_LIB) -lusb
```

Otherwise IOC compilation will report errors:

```bash
libseabreeze.so: undefined reference to `usb_close'
libseabreeze.so: undefined reference to `usb_bulk_write'
libseabreeze.so: undefined reference to `usb_claim_interface'
libseabreeze.so: undefined reference to `usb_find_busses'
libseabreeze.so: undefined reference to `usb_device'
libseabreeze.so: undefined reference to `usb_init'
libseabreeze.so: undefined reference to `usb_get_string_simple'
libseabreeze.so: undefined reference to `usb_clear_halt'
libseabreeze.so: undefined reference to `usb_busses'
libseabreeze.so: undefined reference to `usb_open'
libseabreeze.so: undefined reference to `usb_find_devices'
libseabreeze.so: undefined reference to `usb_bulk_read'
libseabreeze.so: undefined reference to `usb_reset'
```

## 1.4 Modify source files (in some OS)

In `./src/common/Log.cpp`, add `{}` to `if` statement at line 163 and line 180, otherwise it may report:

```bash
error: this ‘if’ clause does not guard... [-Werror=misleading-indentation]
```

## 1.5 Make

If `libusb-dev` is not installed, it reports:

```bash
Error: NativeUSBLinux.c:35:17: fatal error: usb.h: No such file or directory 
```

## 1.6 Expose libseabreeze.so

Either set system variable:

```bash
export LD_LIBRARY_PATH="$PWD/lib"
```

Or install `libseabreeze.so`into a system library directory like `/usr/local/lib` that `ld.so` knows about.

# 2. Build the IOC

## 2.1 Prerequisitions

QEPro IOC depends on asyn, busy, autosave.

## 2.2 Download the package

<https://bitbucket.org/europeanspallationsource/m-epics-qeproasyn/>{:target="_blank"}

## 2.3 Create IOC

Copy source files in `Db/` and `src/` in `m-epics-qeproasyn/` to corresponding IOC directories.

## 2.4 Modify Makefiles

- qeproApp/src/Makefile

``` Makefile
TOP=../..

include $(TOP)/configure/CONFIG
#----------------------------------------
#  ADD MACRO DEFINITIONS AFTER THIS LINE
#=============================

#=============================
# Build the IOC application

PROD_IOC = qepro
# qepro.dbd will be created and installed
DBD += qepro.dbd

# qepro.dbd will be made up from these files:
qepro_DBD += base.dbd
#qepro_DBD += $(ASYN)/dbd/asyn.dbd
qepro_DBD += asyn.dbd
qepro_DBD += busyRecord.dbd

# Include dbd files from all support applications:
qepro_DBD += drvUSBQEProSupport.dbd

# Add all the support libraries needed by this IOC
qepro_LIBS += asyn
qepro_LIBS += busy
qepro_SYS_LIBS += usb-1.0
qepro_SYS_LIBS += usb
#qepro_LIBS += seabreeze

# qepro_registerRecordDeviceDriver.cpp derives from qepro.dbd
qepro_SRCS += qepro_registerRecordDeviceDriver.cpp
qepro_SRCS += drvUSBQEPro.cpp
qepro_SRCS += drvUSBQEProOBP.cpp

# Build the main IOC entry point on workstation OSs.
qepro_SRCS_DEFAULT += qeproMain.cpp
qepro_SRCS_vxWorks += -nil-

#PROD_LDFLAGS += -WL,-rpath,/home/liji/iocs/seabreeze-3.0.11/SeaBreeze/lib
PROD_SYS_LIBS += seabreeze

# Add support from base/src/vxWorks if needed
#qepro_OBJS_vxWorks += $(EPICS_BASE_BIN)/vxComLibrary

# Finally link to the EPICS Base libraries
qepro_LIBS += $(EPICS_BASE_IOC_LIBS)

USR_CPPFLAGS += -I/home/liji/data/epics/lib/seabreeze/3.0.11/SeaBreeze/include -DLINUX
SEABREEZE_DIR = /home/liji/data/epics/lib/seabreeze/3.0.11/SeaBreeze/lib
USR_LDFLAGS += -L$(SEABREEZE_DIR) -lseabreeze
USR_LDFLAGS += -lusb
#===========================

include $(TOP)/configure/RULES
#----------------------------------------
#  ADD RULES AFTER THIS LINE
```

- qeproApp/Db/Makefile

```Makefile
TOP=../..
include $(TOP)/configure/CONFIG
#----------------------------------------
#  ADD MACRO DEFINITIONS AFTER THIS LINE

#----------------------------------------------------
# Create and install (or just install) into <top>/db
# databases, templates, substitutions like this
DB += qepro.template

#----------------------------------------------------
# If <anyname>.db template is not named <anyname>*.template add
# <anyname>_template = <templatename>

include $(TOP)/configure/RULES
#----------------------------------------
#  ADD RULES AFTER THIS LINE
```

## 2.5 Make

```
Error: make[3]: *** No rule to make target '../../../lib/linux-x86_64/libseabreeze.a', needed by 'qepro'.  Stop.
```

This error is caused by incorrect definition in `src/Makefile`. Reference to the example listed above.

# Appendix

## I. System configuration

To be able to access the device from the USB port, add a file (e.g., 10-qepro.rules) to /etc/udev/rules.d/:

```
SUBSYSTEMS=="usb", ATTRS{idVendor}=="2457",ATTRS{idProduct}=="4004",GROUP="root",MODE="0666"
```
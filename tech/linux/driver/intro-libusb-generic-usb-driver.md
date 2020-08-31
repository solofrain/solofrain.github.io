 通用usb驱动libusb介绍和使用示例 
 ==

 Orininal: <https://www.cnblogs.com/image-eye/archive/2011/08/30/2159897.html>{:target="_blank"}

 小知识：

```
sudo insmod /lib/modules/2.6.22-14-generic/kernel/drivers/usb/serial/usbserial.ko vendor=0x8086 product=0xd001
```

同时插上ttyUSB0和ttyUSB1(ch341)，obm可以将dkb下载下去，但是自动重起之后,就不能下载接下来的东西了,所以应该,需要`close(ttyUSB0_handle)`；然后进行接下来的下载,分别调用两次不过应该自动关闭了,所以可能还是不能同时插上ttyUSB0和ttyUSB1

`lsusb` 显示usb设备的vendor和product
比如：

```
b074@gliethttp:~$ lsusb
Bus 002 Device 001: ID 0000:0000 
Bus 001 Device 116: ID 8086:d001 Intel Corp.
Bus 001 Device 003: ID 413c:2105 Dell Computer Corp.
Bus 001 Device 002: ID 0461:4d15 Primax Electronics, Ltd
Bus 001 Device 001: ID 0000:0000
```

其中`Bus 001 Device 116: ID 8086:d001 Intel Corp. `就是`vendor=0x8086`和`product=0xd001`

可以使用dmesg来查看具体的是ttyUSB0还是ttyUSB1了

```
pidof hello.exe
pidof bash
```

显示进程的pid值

波特率：

```
#define  B0    0000000        /* hang up */
#define  B50    0000001
#define  B75    0000002
#define  B110    0000003
#define  B134    0000004
#define  B150    0000005
#define  B200    0000006
#define  B300    0000007
#define  B600    0000010
#define  B1200    0000011
#define  B1800    0000012
#define  B2400    0000013
#define  B4800    0000014
#define  B9600    0000015
#define  B19200    0000016
#define  B38400    0000017
#define EXTA B19200
#define EXTB B38400
#define CSIZE    0000060
#define   CS5    0000000
#define   CS6    0000020
#define   CS7    0000040
#define   CS8    0000060
#define CSTOPB    0000100
#define CREAD    0000200
#define PARENB    0000400
#define PARODD    0001000
#define HUPCL    0002000
#define CLOCAL    0004000
#define CBAUDEX 0010000
#define    BOTHER 0010000
#define    B57600 0010001
#define   B115200 0010002
#define   B230400 0010003
#define   B460800 0010004 //有些CDMA使用该波特率
#define   B500000 0010005
#define   B576000 0010006
#define   B921600 0010007
#define  B1000000 0010010
#define  B1152000 0010011
#define  B1500000 0010012
#define  B2000000 0010013
#define  B2500000 0010014
#define  B3000000 0010015
#define  B3500000 0010016
#define  B4000000 0010017
```

Developing Linux Device Drivers using Libusb API
---

Written by vikram_cvk - 2004-07-16 18:05

# Introduction

We often come across a situation where a USB device which runs perfectly on Windows platform does not even get detected on Linux. Lack of support for USB devices is one of the reason why some people don't embrace Linux. Now there is a new API by name Libusb which helps the developers to develop USB device drivers on the fly!

# What is libusb

Libusb is a high-level language API which conceals low-level kernel interactions with the USB modules. It provides a set of function which are adequate to develop a device driver for a USB device from the Userspace.

## Libusb is not complex

For any wannabe Linux Kernel programmers developing device driver as a Kernel module is a herculean task. Developing kernel modules requires fair degree of proficiency in 'C' language and also good idea of kernel subsystems, data structures etc. All these are enough to put-off a developer from venturing into Device Driver programming.Libusb has been designed to address this shortcoming. Simplified interface allows developers to develop USB drivers from the userspace . Libusb library functions provide high level abstraction to the Kernel structures and allows the developers to have access to these structures through the USBFS(USBfilesystem).

## It's Cross-platform

Beauty of Libusb lies in its cross platform functionality. Driver written for one platform could be easily ported onto another platform with little or no changes, currently following operating systems are supported by Libusb.

- Linux
- FreeBSD
- Darwin
- OS X

This HOWTO focuses on how Libusb can be used on Linux platform. For information about other platforms goto <http://http://libusb.sourceforge.net/>{:target="_blank"}.


# LIBUSB ON LINUX

Linux is the most popular platform for the Libusb API,the reason being growing popularity of Linux as a stable OS. On Linux Libusb makes of the USBFS file system. by default USBFS is automatically mounted when the system is booted.

What is USBFS
USBFS is a filesystem specifically designed for USB devices, by default this filesystem gets mounted when the system is booted and it can be found at /proc/bus/usb/. This filesystem consists of information about all the USB devices that are connected to the computer.Libusb makes use of this filesystem to interact with the USB devices.

Following C program can be a stepping stone into the world of Libusb.This program can be used to gather all the technical/hardware details of a USB device connected to the computer ,ensure that some USB device is connected into the USB port.

Details like Vendor-Id , Product-Id ,Endpoint addresses of a USB device is of paramount importance for a device driver developer.

```C
/* testlibusb.c */

#include
#include


void print_endpoint(struct usb_endpoint_descriptor *endpoint)
{
    printf(" bEndpointAddress: %02xh\n", endpoint->bEndpointAddress);
    printf(" bmAttributes: %02xh\n", endpoint->bmAttributes);
    printf(" wMaxPacketSize: %d\n", endpoint->wMaxPacketSize);
    printf(" bInterval: %d\n", endpoint->bInterval);
    printf(" bRefresh: %d\n", endpoint->bRefresh);
    printf(" bSynchAddress: %d\n", endpoint->bSynchAddress);
}


void print_altsetting(struct usb_interface_descriptor *interface)
{
    int i;

    printf(" bInterfaceNumber: %d\n", interface->bInterfaceNumber);
    printf(" bAlternateSetting: %d\n", interface->bAlternateSetting);
    printf(" bNumEndpoints: %d\n", interface->bNumEndpoints);
    printf(" bInterfaceClass: %d\n", interface->bInterfaceClass);
    printf(" bInterfaceSubClass: %d\n", interface->bInterfaceSubClass);
    printf(" bInterfaceProtocol: %d\n", interface->bInterfaceProtocol);
    printf(" iInterface: %d\n", interface->iInterface);

    for (i = 0; i < interface->bNumEndpoints; i++)
        print_endpoint(&interface->endpoint);
}


void print_interface(struct usb_interface *interface)
{
    int i;

    for (i = 0; i < interface->num_altsetting; i++)
        print_altsetting(&interface->altsetting);
}


void print_configuration(struct usb_config_descriptor *config)
{
    int i;

    printf(" wTotalLength: %d\n", config->wTotalLength);
    printf(" bNumInterfaces: %d\n", config->bNumInterfaces);
    printf(" bConfigurationValue: %d\n", config->bConfigurationValue);
    printf(" iConfiguration: %d\n", config->iConfiguration);
    printf(" bmAttributes: %02xh\n", config->bmAttributes);
    printf(" MaxPower: %d\n", config->MaxPower);

    for (i = 0; i < config->bNumInterfaces; i++)
        print_interface(&config->interface);
}


int main(void)
{
    struct usb_bus *bus;
    struct usb_device *dev;

    usb_init();
    usb_find_busses();
    usb_find_devices();

    printf("bus/device idVendor/idProduct\n");

    for (bus = usb_busses; bus; bus = bus->next) {
        for (dev = bus->devices; dev; dev = dev->next) {
            int ret, i;
            char string[256];
            usb_dev_handle *udev;

            printf("%s/%s %04X/%04X\n", bus->dirname, dev->filename,
            dev->descriptor.idVendor, dev->descriptor.idProduct);

            udev = usb_open(dev);
            if (udev) {
                if (dev->descriptor.iManufacturer) {
                    ret = usb_get_string_simple(udev, dev->descriptor.iManufacturer, string, sizeof(string));
                    if (ret > 0)
                        printf("- Manufacturer : %s\n", string);
                    else
                        printf("- Unable to fetch manufacturer string\n");
                }

                if (dev->descriptor.iProduct) {
                    ret = usb_get_string_simple(udev, dev->descriptor.iProduct, string, sizeof(string));
                    if (ret > 0)
                        printf("- Product : %s\n", string);
                    else
                        printf("- Unable to fetch product string\n");
                }

                if (dev->descriptor.iSerialNumber) {
                    ret = usb_get_string_simple(udev, dev->descriptor.iSerialNumber, string, sizeof(string));
                    if (ret > 0)
                        printf("- Serial Number: %s\n", string);
                    else
                        printf("- Unable to fetch serial number string\n");
                }

                usb_close (udev);
            }

            if (!dev->config) {
                printf(" Couldn't retrieve descriptors\n");
                continue;
            }

            for (i = 0; i < dev->descriptor.bNumConfigurations; i++)
                print_configuration(&dev->config);
        }
    }

    return 0;
}
```

The above program should be compiled as

```
(root$)gcc -o usbdevice_details testlibusb.c -I/usr/local/include -L. -lnsl -lm -lc -L/usr/local/lib -lusb

(root$)./usbdevice_details
```

Following is the output of the above command ,its the listing of a USB pen drive connected to my system.

The first line displays the bus-name/device-name & device-id/product-id and rest of the listing is self-descriptive.

```
001/004 0EA0/2168
- Manufacturer : USB
- Product : Flash Disk
- Serial Number: 4CE45C4E403EE53D
wTotalLength: 39
bNumInterfaces: 1
bConfigurationValue: 1
iConfiguration: 0
bmAttributes: 80h
MaxPower: 100
bInterfaceNumber: 0
bAlternateSetting: 0
bNumEndpoints: 3
bInterfaceClass: 8
bInterfaceSubClass: 6
bInterfaceProtocol: 80
iInterface: 0
bEndpointAddress: 81h
bmAttributes: 02h
wMaxPacketSize: 64
bInterval: 0
bRefresh: 0
bSynchAddress: 0
bEndpointAddress: 02h
bmAttributes: 02h
wMaxPacketSize: 64
bInterval: 0
bRefresh: 0
bSynchAddress: 0
bEndpointAddress: 83h
bmAttributes: 03h
wMaxPacketSize: 2
bInterval: 1
bRefresh: 0
bSynchAddress: 0
```

Before executing the above program download the current version of Libusb library from <http://http://libusb.sourceforge.net/>. The above program can also be found under the tests directory of Libusb directory (after u install it)


Now I will explain in brief some of the functions and attributes dealt in the above program.


- usb_init() - Used to initialize Libusb and establish connection with kernel structures .

- usb_find_busses() - Looks for all the USB busses on the computer.

- usb_find_devices() - Looks for all the USB devices connected to the computer.

- usb_open(dev) - Opens the device 'dev' which is given as argument to this function.

- usb_get_string_simple() - Used to extract the string descriptor of the device taken argument.

Important attributes of USB devices useful in device driver coding

Configuration and Endpoints are one of the two important descriptors of any USB device. These descriptors are defined using the `struct usb_config_descriptor` and `struct_usb_endpoint_descriptor` respectively .

- dev->descriptor.idVendor -  Reveals the Vendor-Id of the USB device connected to the system.

- dev->descriptor.idProduct - Reveals the Product-Id of the USB device connected to the system.

- dev->descriptor.iManufacturer - Reveals the name of the Manufacturer USB device connected to the system.

- EndpointAddress:Combination of endpoint address and endpoint direction on a USB device.

- InterfaceNumber : One of the several interfaces that is allocated to the connected USB device.

- AlternateSetting:This is part of the a single interface allocated to the USB device.


Prerequisites for Libusb programming:

- Linux system with Kernel 2.4 above series.
- Proficiency in C language.
- Good understanding of USB device internals.
- Idea about USBFS.

Hope this HOWTO has enlightened you about Libusb API and I expect this HOWTO will give you a head start in your device driver programming endeavor .This HOWTO is just an introduction to Libusb ,for complete documentation please goto <http://libusb.sourceforge.net/>.

About Myself

My name is Vikram C , I'm a linux freak and currently working as Linux developer in the city of Hyderabad India.You can reach me at vikram_147@hotmail.com / vikram@asrttechnologies.com

//================================================
2008年03月19日 星期三 10:31

驱动开发向来是内核开发中工作量最多的一块，随着USB设备的普及，大量的USB设备的驱动开发也成为驱动开发者手头上做的最多的事情。本文主要介绍 Linux平台下基于libusb的驱动开发，希望能够给从事Linux驱动开发的朋友带来些帮助，更希望能够给其他平台上的无驱设计带来些帮助。文章是我在工作中使用libusb的一些总结，难免有错误，如有不当的地方，还请指正。

Linux 平台上的usb驱动开发，主要有内核驱动的开发和基于libusb的无驱设计。

对于内核驱动的大部分设备，诸如带usb接口的hid设备，linux本身已经自带了相关的驱动，我们只要操作设备文件便可以完成对设备大部分的操作，而另外一些设备，诸如自己设计的硬件产品，这些驱动就需要我们驱动工程师开发出相关的驱动了。内核驱动有它的优点，然而内核驱动在某些情况下会遇到如下的一些问题：

>1 当使用我们产品的客户有2.4内核的平台，同时也有2.6内核的平台，我们要设计的驱动是要兼容两个平台的，就连makefile 我们都要写两个。

>2 当我们要把linux移植到嵌入平台上，你会发现原先linux自带的驱动移过去还挺大的，我的内核当然是越小越好拉，这样有必要么。这还不是最郁闷的地方，如果嵌入平台是客户的，客户要购买你的产品，你突然发现客户设备里的系统和你的环境不一样，它没有你要的驱动了，你的程序运行不了，你会先想：“没关系，我写个内核驱动加载一下不就行了。”却发现客户连insmod加载模块的工具都没移植，那时你就看看老天，说声我怎么那么倒霉啊，客户可不想你动他花了n时间移植的内核哦

>3 花了些功夫写了个新产品的驱动，挺有成就感啊，代码质量也是相当的有水准啊。正当你沉醉在你的代码中时，客服不断的邮件来了，“客户需要2.6.5内核的驱动，config文件我已经发你了” “客户需要双核的 2.6.18-smp 的驱动” “客户的平台是自己定制的是2.6.12-xxx ”。你恨不得把驱动的源代码给客户，这样省得编译了。你的一部分工作时间编译内核，定制驱动。

有问题产生必然会有想办法解决问题的人， libusb的出现给我们带来了某些方便，即节约了我们的时间，也降低了公司的成本。 所以在一些情况下，就可以考虑使用libusb的无驱设计了。

下面我们就来详细讨论一下libusb, 并以写一个hid设备的驱动来讲解如何运用libusb,至于文章中涉及的usb协议的知识，限于篇幅，就不详细讲解了，相关的可自行查看usb相关协议。

# 1. libusb 介绍

   libusb 设计了一系列的外部API 为应用程序所调用，通过这些API应用程序可以操作硬件，从libusb的源代码可以看出，这些API 调用了内核的底层接口，和kernel driver中所用到的函数所实现的功能差不多，只是libusb更加接近USB 规范。使得libusb的使用也比开发内核驱动相对容易的多。

Libusb 的编译安装请查看Readme,这里不做详解

# 2. libusb 的外部接口

## 2.1 初始化设备接口

这些接口也可以称为核心函数，它们主要用来初始化并寻找相关设备。

- **usb_init()**

    函数定义： `void usb_init(void);`

    从函数名称可以看出这个函数是用来初始化相关数据的，这个函数大家只要记住必须调用就行了，而且是一开始就要调用的.

- **usb_find_busses()**

    函数定义：` int usb_find_busses(void);`

    寻找系统上的usb总线，任何usb设备都通过usb总线和计算机总线通信。进而和其他设备通信。此函数返回总线数。

- **usb_find_devices()**

    函数定义： `int usb_find_devices(void);`

    寻找总线上的usb设备，这个函数必要在调用usb_find_busses()后使用。以上的三个函数都是一开始就要用到的，此函数返回设备数量。

- **usb_get_busses()**

    函数定义： `struct usb_bus *usb_get_busses(void);`

    这个函数返回总线的列表，在高一些的版本中已经用不到了，这在下面的实例中会有讲解

## 2.2 操作设备接口

- **usb_open()**

    函数定义： `usb_dev_handle *usb_open(struct *usb_device dev);`

    打开要使用的设备。在对硬件进行操作前必须要调用 `usb_open` 来打开设备，这里大家看到有两个结构体 `usb_dev_handle` 和 `usb_device` 是我们在开发中经常碰到的，有必要把它们的结构看一看。在libusb 中的`usb.h`和`usbi.h`中有定义。

    这里我们不妨理解为返回的 `usb_dev_handle` 指针是指向设备的句柄，而形参里输入就是需要打开的设备。

-   **usb_close()**

    函数定义： `int usb_close(usb_dev_handle *dev);`

    与usb_open相对应，关闭设备，是必须调用的, 返回0成功，<0 失败。

-   **usb_set_configuration()**

    函数定义： `int usb_set_configuration(usb_dev_handle *dev, int configuration);`

    设置当前设备使用的`configuration`。参数`configuration` 是你要使用的`configurtation descriptoes`中的`bConfigurationValue`, 返回`0`成功，`<0`失败( 一个设备可能包含多个`configuration`,比如同时支持高速和低速的设备就有对应的两个`configuration`,详细可查看usb标准)。

-   **usb_set_altinterface()**

    函数定义： `int usb_set_altinterface(usb_dev_handle *dev, int alternate);`

    和名字的意思一样，此函数设置当前设备配置的`interface descriptor`。参数`alternate`是指`interface descriptor`中的`bAlternateSetting`。返回`0`成功，`<0`失败

-   **usb_resetep()**

    函数定义： `int usb_resetep(usb_dev_handle *dev, unsigned int ep);`

    复位指定的`endpoint`。参数`ep` 是指`bEndpointAddress`。这个函数不经常用，被下面介绍的`usb_clear_halt`函数所替代。

-   **usb_clear_halt()**

    函数定义： `int usb_clear_halt (usb_dev_handle *dev, unsigned int ep);`

    复位指定的`endpoint`。参数`ep` 是指`bEndpointAddress`。这个函数用来替代`usb_resetep`。

-   **usb_reset()**

    函数定义：`int usb_reset(usb_dev_handle *dev);`

    这个函数现在基本不怎么用，不过这里我也讲一下，和名字所起的意思一样，这个函数reset设备，因为重启设备后还是要重新打开设备，所以用`usb_close`就已经可以满足要求了。

-   **usb_claim_interface()**

    函数定义： `int usb_claim_interface(usb_dev_handle *dev, int interface);`

    注册与操作系统通信的接口，这个函数必须被调用，因为只有注册接口，才能做相应的操作。

    `Interface` 指 `bInterfaceNumber`。 (下面介绍的`usb_release_interface` 与之相对应，也是必须调用的函数)

-   **usb_release_interface()**

    函数定义： int usb_release_interface(usb_dev_handle *dev, int interface);

    注销被`usb_claim_interface`函数调用后的接口，释放资源，和`usb_claim_interface`对应使用。

## 2.3 控制传输接口

-   **usb_control_msg()**

    函数定义：`int usb_control_msg(usb_dev_handle *dev, int requesttype, int request, int value, int index, char *bytes, int size, int timeout);`

    从默认的管道发送和接受控制数据。

-   **usb_get_string()**

    函数定义： `int usb_get_string(usb_dev_handle *dev, int index, int langid, char *buf, size_t buflen);`

-   **usb_get_string_simple()**

    函数定义： `int usb_get_string_simple(usb_dev_handle *dev, int index, char *buf, size_t buflen);`

-   **usb_get_descriptor()**

    函数定义： `int usb_get_descriptor(usb_dev_handle *dev, unsigned char type, unsigned char index, void *buf, int size);`

-   **usb_get_descriptor_by_endpoint()**

    函数定义： `int usb_get_descriptor_by_endpoint(usb_dev_handle *dev, int ep, unsigned char type, unsigned char index, void *buf, int size);`

## 2.4 批传输接口

-   **usb_bulk_write()**

    函数定义： `int usb_bulk_write(usb_dev_handle *dev, int ep, char *bytes, int size, int timeout);`
    
-    **usb_interrupt_read()**

    函数定义： `int usb_interrupt_read(usb_dev_handle *dev, int ep, char *bytes, int size, int timeout);`

## 2.5 中断传输接口

- **usb_bulk_write()**

    函数定义： `int usb_bulk_write(usb_dev_handle *dev, int ep, char *bytes, int size, int timeout);`

- **usb_interrupt_read()**

    函数定义： `int usb_interrupt_read(usb_dev_handle *dev, int ep, char *bytes, int size, int timeout);`

基本上libusb所经常用到的函数就有这些了，和usb协议确实很接近吧。下面我们实例在介绍一个应用。

//----------------===================================

# 3. Libusb库的使用

使用libusb之前你的Linux系统必须装有usb文件系统，这里还介绍了使用hiddev设备文件来访问设备，目的在于不仅可以比较出usb的易用性，还提供了一个转化成libusb驱动的案例。

## 3.1 find设备

任何驱动第一步首先是寻找到要操作的设备，我们先来看看HID驱动是怎样寻找到设备的。我们假设寻找设备的函数`Device_Find`(注：代码只是为了方便解说，不保证代码的健全)。我们简单看一下使用hid驱动寻找设备的实现，然后在看一下libusb是如何寻找设备的。

```C
int Device_Find()
{
    char dir_str[100];   /* 这个变量我们用来保存设备文件的目录路径 */
    char hiddev[100];    /* 这个变量用来保存设备文件的全路径 */
    DIR dir;              
    /* 申请的字符串数组清空，这个编程习惯要养成 */
    memset (dir_str, 0 , sizeof(dir_str));
    memset (hiddev, 0 , sizeof(hiddev));
    /* hiddev 的设备描述符不在/dev/usb/hid下面，就在/dev/usb 下面
       这里我们使用opendir函数来检验目录的有效性
       打开目录返回的值保存在变量dir里，dir前面有声明
    */

    dir=opendir("/dev/usb/hid");
    if(dir){
        /* 程序运行到这里，说明存在 /dev/usb/hid 路径的目录 */
        sprintf(dir_str,"/dev/usb/hid/");
        closedir(dir);
    }else{
        /* 如果不存在hid目录，那么设备文件就在/dev/usb下 */
        sprintf(dir_str,"/dev/usb/");
    }

    /* DEVICE_MINOR 是指设备数，HID一般是16个 */
    for(i = 0; i < DEVICE_MINOR; i++) {
    /* 获得全路径的设备文件名，一般hid设备文件名是hiddev0 到 hiddev16 */
        sprintf(hiddev, "%shiddev%d", dir_str,i);
       /* 打开设备文件,获得文件句柄 */
       fd = open(hiddev, O_RDWR);
       if(fd > 0) {
           /* 操作设备获得设备信息 */
           ioctl(fd, HIDIOCGDEVINFO, &info);
   
           /* VENDOR_ID 和 PRODUCT_ID 是标识usb设备厂家和产品ID,驱动都需要这两个参数来寻找设备,到此我们寻找到了设备 */
           if(info.vendor== VENDOR_ID && info.product== PRODUCT_ID) {
                /* 这里添加设备的初始化代码 */
                  
               device_num++;   /* 找到的设备数 */
           }
           close(fd);
       }
    }
    return device_num;         /* 返回寻找的设备数量 */
}
```

我们再来看libusb是如何来寻找和初始化设备。

```C
int Device_Find()
{
    struct usb_bus             *busses;
    int                           device_num = 0;
    device_num = 0;       /* 记录设备数量 */
   
    usb_init();            /* 初始化 */
    usb_find_busses();   /* 寻找系统上的usb总线 */
    usb_find_devices(); /* 寻找usb总线上的usb设备 */
   
    /* 获得系统总线链表的句柄 */
    busses = usb_get_busses();
    struct usb_bus       *bus;

    /* 遍历总线 */
    for (bus = busses; bus; bus = bus->next) {
        struct usb_device *dev;

        /* 遍历总线上的设备 */
        for (dev = bus->devices; dev; dev = dev->next) {
            /* 寻找到相关设备， */
            if(dev->descriptor.idVendor==VENDOR_ID&& dev->descriptor.idProduct == PRODUCT_ID) {
                /* 这里添加设备的初始化代码 */
                  
                device_num++;   /* 找到的设备数 */
            }
        }
    }
    return device_num;        /* 返回设备数量 */
}
```

注：在新版本的libusb中，`usb_get_busses`就可以不用了 ，这个函数是返回系统上的usb总线链表句柄
这里我们直接用`usb_busses`变量，这个变量在usb.h中被定义为外部变量。
所以可以直接写成这样：

```C
struct usb_bus    *bus;
        for (bus = usb_busses; bus; bus = bus->next) {
               struct usb_device *dev;
        for (dev = bus->devices; dev; dev = dev->next) {
           /* 这里添加设备的初始化代码 */
        }
}
```

## 3.2 打开设备

假设我们定义的打开设备的函数名是`Device_open`。

```C
/* 使用hid驱动打开设备 */
int Device_Open()
{
    int handle;
    /* 传统HID驱动调用,通过open打开设备文件就可 */
    handle = open(“hiddev0”, O_RDONLY);
}

/* 使用libusb打开驱动 */
int Device_Open()
{
    /* LIBUSB 驱动打开设备，这里写的是伪代码，不保证代码有用 */
    struct usb_device*    udev;
    usb_dev_handle*        device_handle;
    /* 当找到设备后，通过usb_open打开设备，这里的函数就相当open 函数 */
    device_handle = usb_open(udev);
}
```

## 3.3 读写设备和操作设备

假设我们的设备使用控制传输方式，至于批处理传输和中断传输限于篇幅这里不介绍。我们这里定义三个函数： Device_Write, Device_Read, Device_Report。

- `Device_Report` 功能发送接收函数
- `Device_Write` 功能写数据
- `Device_Read`   功能读数据

`Device_Write`和`Device_Read`调用`Device_Report`发送写的信息和读的信息，开发者根据发送的命令协议来设计，我们这里只简单实现发送数据的函数。

假设我们要给设备发送72字节的数据，头8个字节是报告头，是我们定义的和设备相关的规则，后64位是数据。

```C
/* HID驱动的实现(这里只是用代码来有助理解，代码是伪代码) */
int Device_Report(int fd, unsigned char *buffer72)
{
    int ret; /* 保存ioctl函数的返回值 */
    int index;
    unsigned char send_data[72]; /* 发送的数据 */
    unsigned char recv_data[72]; /* 接收的数据 */
    struct hiddev_usage_ref uref; /* hid驱动定义的数据包 */
    struct hiddev_report_info rinfo; /* hid驱动定义的 */

    memset(send_data, 0, sizeof(send_data));
    memset(recv_data, 0, sizeof(recv_data));
    memcpy(send_data, buffer72, 72);
   /* 这在发送数据之前必须调用的，初始化设备 */
    ret = ioctl(fd, HIDIOCINITREPORT, 0);
    if( ret !=0) {
        return NOT_OPENED_DEVICE;/* NOT_OPENED_DEVICE 属于自己定义宏 */
    }
    /* HID设备每次传输一个字节的数据包 */
    for(index = 0; index < 72; index++) {
        /* 设置发送数据的状态 */
        uref.report_type = HID_REPORT_TYPE_FEATURE;
        uref.report_id = HID_REPORT_ID_FIRST;
        uref.usage_index = index;
        uref.field_index = 0;
        uref.value = send_data[index];
        ioctl(fd, HIDIOCGUCODE, &uref);
        ret=ioctl(fd, HIDIOCSUSAGE, &uref);
        if(ret != 0 ){
           return UNKNOWN_ERROR;
    }
}
/* 发送数据 */
rinfo.report_type = HID_REPORT_TYPE_FEATURE;
rinfo.report_id = HID_REPORT_ID_FIRST;
rinfo.num_fields = 1;
ret=ioctl(fd, HIDIOCSREPORT, &rinfo);   /* 发送数据 */
if(ret != 0) {
        return WRITE_REPORT;
}
/* 接受数据 */
ret = ioctl(fd, HIDIOCINITREPORT, 0);
for(index = 0; index < 72; index++) {
    uref.report_type = HID_REPORT_TYPE_FEATURE;
    uref.report_id = HID_REPORT_ID_FIRST;
    uref.usage_index = index;
    uref.field_index = 0;
    ioctl(fd, HIDIOCGUCODE, &uref);
    ret = ioctl(fd, HIDIOCGUSAGE, &uref);
    if(ret != 0 ) {
        return UNKNOWN_ERROR;
    }
    recv_data[index] = uref.value;
}
memcpy(buffer72, recv_data, 72);
return SUCCESS;
}
```

```C
/* libusb驱动的实现 */
int Device_Report(int fd, unsigned char *buffer72)
{
    /* 定义设备句柄 */
    usb_dev_handle* Device_handle;
   
    /* save the data of send and receive */
    unsigned char   send_data[72];
    unsigned char   recv_data[72];
   
    int              send_len;
    int             recv_len;
   
    /* 数据置空 */
    memset(send_data, 0 , sizeof(send_data));
    memset(recv_data, 0 , sizeof(recv_data));
   
    /* 这里的g_list是全局的数据变量，里面可以存储相关设备的所需信息，当然我们也可以从函数形参中传输进来，设备的信息在打开设备时初始化，我们将在后面的总结中详细描述一下 */
    Device_handle = (usb_dev_handle*)(g_list[fd].device_handle);
    if (Device_handle == NULL) {
        return NOT_OPENED_DEVICE;
    }

    /* 这个函数前面已经说过，在操作设备前是必须调用的, 0是指用默认的设备 */
    usb_claim_interface(Device_handle, 0);
    /* 发送数据，所用到的宏定义在usb.h可以找到，我列出来大家看一下
        #define USB_ENDPOINT_OUT       0x00
        #define USB_TYPE_CLASS     (0x01 << 5)
        #define USB_RECIP_INTERFACE 0x01
        
        #define HID_REPORT_SET       0x09 */

    send_len = usb_control_msg(Device_handle,
                               USB_ENDPOINT_OUT + USB_TYPE_CLASS + USB_RECIP_INTERFACE,
                               HID_REPORT_SET,
                               0x300,
                               0,
                               send_data, 72, USB_TIMEOUT);

    /* 发送数据有错误 */
    if (send_len < 0) {
        return WRITE_REPORT;
    }
    if (send_len != 72) {
        return send_len;
    }

    /* 接受数据
        #define USB_ENDPOINT_IN         0x80
        #define USB_TYPE_CLASS          (0x01 << 5)
        #define USB_RECIP_INTERFACE        0x01
        #define HID_REPORT_GET          0x01
        */
    recv_len = usb_control_msg(Device_handle,
                               USB_ENDPOINT_IN + USB_TYPE_CLASS + USB_RECIP_INTERFACE,
                               HID_REPORT_GET,
                               0x300,
                               0,
                               recv_data, 72, USB_TIMEOUT);                    
    if (recv_len < 0) {
        printf("failed to retrieve report from USB device!\n");
        return READ_REPORT;
    }
   
    if (recv_len != 72) {
        return recv_len;
    }
      
    /* 和usb_claim_interface对应 */
    usb_release_interface(RY2_handle, 0);
    memcpy(buffer72, recv_data, 72);
    return SUCCESS;
}
```

## 3.4 关闭设备

假设我们定义的关闭设备的函数名是`Device_Close()`

```C
/* 使用hid驱动关闭设备 */
int Device_Close()
{
    int handle;
   
    handle = open(“hiddev0”, O_RDONLY);

    /* 传统HID驱动调用,通过close()设备文件就可 */
    close( handle );
}
```

```C
/* 使用libusb关闭驱动 */
int Device_Close()
{
    /* LIBUSB 驱动打开设备，这里写的是伪代码，不保证代码有用 */
    struct usb_device*    udev;
    usb_dev_handle*        device_handle;

    device_handle = usb_open(udev);

    /* libusb库使用usb_close关闭程序 */
    usb_close(device_handle);
}
```

## 3.5  libusb的驱动框架

前面我们看了些主要的libusb函数的使用，这里我们把前面的内容归纳下：
一般的驱动应该都包含如下接口：

- Device_Find();   /* 寻找设备接口 */
- Device_Open(); /* 打开设备接口 */
- Device_Write(); /* 写设备接口 */
- Device_Read(); /* 读设备接口 */
- Device_Close(); /* 关闭设备接口 */

具体代码如下：

```C
#include <usb.h>
/* usb.h这个头文件是要包括的，里面包含了必须要用到的数据结构 */
/* 我们将一个设备的属性用一个结构体来概括 */
typedef struct
{
    struct usb_device*    udev;
    usb_dev_handle*        device_handle;
    /* 这里可以添加设备的其他属性，这里只列出每个设备要用到的属性 */
} device_descript;
/* 用来设置传输数据的时间延迟 */
#define USB_TIMEOUT     10000

/* 厂家ID 和产品 ID */
#define VENDOR_ID    0xffff    
#define PRODUCT_ID   0xffff

/* 这里定义数组来保存设备的相关属性，DEVICE_MINOR可以设置能够同时操作的设备数量，用全局变量的目的在于方便保存属性 */
#define DEVICE_MINOR 16
int     g_num;
device_descript g_list[ DEVICE_MINOR ];

/* 我们写个设备先找到设备，并把相关信息保存在 g_list 中 */
int Device_Find()
{
    struct usb_bus    *bus;
    struct usb_device *dev;
    g_num = 0;
    usb_find_busses();
    usb_find_devices();
   
    /* 寻找设备 */
    for (bus = usb_busses; bus; bus = bus->next) {
        for (dev = bus->devices; dev; dev = dev->next) {
            if(dev->descriptor.idVendor==VENDOR_ID&& dev->descriptor.idProduct == PRODUCT_ID) {
                /* 保存设备信息 */
                if (g_num < DEVICE_MINOR) {
                    g_list[g_num].udev = dev;  
                    g_num ++;
                }              
            }       
        }
    }
   
    return g_num;
}

/* 找到设备后，我们根据信息打开设备 */
int Device_Open()
{
    /* 根据情况打开你所需要操作的设备，这里我们仅列出伪代码 */
    if(g_list[g_num].udev != NULL) {
        g_list[g_num].device_handle = usb_open(g_list[g_num].udev);
}
}
/* 下面就是操作设备的函数了，我们就不列出来拉，大家可以参考上面的介绍 */
int DeviceWite(int handle)
{
    /* 填写相关代码，具体查看设备协议*/
}
int DeviceOpen(int handle)
{
    /* 填写相关代码，具体查看设备协议 */
}
/* 最后不要忘记关闭设备 */
void Device_close(int handle)
{
    /* 调用usb_close */
}
```

# 4. 小结

到此，使用libusb进行驱动开发介绍完了，通过对库所提供的API的使用可以体会到libusb的易用性。
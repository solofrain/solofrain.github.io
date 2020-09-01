QE-Pro
==

# 1. Seabreeze library (3.0.11)

- src/native/usb/linux/NativeUSBLinux.c

    ```C
    55 typedef struct {
    56     long deviceID;  /* Unique ID for device.  Assigned by this driver */
    57     struct usb_dev_handle *dev;
    58 } __usb_interface_t;
    59 
    60 typedef struct {
    61     long deviceID;  /* Unique ID for device.  Assigned by this driver. */
    62     __usb_interface_t *handle;    /* Pointer to USB interface instance */
    63     /* These paths could probably be made dynamic if they occupy too much space */
    64     char bus_location[PATH_MAX + 1];       /* effective bus directory */
    65     char device_location[PATH_MAX + 1];    /* Location of device relative to bus */
    66     unsigned short vendorID;
    67     unsigned short productID;
    68     unsigned char valid;    /* Whether this struct is valid */
    69     unsigned char mark;     /* Used to determine if device is still present */
    70 } __device_instance_t;

    104 static __device_instance_t *__lookup_device_instance_by_ID(long deviceID)

    123 static __device_instance_t *__lookup_device_instance_by_location(const char *bus_location,
    124         const char *device_location)

    146 static __device_instance_t *__add_device_instance(const char *bus_location,
    147         const char *device_location, int vendorID, int productID)

    167 static void __purge_unmarked_device_instances(int vendorID, int productID)

    211 static void __close_and_dealloc_usb_interface(__usb_interface_t *usb)
    ...
    220            usb_reset(usb->dev);
    221
    222            usb_close(usb->dev);

    232 static int  __probe_devices()

    253 int
    254 USBProbeDevices(int vendorID, int productID, unsigned long *output,
    255         int max_devices)

    352 void *
    353 USBOpen(unsigned long deviceID, int *errorCode) {
    ...
    383     for(bus = usb_busses; bus; bus = bus->next) {
    384         for(device = bus->devices; device; device = device->next) {
    ...
    390                 deviceHandle = usb_open(device);
    ...
    396                 interface = device->config->interface->altsetting->bInterfaceNumber;
    397                 int claim_err = usb_claim_interface(deviceHandle, interface);
    ...
    403                     usb_close(deviceHandle);
    
    486  int
    487 USBClose(void * deviceHandle)
    ...
    496     usb = (__usb_interface_t *) deviceHandle;
    ...
    504     __close_and_dealloc_usb_interface(usb);
    ```

    ```C
    /** A USB bus */
    struct usb_bus {
            /** Name */
            const char *name;
            /** Underlying hardware device */
            struct device *dev;
            /** Host controller operations set */
            struct usb_host_operations *op;
    
            /** Largest transfer allowed on the bus */
            size_t mtu;
            /** Address in-use mask
            *
            * This is used only by buses which perform manual address
            * assignment.  USB allows for addresses in the range [1,127].
            * We use a simple bitmask which restricts us to the range
            * [1,64]; this is unlikely to be a problem in practice.  For
            * comparison: controllers which perform autonomous address
            * assignment (such as xHCI) typically allow for only 32
            * devices per bus anyway.
            */
            unsigned long long addresses;
    
            /** Root hub */
            struct usb_hub *hub;
    
            /** List of USB buses */
            struct list_head list;
            /** List of devices */
            struct list_head devices;
            /** List of hubs */
            struct list_head hubs;
    
            /** Host controller operations */
            struct usb_bus_host_operations *host;
            /** Host controller private data */
            void *priv;
    };

    typedef struct usb_dev_handle {
        int fd;
        struct usb_bus *bus;
        struct usb_device *device;
        int config;
        int interface;
        int altsetting;
        /* Added by RMT so implementations can store other per-open-device data */
        void *impl_info;
    } usb_dev_handle;
    ```

- The issue is most likely that interface is claimed by another Linux driver. call  `libusb_detach_kernel_driver()` and specify the interface number and then you should be able to connect it.
- Did you call libusb_set_configuration() before `libusb_claim_interface()`? This must be called even if there is only one configuration in the descriptor.

SORRY, THIS DOES NOT WORK! Let me know if you are able to make it work. VirtualHere commerical solution (https://www.virtualhere.com/) worked just fine...

# Bluetooth over USB/IP
I've been running zigbee2mqtt on my RT-AX86U Pro for a while now and it's been wonderful as my router is in excellent place signal wise and my home automation server is in very bad one. Recently I wanted to use Bluetooth USB stick but the same problem applies. However, unlike zigbee2mqtt which can be ran on router itself, running Bluetooth anything would be probably too much to ask. But how about sharing USB stick itself over TCP/IP? Turns out that too works nicely. 

## Install entware
https://github.com/jarpatus/asuswrt_scripts/blob/main/README.md

## Create swap
https://github.com/jarpatus/asuswrt_scripts/blob/main/swapfile.md

Note: On models having 1GB RAM swap may not be necessary, however perhaps it's best to use it just to be on safe side.

## USB/IP driver 
https://github.com/jarpatus/asuswrt_scripts/blob/main/kernel_modules.md

```
CONFIG_USBIP_CORE=m
```

When you compile modules, you will get prompted what to do for sub-modules, mark Host driver to be compiled as module. 

## usbip
Install usbip:

```
opkg install usbip-server usbip-client
```

Load required modules:

```
insmod /opt/opt/modules/usbip-core.ko
insmod /opt/opt/modules/usbip-host.ko
```

Start usbipd:
```
/opt/etc/init.d/S74usbipd start
```

List devices, take a note of busid of your USB stick:

```
usbip list -l
```

Bind your USB stick i.e.:
```
usbip bind -b 1-2.1
```


```
```





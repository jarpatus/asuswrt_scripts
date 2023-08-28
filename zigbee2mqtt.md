# Run zigbee2mqtt on Asuswrt / Asuswrt-Merlin
If Asus router is ran as AP only then there is more than enough available power to run zigbee2mqtt on router. This could be useful since routers are usually on good spot and you don't have to dedicate yet another rpi zero or something for the task. Use fast USB stick and remember to use USB extension cable for the zigbee stick!

## Entware
Install entware, if using Asuswrt-Merlin see official wiki (https://github.com/RMerl/asuswrt-merlin.ng/wiki/), if running stock then see instructions (https://github.com/jarpatus/asuswrt_scripts/blob/main/entware_on_stock.md).

## Swap
There may not be enough memory to compile nodejs modules so create swapfile as instructed in Asuswrt-Merlin wiki or if using stock then use mkswap and write custom init file to enable swap on boot (TODO).  

## USB-to-serial driver
Most likely kernel won't contain driver for USB-to-serial adapter used by you zigbee stick. If using Asuswrt-Merlin then follow official wiki on how to download and compile custom firmware. I found it easiest to use docker. Set driver to be compiled as a module in config_base.6a i.e. change `# CONFIG_USB_SERIAL_CP210X is not set` is not set to `CONFIG_USB_SERIAL_CP210X=m`. Compile but there is no need to actually upload custom firmware to your router, just scp resulting module to the router e.g. drivers/usb/serial/cp210x.ko to /opt/opt/cp210x or so.

## Node.js
```
opkg install node node-npm
mkdir /opt/opt
mkdir /opt/opt
```



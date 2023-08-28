# Run zigbee2mqtt on Asuswrt / Asuswrt-Merlin
If Asus router is ran as AP only then there is more than enough available power to run zigbee2mqtt on router. This could be useful since routers are usually on good spot and you don't have to dedicate yet another rpi zero or something for the task. Use fast USB stick and remember to use USB extension cable for the zigbee stick!

## Entware
Install entware, if using Asuswrt-Merlin see official wiki (https://github.com/RMerl/asuswrt-merlin.ng/wiki/), if running stock then see instructions (https://github.com/jarpatus/asuswrt_scripts/blob/main/entware_on_stock.md).

## Swap
There may not be enough memory to compile nodejs modules so create swapfile as instructed in Asuswrt-Merlin wiki or if using stock then use mkswap and write custom init file to enable swap on boot.  

## USB-to-serial driver
Most likely kernel won't contain driver for USB-to-serial adapter used by you zigbee stick. If using Asuswrt-Merlin then follow official wiki on how to download and compile custom firmware. I found it easiest to use docker. Set driver to be compiled as a module in config_base.6a i.e. change `# CONFIG_USB_SERIAL_CP210X is not set` is not set to `CONFIG_USB_SERIAL_CP210X=m`. Compile but there is no need to actually upload custom firmware to your router, just scp resulting module to the router e.g. drivers/usb/serial/cp210x.ko to /opt/opt/cp210x or so.

Then remove option module (if loaded and load usbserial and USB-to-serial modules i.e.:
```
rmmod option
modprobe usbserial
insmod /opt/opt/cp210x/cp210x.ko
```

## Node.js
Install Node.js required by the zigbee2mqtt and git for pulling latest zigbee2mqtt.

```
opkg install node node-npm git-http
```

## zigbee2mqtt
Install zigbee2mqtt.

```
mkdir /opt/opt
git clone --depth 1 https://github.com/Koenkk/zigbee2mqtt.git /opt/opt/zigbee2mqtt
cd /opt/opt/zigbee2mqtt
npm ci
```

Start zigbee2mqtt.
```
npm start
```

## Start on boot
To load kernel modules and start zigbee2mqtt on boot create init file /opt/etc/init.d/S50zigbee2mqtt containing following script:

```
```




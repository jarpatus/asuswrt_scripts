# Run zigbee2mqtt on Asuswrt / Asuswrt-Merlin
In many Asus routers there is more than enough available power to run zigbee2mqtt on router. This could be useful since routers are usually on good spot and you don't have to dedicate yet another rpi zero or something for the task. Use fast USB stick and remember to use USB extension cable for the zigbee stick!

## Install entware
Install entware, if using Asuswrt-Merlin see official wiki (https://github.com/RMerl/asuswrt-merlin.ng/wiki/). If running stock then see instructions (https://github.com/jarpatus/asuswrt_scripts/blob/main/entware_on_stock.md).

## Swap
There may not be enough memory to compile nodejs modules. Create swapfile as instructed in official Asuswrt-Merlin wiki. If using stock firmware and only if using stock then take following steps:

- Create swapfile 
```
dd if=/dev/zero of=/opt/swapfile bs=1M count=4096
mkswap /opt/swapfile
```

- Create init file /opt/etc/init.d/01swap which enables swap on boot:
```
#!/bin/sh

DESC=swap
SWAPFILE=/opt/swapfile

ACTION=$1

ansi_red="\033[1;31m";
ansi_white="\033[1;37m";
ansi_green="\033[1;32m";
ansi_std="\033[m";

start() {
	echo -e -n "$ansi_white Starting $DESC... $ansi_std"
	swapon $SWAPFILE
	if [ $? -ne 0 ]; then
		echo -e "            $ansi_red failed. $ansi_std"
		logger "Failed to start $DESC from $CALLER."
		return 255
	else
		echo -e "            $ansi_green done. $ansi_std"
		logger "Started $DESC from $CALLER."
		return 0
	fi
}

stop() {
	echo -e -n "$ansi_white Shutting down $DESC... $ansi_std"
	swapoff $SWAPFILE
	if [ $? -ne 0 ]; then
		echo -e "       $ansi_red failed. $ansi_std"
		return 255
	else
		echo -e "       $ansi_green done. $ansi_std"
		return 0
	fi
}

check() {
	echo -e -n "$ansi_white Checking $DESC... "
		if cat /proc/swaps | grep swapfile > /dev/null; then
		echo -e "            $ansi_green alive. $ansi_std";
		return 0
	else
		echo -e "            $ansi_red dead. $ansi_std";
		return 1
	fi
}

case $ACTION in
	start)
		check || start
		;;
	stop | kill )
		check && stop
		;;
	restart)
		check > /dev/null && stop
		start
		;;
	check)
		check
		;;
	reconfigure)
		stop
		;;
	*)
		echo -e "$ansi_white Usage: $0 (start|stop|restart|check|kill|reconfigure)$ansi_std"
		exit 1
		;;
esac

```


to enable swap on boot:

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
#!/bin/bash

ENABLED=yes
PRECMD="rmmod option ; modprobe usbserial ; insmod /opt/opt/cp210x/cp210x.ko"
PROCS=node
ARGS="/opt/opt/zigbee2mqtt/index.js"
PREARGS=""
DESC="zigbee2mqtt"
PATH=/opt/sbin:/opt/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

. /opt/etc/init.d/rc.func
```





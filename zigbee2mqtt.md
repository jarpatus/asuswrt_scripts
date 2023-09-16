# zigbee2mqtt
In many Asus routers there is more than enough available power to run zigbee2mqtt on router. This could be useful since routers are usually on good spot and you don't have to dedicate yet another rpi zero or something for the task. Use fast USB stick and remember to use USB extension cable for the zigbee stick!

## Install entware
https://github.com/jarpatus/asuswrt_scripts/blob/main/README.md.

## Create swap
https://github.com/jarpatus/asuswrt_scripts/blob/main/swapfile.md.

Note: On my RT-AX86U Pro with 1GB memory, no swap were needed. Memory consumption were 610 MB when compiling. 

## USB-to-serial driver
Most likely kernel won't contain driver for USB-to-serial adapter used by you zigbee stick. Building new modules for stock firmware is bit black magic and there seem to be no clear instructions? I had success to build cp210x module by steps something like this:

- Get GPL sources for your router from Asus
- Use docker and following image for build environment https://hub.docker.com/r/gnuton/asuswrt-merlin-toolchains-docker
- Start container in asuswrt/ folder
- Install some required packages:
```
sudo apt install gdisk
```

- Prepare building environment

```
ln -s release/src-rt-5.04axhnd.675x/toolchains/ /opt
export LD_LIBRARY_PATH=/opt/toolchains/crosstools-arm-gcc-9.2-linux-4.19-glibc-2.30-binutils-2.32/usr/lib
export TOOLCHAIN_BASE=/opt/toolchains
export PATH=/opt/toolchains/crosstools-arm-gcc-9.2-linux-4.19-glibc-2.30-binutils-2.32/usr/bin:/opt/toolchains/crosstools-aarch64-gcc-9.2-linux-4.19-glibc-2.30-binutils-2.32/usr/bin:/projects/hnd/tools/linux/hndtools-armeabi-2011.09/bin:$PATH
```

  
```
```



- Make some required top level stuff like maketargets command (probably am using this wrong):
- ```
source /home/docker/envs/bcm-hnd-ax-4.19.sh 
cd release/src-rt
make rt-ax86u
```

 
- 

is a bit tricky and is not instructed here, but in essence you need to download GPL pakcage for your router which contains kernel sources and setup suitable environment perhaps using docker and some ready made environment from docker hub.  

If using Asuswrt-Merlin then follow official wiki on how to download and compile custom firmware. I found it easiest to use docker. Set required driver to be compiled as a module in config_base.6a i.e. change `# CONFIG_USB_SERIAL_CP210X is not set` to `CONFIG_USB_SERIAL_CP210X=m`. Compile kernel `make kernel` and copy resulting module to the router e.g. copy drivers/usb/serial/cp210x.ko from compiled kernel to /opt/opt/cp210x on your router.

Then load required module:
```
insmod /opt/opt/cp210x/cp210x.ko
```

Looks like it is possible to use module compuiled for Asuswrt-Merlin for stock firmware as well, but YMMV.

## Node.js
Install Node.js as zigbee2mqtt is build on it. Also install git for pulling zigbee2mqtt sources and daemonize for running zigbee2mqtt as a daemon.

```
opkg install node node-npm git-http daemonize
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
#!/bin/sh

DESC=zigbee2mqtt
DIR=/opt/opt/zigbee2mqtt
PIDFILE=/tmp/$DESC.pid

ACTION=$1
CALLER=$2

ansi_red="\033[1;31m";
ansi_white="\033[1;37m";
ansi_green="\033[1;32m";
ansi_std="\033[m";

start() {
        echo -e -n "$ansi_white Starting $DESC... $ansi_std"
        insmod /opt/opt/cp210x/cp210x.ko 2> /dev/null
        daemonize -c $DIR -p $PIDFILE /opt/bin/node index.js
        if [ $? -eq 0 ]; then
                echo -e "            $ansi_green done. $ansi_std"
                logger "Started $DESC from $CALLER."
                return 0
        else
                echo -e "            $ansi_red failed. $ansi_std"
                logger "Failed to start $DESC from $CALLER."
                return 255
        fi
}

stop() {
        echo -e -n "$ansi_white Shutting down $DESC... $ansi_std"
        kill `cat $PIDFILE`
        while kill -0 `cat $PIDFILE` 2> /dev/null; do
            sleep 1
        done
        if [ $? -eq 0 ]; then
                echo -e "       $ansi_green done. $ansi_std"
                return 0
        else
                echo -e "       $ansi_red failed. $ansi_std"
                return 255
        fi
}

_kill() {
        echo -e -n "$ansi_white Killing $DESC... $ansi_std"
        kill -9 `cat $PIDFILE`
        if [ $? -eq 0 ]; then
                echo -e "             $ansi_green done. $ansi_std"
                return 0
        else
                echo -e "             $ansi_red failed. $ansi_std"
                return 255
        fi
 }

check() {
        echo -e -n "$ansi_white Checking $DESC... "
        test -f $PIDFILE && kill -0 `cat $PIDFILE` 2> /dev/null
        if [ $? -eq 0 ]; then
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
        stop)
                check && stop
                ;;
        kill)
                check && _kill
                ;;
        restart)
                check && stop
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

# zigbee2mqtt
In many Asus routers there is more than enough available power to run zigbee2mqtt on router. This could be useful since routers are usually on good spot and you don't have to dedicate yet another rpi zero or something for the task. Use fast USB stick and remember to use USB extension cable for the zigbee stick!

## Install entware
https://github.com/jarpatus/asuswrt_scripts/blob/main/README.md

## Create swap
https://github.com/jarpatus/asuswrt_scripts/blob/main/swapfile.md

Note: On models having 1GB RAM swap may not be necessary, however perhaps it's best to use it just to be on safe side.

## USB-to-serial driver
https://github.com/jarpatus/asuswrt_scripts/blob/main/kernel_modules.md

## Node.js
Install Node.js as zigbee2mqtt is build on it. Also install git for pulling zigbee2mqtt sources and daemonize for running zigbee2mqtt as a daemon.

```
opkg install node node-npm git-http daemonize
```

Note: At least on RT-AX86U Pro node.js version v18.17.1 seems to be unstable and crash when doing any I/O heavy operations. Version v16.19.1 from archive seems to work just fine (https://bin.entware.net/aarch64-k3.10/archive/):

```
cd /opt/tmp
wget https://bin.entware.net/aarch64-k3.10/archive/node_v16.19.1-1_aarch64-3.10.ipk https://bin.entware.net/aarch64-k3.10/archive/node-npm_v16.19.1-1_aarch64-3.10.ipk
opkg install node_v16.19.1-1_aarch64-3.10.ipk node-npm_v16.19.1-1_aarch64-3.10.ipk git-http daemonize
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

- Make it executable
```
chmod +x /opt/etc/init.d/S50zigbee2mqtt
```

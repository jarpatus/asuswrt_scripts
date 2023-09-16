# Swap
There may not be enough memory to do everything (i.e. if you try to install zigbee2mqtt, it will compile some node.js modules and could run out of memory) so swap file may be needed:

- Create swapfile 
```
dd if=/dev/zero of=/opt/swapfile bs=1M count=4096
mkswap /opt/swapfile
```

- Create init file /opt/etc/init.d/01swapfile which enables swapfile on boot:
```
#!/bin/sh

DESC=swapfile
SWAPFILE=/opt/swapfile

ACTION=$1
CALLER=$2

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

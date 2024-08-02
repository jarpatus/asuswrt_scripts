# Compile additional kernel modules
Stock firmware does not include support for too many USB devices and it may be necessary to compile new modules. Building new modules for stock firmware is bit tricky and poorly documented and exact steps seems to vary withing each firmware release, but these steps were successfull with RT-AX86U Pro version 3.0.0.4.388_24199:

- Get GPL sources for your router from Asus, router's support page, choose Driver & Tools and Others as OS
- Extract sources to some Linux machine (VM if you have to)
- Start docker container from gnuton's image (https://hub.docker.com/r/gnuton/asuswrt-merlin-toolchains-docker) in asuswrt/ folder:
```
cd asuswrt/
docker run -it --rm -v "$PWD:/build" gnuton/asuswrt-merlin-toolchains-docker /bin/bash
```

- From docker container prepare building environment and go to correct release directory i.e.:
```
source /home/docker/envs/bcm-hnd-ax-4.19.sh
cd /build/release/src-rt-5.04axhnd.675x/
```

- Run make with correct router model number as an argument (see chip_profile.mak for list of available model numbers). In case you already ran make once, chip_profile.tmp must be removed first:
```
rm chip_profile.tmp
make RT-AX86U_PRO
```

This will create chip_profile.tmp containing build details of your router and will try to build whole firmware. As we do not need whole firmware, Ctrl+C and just build kernel image and modules instead:
```
make RT-AX86U_PRO image
make RT-AX86U_PRO modules
```

For me both commands produced an error at some point (missing version.h), but got far enough to actually compile kernel image and modules. Edit kernel/linux-4.19/.config and add modules you want to, e.g.:

```
CONFIG_USB_SERIAL_CP210X=m
```

Recompile modules (probably you could just compile one module and also without pre-compiling anything, but at least this worked for me) and ssh them to your router i.e.:

```
cat ./targets/94912GW/modules/lib/modules/4.19.183/kernel/drivers/usb/serial/cp210x.ko | ssh user@rt-ax86u "cat > /opt/opt/modules/cp210x.ko"
```

SSH to your router and load module e.g.:

```
modprobe usbserial
insmod /opt/opt/modules/cp210x.ko
```

You may or may not have to recompile module with new firmware releases.


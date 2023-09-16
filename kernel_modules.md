# Compile additional kernel modules
Stock firmware does not include support for too many USB devices and it may be necessary to compile new modules. Building new modules for stock firmware is bit tricky and poorly documented (?), but these steps were successfull with RT-AX86U Pro (3.0.0.4.388_23565):

- Get GPL sources for your router from Asus, router's support page, choose operating system Other
- Extract to some Linux box, VM or so
- Use gnuton's docker image: https://hub.docker.com/r/gnuton/asuswrt-merlin-toolchains-docker
- Start docker container in asuswrt/ folder
- Prepare building environment and go to correct release directory i.e.:
```
source /home/docker/envs/bcm-hnd-ax-4.19.sh
cd release/src-rt-5.04axhnd.675x/
```

At this point nothing seemed to work as it should have, until I found out that Makefile is missing one crucial row (?!) which should be added to the top of the Makefile if missing. Also keep in mind that you run make even once and it fails, it may have done lot's of settings files already and make clean seems not to work properly either -> start from scratch.  

```
include ./chip_profile.mak
```

Now it should be possible to build kernel and modules:

```
make kernel RT-AX86U_PRO image
make kernel RT-AX86U_PRO modules
```

For me both commands produced an error at some point (missing version.h), but got far enough to actually compile kernel image and modules. Edit kernel/linux-4.19/.config and add modules you want to, e.g.:

```
CONFIG_USB_SERIAL_CP210X=m
```

Recompile modules (probably you could just compile one module and also without pre-compiling anything, but at least this worked for me) and ssh them to your router i.e.:

```
cat ./targets/94912GW/modules/lib/modules/4.19.183/kernel/drivers/usb/serial/cp210x.ko | ssh user@rt-ax86u "cat > /opt/opt/cp210x.ko"
```

SSH to your router and load module:

```
modprobe usbserial
insmod /opt/opt/cp210x.ko
```

You may or may not have to recompile module with new firmware releases.


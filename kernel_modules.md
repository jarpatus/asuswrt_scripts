# Compile additional kernel modules
Stock firmware does not include that many modules and it may be necessary to compile new modules e.g. for USB devices like USB-to-serial adapters. Building new modules for stock firmware is bit black magic and poorly documented (?) but these steps were successfull with RT-AX86U Pro and 3.0.0.4.388_23565:

- Get GPL sources for your router from Asus (router support page, operating system Other )
- Extract
- Use gnuton's docker image: https://hub.docker.com/r/gnuton/asuswrt-merlin-toolchains-docker
- Start container in asuswrt/ folder
- Prepare building environment and go to correct release directory e.g.:
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

For me both commands produced an error at some point, but got far enough to actually compile kernel image and modules. Edit kernel/linux-4.19/.config and add modules you want to, e.g.:

```
CONFIG_USB_SERIAL_CP210X=m
```

Recompile modules and ssh them to your router:

```

```




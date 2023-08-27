# Install entware on stock Asuswrt firmware
Usually Asuswrt-Merlin firmware is used when customizing Asus routers (you really should use it) but if for reason or another you want to stay stock but still use entware, it is possible. There are guides out there which do utilize script_usbmount setting in nvram to run custom script on USB mount but it is possible to do it without nvram so you can plug USB stick to factory resetted router and it just works.

# Compatibility
Tested with RT-AC86U.

# Background
Entware likes to be installed to /opt. On stock Asuswrt /opt is symlinkked to /tmp/opt, which does not exist. On the other hand, when USB stick is mounted then stock firmware checks if asusware.arm/.asusrouter file exists on the stick and if it does, firmware will symlink /tmp/opt to /tmp/mnt/.../asusware.arm so /opt now points to asusware.arm directory on your USB stick. Additionally asusware.arm/.asusrouter file will be executed and /opt/sbin, /opt/bin, /opt/usr/sbin and /opt/usr/bin will be added to the path.

This behaviour can be witnessed by installing Download Master from GUI and checking contents of the USB stick afterwards.

# Entware installation
- Format USB stick to ext3 file system if not already ext3.
- Create asusware.arm/.asusrouter file to USB stick and chmod to 777:
```
cd /tmp/mnt/.../
mkdir asusware.arm
chmod 777 asusware.arm/
touch asusware.arm/.asusrouter
chmod 777 asusware.arm/.asusrouter
```
Reboot and check that /opt now really is symlink to asusware.arm on your USB stick.
- Install entware
```
cd /opt
wget -O - http://bin.entware.net/aarch64-k3.10/installer/generic.sh | sh
```
Check contents of /opt to validate. Reboot and see that entware installation survives the boot.

# Entware initialization
Stock firmware will also run all init files from /opt/etc/init.d on start and create some kind of pseudo PID file (?) named /opt/initfile.1. So in example /opt/etc/init.d/S99debug will be run and /opt/S99debug.1 will be created. When debugging please notice that /tmp seems to be cleared (?) after init so if you try to validate init by creating files to /tmp you won't see them. Use /opt/tmp instead. 

Entware does want to /opt/etc/init.d/rc.unslung to be executed on start but it also just start init files. So rc.unslung can be left alone and everything still should work. I realize that this could be source of compatibility problems so YMMV (when tested, at least lighttpd seems to init just fine).

# Sources and links
- https://github.com/Entware/Entware/wiki




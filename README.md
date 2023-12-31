# Entware on stock Asuswrt
Usually Asuswrt-Merlin firmware is used when customizing Asus routers (you really should use it) but if for reason or another you want to stay stock and still use entware, it is possible. There are guides out there which do utilize script_usbmount setting in nvram to run custom script on USB mount, but it is possible to do it without nvram as well. Also according to some sources these nvram variables do longer work with some models and newest firmwares.

This guide taps into stock Asuswrt mechanism to install additional features like Download Master and mounts custom entware installation on top of stock Asuswrt. No need for nvram changes and it is possible to plug USB stick to factory resetted router and it just works.

# Compatibility
Tested with RT-AX86U Pro. YMMV.

# Background
Entware must to be installed to /opt. On stock Asuswrt /opt is either symlinkked to /tmp/opt, which does not exist, or subfolders under /opt are symlinkked to /tmp/opt/... which do not exist either. Looks like older models symlink whole /opt and newer models folders under /opt.

When an USB stick is mounted then stock firmware checks if asusware.arm/.asusrouter file exists on the stick and if it does, firmware will symlink /tmp/opt to /tmp/mnt/sdaX/asusware.arm so /opt now points to asusware.arm directory on your USB stick. 

Additionally /opt/sbin, /opt/bin, /opt/usr/sbin and /opt/usr/bin will be added to the path. Seems that in the past asusware.arm/.asusrouter file was executed as well but that seems no longer to be the case. 

Stock firmware will then do initialization by running /usr/sbin/app_init_run.sh which executes service specific init files from /opt/etc/init.d if service has been set as enabled in /opt/lib/ipkg/info/service.control file.

This behaviour can be witnessed by installing Download Master from GUI and checking contents of the USB stick afterwards. Like said earlier, exact behaviour seems to differ from model to model or firmware to firmware so adjustments may be necessary .

# Prepare USB stick
From now own it is assumed that your USB stick device file is /dev/sda. If not adjust all commands accordingly.

- Create one Linux partition to USB stick and format it as ext3 file system (sda is your USB stick device, usually sda but doublce check, also it is assumed you know how to use fdisk).
```
fdisk /dev/sda
mkfs.ext3 /dev/sda1
```

- Reboot and check that USB stick was mounted to /tmp/mnt/sda1
- Create empty asusware.arm/.asusrouter file which triggers stock initialization process:
```
cd /tmp/mnt/sda1/
mkdir asusware.arm 
touch asusware.arm/.asusrouter
```

- Create custom initialization and control files which will mount entware folder as new /opt:
```
cd /tmp/mnt/sda1/
mkdir asusware.arm/etc asusware.arm/etc/init.d asusware.arm/lib asusware.arm/lib/ipkg asusware.arm/lib/ipkg/info entware
echo -e "#!/bin/sh\nmount -o bind /tmp/mnt/sda1/entware /opt\n/opt/etc/init.d/rc.unslung start" > asusware.arm/etc/init.d/S50entware
echo "Enabled: yes" > asusware.arm/lib/ipkg/info/entware.control
```

- Reboot the router and double check that /tmp/opt is now symlinked to /tmp/mnt/sda1/asusware.arm and that /dev/sda1 is now mounted to /opt (to be exact, entware folder from it).
```
ls -ld /tmp/opt
mount |grep sda1
```

# Entware installation

- Install entware
```
cd /opt
wget -O - http://bin.entware.net/aarch64-k3.10/installer/generic.sh | sh
```
Check contents of /opt to validate. Reboot and see that entware installation survives the boot.

- Install usefull packages i.e.:
```
opkg install htop nano
```

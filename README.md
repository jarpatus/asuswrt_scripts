# Install entware on stock Asuswrt firmware
Usually Asuswrt-Merlin firmware is used when customizing Asus routers (you really should use it) but if for reason or another you want to stay stock but still use entware, it is possible. There are guides out there which do utilize script_usbmount setting in nvram to run custom script on USB mount but it is possible to do it without nvram so you can plug USB stick to factory resetted router and it just works.

# Compatibility
Tested with RT-AC86U.

# Background
Entware likes to be installed to /opt/entware. On stock Asuswrt /opt is symlinkked to /tmp/opt, which does not exist. On the other hand, when USB stick is mounted then stock firmware checks if asusware.arm/.asusrouter file exists and if it does, it will symlink /tmp/opt to /tmp/mnt/sdaX/asusware.arm so /opt now points top asusware.arm directory on your USB stick. Additionally asusware.arm/.asusrouter file will be executed which allows initializing entware.

This behaviour can be witnessed by installing Download Master from GUI and checking contents of the USB stick afterwards.

# Entware installation
- Format USB stick to ext3 file system if not already ext3.
- Create asusware.arm/.asusrouter file and chmod file and folder to 777:
```
mkdir asusware.arm
chmod 777 asusware.arm/
echo -e "#!/bin/sh\ntouch /tmp/success" > asusware.arm/.asusrouter
chmod 777 asusware.arm/.asusrouter
```

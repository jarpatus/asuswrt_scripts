# Run zigbee2mqtt on Asuswrt / Asuswrt-Merlin
If Asus router is ran as AP only then there is more than enough available power to run zigbee2mqtt on router. This could be useful since routers are usually on good spot and you don't have to dedicate yet another rpi zero or something for the task. Use fast USB stick and remember to use USB extension cable!

## Entware
Install entware, if using Asuswrt-Merlin see official documentation (https://github.com/RMerl/asuswrt-merlin.ng/wiki/), if running stock then see instructions (https://github.com/jarpatus/asuswrt_scripts/blob/main/entware_on_stock.md).

## Swap
There may not be enough memory to compile nodejs modules so create swapfile as instructed in Asuswrt-Merlin wiki or if using stock then use mkswap and write custom init file to enable swap on boot (TODO).  

```
opkg install node node-npm
mkdir /opt/opt
mkdir /opt/opt
```



# RaspiWRT-openwrt-bk

### This is the backup repo for openwrt on Raspberry pi 1B
https://openwrt.org/toh/raspberry_pi_foundation/raspberry_pi

#### 1. Write the openwrt image to SD card. 
If you are using [Edimax N150 Wi-Fi Nano USB Adapter](https://www.edimax.com/edimax/merchandise/merchandise_detail/data/edimax/global/home_legacy_wireless_adapters/ew-7811un/), you need to build [**kmod-rtl8192cu**](https://openwrt.org/packages/pkgdata/kmod-rtl8192cu) package into the image file.

*reference* : https://www.reddit.com/r/openwrt/comments/gfexrb/edimax_raspberry_pi/

Click [here](https://firmware-selector.openwrt.org/) and to search your device.

Click customize package and add **kmod-rtl8192cu** into the list.

[**openwrt-23.05.0-5d7a7f24d1fe-bcm27xx-bcm2708-rpi-ext4-factory.img.gz**](./openwrt-23.05.0-5d7a7f24d1fe-bcm27xx-bcm2708-rpi-ext4-factory.img.gz) is openwrt 23.05 with default packages and kmod-rtl8192cu.

on PC :
```shell
gunzip openwrt-23.05.0-5d7a7f24d1fe-bcm27xx-bcm2708-rpi-ext4-factory.img.gz
```

You will get **openwrt-23.05.0-5d7a7f24d1fe-bcm27xx-bcm2708-rpi-ext4-factory.img**

You can use **dd** or raspberry pi image writer to write the image to your SD card

#### 2. Basic access point(ap) setting for RaspiWRT
This setting uses *LAN port* for WAN and use *wlan0* to share internet to other wifi devices.

And WAN is use dhcp to get the WAN IP.

1. insert SD card with openwrt image into raspberry pi
2. power on raspberry pi and connect it to your PC with LAN Cable.
3. Set PC ip as **192.168.1.100** and connect raspberry pi with ssh

on PC :
```shell
ssh root@192.168.1.1
```

Before restore config files, backup original config files.

on rasperry pi(192.168.1.1) :
```shell
cp /etc/config/firewall /etc/config/firewall.bk
cp /etc/config/network /etc/config/network.bk
cp /etc/config/wireless /etc/config/wireless.bk
cp /etc/config/system /etc/config/system.bk
```

Scp config files into raspberry pi to restore config files.

on PC :
```shell
scp ./rootfs/etc/config/firewall root@192.168.1.1:/etc/config/
scp ./rootfs/etc/config/network root@192.168.1.1:/etc/config/
scp ./rootfs/etc/config/wireless root@192.168.1.1:/etc/config/
scp ./rootfs/etc/config/system root@192.168.1.1:/etc/config/
scp ./rootfs/etc/freememory.sh root@192.168.1.1:/etc/
scp ./profile root@192.168.1.1:~/.profile
reboot
```

After reboot, you will see the wifi signal as below.

The SSID is **RaspiWRT** , connection password is **open-wrt** , local hostname is **raspiwrt**

unplug LAN cable and connect to raspberry pi(RaspiWRT) via wifi.

*reference* : https://note.com/tango9512357/n/nad6f93d635dc

#### 3. Extend your storage size to filesystem

login to raspbery pi(RaspiWRT)
on PC :
```shell
ssh root@192.168.10.1
```

on RaspiWRT(192.168.10.1) :
```shell
opkg update
opkg install cfdisk resize2fs tune2fs lsblk
cfdisk /dev/mmcblk0
```

resize **mmcblk02** partition then write. After exit, reboot the RaspiWRT.

login to raspbery pi(RaspiWRT) again
on PC :
```shell
ssh root@192.168.10.1
```

on RaspiWRT(192.168.10.1) :
```shell
mount -o remount,ro /
tune2fs -O^resize_inode /dev/mmcblk0p2
e2fsck -f /dev/mmcblk0p2
tune2fs -O^resize_inode /dev/mmcblk0p2
fsck.ext4 /dev/mmcblk0p2 
resize2fs /dev/mmcblk0p2
df -h
```

After **df -h**, you can see your filesystem size is extended.

*reference* : https://github.com/rahulelex/resize-storage-on-openwrt-raspberry-pi


#### 4. Install general package (in RaspiWRT)

on RaspiWRT(192.168.10.1) :
``` shell
opkg install block-mount ca-bundle ca-certificates f2fsck fuse-utils glib2 kmod-usb-storage kmod-usb2 kmod-usb3 usbutils dropbearconvert
opkg install bzip2 rename rsync sudo tree unzip unrar whereis nano vim lsof curl lua htop luci-theme-material perl bc 
chmod +x /etc/freememory.sh
crontab -e
```
> 0 */2 * * 1 /etc/freememory.sh

Or just restore with **list-installed.txt**. (just check the txt file for what will be installed)

on PC : 
``` shell
scp ./rootfs/list-installed.txt root@192.168.10.1:/tmp
``` 

on RaspiWRT(192.168.10.1) :
``` shell
opkg update
cat /tmp/list-installed.txt | xargs opkg install
```

**bk_pkg_list.sh** is the script to back up your opkg installed list.

#### 5. Install avahi daemon

on RaspiWRT(192.168.10.1) :
``` shell
opkg update
opkg install avahi-dbus-daemon avahi-utils
```
The local domain is **raspiwrt.local**.

#### 6. Install aria2
on RaspiWRT(192.168.10.1) :
``` shell
opkg update
opkg install aria2 luci-app-aria2
mkdir -p /root/share/downloads
chmod 777 -R /root/share/downloads
cd /tmp
wget --no-check-certificate https://github.com/binux/yaaw/zipball/master -O yaaw.zip
unzip yaaw.zip
mv binux-yaaw-*  /www/yaaw
rm yaaw.zip
```

on PC : 
``` shell
scp ./rootfs/etc/config/aria2 root@192.168.10.1:/etc/config/
``` 
 The default download folder is **/root/share/downloads**
 
 Remote site is **http://192.168.10.1/yaaw**

#### 7. Install adblock fast
on RaspiWRT(192.168.10.1) :
``` shell
opkg update
opkg install adblock-fast luci-app-adblock-fast
uci set adblock-fast.config.enabled='1'
uci commit adblock-fast
```

on PC : 
``` shell
scp ./rootfs/etc/config/adblock-fast root@192.168.10.1:/etc/config/
``` 

#### 8. Install ttyd
on RaspiWRT(192.168.10.1) :
``` shell
opkg update
opkg install ttyd luci-app-ttyd
```

on PC : 
``` shell
scp ./rootfs/etc/config/ttyd root@192.168.10.1:/etc/config/
``` 

The ttyd port is **800** and only available under __192.168.10.*__

Access **http://192.168.10.1:800** for ttyd.


#### 9. Install samba4 server
on RaspiWRT(192.168.10.1) :
``` shell
opkg update
opkg install samba4-server luci-app-samba4
```

on PC : 
``` shell
scp ./rootfs/etc/config/samba4 root@192.168.10.1:/etc/config/
``` 

The share folder is set as **/root/share** and guest ok.

#### 10. Install minidlna server
on RaspiWRT(192.168.10.1) :
``` shell
opkg update
opkg install minidlna luci-app-minidlna 
```

on PC : 
``` shell
scp ./rootfs/etc/config/minidlna root@192.168.10.1:/etc/config/
``` 

The media scan path is set as **/root/share**.


#### 11. Install sshtunnel client
on RaspiWRT(192.168.10.1) :
``` shell
opkg update
opkg install sshtunnel luci-app-sshtunnel 
```

``` shell
cd ~
mkdir .ssh
chmod 700 .ssh/
dropbearkey -t rsa -f /root/.ssh/id_dropbear
```
That last command will print the public key to the console, which we can copy and paste into a file:
``` shell
vi ~/.ssh/id_rsa.pub
```
The same public key can also be copied into ~/.ssh/authorized_keys on ssh server we want to connect to.
``` shell
scp -p [port] ~/.ssh/id_rsa.pub [account]@[my.ssh.server]:~/.ssh/authorized_keys
```
The Dropbear key needs to be converted, after installing the tool to do that:
``` shell
dropbearconvert dropbear openssh ~/.ssh/id_dropbear ~/.ssh/id_rsa
``` 
Now you can log to ssh server without input password.
``` shell
ssh -p [port] [account]@[my.ssh.server]
``` 

Then to restore sshtunnel config 

on PC :
``` shell
scp ./rootfs/etc/config/sshtunnel root@192.168.10.1:/etc/config
```
default is pios server, check config for more detail.

proxy port is **1234**.

Assign socket v5 proxy as **socket://192.68.10.1:1234**

_reference_ : https://blog.thestateofme.com/2022/10/26/socks-proxy-ssh-tunnels-on-openwrt/


#### 11. Install ddns
on RaspiWRT(192.168.10.1) :
``` shell
opkg update
opkg install ddns-scripts luci-app-ddns ddns-scripts-noip
```


#### 11. Install convenient luci app
on RaspiWRT(192.168.10.1) :
``` shell
opkg update
opkg install luci-app-acl luci-app-wifischedule luci-app-commands 
```

#### 12. Install polipo
on RaspiWRT(192.168.10.1) :
``` shell
opkg update
opkg install polipo luci-app-polipo	
```

on PC :
``` shell
scp ./rootfs/etc/config/polipo root@192.168.10.1:/etc/config
```
http proxy port is **4321**. http proxy is based on socket5 proxy which provied by **sshtunnel**.

Assign http proxy as **http://192.168.10.1:4321** on client side. 

_reference_ : https://blog.thestateofme.com/2022/10/26/socks-proxy-ssh-tunnels-on-openwrt/


#### 12. Install alist
on RaspiWRT(192.168.10.1) :
``` shell
sh -c "$(curl -ksS https://raw.githubusercontent.com/sbwml/luci-app-alist/master/install.sh)"
```
This script will detect your prefered language, locale and CPU architechture, then install the following ipks.

It will take about 40~50MB storage for installation.

1. alist*.ipk
2. luci-app-alist*.ipk
3. luci-i18n*.ipk

on PC :
``` shell
scp ./rootfs/etc/config/alist root@192.168.10.1:/etc/config
```
the config file set access port as **8080** and lan access only.  Access **http://192.168.10.1:8080** for alist.

Go to _Storage_ tab to setup monunt path first.

_reference_ : https://github.com/sbwml/luci-app-alist/

### Appedix

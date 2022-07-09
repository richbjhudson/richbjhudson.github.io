---
layout: post
title:  "Linux OpenSUSE - Hyper-V Enhanced Session"
date:   2022-07-09 18:59:00 +0000
categories: Linux
---
# Assumptions
- Linux OpenSUSE Virtual Machine is already configured and working using console connection.
- GNOME desktop environment is set.

# Steps
- Install the hyper-v-enhanced-session package:
```
sudo zypper install hyper-v-enhanced-session
```
- [Configuring Xorg as the default GNOME session](https://docs.fedoraproject.org/en-US/quick-docs/configuring-xorg-as-default-gnome-session/).
- Create certificate and private key for xrdp:
```
openssl req -x509 -newkey rsa:2048 -nodes -keyout key.pem -out cert.pem -days 365
```
- Edit `sudo vi /etc/xrdp/xrdp.ini` file to use the `cert.pem` and `key.pem` files from previous step.
- Create xrdp session preference file `startwm.sh`:
```
cp /etc/xrdp/startwm.sh.userwindowmanager-sample ~/
cd ~
mv startwm.sh.userwindowmanager-sample startwm.sh
``` 
- Edit `vi startwm.sh` by uncommenting `PREF_SESSION='gnome'`
- Reboot computer.
---
layout: post
title:  "Linux OpenSUSE - Hyper-V Enhanced Session"
date:   2022-07-08 20:00 +0000
categories: Linux
---
# Assumptions
- A Linux OpenSUSE Virtual Machine is already configured and working using a console connection.
- A GNOME desktop environment is being used.

# Steps
- Install the hyper-v-enhanced-session package:
```
sudo zypper install hyper-v-enhanced-session
```
- [Configuring Xorg as the default GNOME session](https://docs.fedoraproject.org/en-US/quick-docs/configuring-xorg-as-default-gnome-session/).
- Create a certificate and private key for xrdp:
```
openssl req -x509 -newkey rsa:2048 -nodes -keyout key.pem -out cert.pem -days 365
```
- Edit `sudo vi /etc/xrdp/xrdp.ini` file to use the `cert.pem` and `key.pem` files from previous step.
- Create a xrdp session preference file `startwm.sh` in the user's home directory:
```
cp /etc/xrdp/startwm.sh.userwindowmanager-sample ~/
cd ~
mv startwm.sh.userwindowmanager-sample startwm.sh
``` 
- Edit `vi startwm.sh` by uncommenting `PREF_SESSION='gnome'`
- Reboot the computer for the settings to apply.
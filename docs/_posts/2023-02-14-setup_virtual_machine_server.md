---
layout: post
title:  "Linux - How to Setup Virtual Machine Server"
date:   2023-02-14 16:00 +0000
categories: Linux
---
# Prerequisites
- *Ubuntu 22.04.1 LTS* installed on a computer. If you are using a Virtual Machine you must ensure that nested virtualisation is supported and enabled.
- A Workstation to connect to the Virtual Machine Server.
- The Virtual Machine Server should have 2 NICs, 1 for SSH connectivity and a 2nd that can be used to configure a network bridge device. If you are using a Virtual Machine you must ensure that MAC address spoofing is allowed for the 2nd NIC.
- A DHCP Server on the same network as the 2nd NIC.
- Adequate storage to store vDisks and VM Images. These instructions assume that you have created a new logical volume for this purpose.

# Steps
## Virtual Machine Server Setup
- Install packages and check libvirtd is running:
    - `sudo apt update`
    - `sudo apt install bridge-utils libvirt-clients libvirt-daemon-system qemu-system-x86`
    - `systemctl status libvirtd`
- Configure logical volume for vDisk and VM Image storage:
    - `sudo vi /etc/fstab` and add the following line: 
    `/dev/vg-images/lv-images   /var/lib/libvirt/images ext4    defaults    0   1`
    - Mount the filesystem using `sudo mount -a`
    - Grant the *kvm* group permission to `/var/lib/libvirt/images` using:
        - `sudo chown :kvm /var/lib/libvirt/images`
        - `sudo chmod g+rw /var/lib/libvirt/images`
        - `sudo systemctl restart libvirtd`
        - `sudo systemctl status libvirtd`
- Grant your user permission to manage the Virtual Machine Server:
    - `sudo usermod -aG kvm rich`
    - `sudo usermod -aG libvirt rich`
- Update the network configuration to include a network bridge device so that Virtual Machine may connect to the network - `sudo vi /etc/netplan/00-installer-config.yaml`:

```
network:
  version: 2
  ethernets:
    eth0:
      addresses:
        - 192.168.101.83/24
      nameservers:
        addresses: [192.168.101.81]
      routes:
        - to: default
          via: 192.168.101.1
    eth1:
      dhcp4: false

  bridges:
    br0:
      interfaces: [eth1]
      dhcp4: true
      parameters:
        stp: false
        forward-delay: 0
```
    
    - `sudo netplan apply`

## Virtual Machine Manager Workstation Setup
- Install Virtual Machine Manager to connect to Virtual Machine Server:
    - `sudo apt install ssh-askpass virt-manager`
    - Connect to the Virtual Machine Server using SSH before you attempt to use Virtual Machine Manager.
    - Open Virtual Machine Manager and *Add Connection*
    <img src="/linux_screenshots/vmm.png" alt="vmm">
- Create a storage pool to store ISOs:
    - Right-click the *Connection> Details* then select the *Storage* tab.
    - *Create a storage pool* with *Target Path: /var/lib/libvirt/images/ISO*.

## Virtual Machine Server ISO Library Setup
- Make sure the permissions are set correctly for `/var/lib/libvirt/images/ISO`:
    - `sudo chown root:kvm /var/lib/libvirt/images/ISO`
    - `sudo chmod g+rw /var/lib/libvirt/images/ISO`
- Download ISO Image to ready to create a VIrtual Machine:
    - `cd /var/lib/libvirt/images/ISO`
    - `wget https://releases.ubuntu.com/22.04.1/ubuntu-22.04.1-live-server-amd64.iso`


## Virtual Machine Manager Workstation Virtual Machine Deployment
- Within Virtual Machine Manager, right-click a Connection and create *New VM*.
  - Use the ISO Library path to select an ISO.
  - A *Bridge device...* with *Device name: br0* should be set for your *Network selection*.
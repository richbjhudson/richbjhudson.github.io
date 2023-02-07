---
layout: techNote
category: Linux - Ubuntu Server
title: Operations and Deployment
---
## Configure kernel parameters, persistent and non-persistent
- Display kernel parameters settings: `sysctl -a` 
- Configure a kernel parameter at runtime: `sudo sysctl net.ipv4.ip_forward=1`
- Configure a kernel parameter persistently within `/etc/sysctl.conf` and apply the configuration without a reboot using `sysctl -p`.

## Diagnose, identify, manage, and troubleshoot processes and services

## Manage or schedule jobs for executing commands

## Search for, install, validate, and maintain software packages or repositories

## Recover from hardware, operating system, or filesystem failures

## Manage Virtual Machines (libvirt)

## Configure container engines, create and manage containers

## Create and enforce MAC using SELinux

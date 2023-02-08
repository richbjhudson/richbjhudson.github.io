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
### Processes
- `htop` is an interactive process viewer:
<img src="/linux_screenshots/htop.png" alt="htop">
- You may view the *nice value* (default 0) and *priority* (default 80) of a process using `ps -l`. 
    - The priority of an existing and new process can be managed using `nice -n +20 top` and `sudo renice -n 0 1190` respectively.
    - The niceness value may range between -20 to +19, the value is added to the priority (80 + nice value) so a high nice value will lower the priority of a process.
- You may send a *signal* to a process to notify the instruct the process to take an action, `kill -l` displays the signal types.
    - `kill 1190` may be used to send a signal to a process ID, it uses the default signal SIGTERM that attempts to gracefully terminate a process. You may force a process to terminate using `kill -9 1190`.
    - `killall top` may be used to send a signal to processes by name.
- <kbd>Ctrl</kbd> + <kbd>Z</kbd> places a process/*job* into a stopped state.
    - `jobs` displays the status of running processes/ jobs.
    - From here you can continue to run a job in the background using `bg 1` or in the forground using `fg 1`. `fg` without a job number brings the most recent job to the foreground. 
    - <kbd>Ctrl</kbd> + <kbd>C</kbd> terminates a process.

### Services
- A *daemon/ service* is 

## Manage or schedule jobs for executing commands

## Search for, install, validate, and maintain software packages or repositories

## Recover from hardware, operating system, or filesystem failures

## Manage Virtual Machines (libvirt)

## Configure container engines, create and manage containers

## Create and enforce MAC using SELinux

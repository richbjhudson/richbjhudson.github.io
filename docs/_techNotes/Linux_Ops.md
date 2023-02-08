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
- You may send a *signal* to a process to instruct the process to take an action, `kill -l` displays the signal types.
    - `kill 1190` may be used to send a signal to a process ID, it uses the default signal SIGTERM that attempts to gracefully terminate a process. You may force a process to terminate using `kill -9 1190`.
    - `killall top` may be used to send a signal to processes by name.
- <kbd>Ctrl</kbd> + <kbd>Z</kbd> places a process/*job* into a stopped state.
    - `jobs` displays the status of running processes/ jobs.
    - From here you can continue to run a job in the background using `bg 1` or in the forground using `fg 1`. `fg` without a job number brings the most recent job to the foreground. 
    - <kbd>Ctrl</kbd> + <kbd>C</kbd> terminates a process.

### Services
- You may perform service/daemon management tasks using `systemctl`.
    - `systemctl list-units -t service` display all active services.
    - `systemctl status ssh` shows the status of a service including recent log information that may also be gathered using `journalctl -u ssh`.
    - You can change the running state of a service using `systemctl [start/stop] ssh` and change the startup mode with `systemctl [enable/disable] ssh`.

## Manage or schedule jobs for executing commands
- *cron* automatically runs commands in the background under the user context that created the job.
- `sudo crontab -u root -l` may be used to view system-wide scheduled jobs for a user.
- You can configure cron for the logged in user with `crontab -e` that will create a configuration file under `/var/spool/cron/crontabs/rich`.

## Search for, install, validate, and maintain software packages or repositories
### APT
- `sudo apt update` is used to download package information from sources configured in `/etc/apt/source.list.d`.
- `apt search htop` is used to search for available packages. Whereas `apt list --installed` shows the packages already installed.
- You may add, remove or remove a package and its configuration with the following command `apt [install/remove/purge] htop`.
- You can install a specific version of a package using `sudo apt install kubelet=1.25.5-00` and then prevent the package from being upgraded using `sudo apt-mark hold kubelet`.
- `apt show htop` display verbose information about a package including dependencies.
- `sudo apt upgrade` can be used to upgrade existing packages, `sudo apt dist-upgrade` in addition upgrades the Kernel and adds/removes dependant packages automatically.
- `apt` is a frontend to `dpkg`. `dpkg -V htop` is a useful command to verify package installation.

### SNAP
- A more modern approach to packaging software is `snap` that allows an application and its dependencies to be bundled together so that it works across Linux distros.
- You can serach for available packages using `snap find firefox`.
- I package may be installed, removed or updated with the following commands `sudo snap [install/remove/refresh] firefox`. If `sudo snap refresh` is run without specifying a package name all packages on the system are updated.

## Recover from hardware, operating system, or filesystem failures


## Manage Virtual Machines (libvirt)

## Configure container engines, create and manage containers

## Create and enforce MAC using SELinux

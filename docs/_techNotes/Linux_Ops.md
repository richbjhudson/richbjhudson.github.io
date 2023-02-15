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
- `apt show htop` displays verbose information about a package including dependencies.
- `sudo apt upgrade` can be used to upgrade existing packages, `sudo apt dist-upgrade` in addition upgrades the Kernel and adds/removes dependant packages automatically.
- `sudo apt-get clean` may be used to clear the *apt cache* to reclaim valuable diskspace. 
- `apt` is a frontend to `dpkg`. `dpkg -V htop` is a useful command to verify package installation.

### SNAP
- A more modern approach to packaging software is `snap` that allows an application and its dependencies to be bundled together so that it works across Linux distros.
- You can search for available packages using `snap find firefox`.
- A package may be installed, removed or updated with the following command `sudo snap [install/remove/refresh] firefox`. If `sudo snap refresh` is run without specifying a package name all packages on the system are updated.

## Recover from hardware, operating system, or filesystem failures
### Live Media 
- Boot Virtual Machine from ISO image and choose *Try or Install Ubuntu Server*.
    - From the *Welcome!* screen select *Help* followed by *Enter Shell*.
    - Determine the logical volume path of the existing filesystem with `lvdisplay`.
    - Make a directory ready to mount the filesystem using `sudo mkdir /mnt/sysimage`.
    - Mount the existing filesystem `sudo mount /dev/ubuntu-vg/ubuntu-lv /mnt/sysimage`
    - Change context into the existing filesystem using `sudo chroot /mnt/sysimage`.
    - Perform maintenance task e.g. reset root password: `passwd root`.

### Grub Menu
- *Single User Mode* is typically used when Ubuntu can't boot normally. *Emergency Mode* is used as a last attempt to retrieve data before re-install. 
- Power off and then on your Virtual Machine for the *GRUB Menu* to display, select the *Ubuntu* option and press <kbd>E</kbd>.
- Enter *emergency/ single user boot mode* by appending `emergency` or `single` (and a space) to the kernel command that starts with `linux` respectively. Then <kbd>F10</kbd> to boot.

### Check and Repair Filesystem
- `sudo fsck.ext4 /dev/sda1` may be used to run a filesystem check against an unmounted filesystem.
- `sudo touch /forcefsck` may be used to run a filesystem check against a mounted filesystem at the next reboot.
- You can review `fsck` activity in `/run/initramfs/fsck.log`.

## Manage Virtual Machines (libvirt)
- Libvirt is a toolkit/ API used to interact with Virtual Machine Servers.
- `virsh list` may display running VMs.
- `virsh [start/shutdown/suspend/resume] VM01` may be used to change the state of a Virtual Machine.

## Configure container engines, create and manage containers


## Create and enforce MAC using SELinux

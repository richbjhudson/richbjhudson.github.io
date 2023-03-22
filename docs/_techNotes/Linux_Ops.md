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
- `dpkg -S pstree` can be used to identify which package a command was installed from.

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
- [Setup a Virtual Machine Server]({{ site.baseurl }}/linux/2023/02/14/setup_virtual_machine_server/)
- Libvirt is a toolkit/ API used to interact with Virtual Machine Servers.
- `virsh list` may display running VMs.
- `virsh [start/shutdown/suspend/resume] VM01` may be used to change the state of a Virtual Machine.

## Configure container engines, create and manage containers
### Docker - Setup
- Install Docker:
```
sudo apt install docker.io
systemctl status docker
```
- Allow your user account to run *docker* commands without *sudo*: `sudo usermod -aG docker rich` *Note: You will have to logoff and then back in again for the permission to take effect.*

### Docker - Managing Container Images
- Search for a *docker* image using `docker search ubuntu`. You can findout further information about an image by browsing [dockerhub](https://hub.docker.com/).
- Download a local copy of an image for use `docker pull ubuntu`.
- Show local images `docker images` and obtain the *IMAGE ID* for subsequent commands.
- Remove an image `docker rmi 12a`.
- You can create a *container image* by capturing the running state of a container with `docker commit c7 ubuntu/apache2:1.0`. `docker ps` would be used to obtain the *CONTAINER ID*.
    - `docker images` will confirm that a new image has been created.
    - You can run a new container using the image `docker run -dit -p 8081:80 ubuntu/apache2:1.0 /bin/bash`.
    - Attach your shell to the container `docker attach 7e`, start apache2 `/etc/init.d/apache2 start` and then exit the interactive shell with <kbd>Ctrl</kbd> then press <kbd>P</kbd> followed by <kbd>Q</kbd>.
    - Confirm that *apache2* is exposed on Host port 8081 `curl http://localhost:8081`.
- Automate the process of building a *container image*.
    - Create a new directory `mkdir apache_custom`.
    - Within the new directory `vi Dockerfile` and add the following:
    ```
    FROM ubuntu
    MAINTAINER Rich <richard.b.j.hudson@gmail.com>
    # Avoid confirmation messages
    ARG DEBIAN_FRONTEND=noninteractive
    # Update the container's packages
    RUN apt update; apt dist-upgrade -y
    # Install Apache and VIM
    RUN apt install -y apache2 vim-nox
    # Start Apache
    ENTRYPOINT apachectl -D FOREGROUND
    ``` 
    - Within the new directory, build an image using the *Dockerfile* `docker build -t ubuntu/apache_custom:1.0 .`
    - Run a container using the image `docker run -dit -p 8080:80 ubuntu/apache_custom:1.0`

### Docker - Managing Containers
- Runs the ubuntu image as a container in interactive mode `docker run -it ubuntu /bin/bash`.
    - Hold onto <kbd>Ctrl</kbd> then press <kbd>P</kbd> followed by <kbd>Q</kbd> to exit the interactive shell without terminating the container.
- List running containers `docker ps` or list all containers `docker ps -all`. This set of commands can be used to obtain the *CONTAINER ID* for use in subsequent commands.
- Attach your shell to a running container `docker attach 3bdd9cd79b2a`. If you exit the container shell the container will be stopped.
- You can change the running state of the container using `docker [start/stop] 3bdd9cd79b2a`
- Remove a containers using `docker rm 3bdd9cd79b2a`. *Note: The container has to be in a stopped state.* 
- You can execute a container in background mode and expose a listening port from within the container to the host. For example, Host port 8080 is redirected to port 80 within the container  `docker run -dit -p 8080:80 ubuntu /bin/bash`.
    - You can install and start apache2 within the container to test with the following commands:
    ```
    docker ps
    docker attach f9
    apt update
    apt install apache2
    /etc/init.d/apache2 start
    ``` 
    *Note: There is no init system inside a container so you cannot run systemctl commands.*
    - Hold onto <kbd>Ctrl</kbd> then press <kbd>P</kbd> followed by <kbd>Q</kbd> to exit the interactive shell without terminating the container.
    - Confirm that *apache2* is exposed on Host port 8080 `curl http://localhost:8080`.
- You can create a container in a stopped state with `docker create -p 8080:80 ubuntu`.

### LXD - Setup
- Install LXD:
```
sudo snap install lxd
snap list lxd
```
- Allow your user account to run *lxd and lxc* commands without *sudo*: `sudo usermod -aG lxd rich` *Note: You will have to logoff and then back in again for the permission to take effect.*
- Initialise the setup of *LXD* e.g. setup a storage pool, using `lxd init`.

### LXD - Managing Container Images
- Pull a container image and run a container `lxc launch ubuntu:22.04 container01`.
    - LXD does not have a concept of layers and all changes made are persistent.
- List container images `lxc image list`, obtain the *FINGERPRINT* for further commands.
- Remove a container image `lxc image delete 6d`. 

### LXD - Managing Containers
- List containers `lxc list`, and obtain the container name for use with further commands.
- You can change the running state of the container using `lxc [start/stop/delete] container01`.
- Open a shell to a container using `lxc exec container01 bash`.
    - You can open a shell to a container with another user with `lxc exec container01 -- su --login ubuntu`.
- Set a container to boot when the LXD Host Server starts with `lxc config set container01 boot.autostart 1`.
- You can use a network bridge connection *br0* to provide outside access to your containers in the same way that we did to VMs using KVM - please see: [Virtual Machine Server Setup]({{ site.baseurl }}/linux/2023/02/14/setup_virtual_machine_server#Virtual-Machine-Server-Setup).
    - Create a *lxc profile* ready to configure the container networking `lxc profile create bridge-profile`.
    - Edit the *lxc profile* to to map a NIC (eth1) within the container to the LXD Host Server network bridge device (br0) `lxc profile edit bridge-profile` - the configuration should look similar to below:
    ```
    config: {}
    description: Network Bridge Connection Profile
    devices:
        eth1:
            name: eth1
            nictype: bridged
            parent: br0
            type: nic
    name: bridge-profile
    used_by: []
    ```
    - Run the container using the *lxc profile* `lxc launch ubuntu:22.04 container02 -p default -p bridge-profile`. *Note: The default profile is loaded first, followed by the bridge-profile profile to ensure the latter overrides any conflicting settings.*
    - Connect to the container and install apache2:
    ```
    lxc exec container02 bash
    apt install apache2
    exit
    ```
    - Obtain the IP Address of the container `lxc list` and test connectivity `curl http://192.168.101.112`. *Note: You can add a lxc profile to an existing container using `lxc profile add container01 bridge-profile`.*

### Kubernetes
- [Setup a K8s Cluster]({{ site.baseurl }}/linux/2023/02/15/setup_k8s/)
- [Deploy Containers to a K8s Cluster]({{ site.baseurl }}/linux/2023/02/17/deploy_containers_to_k8s/)

## Create and enforce MAC using SELinux 
- Disable *apparmor* before you enable *SELinux*:
```
sudo systemctl stop apparmor
sudo systemctl disable apparmor
sudo systemctl status apparmor
```
- Display the status of *SELinux* using either `getenforce` or `sestatus`. To use the tools you may need to install *selinux-utils*, *policycoreutils* and *selinux-basics*: 
```
sudo apt install selinux-basics selinux-utils policycoreutils  
```
- At this stage `sestatus` will report *disabled*, enable SELinux using `sudo selinux-activate` and then reboot.
- You may set the *SELinux Mode* for runtime using `sudo setenforce [Enforcing/ Permissive]`, it will default to *Permissive* after SELinux has been enabled. 
- You may set the *SELinux Mode* persistently by editing `sudo vi /etc/selinux/config` and adding:
```
SELINUX=disabled 
```
- `-Z` option often works with existing commands to display context/ labels, for example `ps axZ | grep apache2` and `ls -Z /var/www/html` shows that the *apache2* process runs in the context of *httpd_t* which is why it is allowed to access the files with a label of *httpd_sys_content_t*. 
- The files within a directory will inherit the context of the directory. This can be set using `semanage fcontext -a -t httpd_sys_content_t /var/www/html`.
    - Here is an example where a new file will inherit label *httpd_sys_content_t*:
    ```
    cd /var/www/html
    sudo sh -c "echo file 1 > ./file1.html"
    sudo setenforce Enforcing
    curl http://localhost/file1.html
    ```
- If a file is created in another directory and moved, it will not inherit the label from the destination directory and will need to be changed.
    - Here is an example where the label is changed from *user_home_t* to *httpd_sys_content_t*:
    ```
    sudo sh -c "echo file2 >/root/file2.html"
    sudo mv /root/file2.html /var/www/html
    sudo setenforce Enforcing
    curl http://localhost/file2.html
    ```
    - At this stage you will receive a *403 Forbidden* message and need to change the label on the file.
    ```
    sudo chcon -t httpd_sys_content_t file2.html
    curl http://localhost/file2.html
    ```
    - An alternative approach is to force the files within a directory to inherit the directory label using `sudo restorecon -RFv /var/www/html`.
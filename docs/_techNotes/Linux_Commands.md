---
layout: techNote
category: Linux - Ubuntu Server
title: Essential Commands
---
## Basic [Git](https://mirrors.edge.kernel.org/pub/software/scm/git/) Operations
- You can get help on a specific `git` command using `git help commit` or alternatively view the [Git User Manual](https://mirrors.edge.kernel.org/pub/software/scm/git/docs/user-manual.html) and [git-scm Documentation](https://git-scm.com/doc)
- `-s` option with each commit adds a *Signed-off-by:* entry required when using [DCO](https://wiki.linuxfoundation.org/dco) to document code contributions.
- `git gui` provides a graphical interface to `git`.

### Git Commands
- Create a local git repository - the version control information is stored in the *.git* subdirectory:
```
mkdir gitdir
cd gitdir
git init
ls -l .git
```
- Add new files to staging area ready to commit:

```
echo "This is testfile1" > testfile1.txt
echo "This is testfile2" > testfile2.txt
git add .
git status
```

- Set who is responsible for the repository:
```
git config --global user.name "Richard Hudson"
git config --global user.email "richard.b.j.hudson@gmail.com"
```
*Note: You may omit the `--global` option to set on a per repository basis.*

- Commit changes to files:

```
echo "Line2" >> testfile1.txt
cat testfile1.txt
git diff testfile1.txt
git add testfile1.txt
git status
git commit -m "Initial commit"
git status
```
*Note: You may sign off a git commit using `git commit -m "Initial commit" -s`.*

- View *git commit* history using `git log`. 
- Create and switch to a *main* branch to replace *master*:

```
git checkout -b main
git branch
git branch -d master
git branch
```

- Rename *master* branch to *main*:

```
git checkout master
git branch -m master main
git push -u origin main
git symbolic-ref refs/remotes/origin/HEAD refs/remotes/origin/main
git branch -a
```
*Note: You could simply create a new main branch and then ignore the master branch.*

## <a id="create-configure-and-troubleshoot-services"></a>Create, configure, and troubleshoot services
- [Systemd](https://manpages.ubuntu.com/manpages/bionic/man1/systemd.1.html) configures the environment and starts processes ready for user login. It replaces shell scripts with programs and provides on-demand daemon starting and can track processes using cgroups. 
- `systemctl` displays everything that *systemd* controls.
- You can list units of type *service* using `systemctl list-units -t service`.
- `systemctl status apache2` is used to display the status of a service.
- You can change the state of a service with `sudo systemctl [start/stop/restart] apache2` and reload its configuration using `sudo systemctl reload apache2`.
- `systemctl [enable/ disable] apache2` is used to set a service to start at boot time.

## <a id="monitor-and-troubleshoot-system-performance-and-services"></a>Monitor and troubleshoot system performance and services
### System Monitoring
<table>
<tr><th>Command</th><th>Description</th></tr>  
<tr><td>journalctl -u ssh</td><td>Query the systemd journal for a service.</td></tr>
<tr><td>tail -f /var/log/syslog</td><td>Some daemons may not have their own log file and may place logs in syslog.</td></tr>
<tr><td>dmesg</td><td>Useful to see hardware issues logged at the kernel level.
</td></tr>
</table>

### Process Monitoring
<table>
<tr><th>Command</th><th>Description</th></tr>  
<tr><td>htop</td><td>Interactive process viewer with mouse support.</td></tr>
<tr><td>pstree</td><td>Display a tree of processes and highlights targeted PID.</td></tr>
<tr><td>ps -elf</td><td>Report a snapshot of the current processes.</td></tr>
<tr><td>uptime</td><td>Tell how long the system has been running.</td></tr>
</table>

### Memory Monitoring
<table>
<tr><th>Command</th><th>Description</th></tr>  
<tr><td>free -h</td><td>Display amount of free and used memory in the system.</td></tr>
<tr><td>vmstat</td><td>Report virtual memory statistics.</td></tr>
</table>

## Determine application and service specific constraints
- [How to troubleshoot services]({{ site.baseurl }}/techNotes/Linux_Commands#create-configure-and-troubleshoot-services)
- [How to troubleshoot system performance issues]({{ site.baseurl }}/techNotes/Linux_Commands#monitor-and-troubleshoot-system-performance-and-services)
- [How to troubleshoot a SELinux label issue]({{ site.baseurl }}/techNotes/Linux_Ops/#create-and-enforce-mac-using-selinux)
- Most applications will have their own log file located in `/var/log`.
- `help ulimit` may be used to provide control over the resources available to the shell and processes.
    - To make changes effective for all logged-in users amend `/etc/security/limits.conf`.

## Troubleshoot diskspace issues
- `df -Th` displays filesystem usage with its type and human readable size.
- Once you have identified a filesystem that is running out of space, you may examine disk usage for the current directory using `du -hsc *`.
- `df -i` displays the number of inodes (metadata describing stored data) in use. A filesystem may report as full if an inode limit is hit. You would need to delete files to reclaim inodes.
- `ncdu -x /var` provides a tree view of your filesystem and <kbd>D</kbd> allows you to delete the selected file or directory.

## Work with SSL certificates
- [How to Setup an Apache Web Server]({{ site.baseurl }}/linux/2023/03/24/setup_apache2/)
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

## Create, configure, and troubleshoot services
- [Systemd](https://manpages.ubuntu.com/manpages/bionic/man1/systemd.1.html) configures the environment and starts processes ready for user login. It replaces shell scripts with programs and provides on-demand daemon starting and can track processes using cgroups. 
- `systemctl` displays everything that *systemd* controls.
- You can list units of type *service* using `systemctl list-units -t service`.
- `systemctl status apache2` is used to display the status of a service.
- You can change the state of a service with `sudo systemctl [start/stop/restart] apache2` and reload its configuration using `sudo systemctl reload apache2`.
- `systemctl [enable/ disable] apache2` is used to set a service to start at boot time.

## Monitor and troubleshoot system performance and services
### System Monitoring
<table>
<tr><th>Command</th><th>Description</th><th>Package</th></tr>  
<tr><td></td><td></td><td></td></tr>
<tr><td></td><td></td><td></td></tr>
</table>

### Process Monitoring
<table>
<tr><th>Command</th><th>Description</th><th>Package</th></tr>  
<tr><td>htop</td><td>Interactive process viewer</td><td>htop</td></tr>
<tr><td>pstree</td><td>Display a tree of processes</td><td>psmisc</td></tr>
<tr><td>ps</td><td>Report a snapshot of the current processes.</td><td>procps</td></tr>
<tr><td>uptime</td><td>Tell how long the system has been running.</td><td>procps</td></tr>
</table>

### Memory Monitoring
<table>
<tr><th>Command</th><th>Description</th><th>Package</th></tr>  
<tr><td></td><td></td><td></td></tr>
<tr><td></td><td></td><td></td></tr>
</table>

## Determine application and service specific constraints

## Troubleshoot diskspace issues

## Work with SSL certificates

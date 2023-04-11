---
layout: techNote
category: Linux - Ubuntu Server
title: Users and Groups
---
## Create and manage local user and group accounts
- Each user account is stored on a line in `/etc/passwd` with the following attributes: *Username:Password:UID:GID:Comment:Home:Shell*.
    - *Password* typically includes a placeholder value of *X* that indicates that the user's password is stored in `/etc/shadow`.
    -  `/etc/shadow` has permission settings of 400 as opposed to 644.
- Add a new user account using `sudo useradd -d /home/dicky -m dicky -s /bin/bash`. 
*Note: The `-m` option creates a new home directory if it does not exist.*
    - Set the new user's password with `sudo passwd dicky`.
    - The contents of the user's new home directory is pulled from `/etc/skel`.
- Delete a user with `sudo userdel -r dicky`. 
*Note: The `-r` removes the home directory*

## Manage personal and system-wide environment profiles

## Configure user resource limits

## Configure and manage ACLs

## Configure the system to use LDAP user and group accounts

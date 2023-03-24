---
layout: post
title:  "Linux - How to Setup an Apache Web Server"
date:   2023-03-24 13:30 +0000
categories: Linux
---
# Installation
- Install *apache2*:
```
sudo apt install apache2
systemctl status apache2
lynx http://192.168.101.81
```
- The main configuration file is `/etc/apache2/apache2.conf`, the document root is `/var/www/html` with a default landing page of `/var/www/html/index.html`.

# Configuring an Additional Site
- You can host multiple website using `/etc/apache2/sites-available` that includes configuration files for each website configuration with a matching criteria. If hosting single site then `000-default.conf` will suffice.
- Copy the `/etc/apache2/sites-available/000-default.conf` file. 
  - Amend the *VirtualHost value* to match an additional hostname or IP address, for example:
```
<VirtualHost 192.168.101.81:80>
```
```
<VirtualHost *:80>
ServerName: hexale.net
```
```
<VirtualHost *:443>
ServerName: hexale.net:443
```
  - Amend the *DocumentRoot value* to a subdirectory of `/var/www`.
  - Amend the *ErrorLog and CustomLog values* to include the Site name.
- You can enable a site configuration using `sudo a2ensite hexale.net.conf` and than reload the apache2 configuration using `sudo systemctl reload apache2`. `a2dissite` may be used to disable a site configuration.
*Note: The commands add / delete a symbolic link to the .conf file in `/etc/apache2/sites-enabled` to enable/ disable the site.*

# Installing Additional Apache Modules
-  `apt search libapache2-mod` to see a list of modules that may be installed.
- `sudo apt install libapache2-mod-php8.1` will install a module to support *PHP*.
- `a2enmod` will list all available modules and `sudo a2enmod php8.1` would install a specific module. `a2dismod` may be used to disable a module.
- Once a module is enabled you must restart the *apache2* service using `sudo systemctl restart apache2`.
- `apache2 -l` will display builtin modules.

# Secure a Site with TLS
- Enable SSL Apache Module:
```
sudo a2enmod ssl
sudo systemctl restart apache2
```
- Generate a certificate
  - Self-signed certificate example:
  ```
  sudo mkdir /etc/apache2/certs 
  sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/certs/mysite.key -out /etc/apache2/certs/mysite.crt
  ```
  - 3rd Party Certificate Authority example:
  ```
  sudo openssl req -new -newkey rsa:2048 -nodes -keyout server.key -out server.csr
  ```
  - Edit `/etc/apache2/sites-available/default-ssl.conf` and set the *SSLCertificateFile and SSLCertificateKeyFile values* as follows:
  ```
  SSLCertificateFile /etc/apache2/certs/mysite.crt
  SSLCertificateKeyFile /etc/apache2/certs/mysite.key
  ```
  - Enable the site configuration:
  ```
  sudo a2ensite default-ssl.conf
  sudo systemctl reload apache2
  lynx https://192.168.101.81
  ``` 
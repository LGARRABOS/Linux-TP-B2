# TP2 : Gestion de service
## I. Un premier serveur web
### 1. Installation

🌞 Installer le serveur Apache

```
[luc@ServeurWeb ~]$ sudo dnf install httpd
[...]
Installed:
  apr-1.7.0-11.el9.x86_64                apr-util-1.6.1-20.el9.x86_64           apr-util-bdb-1.6.1-20.el9.x86_64
  apr-util-openssl-1.6.1-20.el9.x86_64   httpd-2.4.51-7.el9_0.x86_64            httpd-filesystem-2.4.51-7.el9_0.noarch
  httpd-tools-2.4.51-7.el9_0.x86_64      mailcap-2.1.49-5.el9.noarch            mod_http2-1.15.19-2.el9.x86_64
  mod_lua-2.4.51-7.el9_0.x86_64          rocky-logos-httpd-90.11-1.el9.noarch

Complete!
```

🌞 Démarrer le service Apache

```
[luc@ServeurWeb httpd]$ sudo systemctl status httpd
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
     Active: active (running) since Tue 2022-11-15 10:16:17 CET; 35s ago
       Docs: man:httpd.service(8)
   Main PID: 33851 (httpd)
     Status: "Total requests: 0; Idle/Busy workers 100/0;Requests/sec: 0; Bytes served/sec:   0 B/sec"
      Tasks: 213 (limit: 5907)
     Memory: 24.7M
        CPU: 95ms
     CGroup: /system.slice/httpd.service
             ├─33851 /usr/sbin/httpd -DFOREGROUND
             ├─33852 /usr/sbin/httpd -DFOREGROUND
             ├─33853 /usr/sbin/httpd -DFOREGROUND
             ├─33854 /usr/sbin/httpd -DFOREGROUND
             └─33855 /usr/sbin/httpd -DFOREGROUND

Nov 15 10:16:16 ServeurWeb systemd[1]: Starting The Apache HTTP Server...
Nov 15 10:16:16 ServeurWeb httpd[33851]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using fe80::a00:27ff:feba:7>
Nov 15 10:16:17 ServeurWeb systemd[1]: Started The Apache HTTP Server.
Nov 15 10:16:17 ServeurWeb httpd[33851]: Server configured, listening on: port 80
```
```
[luc@ServeurWeb httpd]$ sudo ss -tulpn | grep LISTEN

tcp   LISTEN 0      511                *:80              *:*    users:(("httpd",pid=33855,fd=4) [...]
```

🌞 TEST

```
[luc@ServeurWeb httpd]$ sudo systemctl enable httpd
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
```
```
[luc@ServeurWeb httpd]$ curl localhost:80
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
      /*<![CDATA[*/
[...]
      <footer class="col-sm-12">
      <a href="https://apache.org">Apache&trade;</a> is a registered trademark of <a href="https://apache.org">the Apache Software Foundation</a> in the United States and/or other countries.<br />
      <a href="https://nginx.org">NGINX&trade;</a> is a registered trademark of <a href="https://">F5 Networks, Inc.</a>.
      </footer>

  </body>
</html>
```
### 2. Avancer vers la maîtrise du service

🌞 Le service Apache...

```
[luc@ServeurWeb /]$ sudo cat /etc/systemd/system/multi-user.target.wants/httpd.service
# See httpd.service(8) for more information on using the httpd service.

# Modifying this file in-place is not recommended, because changes
# will be overwritten during package upgrades.  To customize the
# behaviour, run "systemctl edit httpd" to create an override unit.

# For example, to pass additional options (such as -D definitions) to
# the httpd binary at startup, create an override unit (as is done by
# systemctl edit) and enter the following:

#       [Service]
#       Environment=OPTIONS=-DMY_DEFINE

[Unit]
Description=The Apache HTTP Server
Wants=httpd-init.service
After=network.target remote-fs.target nss-lookup.target httpd-init.service
Documentation=man:httpd.service(8)

[Service]
Type=notify
Environment=LANG=C

ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND
ExecReload=/usr/sbin/httpd $OPTIONS -k graceful
# Send SIGWINCH for graceful stop
KillSignal=SIGWINCH
KillMode=mixed
PrivateTmp=true
OOMPolicy=continue

[Install]
WantedBy=multi-user.target
```

🌞 Déterminer sous quel utilisateur tourne le processus Apache

```
[luc@ServeurWeb /]$ sudo cat /etc/httpd/conf/httpd.conf

User apache
Group apache
```
```
[luc@ServeurWeb /]$ ps -ef | grep apache
apache     33852   33851  0 10:16 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache     33853   33851  0 10:16 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache     33854   33851  0 10:16 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache     33855   33851  0 10:16 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
```
```
[luc@ServeurWeb testpage]$ ls -al

-rw-r--r--.  1 root root 7620 Jul  6 04:37 index.html
```

🌞 Changer l'utilisateur utilisé par Apache

```
[luc@ServeurWeb ~]$ sudo useradd web -m -s /sbin/nologin
```
```
[luc@ServeurWeb ~]$ ps -ef | grep web
web        34240   34239  0 10:42 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
web        34241   34239  0 10:42 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
web        34242   34239  0 10:42 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
web        34243   34239  0 10:42 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
```

🌞 Faites en sorte que Apache tourne sur un autre port

```
[luc@ServeurWeb ~]$ sudo firewall-cmd --remove-port=80/tcp --permanent
Warning: NOT_ENABLED: 80:tcp
success

[luc@ServeurWeb ~]$ sudo firewall-cmd --add-port=8080/tcp --permanent
success

[luc@ServeurWeb ~]$ sudo firewall-cmd --reload
success
```
```
[luc@ServeurWeb ~]$ sudo ss -tulpn | grep LISTEN

tcp   LISTEN 0      511                *:8080            *:*    users:(("httpd",pid=34499,fd=4),("httpd",pid=34498,fd=4) [...]
```
## II. Une stack web plus avancée
### A. Base de données

🌞 Install de MariaDB sur db.tp2.linux

```
[luc@BasedeDonnes ~]$ sudo dnf install mariadb-server

[luc@BasedeDonnes ~]$ sudo systemctl enable mariadb

[luc@BasedeDonnes ~]$ sudo systemctl start mariadb

[luc@BasedeDonnes ~]$ mysql_secure_installation
```
```
[luc@BasedeDonnes ~]$ sudo ss -tulpn | grep LISTEN

tcp   LISTEN 0      80                 *:3306            *:*    users:(("mariadbd",pid=35124,fd=19))
```

🌞 Préparation de la base pour NextCloud

```
sudo mysql -u root -p

CREATE USER 'nextcloud'@'10.102.1.11' IDENTIFIED BY 'pewpewpew';

CREATE DATABASE IF NOT EXISTS nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;

GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'10.102.1.11';

FLUSH PRIVILEGES;

```

🌞 Exploration de la base de données

```
[luc@ServeurWeb ~]$ mysql -u nextcloud -h 10.102.1.12 -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 30
Server version: 5.5.5-10.5.16-MariaDB MariaDB Server

Copyright (c) 2000, 2022, Oracle and/or its affiliates.
```

🌞 Trouver une commande SQL qui permet de lister tous les utilisateurs de la base de données

```
MariaDB [(none)]> select User,Host from mysql.user;
+-------------+-------------+
| User        | Host        |
+-------------+-------------+
| nextcloud   | 10.102.1.11 |
| mariadb.sys | localhost   |
| mysql       | localhost   |
| root        | localhost   |
+-------------+-------------+
```

### B. Serveur Web et NextCloud

🌞 Install de PHP

```
[luc@ServeurWeb ~]$ sudo dnf install -y php81-php


Complete!
```

🌞 Install de tous les modules PHP nécessaires pour NextCloud

```
[luc@ServeurWeb ~]$ sudo dnf install -y libxml2 openssl php81-php php81-php-ctype php81-php-curl php81-php-gd php81-php-iconv php81-php-json php81-php-libxml php81-php-mbstring php81-php-openssl php81-php-posix php81-php-session php81-php-xml php81-php-zip php81-php-zlib php81-php-pdo php81-php-mysqlnd php81-php-intl php81-php-bcmath php81-php-gmp


Complete!
```

🌞 Récupérer NextCloud

```
[luc@ServeurWeb tp2_nextcloud]$ ls -all
total 132
drwxr-xr-x. 14 apache apache  4096 Nov 15 12:14 .
drwxr-xr-x.  5 root   root      54 Nov 15 12:03 ..
drwxr-xr-x. 47 apache apache  4096 Oct  6 14:47 3rdparty
drwxr-xr-x. 50 apache apache  4096 Nov 15 12:13 apps
-rw-r--r--.  1 apache apache 19327 Nov 15 12:13 AUTHORS
```

🌞 Adapter la configuration d'Apache

```
[luc@ServeurWeb conf.d]$ cat nextcould.conf
<VirtualHost *:80>
  DocumentRoot /var/www/tp2_nextcloud/ # on indique le chemin de notre webroot
  ServerName  web.tp2.linux # on précise le nom que saisissent les clients pour accéder au service
  <Directory /var/www/tp2_nextcloud/> # on définit des règles d'accès sur notre webroot
    Require all granted
    AllowOverride All
    Options FollowSymLinks MultiViews
    <IfModule mod_dav.c>
      Dav off
    </IfModule>
  </Directory>
</VirtualHost>
```
🌞 Redémarrer le service Apache pour qu'il prenne en compte le nouveau fichier de conf

```
[luc@ServeurWeb conf.d]$ sudo systemctl status httpd
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
    Drop-In: /usr/lib/systemd/system/httpd.service.d
             └─php81-php-fpm.conf
     Active: active (running) since Tue 2022-11-15 12:36:11 CET; 51s ago
       Docs: man:httpd.service(8)
   Main PID: 37184 (httpd)
     Status: "Total requests: 0; Idle/Busy workers 100/0;Requests/sec: 0; Bytes served/sec:   0 B/sec"
      Tasks: 213 (limit: 5907)
     Memory: 22.8M
        CPU: 76ms
     CGroup: /system.slice/httpd.service
             ├─37184 /usr/sbin/httpd -DFOREGROUND
             ├─37185 /usr/sbin/httpd -DFOREGROUND
             ├─37186 /usr/sbin/httpd -DFOREGROUND
             ├─37187 /usr/sbin/httpd -DFOREGROUND
             └─37188 /usr/sbin/httpd -DFOREGROUND
```
### C. Finaliser l'installation de 

🌞 Exploration de la base de données

```
[luc@BasedeDonnes ~]$ sudo mysql -u root -p

USE nextcloud;

MariaDB [nextcloud]> SHOW TABLES;

124 rows in set (0.001 sec)
```
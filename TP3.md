# TP3 : Amélioration de la solution 
# Module 1 : Reverse Proxy 
## Setup
```
[luc@Proxy /]$ sudo dnf install nginx

[luc@Proxy /]$ sudo systemctl enable nginx

[luc@Proxy /]$ sudo firewall-cmd --add-port=80/tcp -- permanent

sudo cat /etc/nginx/conf.d/n.conf
server {
    server_name web.tp2.linux;
    listen 80;

    location / {
        proxy_set_header  Host $host;
        proxy_set_header  X-Real-IP $remote_addr;
        proxy_set_header  X-Forwarded-Proto https;
        proxy_set_header  X-Forwarded-Host $remote_addr;
        proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_pass http://10.102.1.11:80;
    }
    location /.well-known/carddav {
      return 301 $scheme://$host/remote.php/dav;
    }
    location /.well-known/caldav {
      return 301 $scheme://$host/remote.php/dav;
    }
}
```

```
[luc@web /]$ sudo cat /var/www/tp2_nextcloud/config/config.php
<?php
$CONFIG = array (
  'instanceid' => 'ocouu1crrs7j',
  'passwordsalt' => 'SbEMS0VNqf9zZ3yyl7AoOnM+2SgeDF',
  'secret' => '12BWKLCUcwXItRZOYlYqqCj3Ry2vqdD48iQzytOyo4wIZ5MC',
  'trusted_domains' =>
  array (
    0 => 'web.tp2.linux',
  ),
  'datadirectory' => '/var/www/tp2_nextcloud/data',
  'dbtype' => 'mysql',
  'version' => '25.0.0.15',
  'overwrite.cli.url' => 'http://web.tp2.linux',
  'dbname' => 'nextcloud',
  'dbhost' => '10.102.1.12:3306',
  'dbport' => '',
  'dbtableprefix' => 'oc_',
  'mysql.utf8mb4' => true,
  'dbuser' => 'nextcloud',
  'dbpassword' => 'pewpewpew',
  'installed' => true,
  'proxy' => '10.102.1.13',
  'overwriteprotocol' => 'https',
);
```
Bonus:
```
[luc@web ~]$ sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.102.1.1" service name=ssh accept'

[luc@web ~]$ sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.102.1.13" port port=80 protocol=tcp accept'

[luc@web ~]$ sudo firewall-cmd --permanent --add-rich-rule='rule protocol value=icmp reject'
```
## III. HTTPS

```
[luc@Proxy /]$ sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/nginx-selfsigned.key -out /etc/ssl/certs/nginx-selfsigned.crt

[luc@Proxy /]$ sudo openssl dhparam -out /etc/nginx/dhparam.pem 4096
```
```
[luc@Proxy /]$ sudo cat /etc/nginx/snippets/self-signed.conf
ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
```
```
[luc@Proxy /]$ sudo cat /etc/nginx/snippets/ssl-params.conf
ssl_protocols TLSv1.3;
ssl_prefer_server_ciphers on;
ssl_dhparam /etc/nginx/dhparam.pem;
ssl_ciphers EECDH+AESGCM:EDH+AESGCM;
ssl_ecdh_curve secp384r1;
ssl_session_timeout  10m;
ssl_session_cache shared:SSL:10m;
ssl_session_tickets off;
ssl_stapling on;
ssl_stapling_verify on;
resolver 8.8.8.8 8.8.4.4 valid=300s;
resolver_timeout 5s;
# Disable strict transport security for now. You can uncomment the following
# line if you understand the implications.
#add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;
add_header X-XSS-Protection "1; mode=block";
```
```
[luc@Proxy /]$ cat /etc/nginx/conf.d/prox.conf
server {
    server_name web.tp2.linux;
    listen 443 ssl;
    listen [::]:443 ssl;
    include snippets/self-signed.conf;
    include snippets/ssl-params.conf;

    location / {
        proxy_set_header  Host $host;
        proxy_set_header  X-Real-IP $remote_addr;
        proxy_set_header  X-Forwarded-Proto https;
        proxy_set_header  X-Forwarded-Host $remote_addr;
        proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_pass http://10.102.1.11:80;
    }

    location /.well-known/carddav {
      return 301 $scheme://$host/remote.php/dav;
    }

    location /.well-known/caldav {
      return 301 $scheme://$host/remote.php/dav;
    }
}

server {
     listen 80;
     server_name web.tp2.linux;
     return 301 https://$host$request_uri;
}
```
```
[luc@Proxy /]$ sudo firewall-cmd --add-port=443/tcp --permanent
[luc@Proxy /]$ sudo firewall-cmd --reload
```

# Module 5 : Monitoring

```
[luc@web /]$ wget -O /tmp/netdata-kickstart.sh https://my-netdata.io/kickstart.sh && sh /tmp/netdata-kickstart.sh
```

```
[luc@web /]$ sudo cat /etc/netdata/health.d/cpu_usage.conf
alarm: cpu_usage
on: system.cpu
lookup: average -3s percentage foreach user,system
units: %
every: 10s
warn: $this > 50
crit: $this > 80
info: CPU utilization of users on the system itself.
```
```
[luc@web /]$ sudo cat /etc/netdata/health_alarm_notify.conf
###############################################################################
# sending discord notifications

# note: multiple recipients can be given like this:
#                  "CHANNEL1 CHANNEL2 ..."

# enable/disable sending discord notifications
SEND_DISCORD="YES"

# Create a webhook by following the official documentation -
# https://support.discordapp.com/hc/en-us/articles/228383668-Intro-to-Webhooks
DISCORD_WEBHOOK_URL="https://discord.com/api/webhooks/1043123742206857276/HsvknKmZvO1D2QUZKY0GO-S0HXA0m1vFeWXqPJYdmvYcZXPvmtA_WE5-YWfhLQ8UKT9x"

# if a role's recipients are not configured, a notification will be send to
# this discord channel (empty = do not send a notification for unconfigured
# roles):
DEFAULT_RECIPIENT_DISCORD="alarms"

role_recipients_discord[sysadmin]="${DEFAULT_RECIPIENT_DISCORD}"
role_recipients_discord[dba]="${DEFAULT_RECIPIENT_DISCORD}"
role_recipients_discord[webmaster]="${DEFAULT_RECIPIENT_DISCORD}"
```

# Module 2 : Réplication de base de données

Master Machine:
```
[luc@Master /]$ cat /etc/my.cnf.d/mariadb-server.cnf

# Allow server to accept connections on all interfaces.
#
bind-address=0.0.0.0
#
[mariadb]

# This group is only read by MariaDB-10.5 servers.
# If you use the same .cnf file for MariaDB of different versions,
# use this group for options that older servers don't understand
log-bin
server_id=1
log-basename=master1
binlog-format=mixed
[mariadb-10.5]
```
```
MariaDB [(none)]> CREATE USER 'slave'@'10.102.1.14' identified by 'toto';
Query OK, 0 rows affected (0.003 sec)


MariaDB [(none)]> GRANT REPLICATION SLAVE ON *.* TO 'slave'@'10.102.1.14';
Query OK, 0 rows affected (0.003 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.000 sec)



MariaDB [(none)]> SHOW MASTER STATUS;
+--------------------+----------+--------------+------------------+
| File               | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+--------------------+----------+--------------+------------------+
| master1-bin.000001 |     1369 |              |                  |
+--------------------+----------+--------------+------------------+
1 row in set (0.000 sec)
```
Slave Machine:
```
cat /etc/my.cnf.d/mariadb-server.cnf

# Allow server to accept connections on all interfaces.
#
bind-address=0.0.0.0
#
[mariadb]

# This group is only read by MariaDB-10.5 servers.
# If you use the same .cnf file for MariaDB of different versions,
# use this group for options that older servers don't understand
log-bin
server_id=2
log-basename=slave1
binlog-format=mixed
[mariadb-10.5]
```
```
MariaDB [(none)]> STOP SLAVE;
Query OK, 0 rows affected, 1 warning (0.000 sec)


MariaDB [(none)]> CHANGE MASTER TO MASTER_HOST = '10.102.1.12', MASTER_USER = 'slave', MASTER_PASSWORD = 'toto', MASTER_LOG_FILE = 'master1-bin.000001', MAS
TER_LOG_POS = 1369;
Query OK, 0 rows affected (0.007 sec)


MariaDB [(none)]> START SLAVE;
Query OK, 0 rows affected (0.001 sec)
```

Ajout Nexcloud to Slave:

```
[luc@Master ~]$ sudo mysqldump -u root nextcloud > toto.sql

[luc@Master ~]$ scp toto.sql luc@10.102.1.14:/tmp/
```
```
[luc@Slave ~]$ sudo mv /tmp/toto.sql ./

[luc@Slave /]$ cat nano ./toto.sql
cat: nano: No such file or directory
CREATE DATABASE IF NOT EXISTS nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
use nextcloud;


-- MariaDB dump 10.19  Distrib 10.5.16-MariaDB, for Linux (x86_64)
--
-- Host: localhost    Database: nextcloud
-- ------------------------------------------------------
-- Server version       10.5.16-MariaDB-log


[luc@Slave /]$ sudo mysql -u root < toto.sql
```
```
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| nextcloud          |
| performance_schema |
+--------------------+
4 rows in set (0.000 sec)
```

# Module 7 : Fail2Ban

```
[luc@Proxy ~]$ sudo dnf install epel-release -y
Complete!

[luc@Proxy ~]$ sudo dnf install fail2ban -y
Complete!
```

```
[luc@Proxy jail.d]$ cat 00-firewalld.conf
# This file is part of the fail2ban-firewalld package to configure the use of
# the firewalld actions as the default actions.  You can remove this package
# (along with the empty fail2ban meta-package) if you do not use firewalld
[DEFAULT]
banaction = firewallcmd-rich-rules
banaction_allports = firewallcmd-rich-rules

[sshd]
enabled = true
maxretry = 3
findtime = 60
bantime = 1200
```

# Module 6 : Automatiser le déploiement

```
#!/bin/bash

dnf install httpd -y

dnf config-manager --set-enabled crb -y
dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm -y
dnf module list php -y
dnf module enable php:remi-8.1 -y
dnf install -y php81-phpdnf install -y libxml2 openssl php81-php php81-php-ctype php81-php-curl php81-php-gd php81-php-iconv php81-php-json php81-php-libxml php81-php-mbstring php81-php-openssl php81-php-posix php81-php-session php81-php-xml php81-php-zip php81-php-zlib php81-php-pdo php81-php-mysqlnd php81-php-intl php81-php-bcmath php81-php-gmp
mkdir /var/www

curl -o /nextcloud.zip https://download.nextcloud.com/server/prereleases/nextcloud-25.0.0rc3.zip

dnf install unzip -y

unzip /nextcloud.zip -d /var/www/
echo "<VirtualHost *:80>
  DocumentRoot /var/www/nextcloud/
  ServerName $HOSTNAME
  <Directory /var/www/nextcloud/>
    Require all granted
    AllowOverride All
    Options FollowSymLinks MultiViews
    <IfModule mod_dav.c>
      Dav off
    </IfModule>
  </Directory>
</VirtualHost>" > /etc/httpd/conf.d/nextcloud.conf
firewall-cmd --add-port=80/tcp --permanent
firewall-cmd --reload
systemctl restart httpd

```
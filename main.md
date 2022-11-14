# TP1 : (re)Familiaration avec un système GNU/Linux
## 0. Préparation de la machine

🌞 Setup de deux machines Rocky Linux configurées de façon basique.


Un accès internet (via la carte NAT)
```
[luc@node1 ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=113 time=24.3 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=113 time=29.3 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=113 time=23.1 ms
```
```
[luc@node2 ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=113 time=23.4 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=113 time=25.6 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=113 time=25.3 ms
```

Un accès à un réseau local (les deux machines peuvent se ping) (via la carte Host-Only)
```
[luc@node1 ~]$ ping 10.101.1.12
PING 10.101.1.12 (10.101.1.12) 56(84) bytes of data.
64 bytes from 10.101.1.12: icmp_seq=1 ttl=64 time=0.886 ms
64 bytes from 10.101.1.12: icmp_seq=2 ttl=64 time=0.951 ms
64 bytes from 10.101.1.12: icmp_seq=3 ttl=64 time=0.971 ms
```
```
[luc@node2 ~]$ ping 10.101.1.11
PING 10.101.1.11 (10.101.1.11) 56(84) bytes of data.
64 bytes from 10.101.1.11: icmp_seq=1 ttl=64 time=0.601 ms
64 bytes from 10.101.1.11: icmp_seq=2 ttl=64 time=0.745 ms
64 bytes from 10.101.1.11: icmp_seq=3 ttl=64 time=0.818 ms
```
Les machines doivent avoir un nom
Commande: `hostnamectl set-hostname`

Utiliser 1.1.1.1 comme serveur DNS
```
[luc@node1 ~]$ dig ynov.com

; <<>> DiG 9.16.23-RH <<>> ynov.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 19051
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;ynov.com.                      IN      A

;; ANSWER SECTION:
ynov.com.               300     IN      A       104.26.11.233
ynov.com.               300     IN      A       172.67.74.226
ynov.com.               300     IN      A       104.26.10.233

;; Query time: 30 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Mon Nov 14 12:03:16 CET 2022
;; MSG SIZE  rcvd: 85

```
```
[luc@node2 ~]$ dig ynov.com

; <<>> DiG 9.16.23-RH <<>> ynov.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 17747
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;ynov.com.                      IN      A

;; ANSWER SECTION:
ynov.com.               300     IN      A       104.26.10.233
ynov.com.               300     IN      A       104.26.11.233
ynov.com.               300     IN      A       172.67.74.226

;; Query time: 31 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Mon Nov 14 12:05:08 CET 2022
;; MSG SIZE  rcvd: 85

```

Les machines doivent pouvoir se joindre par leurs noms respectifs
```
[luc@node1 ~]$ ping node2.tp1.b2
PING node2.tp1.b2 (10.101.1.12) 56(84) bytes of data.
64 bytes from node2.tp1.b2 (10.101.1.12): icmp_seq=1 ttl=64 time=0.491 ms
64 bytes from node2.tp1.b2 (10.101.1.12): icmp_seq=2 ttl=64 time=1.10 ms
64 bytes from node2.tp1.b2 (10.101.1.12): icmp_seq=3 ttl=64 time=0.849 ms
```
```
[luc@node2 ~]$ ping node1.tp1.b2
PING node1.tp1.b2 (10.101.1.11) 56(84) bytes of data.
64 bytes from node1.tp1.b2 (10.101.1.11): icmp_seq=1 ttl=64 time=0.290 ms
64 bytes from node1.tp1.b2 (10.101.1.11): icmp_seq=2 ttl=64 time=0.746 ms
64 bytes from node1.tp1.b2 (10.101.1.11): icmp_seq=3 ttl=64 time=0.255 ms
```
Le pare-feu est configuré pour bloquer toutes les connexions exceptées celles qui sont nécessaires
```
[luc@node1 ~]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: cockpit dhcpv6-client ssh
  ports:
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```
```
[luc@node2 ~]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: cockpit dhcpv6-client ssh
  ports:
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

## I. Utilisateurs
### 1. Création et configuration

🌞 Ajouter un utilisateur à la machine, qui sera dédié à son administration
```
[luc@node1 ~]$ sudo useradd Admin -m -s /bin/bash


[luc@node1 ~]$ cat /etc/passwd
Admin:x:1001:1001::/home/Admin:/bin/bash
```
```
[luc@node2 ~]$ sudo useradd Admin -m -s /bin/bash


[luc@node1 ~]$ cat /etc/passwd
Admin:x:1001:1001::/home/Admin:/bin/bash
```
🌞 Créer un nouveau groupe admins qui contiendra les utilisateurs de la machine ayant accès aux droits de root via la commande sudo.

```
[luc@node1 ~]$ sudo visudo /etc/sudoers

%admins  ALL=(ALL)       ALL
```
```
[luc@node2 ~]$ sudo visudo /etc/sudoers

%admins  ALL=(ALL)       ALL
```
🌞 Ajouter votre utilisateur à ce groupe admins

Ajoutez un utilisateur a un groupe: `sudo usermod -g admins Admin`
```
[Admin@node1 ~]$ sudo ls

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for Admin:
```
```
[Admin@node2 ~]$ sudo ls

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for Admin:
```
### 2. SSH
```
PS C:\Users\garra> ssh luc@10.101.1.11
Last login: Mon Nov 14 12:42:38 2022
[luc@node1 ~]$
```
```
PS C:\Users\garra> ssh luc@10.101.1.12
Last login: Mon Nov 14 11:54:52 2022 from 10.101.1.1
[luc@node2 ~]$
```

## II. Partitionnement
### 1. Préparation de la VM
### 2. Partitionnement

🌞 Utilisez LVM pour...

-agréger les deux disques en un seul volume group
```
[luc@node1 ~]$ sudo pvcreate /dev/sdb
  Physical volume "/dev/sdc" successfully created.
  
  
[luc@node1 ~]$ sudo pvcreate /dev/sdc
  Physical volume "/dev/sdc" successfully created
```
-créer 3 logical volumes de 1 Go chacun
```
[luc@node1 ~]$ sudo vgcreate data /dev/sdb
  Volume group "data" successfully created
  

[luc@node1 ~]$ sudo vgextend data /dev/sdc
  Volume group "data" successfully extended
```
```
[luc@node1 ~]$ sudo lvcreate -L 1G data -n a
  Logical volume "a" created.
[luc@node1 ~]$ sudo lvcreate -L 1G data -n b
  Logical volume "b" created.
[luc@node1 ~]$ sudo lvcreate -L 1G data -n c
  Logical volume "c" created.
```
-formater ces partitions en ext4
```
[luc@node1 ~]$ sudo mkfs -t ext4 /dev/data/a
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 262144 4k blocks and 65536 inodes
Filesystem UUID: 918166c7-f3fb-4f37-930b-600f90433f8f
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376

Allocating group tables: done
Writing inode tables: done
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done

[luc@node1 ~]$ sudo mkfs -t ext4 /dev/data/b
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 262144 4k blocks and 65536 inodes
Filesystem UUID: fa454f21-448d-4e66-b3de-0b257fdb3c99
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376

Allocating group tables: done
Writing inode tables: done
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done

[luc@node1 ~]$ sudo mkfs -t ext4 /dev/data/c
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 262144 4k blocks and 65536 inodes
Filesystem UUID: c48c025b-2d58-4ef0-96f4-4528f950c566
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376

Allocating group tables: done
Writing inode tables: done
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done
```
-monter ces partitions pour qu'elles soient accessibles aux points de montage /mnt/part1, /mnt/part2 et /mnt/part3.
```
[luc@node1 ~]$ sudo mount /dev/data/a /mnt/part1
[luc@node1 ~]$ sudo mount /dev/data/b /mnt/part2
[luc@node1 ~]$ sudo mount /dev/data/c /mnt/part3
```
🌞 Grâce au fichier /etc/fstab, faites en sorte que cette partition soit montée automatiquement au démarrage du système.

```
/dev/data/a /mnt/part1 ext4 defaults 0 0
/dev/data/b /mnt/part2 ext4 defaults 0 0
/dev/data/c /mnt/part3 ext4 defaults 0 0
```
```
[luc@node1 ~]$ sudo umount /mnt/part1
[luc@node1 ~]$  sudo mount -av
/                        : ignored
/boot                    : already mounted
none                     : ignored
mount: /mnt/part1 does not contain SELinux labels.
       You just mounted a file system that supports labels which does not
       contain labels, onto an SELinux box. It is likely that confined
       applications will generate AVC messages and not be allowed access to
       this file system.  For more details see restorecon(8) and mount(8).
/mnt/part1               : successfully mounted
/mnt/part2               : already mounted
/mnt/part3               : already mounted
```

## III. Gestion de services
### 1. Interaction avec un service existant

🌞 Assurez-vous que...
```
[luc@node1 ~]$ systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
     Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor pre>
     Active: active (running) since Mon 2022-11-14 14:54:39 CET; 27min ago
       Docs: man:firewalld(1)
   Main PID: 652 (firewalld)
      Tasks: 2 (limit: 5907)
     Memory: 42.8M
        CPU: 1.624s
     CGroup: /system.slice/firewalld.service
             └─652 /usr/bin/python3 -s /usr/sbin/firewalld --nofork --nopid

```

### 2. Création de service
#### A. Unité simpliste

🌞 Créer un fichier qui définit une unité de service

```
[luc@node1 ~]$ cat /etc/systemd/system/web.service
[Unit]
Description=Very simple web service

[Service]
ExecStart=/usr/bin/python3 -m http.server 8888

[Install]
WantedBy=multi-user.target
```
```
[luc@node1 ~]$ sudo firewall-cmd --add-port=8888/tcp --permanent

[luc@node1 ~]$ sudo firewall-cmd --reload
```
```
[luc@node1 ~]$ sudo systemctl status web
● web.service - Very simple web service
     Loaded: loaded (/etc/systemd/system/web.service; enabled; vendor preset: disab>
     Active: active (running) since Mon 2022-11-14 15:33:01 CET; 11s ago
   Main PID: 1151 (python3)
      Tasks: 1 (limit: 5907)
     Memory: 9.2M
        CPU: 73ms
     CGroup: /system.slice/web.service
             └─1151 /usr/bin/python3 -m http.server 8888

Nov 14 15:33:01 node1.tp1.b2 systemd[1]: Started Very simple web service.
```

🌞 Une fois le service démarré, assurez-vous que pouvez accéder au serveur web

```
[luc@node2 ~]$ curl 10.101.1.11:8888
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>Directory listing for /</title>
</head>
<body>
<h1>Directory listing for /</h1>
<hr>
<ul>
<li><a href="afs/">afs/</a></li>
<li><a href="bin/">bin@</a></li>
<li><a href="boot/">boot/</a></li>
<li><a href="dev/">dev/</a></li>
<li><a href="etc/">etc/</a></li>
<li><a href="home/">home/</a></li>
<li><a href="lib/">lib@</a></li>
<li><a href="lib64/">lib64@</a></li>
<li><a href="media/">media/</a></li>
<li><a href="mnt/">mnt/</a></li>
<li><a href="opt/">opt/</a></li>
<li><a href="proc/">proc/</a></li>
<li><a href="root/">root/</a></li>
<li><a href="run/">run/</a></li>
<li><a href="sbin/">sbin@</a></li>
<li><a href="srv/">srv/</a></li>
<li><a href="sys/">sys/</a></li>
<li><a href="tmp/">tmp/</a></li>
<li><a href="usr/">usr/</a></li>
<li><a href="var/">var/</a></li>
</ul>
<hr>
</body>
</html>
```
#### B. Modification de l'unité

🌞 Préparez l'environnement pour exécuter le mini serveur web Python

```
[luc@node1 www]$ ls -al
total 4
drwxr-xr-x.  3 root root   18 Nov 14 15:40 .
drwxr-xr-x. 20 root root 4096 Nov 14 15:40 ..
drwxr-xr-x.  2 web  root   19 Nov 14 15:41 meow
```

🌞 Modifiez l'unité de service web.service créée précédemment en ajoutant les clauses

```
[luc@node1 ~]$ sudo cat /etc/systemd/system/web.service
[Unit]
Description=Very simple web service

[Service]
ExecStart=/usr/bin/python3 -m http.server 8888
User=web
WorkingDirectory=/var/www/meow

[Install]
WantedBy=multi-user.target
```

🌞 Vérifiez le bon fonctionnement avec une commande curl

```
[luc@node2 ~]$ curl 10.101.1.11:8888
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>Directory listing for /</title>
</head>
<body>
<h1>Directory listing for /</h1>
<hr>
<ul>
<li><a href="testE">testE</a></li>
</ul>
<hr>
</body>
</html>
```
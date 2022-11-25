# TP4 : Conteneurs
## 1. Install

🌞 Installer Docker sur la machine

```
[luc@docker1 ~]$ sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
[sudo] password for luc:
Adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
```
```
[luc@docker1 ~]$ sudo dnf install docker-ce docker-ce-cli containerd.io
Last metadata expiration check: 0:00:07 ago on Thu 24 Nov 2022 11:04:28 AM CET.
Dependencies resolved.

[...]

Complete!
```
```
[luc@docker1 ~]$ sudo systemctl enable docker
Created symlink /etc/systemd/system/multi-user.target.wants/docker.service → /usr/lib/systemd/system/docker.service.

[luc@docker1 ~]$ sudo systemctl start docker
```
```
[luc@docker1 ~]$ sudo systemctl status docker
● docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
     Active: active (running) since Thu 2022-11-24 11:09:32 CET; 27s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 2823 (dockerd)
      Tasks: 7
     Memory: 23.8M
        CPU: 87ms
     CGroup: /system.slice/docker.service
             └─2823 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```
```
[luc@docker1 ~]$ sudo usermod -aG docker luc
```
## 2. Vérifier l'install
---
## 3. Lancement de conteneurs

🌞 Utiliser la commande docker run

```
[luc@docker1 ~]$ docker run --name web -d -m 6m --cpus="0.5" -v /home/luc/index.html:/index.html -v /home/luc/NGINIX.conf:/conf -p 8888:80 nginx
```
## II. Images
🌞 Construire votre propre images

---
## III. docker-compose
🌞 Conteneurisez votre application

```
[luc@docker1 ~]$ docker build . -t scan
[luc@docker1 ~]$ docker compose up
```
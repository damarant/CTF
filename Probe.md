
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/probe

https://tryhackme.com/room/geolocatingimages

https://tryhackme.com/room/kiba
**memo nikto**

```
nmap -Pn -v -O -p- 10.10.231.59
```
```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
443/tcp  open  https
1338/tcp open  wmc-log-svc
1443/tcp open  ies-lm
1883/tcp open  mqtt
8000/tcp open  http-alt
9007/tcp open  ogs-client

```

```
nmap -sVC -p22,80,443,1443,1883,8000,9007 10.10.231.59
```
```
PORT     STATE SERVICE  VERSION
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http     lighttpd 1.4.55
|_http-server-header: lighttpd/1.4.55
|_http-title: 403 Forbidden
443/tcp  open  ssl/http Apache httpd 2.4.41
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: 403 Forbidden
| ssl-cert: Subject: commonName=dev.probe.thm/organizationName=Tester/stateOrProvinceName=Some-State/countryName=US
| Not valid before: 2023-07-18T10:57:05
|_Not valid after:  2024-07-17T10:57:05
| tls-alpn: 
|_  http/1.1
1443/tcp open  ssl/http Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: PHP 7.4.3-4ubuntu2.19 - phpinfo()
| ssl-cert: Subject: commonName=dev.probe.thm/organizationName=Tester/stateOrProvinceName=Some-State/countryName=US
| Not valid before: 2023-07-18T10:57:05
|_Not valid after:  2024-07-17T10:57:05
| tls-alpn: 
|_  http/1.1
1883/tcp open  mqtt
|_mqtt-subscribe: ERROR: Script execution failed (use -d to debug)
8000/tcp open  http     Apache httpd 2.4.41 ((Ubuntu))
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
9007/tcp open  ssl/http Apache httpd 2.4.41 ((Ubuntu))
|_http-generator: WordPress 6.2.2
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Welcome to my Blog &#8211; I am going to be the best blogger
| ssl-cert: Subject: commonName=myblog.thm/organizationName=Internet Widgits Pty Ltd/stateOrProvinceName=Some-State/countryName=US
| Not valid before: 2023-07-08T11:01:57
|_Not valid after:  2024-07-07T11:01:57
| tls-alpn: 
|_  http/1.1

```

Qual è la versione del server Apache?
```
2.4.41
```
Qual è il numero di porta del servizio FTP?
enumeriamo di nuovo
```
nmap -A -p- 10.10.231.59 -T5 -Pn
```
ci era sfuggita una porta
```
1338/tcp  open     ftp      vsftpd 2.0.8 or later
```
```
1338
```
Qual è l'FQDN del sito web ospitato tramite un certificato autofirmato e che contiene informazioni critiche sul server come la home page?
Cercate l'output cname o CommonName nella scansione fatta con nmap
```
1443/tcp open  ssl/http Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: PHP 7.4.3-4ubuntu2.19 - phpinfo()
| ssl-cert: Subject: commonName=dev.probe.thm/organizationName=Tester/stateOrProvinceName=Some-State/countryName=US
```

```
dev.probe.thm
```

Qual è l'indirizzo email associato al certificato SSL utilizzato per firmare il sito web menzionato nella domanda 3?

ci colleghiamo in https a 
```
http://10.10.231.59:443
```
visualizziamo il certificato e troviamo
```
probe@probe.thm
```

Qual è il valore della  **PHP Extension Build** sul server?
dopo qualche prova ci colleghiamo a
```
https://10.10.231.59:1443/
```

```
API20190902,NTS
```

Cos'è il banner per il servizio FTP?
```
nc 10.10.231.59 1338
```

```
THM{WELCOME_101113}
```

Quale software viene utilizzato per gestire il database sul server?
```
gobuster dir -u http://10.10.231.59:8000 -w /usr/share/wordlists/dirb/big.txt
```
```
/phpmyadmin           (Status: 301) [Size: 324] [-
```
```
phpmyadmin
```
Che tipo di Content Management System (CMS) è ospitato sul server?
lo vediamo da nmap
```
WordPress
```
Qual è il numero di versione del CMS ospitato sul server?
```
6.2.2
```
Qual è il nome utente per il pannello di amministrazione del CMS?
enumeriamo utenti con wpscan
```
wpscan --url https://10.10.231.59:9007 --disable-tls-checks --enumerate u
```

```
Fingerprinting the version - Time: 00:00:01 <> (705 / 705) 100.00% Time: 00:00:01
[+] WordPress version 5.7 identified (Latest, released on 2021-03-09).
 | Found By: Unique Fingerprinting (Aggressive Detection)
 |  - https://10.10.231.59:9007/wp-includes/css/buttons.min.css md5sum is 61acbb6ebdd2479dcb66e467e3f1d80f

[+] WordPress theme in use: twentytwentythree
 | Location: https://10.10.231.59:9007/wp-content/themes/twentytwentythree/
 | Readme: https://10.10.231.59:9007/wp-content/themes/twentytwentythree/readme.txt
 | [!] Directory listing is enabled
 | Style URL: https://10.10.231.59:9007/wp-content/themes/twentytwentythree/style.css
 | Style Name: Twenty Twenty-Three
 | Style URI: https://wordpress.org/themes/twentytwentythree
 | Description: Twenty Twenty-Three is designed to take advantage of the new design tools introduced in WordPress 6....
 | Author: the WordPress team
 | Author URI: https://wordpress.org
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 1.1 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - https://10.10.231.59:9007/wp-content/themes/twentytwentythree/style.css, Match: 'Version: 1.1'

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:01 <==> (10 / 10) 100.00% Time: 00:00:01

[i] User(s) Identified:

[+] joomla
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register
```

```
joomla
```

Durante la scansione delle vulnerabilità,  **OSVDB-3092**  rileva un file che potrebbe essere utilizzato per identificare il software del sito di blogging. Qual è il nome del file?

Nikto usa la scansione OSVDB-3092 per impostazione predefinita.

```
 nikto -h https://10.10.231.59:9007 
```

 cerchiamo la riga di informazioni contenente OSVDB-3092.

```
 OSVDB-3092: /license.txt
```

```
license.txt
```

Qual è il nome del software utilizzato sulla porta HTTP standard?
da nmap
```
http-server-header: lighttpd/1.4.55
```

```
lighttpd
```

Qual è il valore del flag associato alla pagina web ospitata sulla porta 8000?

la pagina è vuota enumeriamo in cerca di altre dir
```
gobuster dir -u http://10.10.231.59:8000 -w /usr/share/wordlists/dirb/big.txt
```

```
/contactus
```
ci connettiamo 
```
http://10.10.231.59:8000/contactus/
```

```
THM{xxxxxxxxxxx}
```


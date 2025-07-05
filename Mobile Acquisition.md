
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/mobileacquisition

I dispositivi mobili sono diventati una miniera d'oro per gli investigatori forensi digitali. Vengono utilizzati quotidianamente e possono rivelare una grande quantità di dati personali, comportamentali e di comunicazione.

In questa sala, scoprirete come i dispositivi mobili possano contenere prove cruciali in un'indagine. Imparerete come questi dispositivi si inseriscono nel più ampio processo di analisi forense digitale, quali sono le sfide comuni che rendono difficili queste indagini e perché i gruppi di minaccia (come gli APT) si stanno concentrando sempre di più su di essi.

Esploreremo anche come i produttori di dispositivi mobili proteggono i loro dispositivi, le discussioni legali in corso in ambito più ampio e un'ampia gamma di tecniche di acquisizione dati. Infine, ti unirai a TryAnalyseMe per eseguire un'acquisizione su un dispositivo mobile.

## Obiettivi di apprendimento

- Scopri come i dispositivi mobili possono svolgere un ruolo fondamentale nelle indagini forensi
- Scopri di più sulle discussioni legali e politiche più ampie riguardanti la sicurezza e la privacy dei dispositivi mobili
- Come i produttori proteggono i loro dispositivi dall'acquisizione dati
- Scopri come gli APT stanno cambiando tattica per colpire i dispositivi mobili
- Acquisire familiarità con le tecniche di acquisizione specifiche per dispositivi mobili.

el mondo odierno, i dispositivi mobili rappresentano una tecnologia sempre più potente e ampiamente adottata in tutto il mondo. Questi dispositivi mobili, come gli smartphone, contengono alcuni dei nostri dati e conversazioni più personali, profondamente integrati nella nostra vita quotidiana. Rappresentano una vera e propria miniera di potenziali prove per un analista.

Poiché i dispositivi mobili hanno la potenza di calcolo necessaria per sostituire computer fissi e portatili, contengono un'ampia gamma di dati preziosi, tra cui:

- Registri delle chiamate e delle chat
- GPS e dati di navigazione
- Documenti e download
- Immagini e video
- Cronologia di navigazione
- Cronologia del WiFi
- Dati specifici dell'app

## Panorama dei dispositivi mobili

Quando parliamo di dispositivi mobili, intendiamo tutto, dagli smartphone alla tecnologia indossabile come gli smartwatch. Sebbene questa sala si concentrerà sugli smartphone, i metodi di acquisizione e le varie protezioni trattate si estendono anche a questi.

È un dibattito vecchio come il mondo: Android contro iPhone. Infatti, al momento in cui scrivo, Android detiene una quota di mercato del 72% [ [StatCounter](https://gs.statcounter.com/os-market-share/mobile/worldwide) ], grazie a diversi produttori come Samsung, Google, Motorola e Xiaomi, che distribuiscono i loro prodotti con Android installato grazie alle sue possibilità di personalizzazione. Approfondiremo più avanti le specifiche di questi due sistemi operativi, imparando strumenti e tecniche specifiche per entrambi.

## Analisi forense dei dispositivi mobili nel mondo reale

In tutto il mondo, i dispositivi mobili hanno svolto un ruolo fondamentale nelle indagini penali, civili e di risposta agli incidenti. Ciò ha innescato un acceso dibattito tra privacy e poteri investigativi – ne parleremo più avanti. Forse ricorderete le prolifiche indagini di polizia riportate nei notiziari o le aule di tribunale accese, dove i dispositivi mobili sono stati la chiave per risolvere il caso.

Ad esempio, in un famoso caso del 2013 in Sudafrica, dati sanitari come il conteggio dei passi sono stati utilizzati per screditare un'argomentazione di difesa, dimostrando che la persona era effettivamente attiva al momento in cui affermava di non esserlo.

Nel settore privato, gli smartphone sono una tecnologia in continua crescita utilizzata nelle aziende. Dalle operazioni alla logistica, fino alle attività in mobilità (che spesso contengono dati critici per l'azienda). Di conseguenza, i gruppi di cybercriminali stanno prendendo piede e stanno concentrando le loro attività su questi dispositivi, spesso utilizzandoli come punti di accesso all'ambiente aziendale o per la fuga di dati.

## Punto di ingresso

I dispositivi mobili sono un ottimo metodo di accesso iniziale per un aggressore. Quando vai da qualche parte, cosa porti con te? Il portafoglio? Le chiavi? Il telefono, magari?

I telefoni si connettono costantemente a reti diverse. Se hai un telefono aziendale, potresti connetterlo al Wi-Fi di casa. Potresti connettere il tuo telefono al bar o a casa di un amico. Questo offre un'eccellente opportunità per un aggressore di valutare e raccogliere informazioni sui dispositivi su diverse reti. Magari lasci il tuo portatile aziendale al lavoro, ma porti il ​​telefono aziendale a casa; improvvisamente, si crea un punto di accesso alla rete aziendale.

Gli stessi artefatti che sono utili a un analista si estendono anche a un aggressore: creazione di una cronologia del comportamento, identificazione dei collaboratori, ecc. Ora, questa sezione non ha lo scopo di spaventarvi, ma di evidenziare come questi dispositivi siano risorse preziose per un aggressore.

Sebbene i meccanismi di sicurezza dei dispositivi mobili siano sofisticati, non sono infallibili. Ne parleremo a breve.

In quale Paese si è verificato un famoso esempio di utilizzo di dispositivi mobili nelle indagini?
```
South Africa
```

Qual è il termine tecnico per un dispositivo che è diventato il metodo di accesso iniziale di un aggressore?
```
Entrypoint
```

**Sfide con l'analisi forense dei dispositivi mobili**

I dispositivi mobili moderni vantano importanti meccanismi di protezione della sicurezza.  Approfondiremo questi aspetti nel resto del modulo, ma ora ve ne presentiamo alcuni.

Funzionalità come la crittografia completa del disco e la crittografia basata sui file sono diventate uno standard sui dispositivi moderni, richiedendo una qualche forma di autenticazione (PIN/FaceID/impronta digitale, ecc.) per lo sblocco, e i dati possono avere diversi stati in cui sono o non sono leggibili. Ad esempio:

- Prima del primo sblocco (BFU)
- Richiedere l'autenticazione ogni volta
- È necessario autenticarsi solo una volta
- Nessuna autenticazione richiesta

Sebbene la crittografia di dischi e file non sia un ostacolo nuovo nell'informatica forense, presenta notevoli difficoltà di acquisizione . I produttori di dispositivi mobili hanno fatto grandi sforzi per proteggere i dati degli utenti e i loro dispositivi. Elenchiamo alcune delle protezioni comuni che Android e iOS condividono ad alto livello .

|   |   |
|---|---|
|**Meccanismo**|**Spiegazione**|
|Crittografia completa del disco e basata sui file|A meno che non vengano autenticati o aggirati, gli strumenti forensi non possono analizzare i dati memorizzati nel dispositivo. I singoli file possono essere crittografati con chiavi diverse.|
|Chiavi di crittografia isolate|Sia Android che iOS utilizzano un componente hardware dedicato per memorizzare le chiavi di crittografia, incredibilmente difficile da recuperare. È simile al modulo TPM delle schede madri.|
|Processo di avvio sicuro|Garantisce che solo il codice affidabile e verificato del produttore possa essere caricato, prevenendo le manomissioni. Una vecchia tecnica di indagine utilizzava un bootloader personalizzato che aggirava un'ampia gamma di meccanismi di sicurezza.|
|Sandbox|Le applicazioni vengono eseguite in ambienti separati, isolate l'una dall'altra. Approfondiremo questo argomento nel corso del modulo.|
|Pulizia del blocco|Questi dispositivi possono essere configurati per cancellarsi automaticamente dopo un certo numero di tentativi di autenticazione non riusciti (ad esempio, inserimento di PIN non riuscito). Se abilitata, questa opzione impedisce attacchi brute-force.|
|Cancellazione remota|Utilizzando funzioni come "Trova il mio", è possibile cancellare da remoto i dati di un dispositivo rispetto a un altro in caso di furto, ecc.|

Gli smartphone moderni integrano sofisticati meccanismi di protezione che rendono difficile l'acquisizione e l'analisi. I produttori stanno migliorando costantemente questi meccanismi, mentre il gioco del gatto e del topo continua. Esamineremo come l'analisi sia possibile tenendo conto di queste (e altre) protezioni.


## Dibattiti legali

Numerose sfide legali e politiche hanno circondato il processo di indagine sui dispositivi mobili, soprattutto negli ultimi anni. Ad esempio, per quanto riguarda la crittografia, stiamo assistendo a dibattiti sulla tutela della privacy individuale e sull'equilibrio tra sicurezza pubblica e forze dell'ordine.

In tempi molto recenti, al momento in cui scrivo, Apple ha ritirato la sua funzionalità avanzata di protezione dei dati nel Regno Unito dopo che il governo britannico ha intentato una causa legale con Apple per ottenere l'accesso alle forze dell'ordine per aggirare tali meccanismi di protezione. Invece di ottemperare, Apple l'ha rimossa completamente, rimuovendo la crittografia end-to-end per i dati di iCloud per tutti gli utenti del Regno Unito.

Abbiamo assistito, e continueremo a assistere, a eventi come questo in tutto il mondo, con altri Paesi che hanno avanzato richieste legali a fornitori e produttori di tecnologia affinché forniscano l'accesso a dati crittografati su richiesta. Alcuni di questi Paesi hanno ceduto a proposte di legge controverse, mettendo a rischio gli utenti nell'interesse della sicurezza pubblica. È un dibattito dinamico con forti argomentazioni da entrambe le parti.

Gli analisti devono possedere competenze tecniche in materia di informatica forense per dispositivi mobili e avere familiarità con le leggi sulla protezione dei dati e con i processi legislativi dinamici quando analizzano dispositivi mobili, in particolare oltre confine, e devono affrontare controlli sempre più rigorosi.

## Rilevamento del malware
Rilevare comportamenti dannosi sui dispositivi mobili è particolarmente difficile. Come accennato in precedenza, le applicazioni dannose si mascherano da utility legittime ed eseguono trucchetti dietro le quinte. Non è possibile, ad esempio, installare il proprio disassembler preferito sul proprio iPhone per analizzare un'app.

Se a ciò si aggiunge il fatto che gli agenti di monitoraggio (come gli agenti EDR ) stanno ancora cercando di raggiungere i dispositivi mobili, in genere questi dispositivi sono meno monitorati su una rete aziendale rispetto a una postazione di lavoro aziendale.

Questi dispositivi sono progettati per essere il più intuitivi possibile. A causa dei termini di marketing sulla sicurezza che hanno sentito dai produttori, gli utenti spesso hanno una "protezione" inferiore quando cliccano su link, ecc.

Gli smartphone più vecchi potrebbero non disporre di meccanismi di sicurezza altrettanto sofisticati oppure, sfruttando gli 0day, potresti imbatterti in un malware in grado di effettuare il rootkit del dispositivo, camuffandosi completamente.


Quale protezione del produttore impedisce il caricamento di codice non attendibile durante l'avvio?

```
Secure boot process
```

Le chiavi di crittografia sono archiviate nel software o nell'hardware?

```
hardware
```

Gli APT incontrano i dispositivi mobili

I dispositivi mobili sono un bersaglio allettante per un aggressore motivato. Questo articolo illustra come gli aggressori cambiano marcia e si concentrano su questi obiettivi succulenti.

## Malware dell'App Store

Forse ricorderete casi di app dannose che si spacciavano per utility legittime (come gli editor di foto) e che apparivano su app store come Google Play. Sebbene le applicazioni vengano controllate tramite scanner automatici, come dimostrato, proprio come qualsiasi altra cosa nella sicurezza informatica, queste non sono affidabili al 100%. 

Sebbene nella maggior parte dei casi queste app dannose vengano utilizzate per raccogliere informazioni come contatti, registri delle chiamate e contenuto degli appunti, possono anche avere funzionalità molto più nefaste, come la sovrapposizione ad altre app (come il browser) per rubare le credenziali di accesso e agire come spyware. La motivazione principale di applicazioni dannose come queste è raccogliere dati e credenziali su larga scala, solitamente per rivenderli su famigerati mercati clandestini.

Un esempio famoso è un'app di fotoritocco che caricava le foto degli utenti sui sistemi dello sviluppatore senza alcun preavviso o autorizzazione e iniettava adware per generare entrate. Sebbene queste app non siano solitamente mirate, rafforzano l'idea che nulla sia mai sicuro al 100%. Di seguito, analizzeremo alcuni esempi di aggressori motivati ​​che utilizzano applicazioni mirate nei loro attacchi.

## Spyware e sorveglianza

La scoperta di Pegasus, un malware estremamente sofisticato progettato per scopi di sorveglianza, che spesso ottiene l'accesso tramite "interazione zero", è stata in grado di fare cose come:

- Leggere le email, accedere alle foto
- Leggere i messaggi
- Tracciamento tramite GPS
- Registrazione delle chiamate telefoniche, microfono e telecamera senza che l'utente se ne accorga
- Acquisizione delle credenziali
- Avendo pochissime tracce di presenza

Pegasus ha utilizzato una combinazione di attacchi "one click" o "zero click", noti come catena di exploit " [BLASTPASS](https://citizenlab.ca/2023/09/blastpass-nso-group-iphone-zero-click-zero-day-exploit-captured-in-the-wild/) ", che richiedevano all'utente solo di cliccare su un URL (o addirittura di non cliccare affatto! Sfruttando le vulnerabilità nelle applicazioni di sistema come Messaggi, WhatsApp, ecc.) per infiltrarsi nel dispositivo. 

Se desideri saperne di più su questo sofisticato malware, Citizen Lab ha pubblicato numerosi articoli di ricerca che puoi trovare [qui](https://citizenlab.ca/tag/pegasus/) .

Per tranquillizzarvi, strumenti sofisticati come Pegasus non vengono utilizzati con leggerezza. Casi come questo non fanno che alimentare il dibattito tra tecniche etiche e legali di applicazione della legge e la privacy degli utenti.

Altri malware degni di nota utilizzati dagli aggressori includono app trojan bancari come Anubis, Cerberus ed Exodus.

In quale app store sono state trovate applicazioni dannose a disposizione degli utenti?

```
Google Play
```

Come si chiama il malware sofisticato che utilizzava una combinazione di attacchi "one click" e "zero click"?

```
Pegasus
```

## Tecniche di acquisizione

'acquisizione viene eseguita nelle primissime fasi del processo di analisi forense dei dispositivi mobili. L'acquisizione è il processo di raccolta di dati da un dispositivo mobile utilizzando una varietà di tecniche in modo forense. Questo capitolo esplorerà le quattro principali tecniche e strumenti di acquisizione e le modalità di conservazione delle prove e dell'accesso.

## Livelli di acquisizione

Quando parliamo di livelli di acquisizione, ci riferiamo alla profondità e al metodo necessari per estrarre i dati da un dispositivo. Le tecniche e i metodi di acquisizione utilizzati sono determinati da una combinazione di diversi fattori, tra cui quelli elencati di seguito:

- Età del dispositivo (ad esempio versione del sistema operativo installato , aggiornamenti, ecc.)
- Meccanismi di sicurezza in atto
- Accesso autenticato o non autenticato (ovvero bloccato o sbloccato)
- Disponibilità di strumenti per l'esaminatore
- Profondità dei dati che desideriamo recuperare (ad esempio dati eliminati)

Nell'ambito dell'analisi forense sui dispositivi mobili, generalmente si distinguono quattro livelli di acquisizione, riportati nella tabella sottostante.

|   |   |   |   |
|---|---|---|---|
|**Metodo di acquisizione**|**Descrizione**|**Caso d'uso**|**Livello di accesso**|
|Manuale|L'acquisizione manuale comporta l'interazione manuale con il dispositivo per raccogliere informazioni, ad esempio scorrendo i messaggi di chat o scattando foto con un dispositivo alternativo.|Questo è incredibilmente prezioso se il dispositivo è attualmente sbloccato, poiché molti meccanismi di sicurezza sono già stati aggirati.  <br>  <br>Tuttavia, ciò solleva seri problemi di integrità dei dati e di non ripudio.|Accesso minimo ai registri di sistema e ai database.|
|Logico|L'acquisizione logica comporta l'utilizzo delle funzionalità del sistema operativo dei dispositivi mobili (come API, funzionalità di backup, ecc.) per estrarre i dati.|Questo metodo è utile nei casi in cui il dispositivo è bloccato, poiché consente l'autenticazione tramite un altro dispositivo considerato attendibile (dal dispositivo mobile).|Parziale.|
|Sistema di file|L'acquisizione del file system comporta la creazione di una copia completa del file system del dispositivo.|Di solito è necessario sfruttare una vulnerabilità, MDM, effettuare il jailbreak o utilizzare toolkit specializzati per ottenere l'accesso privilegiato al file system.|Sostanziale.|
|Fisico|Un'immagine completa bit per bit del dispositivo, che consente di recuperare i dati eliminati.|Difficile da usare con i dispositivi moderni a causa degli estesi meccanismi di sicurezza; tuttavia, è incredibilmente prezioso, soprattutto con i dispositivi più vecchi.|Completo  (sui dispositivi senza crittografia a riposo ).|

## Mantenere l'accesso

Come per i dispositivi digitali tradizionali, preservare l'accesso è un obiettivo fondamentale per gli analisti. Ad esempio, se un dispositivo è sbloccato, dobbiamo assicurarci che rimanga sbloccato. Un dispositivo sbloccato rappresenta lo scenario migliore per un analista, poiché significa che un determinato numero di meccanismi di sicurezza non è più applicato.

Disabilitare il timer della schermata di blocco, configurabile nelle rispettive impostazioni su Android e iOS, può essere un modo efficace per garantire che il dispositivo resti sbloccato.

Inoltre, è essenziale abilitare la modalità "aereo" sui dispositivi mobili. Ciò impedisce qualsiasi modifica ai dati e, nel caso di iOS, impedisce la cancellazione remota tramite la funzione "Trova il mio".

## Acquisizione manuale

Questo metodo di acquisizione è spesso considerato un metodo preliminare per il recupero dei dati nell'ambito del processo investigativo. L'acquisizione manuale prevede la raccolta di prove navigando sul telefono, aprendo diverse applicazioni o messaggi e scattando foto delle informazioni presenti utilizzando un altro dispositivo.

Sebbene l'acquisizione manuale richieda lo sblocco del dispositivo, può essere un ottimo modo per ottenere rapidamente informazioni chiave. Tuttavia, è importante notare che questo compromette la non ripudiabilità e l'autenticità dei dati, rendendoli potenzialmente inammissibili e inaffidabili. Inoltre, molti artefatti potrebbero non essere rilevati a causa dell'impossibilità di consultare direttamente i log e i file di sistema. 

## Acquisizione logica

L'acquisizione logica prevede l'utilizzo di funzionalità del sistema operativo del dispositivo mobile per estrarre i dati. Questa tecnica è considerata molto più sicura nel preservare l'integrità delle prove, poiché nulla viene sovrascritto o modificato. Per i dispositivi mobili, ciò può comportare l'utilizzo di funzionalità per creare un backup ed esaminarlo.

Tuttavia, sebbene l'acquisizione logica fornisca molti più dati rispetto all'acquisizione manuale, è solo parziale, poiché i backup spesso ammettono tipi di dati specifici, tra cui i file del sistema operativo.

Ciò può essere fatto utilizzando ==strumenti quali 3uTools, Easeus== e  [libimobiledevice](https://libimobiledevice.org/) sulla CLI .

==Utilizzo di libimobiledevice per creare un backup per iOS==

```shell-session
cmnatic@thm-dev$ idevicebackup2 backup --full ./backup

Started "com.apple.mobilebackup2" service on port 49174.
Negotiated Protocol Version 2.1
Starting backup...
Backup will be unencrypted.
Requesting backup from device...
Full backup mode.
[==============================                       ] 55% Finished
Receiving files
[==================================================] 100% (12.6 MB/12.5 MB)
[==================================================] 100% (12.6 MB/12.5 MB)
[==================================================] 100% (12.7 MB/12.5 MB)
[==================================================] 100% (12.7 MB/12.5 MB)
[==================================================] 100% (12.7 MB/12.5 MB)
```

Per Android, questa operazione può essere eseguita utilizzando Android Debug Bridge (ADB):

==Utilizzo di ADB per creare un backup per Android==

```shell-session
cmnatic@thm-dev$ adb backup -apk -shared -all -f backup.ab

Backing up data... Please wait.
Writing android application package (APK) files... 
Writing shared storage files...
Backup Complete!
```

## Acquisizione del file system

Questo metodo di acquisizione fornisce un'estrazione completa del file system stesso. Si tratta di una tecnica molto più completa rispetto ad altre, come l'acquisizione logica. Ad esempio, i dati del sistema operativo saranno inclusi nell'estrazione, consentendo potenzialmente il recupero di dati cancellati e di dati aggiuntivi non inclusi nei backup, consentendo un'analisi completa.

Detto questo, questa tecnica non è priva di difficoltà. L'acquisizione del file system richiede l'accesso privilegiato al dispositivo. Cosa ciò comporti esattamente verrà spiegato nel prossimo passaggio; tuttavia, di solito comporta l'ottenimento dell'accesso root, spesso sfruttando le vulnerabilità del dispositivo per aggirare i meccanismi di sicurezza. Toolkit forensi specializzati come Cellebrite UFED sono in grado di eseguire tale acquisizione.

==Utilizzo di ADB per creare un'estrazione del file system per Android==

```shell-session
cmnatic@thm-dev$ adb pull /data /mnt/android_backup

pull: building file list...
pull: /data/anr/traces.txt -> /mnt/android_backup/anr/traces.txt
pull: /data/system/packages.xml -> /mnt/android_backup/system/packages.xml
pull: /data/system/users/0.xml -> /mnt/android_backup/system/users/0.xml
pull: /data/data/com.android.providers.contacts/databases/contacts2.db -> /mnt/android_backup/data/com.android.providers.contacts/databases/contacts2.db
...
[100%] /data -> /mnt/android_backup
```

_Esempio di utilizzo di ADB per eseguire l'estrazione del file system su un dispositivo Android con root._

Rispondi alle domande seguenti

Se volessi recuperare i dati cancellati, quale metodo di acquisizione dovrei provare?

```
Physical
```

Quale metodo di acquisizione prevede l'utilizzo delle funzionalità del sistema operativo per estrarre i dati?

```
Logical acquisition
```

Come si chiama lo strumento che può essere utilizzato per eseguire un backup di un **Iphone** , tramite la CLI?

```
idevicebackup2
```

Come si chiama lo strumento che può essere utilizzato per eseguire un backup di un **Android** , tramite la CLI?

Si prega di notare che stiamo cercando l'acronimo.

```
ADB
```

## Tecniche di acquisizione specialistiche

## Servizi specialistici

Suite hardware e software specializzate sono disponibili per organizzazioni specifiche. Ad esempio, Cellebrite UFED è una suite hardware e software avanzata per l'acquisizione e l'analisi, accessibile solo alle forze dell'ordine, alle agenzie governative e a organizzazioni simili. Utilizza tecniche sofisticate per aggirare i meccanismi di sicurezza e analizzare i dispositivi mobili.

Altre utilità specialistiche includono Oxygen Suite, che può estrarre dati dal cloud e dati all'interno del dispositivo e aggirare determinati meccanismi di sicurezza.

## Jailbreaking

Il jailbreaking consiste nello sfruttare una vulnerabilità nota nel sistema operativo del dispositivo mobile per ottenere il cosiddetto accesso "a livello root", consentendo il controllo completo del dispositivo. Questa tecnica fornisce un accesso non filtrato al dispositivo, ma lo modifica in modo permanente, quindi non è valida dal punto di vista forense.

Sebbene il jailbreak sia ancora possibile, solitamente è possibile eseguirlo solo su vecchie versioni del sistema operativo, una volta scoperto un exploit, ad esempio se il dispositivo non dispone degli ultimi aggiornamenti.

## Caricamento di avvio personalizzato

Questa tecnica prevede l'avvio del dispositivo mobile con un sistema operativo temporaneo e personalizzato che fornisce un accesso di basso livello al dispositivo e aggira i meccanismi di sicurezza. Si differenzia da altre tecniche, come il jailbreak, in quanto non altera in modo permanente il dispositivo, rendendolo affidabile dal punto di vista forense.

Tuttavia, meccanismi di sicurezza come la crittografia potrebbero essere ancora in atto, soprattutto nei dispositivi moderni che memorizzano le chiavi di crittografia in un componente hardware dedicato. Inoltre, i dispositivi moderni utilizzano la cosiddetta catena di "avvio sicuro" per garantire l'esecuzione solo di codice affidabile e verificato, rendendo molto più difficile il caricamento personalizzato.

## JTAG

Il JTAG (Joint Test Action Group) è stato inizialmente utilizzato per diagnosticare i componenti dei circuiti stampati. Tuttavia, si è fatto strada nell'ambito dell'informatica forense mobile estraendo fisicamente i dati direttamente dai componenti hardware del dispositivo. Il JTAG è ormai considerato obsoleto per i dispositivi mobili moderni, principalmente a causa dei rischi, degli strumenti e delle conoscenze richieste.

## Forza bruta

Sebbene in precedenza avessimo accennato in questa sala al fatto che i dispositivi moderni dispongono di meccanismi di sicurezza contro gli attacchi di forza bruta, come il codice PIN, l'utente deve configurarli. In caso contrario, è possibile utilizzare degli strumenti per indovinare il codice PIN e sbloccare il dispositivo in modo casuale. Sebbene richieda molto tempo, se si è sufficientemente motivati, alla fine si riuscirà a sbloccarlo, anche se tra 10 anni.

I dispositivi mobili solitamente includono protezioni quali limitazione della velocità e blocco per **x** **tempo** dopo **y tentativi** , che aumentano ogni volta che vengono attivate.

## Estrazione delle nuvole

Un approccio molto più basato sulla legge, l'estrazione dal cloud, prevede l'invio di richieste legali a produttori, sviluppatori di app, ecc., per recuperare informazioni o backup specifici dell'applicazione.  Nella maggior parte dei casi, questo può essere un approccio più _efficace_  per il recupero dei dati piuttosto che aggirare i meccanismi di sicurezza. 

Rispondi alle domande seguenti

Come si chiama la tecnica che avvia il dispositivo con un sistema operativo temporaneo, spesso bypassando i meccanismi di sicurezza?

```
Custom Boot Loading
```

Come si chiama la tecnica che sfrutta una vulnerabilità nota all'interno del dispositivo, concedendogli accesso completo o "root"?

```
Jailbreaking
```

## Pratico

Per la parte pratica, sostituirai un analista forense nel team di analisi mobile di TryAnalyseMe. Questa organizzazione privata fornisce servizi di analisi forense su un'ampia gamma di dispositivi. 

Un nuovo caso è arrivato sulla tua scrivania. Il tuo compito è:

- Creare il caso (come investigatore “analista”) 
- Aggiungere l'elemento probatorio utilizzando i dettagli nella seguente tabella
- Eseguire l'acquisizione dei dati
- Esporta il report per recuperare il flag

**Distribuisci il sito statico associato a questa attività** . I ​​dettagli sull'iPhone su cui devi eseguire un'acquisizione sono forniti nella tabella sottostante. **Assicurati di compilarla quando aggiungi l'elemento come prova.**

|   |   |
|---|---|
|**Campo**|**Valore**|
|**IMEI**|358240051111112|
|**Tipo**|Mobile|
|**Colore**|Argento|
|**Sistema operativo**|iOS|
|**Investigatore**|Analista|

Rispondi alle domande seguenti

Dopo aver creato il caso e aggiunto l'iPhone come prova, esegui l'acquisizione dei dati.  
  
Qual è il flag visualizzato al termine dell'acquisizione?

```
THM{MOBILE_xxxxxxx}
```


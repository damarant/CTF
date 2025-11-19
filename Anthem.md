
#tryhackmelabs #laboratorio 

https://tryhackme.com/r/room/anthem?ref=blog.tryhackme.com

Eseguiamo nmap e controlliamo quali porte sono aperte.
```
nmap -Pn -O -sV -v -p- 10.10.180.163
```
```
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
3389/tcp open  ms-wbt-server Microsoft Terminal Services
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
```

Quale porta è per il server web?

```
80
```

Quale porta è per il servizio desktop remoto?

```
3389
```

Quale potrebbe essere una password in una delle pagine controllate dai web crawler?
```
http://10.10.180.163/robots.txt
```
```
UmbracoIsTheBest!
```

Quale CMS utilizza il sito web?
```
http://10.10.180.163/umbraco/#/login
```
```
Umbraco
```

Qual è il dominio del sito web?

```
Anthem.com
```

Come si chiama l'amministratore?  
```
http://10.10.180.163/archive/a-cheers-to-our-it-department/
```
fatto una ricerca su Google della poesia trovata e il primo risultato è stato un nome utente
```
Solomon Grundy
```

Possiamo trovare l'indirizzo email dell'amministratore?
visto che il formato email trovato per CV è JD@anthem.com proviamo
```
sg@anthem.com
```

Cos'è la bandiera 1?
```
view-source:http://10.10.180.163/archive/we-are-hiring/
```

```
THM{L0L_WH0_US3S_M3T4}
```

Cos'è la bandiera 2?
```
view-source:http://10.10.180.163/archive/a-cheers-to-our-it-department/
```

```
THM{G!T_G00D}
```

Cos'è la bandiera 3?
```
http://10.10.180.163/authors/jane-doe/
```
```
THM{L0L_WH0_D15}
```

Cos'è la bandiera 4?

```
view-source:http://10.10.180.163/archive/a-cheers-to-our-it-department/
```

```
THM{AN0TH3R_M3TA}
```

Scopriamo il nome utente e la password per accedere alla casella. (La casella non è su un dominio)

possiamo loggarci con
```
http://10.10.180.163/umbraco/#/login
```
```
sg@anthem.com
```
```
UmbracoIsTheBest!
```

Ottieni l'accesso iniziale alla macchina. Qual è il contenuto di user.txt?
```
xfreerdp /u:sg /p:UmbracoIsTheBest! /v:10.10.180.163
```
**oppure utilizziamo Remmina**
```
THM{N00T_NO0T}
```

Possiamo individuare la password di amministratore?

per l'escalation dei privilegi proviamo a guardare i file modificati di recente usando run(esegui)>recent
troviamo in Backup un file restore.txt senza permessi di lettura
entriamo in proprietà ed aggiungiamo utente SG nel gruppo con tutti i permessi abilitati
ora possiamo leggere il contenuto di restore.txt
```
ChangeMeBaby1MoreTime
```

Aumenta i tuoi privilegi a root, qual è il contenuto del file root.txt?

Andiamo in (quando ci richiede la password inseriamo ChangeMeBaby1MoreTime )
```
C:\Users\Administrator\Desktop
```
troviamo il file root.txt
```
THM{XXXXXXX}
```







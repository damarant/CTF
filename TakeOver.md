
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/takeover

Salve,  
  
sono l'amministratore delegato e uno dei co-fondatori di futurevera. thm . In Futurevera crediamo che il futuro sia nello spazio. Facciamo molta ricerca spaziale e scriviamo blog a riguardo. In passato aiutavamo gli studenti con domande sullo spazio, ma stiamo ricostruendo il nostro supporto.  

[Di recente, degli hacker blackhat ci hanno contattato dicendo che potrebbero impossessarsi del nostro sistema](https://futurevera.thm/) e ci chiedono un lauto riscatto. Aiutateci a scoprire cosa possono impossessarsi.  
  
Il nostro sito web è [https://futurevera.thm](https://futurevera.thm/)

Suggerimento: non dimenticare di aggiungere 10.10.214.87 in /etc/hosts per futurevera.thm ; )

Rispondi alle domande seguenti

Qual è il valore della bandiera?
Questa è una sfida di enumerazione: una volta trovata, ti verrà subito assegnata la bandiera.

```
sudo echo '10.10.214.87 futurevera.thm' >> /etc/hosts
```

```
nmap -Pn -O -v -p- futurevera.thm
```
```
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https
```

```
gobuster dir -u http://futurevera.thm:80/ -w /usr/share/wordlists/dirb/common.txt -t50
```
```
/.htpasswd            (Status: 403) [Size: 279]
/.htaccess            (Status: 403) [Size: 279]
/.hta                 (Status: 403) [Size: 279]
/index.php            (Status: 302) [Size: 0] [--> https://futurevera.thm/]
/server-status        (Status: 403) [Size: 279]
Progress: 4614 / 4615 (99.98%)
```

```
gobuster dir -u https://futurevera.thm:443/ -w /usr/share/wordlists/dirb/common.txt -t50 -k
```
 -k opzione per ignorare il certificato non valido
```
/.htpasswd            (Status: 403) [Size: 280]
/.hta                 (Status: 403) [Size: 280]
/.htaccess            (Status: 403) [Size: 280]
/assets               (Status: 301) [Size: 319] [--> https://futurevera.thm/assets/]
/css                  (Status: 301) [Size: 316] [--> https://futurevera.thm/css/]
/index.html           (Status: 200) [Size: 4605]
/js                   (Status: 301) [Size: 315] [--> https://futurevera.thm/js/]
/server-status        (Status: 403) [Size: 280]
```
Visitiamo
```
https://futurevera.thm/index.html/
```

```
https://futurevera.thm/assets/
```

```
https://futurevera.thm/css/
```

```
https://futurevera.thm/js/
```

niente di che, enumeriamo i sottodomini con ffuf

```
ffuf -w /usr/share/dirb/wordlists/common.txt -H "Host: FUZZ.futurevera.thm" -u https://10.10.214.87 -fs 4605 -c
```
**-H:** Intestazione **“Nome: Valore”** , separata da due punti (per trovare sottodomini senza record DNS)
**-fs:** filtra la dimensione della risposta HTTP (per filtrare in base alla dimensione della risposta)

```
Blog                    [Status: 200, Size: 3838, Words: 1326, Lines: 81]
blog                    [Status: 200, Size: 3838, Words: 1326, Lines: 81]
Support                 [Status: 200, Size: 1522, Words: 367, Lines: 34]
support                 [Status: 200, Size: 1522, Words: 367, Lines: 34]
```

aggiungiamo  i sottodomini **“blog”** e **“support”** al file **hosts** 

```
echo -e "10.10.214.87 blog.futurevera.thm\n10.10.214.87 support.futurevera.thm" | sudo tee -a /etc/hosts
```

visitiamo
```
https://blog.futurevera.thm/
```
e
```
https://support.futurevera.thm/
```

non troviamo nulla di utile neanche nel sorgente,
dopo tante ricerche troviamo tra le informazioni del certificato
```
DNS Name secrethelpdesk934752.support.futurevera.thm
```
aggiungiamolo al file di hosts 
```
sudo echo '10.10.214.87 secrethelpdesk934752.support.futurevera.thm' >> /etc/hosts
```
andiamo su
```
secrethelpdesk934752.support.futurevera.thm
```
ed otteniamo
```
We can\u2019t connect to the server at flag{xxxxxxxxxxxxxxxx}.s3-website-us-west-3.amazonaws.com.
```
ecco la nostra flag
```
flag{xxxxxxxxxxxxxxx}
```



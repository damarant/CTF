
#tryhackmelabs #laboratorio #idor

https://tryhackme.com/room/corridor

Ti sei ritrovato in uno strano corridoio. Riesci a ritrovare la strada da cui sei venuto?  

In questa sfida, esplorerai potenziali vulnerabilità IDOR . Esamina gli endpoint URL a cui accedi durante la navigazione del sito web e annota i valori esadecimali che trovi (assomigliano molto a un _hash_ , non è vero?). Questo potrebbe aiutarti a scoprire posizioni di siti web a cui non avresti dovuto accedere.

Le **IDOR** (Insecure Direct Object References) sono una vulnerabilità di sicurezza nelle applicazioni web che si verifica quando un attaccante può accedere a oggetti o risorse a cui non dovrebbe avere accesso, semplicemente manipolando i parametri della richiesta. Questo tipo di vulnerabilità è spesso legato a un controllo inadeguato delle autorizzazioni.

**Come funzionano le IDOR**

Le IDOR si verificano quando un'applicazione espone riferimenti diretti a oggetti, come file, record di database o altre risorse, senza implementare un adeguato controllo di accesso. Ad esempio, se un URL contiene un identificatore di un oggetto (come un numero di ID) e un utente malintenzionato modifica quel numero, potrebbe accedere a dati riservati.

```
nmap -sVC -vv -p- 10.10.123.162
```
```
80/tcp open  http    syn-ack ttl 63 Werkzeug httpd 2.0.3 (Python 3.10.2)
| http-methods: 
|_  Supported Methods: HEAD OPTIONS GET
|_http-server-header: Werkzeug/2.0.3 Python/3.10.2
|_http-title: Corridor
MAC Address: 02:BC:49:5B:43:A7 (Unknown)

```

andiamo su

http://10.10.123.162/

e troviamo una pagina con tutte porte

come consigliato annotiamo i valori esadecimali

```
c81e728d9d4c2f636f067f89cc14862c
```

```
eccbc87e4b5ce2fe28308fd9f2a7baf3
```

```
a87ff679a2f3e71d9181a67b7542122c
```

```
e4da3b7fbbce2345d7772b0674a318d5
```

```
1679091c5a880faf6fb5e6087eb1b2dc
```

```
8f14e45fceea167a5a36dedd4bea2543
```

```
c51ce410c124a10e0db5e4b97fc2af39
```

```
c20ad4d76fe97759aa27a0c99bff6710
```

```
6512bd43d9caa6e02c990b0a82652dca
```

```
d3d9446802a44259755d38e6d163e820
```

```
45c48cce2e2d7fbdea1afc51c7c6ad26
```

```
c9f0f895fb98ab9159f51fd0297e236d
```

```
c81e728d9d4c2f636f067f89cc14862c
eccbc87e4b5ce2fe28308fd9f2a7baf3
a87ff679a2f3e71d9181a67b7542122c
e4da3b7fbbce2345d7772b0674a318d5
1679091c5a880faf6fb5e6087eb1b2dc
8f14e45fceea167a5a36dedd4bea2543
c51ce410c124a10e0db5e4b97fc2af39
c20ad4d76fe97759aa27a0c99bff6710
6512bd43d9caa6e02c990b0a82652dca
d3d9446802a44259755d38e6d163e820
45c48cce2e2d7fbdea1afc51c7c6ad26
c9f0f895fb98ab9159f51fd0297e236d
```

cerchiamo di capire di che hash si tratta

```
echo 'c81e728d9d4c2f636f067f89cc14862c' > hash
```

```
hashid hash
```

```
--File 'hash'--
Analyzing 'c81e728d9d4c2f636f067f89cc14862c'
[+] MD2 
[+] MD5 
[+] MD4 
[+] Double MD5 
[+] LM 
[+] RIPEMD-128 
[+] Haval-128 
[+] Tiger-128 
[+] Skein-256(128) 
[+] Skein-512(128) 
[+] Lotus Notes/Domino 5 
[+] Skype 
[+] Snefru-128 
[+] NTLM 
[+] Domain Cached Credentials 
[+] Domain Cached Credentials 2 
[+] DNSSEC(NSEC3) 
[+] RAdmin v2.x 

```

dovremmo aver a che fare con md5

andiamo su
```
https://crackstation.net/
```

immettiamo tutti gli hash e avviamo la decodifica

abbiamo a che fare con md5 e tutti gli hash corrispondono a numeri da 2 a 13

proviamo allora a codificare il numero 1 che dovrebbe corrispondere alla porta o pagina1

andiamo su
```
https://cyberchef.io/#recipe=MD5()&input=MQo
```

mettiamo il numero 1 come input e MD5 come codifica otteniamo

```
c4ca4238a0b923820dcc509a6f75849b
```
andiamo su 
```
http://10.10.123.162/c4ca4238a0b923820dcc509a6f75849b
```
ed abbiamo una stanza vuota
proviamo a codificare il numero zero
e quindi andiamo su
```
http://10.10.123.162/cfcd208495d565ef66e7dff9f98764da
```

```
flag{xxxxxxxxxxxxx} 
```
ecco trovata la nostra flag!


https://tryhackme.com/path/outline/pentestplus

**Impatto della formazione sui team**
https://tryhackme.com/room/training

**Impegni del Red Team**
https://tryhackme.com/room/redteamengagements

https://tryhackme.com/room/pythonforcybersecurity

https://tryhackme.com/room/windowslocalpersistence


#laboratorio #tryhackmelabs 

https://tryhackme.com/room/googledorking

Google è probabilmente l'esempio più famoso di "motori di ricerca", voglio dire, chi si ricorda di Ask Jeeves? _rabbrividisce_

Ora, spiegare come funzionano questi "motori di ricerca" potrebbe sembrare un po' paternalistico, ma dietro le quinte c'è molto di più di quello che vediamo. Ancora più importante, possiamo sfruttare questo a nostro vantaggio per trovare ogni sorta di informazioni che un elenco di parole non troverebbe.

 "motori di ricerca" come Google sono enormi indicizzatori, in particolare di contenuti sparsi sul World Wide Web.

Questi elementi essenziali per la navigazione in Internet utilizzano i cosiddetti "crawler" o "spider" per cercare contenuti nel World Wide Web, di cui parlerò nel prossimo esercizio.

## Impariamo a conoscere i crawler

**Cosa sono i crawler e come funzionano?**

Questi crawler scoprono i contenuti attraverso vari metodi. Uno di questi è la scoperta pura, in cui un URL viene visitato dal crawler e le informazioni relative al tipo di contenuto del sito web vengono restituite al motore di ricerca. In effetti, sono moltissime le informazioni che i crawler moderni raccolgono, ma ne parleremo più avanti. Un altro metodo utilizzato dai crawler per scoprire i contenuti è seguire tutti gli URL trovati nei siti web precedentemente scansionati. Un po' come un virus, nel senso che vorrà attraversare/diffondersi ovunque possa.  
 
Una volta che un web crawler rileva un dominio come **mywebsite.com** , ne indicizza l'intero contenuto, cercando parole chiave e altre informazioni varie, ma parlerò di queste informazioni varie più avanti.

Tuttavia,  **i crawler tentano di esplorare, ovvero di scansionare, ogni URL e file che riescono a trovare!** Ad esempio, se " **mywebsite.com** " avesse le stesse parole chiave di prima ("Apple", "Banana" e "Pear"), ma avesse anche un URL verso un altro sito web " **otherwebsite.com** ", il crawler tenterà di esplorare tutto ciò che si trova su quell'URL ( **otherwebsite.com** ) e di recuperare il contenuto di tutto ciò che si trova all'interno di quel dominio.

Indica il termine chiave per definire a cosa serve un "Crawler". Si tratta di una raccolta di risorse e delle relative posizioni.

```
index
```

Come si chiama la tecnica utilizzata dai "motori di ricerca" per recuperare queste informazioni sui siti web?

```
crawling
```

Qual è un esempio del tipo di contenuti che potrebbero essere raccolti da un sito web?

```
keywords
```

## Ottimizzazione dei motori di ricerca

L'ottimizzazione per i motori di ricerca (SEO) è un argomento diffuso e redditizio nei motori di ricerca moderni. Tanto che intere aziende sfruttano il miglioramento del "ranking" SEO di un dominio. In astratto, i motori di ricerca "daranno priorità" ai domini più facili da indicizzare. Ci sono molti fattori che determinano quanto sia "ottimale" un dominio, il che si traduce in qualcosa di simile a un sistema di punteggio.

Ma...chi o cosa regolamenta questi "crawler"?

Oltre ai motori di ricerca che forniscono questi "crawler", sono i proprietari stessi di siti web/server a stabilire quali contenuti i "crawler" possano analizzare. I motori di ricerca vorranno recuperare **tutto** da un sito web, ma ci sono alcuni casi in cui non vorremmo che **tutti** i contenuti del nostro sito web venissero indicizzati! Riesci a pensarne qualcuno...? Che ne dici di una pagina di accesso segreta per gli amministratori? Non vogliamo che **chiunque** possa trovare quella directory, soprattutto tramite una ricerca su Google.

Introduzione a Robots.txt...

## Beepboop - Robots.txt

﻿﻿**Robots.txt**
Simile alle "Sitemap" di cui parleremo più avanti, questo file è la prima cosa indicizzata dai "Crawler" quando si visita un sito web.
 
**Ma di cosa si tratta?**
Questo file deve essere servito nella directory radice, specificata dal webserver stesso. Considerando l'estensione **.txt** , è abbastanza probabile che si tratti di un file di testo.

Il file di testo definisce i permessi che il "Crawler" ha sul sito web. Ad esempio, quale tipo di "Crawler" è consentito (ad esempio, vuoi che solo il "Crawler" di Google indicizzi il tuo sito e non quello di MSN). Inoltre, Robots.txt può specificare quali file e directory vogliamo o non vogliamo che vengano indicizzati dal "Crawler".

Ecco alcune parole chiave...

|            |                                                                                                                                                               |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Keyword    | Funzione                                                                                                                                                      |
| User-agent | Specifica il tipo di "Crawler" che può indicizzare il tuo sito (l'asterisco è un carattere jolly, consentendo **tutti gli "User-agent"**\|                    |
| Allow      | Specificare le directory o i file che il "Crawler"  **può** indicizzare\|                                                                                     |
| Sitemap    | Fornire un riferimento a dove si trova la mappa del sito (migliora la SEO come discusso in precedenza, parleremo delle mappe del sito nel prossimo compito)\| |
| Disallow   | Specificare le directory o i file che il "Crawler"  **non può** indicizzare                                                                                   |
[La " Sitemap](http://mywebsite.com/sitemap.xml) " si trova [all'indirizzo](http://mywebsite.com/sitemap.xml) [http://mywebsite.com/sitemap.xml](http://mywebsite.com/sitemap.xml)

Dove si troverebbe "robots.txt" sul dominio " **ablog.com** "

```
ablog.com/robots.txt
```

Se un sito web avesse una mappa del sito, dove si troverebbe?

```
/sitemap.xml
```

Come potremmo consentire solo a "Bingbot" di indicizzare il sito web?

```
User-agent: Bingbot
```

Come possiamo impedire a un "Crawler" di indicizzare la directory "/dont-index-me/"?  

```
Disallow: /dont-index-me/
```

Qual è l'estensione di un file di configurazione di sistema Unix/Linux che potremmo voler nascondere ai "Crawler"?

```
.conf
```


## Sitemaps

**Mappe del sito**

Paragonabili alle mappe geografiche della vita reale, le "Sitemap" sono proprio questo, ma per i siti web!
  
Le "Sitemap" sono risorse indicative utili per i crawler, poiché specificano i percorsi necessari per trovare i contenuti sul dominio.

Le "Sitemap" sono formattate in XML . 

La presenza di "Sitemap" ha un peso notevole nell'influenzare l'"ottimizzazione" e la popolarità di un sito web. Queste mappe semplificano notevolmente la navigazione dei contenuti per il crawler!

Perché le "Sitemap" sono così vantaggiose per i motori di ricerca?

I motori di ricerca sono pigri! Meglio ancora, hanno molti dati da elaborare. L'efficienza con cui questi dati vengono raccolti è fondamentale. Risorse come le "Sitemap" sono estremamente utili per i "crawler", poiché i percorsi necessari per raggiungere i contenuti sono già forniti! Tutto ciò che il crawler deve fare è estrarre questi contenuti, invece di dover affrontare il processo di ricerca e estrazione manuale. Immagina di usare un elenco di parole per trovare i file invece di indovinarne il nome a caso!


Più un sito web è facile da "scansionare", più è ottimizzato per il "motore di ricerca"

Rispondi alle domande seguenti

Qual è la tipica struttura dei file di una "Sitemap"?

```
xml
```

A quale esempio concreto si può paragonare la parola "Sitemap"?

```
map
```

Assegna un nome alla parola chiave per il percorso seguito per il contenuto di un sito web

```
route
```

## What is Google Dorking?

Come abbiamo già detto, Google scansiona e indicizza moltissimi siti web. Il cittadino medio usa Google per cercare immagini di gatti (io personalmente sono più un amante dei cani...). Sebbene Google abbia già indicizzato molte immagini di gatti, pronte per essere mostrate al cittadino, questo è un utilizzo piuttosto banale del motore di ricerca rispetto a ciò per cui può essere utilizzato.  
Ad esempio, possiamo aggiungere operatori come quelli dei linguaggi di programmazione per aumentare o diminuire i risultati di ricerca, oppure eseguire operazioni aritmetiche!

Supponiamo di voler restringere la nostra query di ricerca, utilizzando le virgolette. Google interpreterà tutto ciò che è compreso tra queste virgolette come esatto e restituirà solo i risultati della frase esatta fornita

**Perfezionamento delle nostre query**

Possiamo usare termini come " **site** " (ad esempio bbc.co.uk) e una query (ad esempio "gchq news") per cercare nel sito specificato la parola chiave che abbiamo fornito, filtrando così i contenuti che potrebbero essere più difficili da trovare altrimenti. Ad esempio, utilizzando "site" e "query" di "bbc" e "gchq", abbiamo modificato l'ordine in cui Google restituisce i risultati.

**Cosa rende il "Google Dorking" così attraente?**

Prima di tutto, e la parte importante, è legale! Sono tutte informazioni indicizzate e disponibili al pubblico. Tuttavia, è qui che entra in gioco la questione della legalità...
 
Ecco alcuni termini comuni che possiamo cercare e combinare:

|           |                                                                                |
| --------- | ------------------------------------------------------------------------------ |
| Term      | Azione                                                                         |
| filetype: | Cerca un file in base alla sua estensione (ad esempio PDF)                     |
| cache:    | Visualizza la versione memorizzata nella cache di Google di un URL specificato |
| intitle:  | La frase specificata DEVE apparire nel titolo della pagina                     |
|           |                                                                                |
Ad esempio, supponiamo di voler usare Google per cercare tutti i PDF su bbc.co.uk:

`site:bbc.co.uk filetype:pdf`

Quale sarebbe il formato utilizzato per interrogare il sito bbc.co.uk sulle difese contro le inondazioni?  

```
site:bbc.co.uk flood defences
```

Quale termine useresti per effettuare una ricerca in base al tipo di file?

```
filetype:
```

Quale termine possiamo usare per cercare le pagine di login?

```
intitle:login
```
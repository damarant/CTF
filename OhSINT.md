
#laboratorio #tryhackmelabs 

https://tryhackme.com/room/ohsint

Quali informazioni è possibile ottenere con un solo file immagine?

**Nota:** questa sfida è stata aggiornata il 01/02/2024. Se state seguendo delle vecchie guide dettagliate, aspettatevi una piccola modifica. Inoltre, il file è disponibile anche su AttackBox, nella `/Rooms/OhSINT`directory.

```
cd /root/Rooms/OhSINT
```

```
ls
```

```
WindowsXP.jpg
```
cerchiamo info sull'immagine trovata
```
exiftool WindowsXP.jpg
```

```
ExifTool Version Number         : 11.88
File Name                       : WindowsXP.jpg
Directory                       : .
File Size                       : 229 kB
File Modification Date/Time     : 2022:09:02 08:48:02+01:00
File Access Date/Time           : 2025:05:13 15:13:14+01:00
File Inode Change Date/Time     : 2022:09:02 08:48:11+01:00
File Permissions                : rw-rw-r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
XMP Toolkit                     : Image::ExifTool 11.27
GPS Latitude                    : 54 deg 17' 41.27" N
GPS Longitude                   : 2 deg 15' 1.33" W
Copyright                       : OWoodflint
Image Width                     : 1920
Image Height                    : 1080
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 1920x1080
Megapixels                      : 2.1
GPS Latitude Ref                : North
GPS Longitude Ref               : West
GPS Position                    : 54 deg 17' 41.27" N, 2 deg 15' 1.33" W

```
facciamo ricerca dell' autore su google
```
OWoodflint
```
Di cosa è composto l'avatar di questo utente?
```
https://x.com/OWoodflint/status/1102220421091463168
```
```
cat
```
In quale città si trova questa persona?
cerchiamo le coordinate trovate

```
Bssid: B4:5D:50:AA:86:41
```
```
GPS Latitude                    : 54 deg 17' 41.27" N
GPS Longitude                   : 2 deg 15' 1.33" W
```
convertiamo coordinate GMS
```
https://www.coordinate-gps.it/conversione-coordinate-gps
```
latitudine
```
54.2947972
```
Longitudine
```
-2.2503694444444444
```

In quale città si trova questa persona?
```
london
```

**importante salvare**
```
https://wigle.net/
darbarbaro@ist-hier.com
darbarbaro
regna1234
```

Qual è l'SSID del WAP a cui si è connesso?
inseriamo BSSID
```
https://wigle.net/search#detailSearch?netid=B4%3A5D%3A50%3AAA%3A86%3A41
```

```
UnileverWiFi
```

Qual è il suo indirizzo email personale?  
lo troviamo su https://github.com/OWoodfl1nt/people_finder
```
OWoodflint@gmail.com
```

Su quale sito hai trovato il suo indirizzo email?

```
github
```

Dove è andato in vacanza?  

su github troviamo un link
https://oliverwoodflint.wordpress.com/
```
New York
```

Qual è la password della persona?
lo troviamo sempre nella pagina wordpress!
```
pennYDr0pper.!
```

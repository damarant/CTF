
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/neighbour

visitiamo la pagina
http://10.10.30.170/
abbiamo un form per loggarci

esploriamo il codice sorgente con ctrl+u

troviamo user e password
```
<!-- use guest:guest credentials until registration is fixed. "admin" user account is off limits!!!!! -->
```

ci logghiamo con le credenziali trovate

```
guest:guest
```

notiamo url `http://10.10.30.170/profile.php?user=guest`

proviamo a sostituirla con admin
```
http://10.10.30.170/profile.php?user=admin
```
ecco trovata la nostra flag!

```
# Hi, **admin**. Welcome to your site. The flag is: flag{xxxxxxxxxxxxxxx}
```
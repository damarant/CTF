#tryhackmelabs #laboratorio 

https://tryhackme.com/room/cyberheroes

Vuoi far parte del club d'élite dei CyberHeroes? Dimostra il tuo valore trovando un modo per accedere!

```
nmap -Pn -O -v -p- 10.10.172.165
```

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```
visitiamo
```
http://10.10.172.165/
```
troviamo
Siamo un gruppo di giovani hacker, sviluppatori, cacciatori di bug, cyber guerrieri e guardiani informatici. Troviamo vulnerabilità nei siti web in modo legale e li salviamo dagli hacker. Unisciti a noi, trova la vulnerabilità nella nostra pagina di login e accedi per unirti a noi. :D

andiamo alla pagina di login 
```
http://10.10.172.165/login.html
```
ispezioniamo il codice e troviamo qualcosa d'interessante
```
  <script>
    function authenticate() {
      a = document.getElementById('uname')
      b = document.getElementById('pass')
      const RevereString = str => [...str].reverse().join('');
      if (a.value=="h3ck3rBoi" & b.value==RevereString("54321@terceSrepuS")) { 
        var xhttp = new XMLHttpRequest();
        xhttp.onreadystatechange = function() {
          if (this.readyState == 4 && this.status == 200) {
            document.getElementById("flag").innerHTML = this.responseText ;
            document.getElementById("todel").innerHTML = "";
            document.getElementById("rm").remove() ;
          }
        };
        xhttp.open("GET", "RandomLo0o0o0o0o0o0o0o0o0o0gpath12345_Flag_"+a.value+"_"+b.value+".txt", true);
        xhttp.send();
      }
      else {
        alert("Incorrect Password, try again.. you got this hacker !")
      }
    }
  </script>
```

il codice verifica se il valore del campo nome utente è uguale a `"h3ck3rBoi"` e se il valore della password è uguale alla stringa `"54321@terceSrepuS"` scritta al contrario, se le credenziali sono corrette, viene creata una nuova richiesta

quindi immettiamo l'username
```
h3ck3rBoi
```
e come password
```
xxxxxxxxxxx
```
otteniamo la nostra flag!
```
#### Congrats Hacker, you made it !! Go ahead and nail other challenges as well :D flag{edb0be532c540b1a150c3a7e85d2466e}
```

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

```

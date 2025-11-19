
#tryhackmelabs #laboratorio 

https://tryhackme.com/room/printnightmarec3kj?ref=blog.tryhackme.com


A quale indirizzo remoto è arrivato il dipendente?

```
20.188.56.147
```

Secondo il PCAP, quale utente restituisce un errore STATUS_LOGON_FAILURE?

```
THM-PRINTNIGHT0\rjones
```

Quale utente riesce a connettersi correttamente a una condivisione SMB?

```
THM-PRINTNIGHT0/gentilguest
```

Qual è la prima condivisione SMB remota a cui è connesso l'endpoint? Qual era il primo nome file? Qual era il secondo? ( **formato** : risposta,risposta,risposta)

```
\\printnightmare.gentilkiwi.com\IPC$,print,srvsvc,spoolss
```
oppure Utilizzare Brim 
```
_path=~smb* OR _path=dce_rpc | cut ts, path, endpoint | sort ts
```
Da quale condivisione SMB remota è stata ottenuta la DLL dannosa? Qual era il percorso per la cartella remota per la prima DLL? E per la seconda?  ( formato : risposta,risposta,risposta)
```
\\printnightmare.gentilkiwi.com\print$,\x64\3mimispool.dll,\W32X86\3\mimispool.dll
```

Qual è stata la prima posizione in cui è stata scaricata la DLL dannosa sull'endpoint? Qual è stata la seconda?

```
_path=~smb* OR _path=dce_rpc AND name=*.dll || sort ts
```

```
C:\\Windows\system32\spool\drivers\X64\3,C:\Windows\system32\spool\drivers\W32X86\3
```

Qual è la cartella che ha il nome del server di stampa remoto a cui l'utente si è connesso? (fornire il percorso completo della cartella)
controlliamo logfile
```
C:\\Windows\System32\spool\SERVERS\printnightmare.gentilkiwi.com
```

Qual è il nome della stampante aggiunta dalla DLL?

```
Kiwi Legit Printer
```

Qual era l'ID di processo per il prompt dei comandi con privilegi elevati? Qual era il suo processo padre? ( **formato** : risposta,risposta)

```
5408,spoolsv.exe
```

Quale comando ha eseguito l'utente per elevare i privilegi?
da process monitor

utilizziamo FullEventLogView

```
net localgroup administrators rjones /add
```



















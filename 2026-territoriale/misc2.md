# OliCyber.IT 2026 - Selezione Territoriale

## [misc] the cantina (253 solves)

sei appena arrivato alla cantina. il droide barista beep-boopa in attesa del tuo ordine.

Author: Andrea Raineri <@Rising>

## Panoramica

La challenge consiste in un server TCP con il quale bisogna interagire eseguendo una specifica serie di comandi in modo da ottenere i comandi. Il server accetta i seguenti comandi:

- `select_coin`
- `select_wallet`
- `authenticate`
- `topup_wallet`
- `list_drinks`
- `buy_drink`

I comandi vanno eseguiti con parametri e ordine specifico che è possibile dedurre tramite l'analisi della cattura di traffico fornita come attachment.

## Soluzione

Analizzando i flussi TCP (follow TCP stream) non si trova un flusso che contenga la flag o esegua la completa serie di comandi corretti per ottenere la flag, ma si possono combinare le informazioni presenti nei vari flussi per dedurre la corretta sequenza di comandi da eseguire.

- Il flusso 2 rivela le risposte per eseguire il comando authenticate con successo e il seguente topup_wallet per caricare i fondi
- Il flusso 3 mostra i drink disponibili, i loro prezzi, in particolare il drink "ESCLUSIVO" (da comprare per ottenere la flag)
- Il flusso 4 rivela nel messaggio di errore all'acquisto del drink "ESCLUSIVO" che è necessario selezionare la moneta OLI
- Il flusso 5 rivela nel messaggio di errore che il wallet 0xBEEF non è valido, si può dedurre che quello corretto sia 0xBABE del flusso 4

Dunque la sequenza corretta di comandi da eseguire è:

```text
select_coin
OLI

select_wallet
0xBABE

authenticate
Han
Vader
Kashyyyk

topup_wallet

buy_drink
Darksaber Distillate
```

## Exploit

```python
# Python script con pwntools

from pwn import *

r = remote('host', 1337)

r.sendline(b'select_coin')
r.sendline(b'OLI')
r.sendline(b'select_wallet')
r.sendline(b'0xBABE')
r.sendline(b'authenticate')
r.sendline(b'Han')
r.sendline(b'Vader')
r.sendline(b'Kashyyyk')
r.sendline(b'topup_wallet')
r.sendline(b'buy_drink')
r.sendline(b'Darksaber Distillate')

print(r.recvall().decode())
```

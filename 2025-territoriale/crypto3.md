# OliCyber.IT 2025 - Selezione territoriale

## [crypto] Secure Passphrase Generator (5 risoluzioni)

Generare passphrase in maniera sicura è molto importante, ma cosa succede se te la dimentichi? Per fortuna il nostro sistema ha pensato anche a questo!

`nc spg.challs.olicyber.it 38002`

Autori: Matteo Protopapa <@matpro>, Lorenzo Demeio <@Devrar>

## Panoramica

La challenge implementa un generatore casuale di passphrase e la possibilità di recuperarla attraverso un token cifrato con AES-CBC. Il token contiene all'interno informazioni sullo username e sulle posizioni delle parole della passphrase all'interno della lista.

I primi 4 elementi della lista sono le 4 parti della flag, ma le parole vengono selezionate dall'indice 4 in poi, evitando quindi di rivelare direttamente la flag.

## Soluzione

Il parsing del token è eseguito inserendo chiave e valore all'interno di un dizionario, per cui, nel caso la stessa chiave fosse presente più volte, questa verrà sovrascritta con l'ultimo valore. Ad esempio, se provassimo a registrare uno username della forma `gabibbo;index0=0;index1=1;index2=2;index3=3` e i 4 indici generati casualmente fossero `r0`, `r1`, `r2` ed `r3`, il contenuto del token sarebbe `username=gabibbo;index0=0;index1=1;index2=2;index3=3;index0=r0;index1=r1;index2=r2;index3=r3`.

Dunque, quando questo viene decodificato, i valori di `index0, ..., index3` vengono correttamente sovrascritti con i valori `r0, ..., r3`. Per evitare che ciò accada, possiamo pensare di troncare il ciphertext che abbiamo a disposizione omettendo la parte con gli indici casuali.

Utilizzando quindi uno username della forma `gabibbo;index0=0;index1=1;index2=2;index3=3;a=bbbbbbbbb`, avremo che `username=gabibbo;index0=0;index1=1;index2=2;index3=3;a=bbbbbbbbb` ha lunghezza esattamente `64 = 4 * 16`. Troncando quindi il token a 80 bytes, 16 di IV e 64 di plaintext, il parsing ci permetterà di ottenere una passphrase composta dalle parti della flag.

L'ultimo problema da risolvere è il padding, dato che il token descritto nel paragrafo prima non ha un padding corretto e quindi la challenge darebbe un'eccezione. Per aggiustare questo basta sostituire l'ultima `b` con `\x01`, ovvero un padding lungo 1 byte.

La strategia finale consiste quindi nel registrare prima un utente con username `gabibbo`, poi uno con username `gabibbo;index0=0;index1=1;index2=2;index3=3;a=bbbbbbbb\x01` e infine troncare il token come descritto sopra e utilizzarlo per richiedere la passphrase.

## Exploit

```py
#!/usr/bin/env python3

import os
from pwn import *

HOST = os.environ.get("HOST", "spg.challs.olicyber.it")
PORT = int(os.environ.get("PORT", 38002))

payload = b"gabibbo;index0=0;index1=1;index2=2;index3=3;a=bbbbbbbb\x01"

with remote(HOST, PORT) as chall:
   chall.sendlineafter(b"> ", b"1")
   chall.sendlineafter(b"? ", b"gabibbo")
   chall.sendlineafter(b"> ", b"1")
   chall.sendlineafter(b"? ", payload)
   chall.recvline()
   token = chall.recvline().decode().split(": ")[1][:5*32]
   chall.sendlineafter(b"> ", b"2")
   chall.sendlineafter(b"? ", token.encode())
   flag = chall.recvline(False).decode().split()[-1].replace("-", "")
   print(flag)
```

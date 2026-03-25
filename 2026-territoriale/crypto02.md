# OliCyber.IT 2026 - Selezione Territoriale

## [crypto] Another One (Time Pad) (7 solves)

...and another one (time pad)

Dunque, il tutor diceva di scegliere una chiave lunga quanto il messaggio, in modo uniformemente casuale tra tutte le stringhe di quella lunghezza... dovrebbe essere tutto a posto!

`nc 10.45.1.2 10927`

Author: Francesco Felet <@PhiQuadro>

## Panoramica

Il server offre queries illimitate ad un encryption oracle il quale, dopo aver prelevato una chiave da `os.urandom()`, restituisce lo xor tra flag e chiave codificate in base64.

## Soluzione

Per ottenere perfect secrecy è necessario che la chiave sia selezionata in modo uniformemente casuale in _tutto il dominio dell'operazione_ scelta per la cifratura. In questo caso lo xor viene effettuato byte per byte, ma i byte di chiave sono caratteri base64 - un sottoinsieme molto ristretto di tutti i 256 possibili valori.

Siano `p` un carattere del testo in chiaro e `k` il corrispondente byte di chiave: noi otteniamo dall'encryption oracle `c = xor(p, k)`. Sfruttando il fatto che `k` sia un carattere base64, possiamo dedurre che le possibili preimmagini siano `p' = xor(c, k_guess)`, per ogni `k_guess` nell'alfabeto base64. Si noti che in questo insieme è sicuramente presente il carattere in chiaro originale.

Ora, chiedendo una nuova cifratura dello stesso carattere (quindi cifrato con `k` probabilmente differente) e ripetendo il ragionamento otterremo un nuovo insieme di candidati `p"`, il quale conterrà sempre il carattere in chiaro originale, ma che in generale sarà differente dall'insieme precedente. Infatti avremo (con un po' di abuso di notazione) `p' = {p xor k1 xor k_ for k_ in A}` e `p" = {p xor k2 xor k_ for k_ in A}` ove `A` è l'alfabeto dei caratteri base64.

Possiamo dunque sperare che per questo alfabeto un numero sufficientemente ampio di questi insiemi intersechino esclusivamente nel testo in chiaro originale. Possiamo verificarlo con un piccolo script:

```py
from string import ascii_lowercase, ascii_uppercase, digits

alph = ascii_lowercase + ascii_uppercase + '+/'
alph = alph.encode()

kalph = ascii_uppercase + ascii_lowercase + digits + '+/'
kalph = kalph.encode()

for c in alph:
    cands = set(list(alph))
    for k in kalph:
        c1 = set()
        for k1 in kalph:
            c1.add(c^k1^k)
        cands &= c1
    print(chr(c), cands)
```

## Exploit

```py
#!/usr/bin/env python3

import logging
import os
from pwn import process, remote
from string import ascii_lowercase, digits, ascii_uppercase
from base64 import b64decode

logging.disable()

HOST = os.environ.get("HOST", "10.45.1.2")
PORT = int(os.environ.get("PORT", 10927))

alph = (ascii_lowercase + digits + ascii_uppercase + '+/').encode()
k_alph = alph

with remote(HOST, PORT) as chall:
    poss = [set(alph) for _ in range(100)]

    lcands = [10]
    while any([lc > 1 for lc in lcands]):
        ctxt = bytes.fromhex(chall.recvline(False).decode().split()[-1])

        chall.recvuntil(b'> ')
        chall.sendline(b'yea')
        if len(poss) > len(ctxt):
            poss = poss[:len(ctxt)]

        for i,c in enumerate(ctxt):
            curr_poss = set()
            for k in k_alph:
                p = c^k
                if p in alph:
                    curr_poss.add(p)
            poss[i] &= curr_poss

        lcands = [len(x) for x in poss]

    print(b64decode(bytes([list(p)[0] for p in poss]) + b'===').decode())

```

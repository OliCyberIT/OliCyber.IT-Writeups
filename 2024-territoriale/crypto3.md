# OliCyber.IT 2024 - Selezione territoriale

## [crypto] Random Cycles (4 risoluzioni)

Sapendo la chiave è facile decifrare. Questa volta sarà più difficile.

Autore: Lorenzo Demeio <@Devrar>

## Panoramica

In questa challenge non ci viene più data la chiave per la decifratura, ma abbiamo la possibilità di fare 10 richieste scegliendo un testo cifrato, che verrà decifrato dal server. Fatto ciò, ci viene chiesto di decifrare un testo casuale per 100 volte.

## Soluzione

Il nostro obiettivo è recuperare la chiave utilizzando le 10 richieste a disposizione. Chiamiamo $n_c$ il numero casuale associato al carattere $c$ della chiave.

Dalla challenge precedente sappiamo che dallo step `i` della cifratura, il carattere in posizione `i` del plaintext rimane invariato. Quindi, scegliendo un testo con tutti caratteri distinti, otteniamo dell'informazione su $n_c$ per ogni $c$ nel testo in chiaro nel modo seguente:

- Supponiamo di inviare il testo `abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWX`.
- L'output sarà qualcosa del tipo `aNihEPVAwqnMOILXHFcvTUsjdefGJbQpWrDzklSKmuyxtgCRoB`
- Siccome dal secondo step in poi il carattere `N` non ha mai cambiato posizione, sappiamo che è arrivato alla seconda posizione al primo step.
- Dato che all'inizio era alla posizione 39, sappiamo $n_a = 11 \mod{49}$.
- Procedendo così otteniamo $n_N \mod{48}$, $n_i \mod{47}$ e così via.

Per ottenere i valori esatti degli $n_c$ combiniamo le informazioni da query diverse. Se abbiamo diverse equazioni della forma $n_a = k_1 \mod{m_1}$ e $n_a = k_2 \mod{m_2}$, usando il Teorema Cinese del Resto (TCR, o CRT) possiamo ottenere $n_a \mod{\text{lcm}(m_1, m_2)}$. Prendendo testi casuali con tutti i caratteri differenti, con probabilità alta avremo abbastanza equazioni per ricostruire ogni $n_c$ modulo qualcosa più grande di 50, ottenendo quindi il suo valore esatto.

Una volta recuperata la chiave possiamo decifrare usando la soluzione della challenge precedente.

## Exploit

```py
import os
from pwn import remote
import random
import string
import math
from sympy.ntheory.modular import crt 

HOST = os.environ.get("HOST", "localhost")
PORT = int(os.environ.get("PORT", 34001))

alph = string.ascii_letters + string.digits
n = 50

def spin(w, n):
    n = n % len(w)
    return w[-n:] + w[:-n]

def decrypt(ct, key):
    pt = ct
    for i in range(n-1, 0, -1):
        k = key[pt[i-1]]
        pt = pt[:i] + spin(pt[i:], -k)
    return pt

chall = remote(HOST, PORT)

key = {x: [(0, 1)] for x in alph}

for _ in range(10):
    payload = ''.join(random.sample(alph, n))

    chall.sendlineafter(b": ", payload.encode())
    ans = chall.recvline().decode().split(": ")[1].strip()

    tmp = payload
    for i in range(n-2):
        k = n-i-1 - tmp[i+1:].index(ans[i+1])
        key[tmp[i]].append((k, n-i-1))
        tmp = tmp[:i+1] + spin(tmp[i+1:], k)

my_key = {x: int(crt([y[1] for y in key[x]], [y[0] for y in key[x]])[0]) % int(math.lcm(*[y[1] for y in key[x]])) for x in alph}

for _ in range(100):
    ct = chall.recvline().decode().split(": ")[1].strip()

    pt = decrypt(ct, my_key)
    chall.sendlineafter(b": ", pt.encode())

print(chall.recvline().decode())
```

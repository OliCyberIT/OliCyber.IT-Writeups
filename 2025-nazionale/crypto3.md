# OliCyber.IT 2025 - Competizione nazionale

## [crypto] Cosmic rays (3 solves)

Conquisteremo lo spazio con una nuova Certificate Authority. Ma i raggi cosmici saranno un problema?

_Il timeout della challenge in remoto è di 300 secondi._

`nc cosmicrays.challs.olicyber.it 38067`

Author: Lorenzo Demeio <@Devrar>

## Panoramica

La challenge implementa una finta Certificate Authority a cui possiamo chiedere di firmare e verificare dei certificati con delle firme DSA. Tuttavia la chiave privata viene ogni volta flippata di un bit, quindi la firma che otteniamo non viene verificata dalla chiave pubblica originale.

## Soluzione

### Scoprire il nuovo bit flippato

Ogni volta che viene flippato un nuovo bit, possiamo scoprire la sua posizione e il suo valore facendo brute force sulla chiave pubblica per vedere quale verifica correttamente la firma ottenuta.

Supponiamo che la chiave privata `private_key` abbia un bit 1 alla posizione `k`. Se la chiave viene aggiornata come `private_key = private_key ^ (1 << k)`, allora la nuova chiave privata sarà identica alla precedente tranne per il bit alla posizione `k` che adesso sarà 0.

Quindi la nuova chiave pubblica corrispondente, che verifica la firma generata con la nuova chiave privata, sarà `pubkey * pow(g, -(1 << k), p) % p`.

Al contrario, se il bit a posizione `k` era originariamente 0, una volta aggiornata, la nuova chiave pubblica corrispondente sarà `pubkey * pow(g, (1 << k), p) % p`.

Possiamo quindi scoprire quale bit è stato flippato e quale era il suo valore brutando la posizione `k` e guardando se la firma verifica sotto la chiave pubblica `pubkey * pow(g, -(1 << k), p) % p` o `pubkey * pow(g, (1 << k), p) % p`

Dato che la chiave privata rimane flippata, dobbiamo tenere anche la chiave pubblica aggiornata con quella che verifica la firma correttamente.

```python
for i in range(160):
    if verify_certificate(certificate, r, s, cur_pubkey*pow(g, (1 << i), p) % p):
        if i in idxs:
            idxs.remove(i)
        cur_pubkey = cur_pubkey*pow(g, (1 << i), p) % p
        break
    elif verify_certificate(certificate, r, s, cur_pubkey*pow(g, -(1 << i), p) % p):
        if i in idxs:
            guess_x |= 1 << i
            idxs.remove(i)
        cur_pubkey = cur_pubkey*pow(g, -(1 << i), p) % p
        break
```

### Ottenere gli ultimi bit

Dato che abbiamo a disposizione solo 500 query e con alta probabilità vengono flippati spesso gli stessi bit, una volta finite le query rimarranno dei bit da indovinare. In media il numero di questi bit è abbastanza basso per cui è possibile implementare un attacco brute force usando come check quando il nostro guess ci dà la chiave pubblica originale.

```python
for bits in range(2**15):
    tmp_x = guess_x
    for idx in idxs:
        tmp_x |= (bits & 1) << idx
        bits >>= 1
    if pow(g, tmp_x, p) == pubkey:
        x = tmp_x
        break
```

## Exploit

```python
#!/usr/bin/env python3

import logging
import os
from pwn import process, remote
from secrets import randbelow
from hashlib import sha256
import time

HOST = os.environ.get("HOST", "todo.challs.todo.it")
PORT = int(os.environ.get("PORT", 34001))

p = 89884656743115796742429711405763364460177151692783429800884652449310979263752253529349195459823881715145796498046459238345428121561386626945679753956400077352882071663925459750500807018254028771490434021315691357123734637046894876123496168716251735252662742462099334802433058472377674408598573487858308054417
q = 1193447034984784682329306571139467195163334221569
g = pow(2, (p-1)//q, p)

def h(m):
    return int.from_bytes(sha256(m).digest()[:20], "big")

def create_certificate(name, domain, private_key):
    certificate = f"The Space CA certifies that {name} owns {domain}"
    k = randbelow(q)
    r = pow(g, k, p) % q
    s = pow(k, -1, q)*(h(certificate.encode()) + r*private_key) % q

    return certificate, (r,s)

def verify_certificate(certificate, r, s, pubkey):
    w = pow(s, -1, q)
    u1 = (w*h(certificate.encode())) % q
    u2 = (r*w) % q
    return r == (pow(g, u1, p)*pow(pubkey, u2, p) % p) % q

with remote(HOST, PORT) as chall:
    chall.recvline()
    pubkey = int(chall.recvline().decode().split(": ")[1])
    guess_x = 0
    name = "Mario"
    domain = "ciao.ciao"
    certificate = f"The Space CA certifies that {name} owns {domain}"
    cur_pubkey = pubkey
    count = 0

    idxs = list(range(160))
    while len(idxs) > 15:
        count += 1
        chall.sendlineafter(b"> ", b"1")
        chall.sendlineafter(b"name: ", name.encode())
        chall.sendlineafter(b"domain: ", domain.encode())
        chall.recvline().decode()
        r,s = eval(chall.recvline().decode().split(": ")[1])
        for i in range(160):
            if verify_certificate(certificate, r, s, cur_pubkey*pow(g, (1 << i), p) % p):
                if i in idxs:
                    idxs.remove(i)
                cur_pubkey = cur_pubkey*pow(g, (1 << i), p) % p
                break
            elif verify_certificate(certificate, r, s, cur_pubkey*pow(g, -(1 << i), p) % p):
                if i in idxs:
                    guess_x |= 1 << i
                    idxs.remove(i)
                cur_pubkey = cur_pubkey*pow(g, -(1 << i), p) % p
                break

    for bits in range(2**15):
        tmp_x = guess_x
        for idx in idxs:
            tmp_x |= (bits & 1) << idx
            bits >>= 1
        if pow(g, tmp_x, p) == pubkey:
            x = tmp_x
            break
    else:
        print("Nope")
        exit()

    cert, (r, s) = create_certificate("Mario", "space.ca", x)
    chall.sendlineafter(b"> ", b"2")
    chall.recvline()
    chall.sendline(cert.encode())
    chall.sendlineafter(b"r: ", str(r).encode())
    chall.sendlineafter(b"s: ", str(s).encode())
    print(chall.recvline(False).decode())

```

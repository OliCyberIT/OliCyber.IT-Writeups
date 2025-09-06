# OliCyber.IT 2024 - Finale Nazionale

## [crypto] Choose your OTP (11 risoluzioni)

Il futuro è già qua: prova il nostro sistema rivoluzionario!

`nc chooseyourotp.challs.olicyber.it 38302`

Author: Matteo Protopapa <@matpro>

## Panoramica

La sfida chiede semplicemente un intero $n$, poi calcola un intero casuale $0\le r \le n$ e calcola lo XOR tra $r$ e la flag. Il risultato viene poi troncato alla lunghezza in bit di $n$.

## Soluzione

Dato che abbiamo completa libertà su $n$, possiamo inviare $n = 2^k$, per valori crescenti di $k$. In questo modo, $n$ ha $k+1$ bit, ma il più significativo è quasi sempre 0. Interrogare il server alcune volte ci dà abbastanza fiducia sul bit $k$-esimo (con il più a sinistra che è lo $0$-esimo), eccetto per i primi 2 bit, che possono essere indovinati poiché la flag termina con `}`.

## Exploit

```python
#!/usr/bin/env python3

import logging
import os
from pwn import remote

logging.disable()

HOST = os.environ.get("HOST", "chooseyourotp.challs.olicyber.it")
PORT = int(os.environ.get("PORT", 1337))

flag = b'\x01'
bits = '01'

r = remote(HOST, PORT)

k = 2
while not flag.startswith(b'flag'):
    recovered = [0, 0]

    for i in range(10):
        inp = 2**k
        r.sendlineafter(b'> ', str(inp).encode())
        recovered_bit = bin(int(r.recvline(False).decode()))[2:].zfill(k+1)
        recovered_bit = int(recovered_bit[0])
        recovered[recovered_bit] += 1

    if recovered[0] > recovered[1]:
        bits = '0' + bits
    else:
        bits = '1' + bits

    flag = int.to_bytes(int(''.join(bits), 2), k//8 + 1, 'big')
    # print(flag)
    k += 1

print(flag.decode())
```
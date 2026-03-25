# OliCyber.IT 2026 - Selezione Territoriale

## [crypto] ECB-MAC (5 solves)

> Look, if you had one shot or one opportunity</br>
> To seize everything you ever wanted in one moment</br>
> Would you capture it or just let it slip?

`nc ecb-mac.challs.olicyber.it 38082`

Author: Lorenzo Demeio <@Devrar>

## Panoramica

La challenge implementa un tentativo di MAC utilizzando la modalità ECB di AES per messaggi di lunghezza massima 16 blocchi. Ogni blocco viene ruotato a sinistra di un numero di bytes pari all'indice del blocco e poi cifrato con AES. I ciphertext vengono a loro volta ruotati e poi xorati assieme per ottenere il tag finale.

Per ottenere la flag dobbiamo riuscire a ricavarci il tag di un messaggio target da 16 blocchi.

## Soluzione

Abbiamo a disposizione una singola query in cui possiamo chiedere il tag di un messaggio. Chiamiamo $B_0, \ldots, B_{15}$ i blocchi del messaggio target. Il risultato delle rotazioni e della cifratura del blocco $B_i$ sono:

$$
C_i = rotL(Enc(rotL(B_i, i)), i).
$$

Notiamo che, se posizioniamo il blocco $B_i$ alla posizione $i-1$, quello che otteniamo è

$$
rotL(Enc(rotL(B_i, i-1)), i-1),
$$

mentre il blocco $B_0$, se posizionato all'indice $15$, diventa

$$
rotL(Enc(rotL(B_0, 15)), 15).
$$

Quindi, se invece di $B_i$ mettiamo il blocco $rotL(B_i, 1)$ in posizione $i-1$, otteniamo

$$
D_i = rotL(Enc(rotL(rotL(B_i, 1), i-1)), i-1) = rotL(Enc(rotL(B_i), i), i-1) = rotR(C_i, 1).
$$

Facendo lo xor di tutti i $D_i$, che è il risultato del tag, si ottiene quindi lo xor di tutti i $C_i$ ruotato a destra di 1. Basta quindi prendere il tag inviatoci dal server e ruotarlo a sinistra di 1 per ottenere il tag del messaggio target.

## Exploit

```py
from pwn import remote
import os
from ecb-mac import target_msg

HOST = os.environ.get("HOST", "ecb-mac.challs.olicyber.it")
PORT = int(os.environ.get("PORT", 38082))

def rotate_left(block, i):
    return block[i:] + block[:i]

blocks = [target_msg[i:i+16] for i in range(0, len(target_msg), 16)]

payload = [rotate_left(blocks[i], 1) for i in range(1, 16)] + [rotate_left(blocks[0], 1)]

with remote(HOST, PORT) as chall:
    chall.sendlineafter(b": ", b"".join(payload).hex().encode())
    mac =  bytes.fromhex(chall.recvline().strip().split(b": ")[1].decode())
    chall.sendlineafter(b": ", rotate_left(mac, 1).hex().encode())
    print(chall.recvline().decode())

```

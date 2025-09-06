# OliCyber.IT 2025 - Competizione nazionale

## [crypto] Colors (89 solves)

Perché usare la matematica quando possiamo mescolare i colori?

Author: Francesco Felet <@PhiQuadro>

## Panoramica

In questa challenge viene effettuato uno scambio di chiavi di Diffie-Hellman, ma anziché le usuali operazioni su $F_p$ si stanno "mescolando dei colori" (4 byte RGBA) tramite somma modulo 256 componente per componente.

## Soluzione

Per risolvere la challenge è sufficiente estrarre i colori dalle immagini fornite (con un color picker, o direttamente da python usando pillow) e fare la differenza modulo 256 componente per componente tra una chiave pubblica e il generatore per ricavare la chiave privata corrispondente. A quel punto basta mescolare la chiave privata ottenuta con l'altra chiave pubblica per ottenere il segreto condiviso con cui decifrare la flag.

In alternativa, dato che le chiavi utilizzate sono lunghe solo 4 byte, è possibile effettuare una ricerca esaustiva direttamente sul cifrato, anche se chiaramente più oneroso rispetto alla soluzione precedente.

## Exploit

```py
#!/usr/bin/env python3
from PIL import Image
from hashlib import sha256
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad

def get_col(file):
    with Image.open(file) as im:
        w, h = im.width, im.height
        for i in range(w):
            for j in range(h):
                pix = im.getpixel((i,j))
                if pix != (255,255,255,255):
                    return pix
    return (255, 255, 255, 255)

def mix(col1, col2):
    return [(c1 + c2) % 256 for c1, c2 in zip(col1, col2)]

with open('ciphertext.txt', 'r') as rf:
    ctxt = bytes.fromhex(rf.read().strip())
    iv, ctxt = ctxt[:16], ctxt[16:]

g, A, B = [get_col(f'{x}.png') for x in 'gAB']
a = [(c1 - c2) % 256 for c1, c2 in zip(A, g)]

shared = mix(B, a)

cipher = AES.new(sha256(bytes(shared)).digest(), AES.MODE_CBC, iv)
print(unpad(cipher.decrypt(ctxt), 16).decode())
```

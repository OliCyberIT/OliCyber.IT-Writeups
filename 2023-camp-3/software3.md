# OliCyber.IT 2023 - Simulazione training camp 3

## [binary] baby-encryption (15 risoluzioni)

La challenge controlla se la stringa inserita corrisponda alla flag, attraverso delle operazioni eseguite sull'input.

In particolare fa 16 volte xor della stringa inserita con una chiave, che ad ogni iterazione viene ruotata di un byte.

### Soluzione

Basta eseguire nuovamente lo xor con le stesse chiavi per riottenere la flag originale dal target definito nel programma

### Exploit

```py
#!/bin/python3

from pwn import xor

i = b"\x27\x2d\x20\x26\x3a\x36\x29\x70\x2d\x72\x70\x1e\x39\x71\x33\x3c"
const = "\xf8\x6f\x86\x83\xc3\x9c\x8b\x35\xf0\xc0\x87\x92\x2e\x41\x2b\x53"

def rot_key():
    global const
    const = const[15] + const[:15]

for _ in range(16):
    i = xor(i, const)
    rot_key()

print(i)

```

# OliCyber.IT 2025 - Competizione nazionale

## [rev] sodiiiiiium (84 solves)

meglio non toccarlo altrimenti esploderà, conosci la formula?

Author: Andrea Raineri <@Rising>

## Panoramica

La challenge ci fornisce un eseguibile che, una volta avviato, ci chiede subito di inserire una formula segreta.
Provando con alcune stringhe di input brevi, riceviamo come risposta che devono essere inseriti più dati.
Aprendo l'eseguibile con Ghidra vediamo che il programma si aspetta una stringa di 64 byte. Dopo che il nostro input viene ricevuto, i byte in posizione pari vengono xorati con il byte `0x20`, dopodiché l'intera stringa viene xorata con il risultato dell'hash `sha512` calcolato su una stringa contenuta all'interno dell’ELF (`tung tung tung sahur VS cappuccino assassino`).
Dopo queste operazioni, il risultato viene confrontato con una sequenza di byte presente nella sezione dati dell’ELF.

Le funzioni per le operazioni crittografiche e la gestione dei buffer di memoria fanno parte della libreria C libsodium, che è linkata staticamente all’interno del binario, ma nessun simbolo di funzione è stato rimosso dall’eseguibile.

## Soluzione

La flag è l'input atteso dall'eseguibile, che stamperà un messaggio di successo se, dopo le trasformazioni applicate, corrisponde alla sequenza di byte memorizzata all’interno. Per calcolare la flag corretta possiamo invertire tutte le operazioni.

## Exploit

```python
from hashlib import sha512

ENC_FLAG = b'\x2c\xe4\x19\x0b\xdf\x7c\x92\x03\x01\x21\x7b\x56\xb1\x1f\x06\x7f\xd3\x61\xbb\x2f\x11\xb0\x4e\x7b\x9a\x75\x91\x2b\x16\xad\xb5\xaf\x96\xe0\x0e\xd7\xec\xbc\xb1\xfc\x5c\xf8\x08\x37\xac\x4f\x40\xd1\x81\xc0\xcd\xf9\xb8\x16\x7c\xe2\xf2\x87\xfc\x69\x3a\x99\x30\xf2'
KEY = b"tung tung tung sahur VS cappuccino assassino"

flag = bytearray([x ^ y for x, y in zip(ENC_FLAG, sha512(KEY).digest())])
for i in range(0, len(flag), 2):
    flag[i] ^= 0x20

print(flag.decode())
```

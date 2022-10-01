# OliCyber.IT 2021 - Competizione nazionale

## [misc-1] Il test di ammissione (60 risoluzioni)

La challenge richiede di settare a 5 tutti i contatori forniti. Ogni contatore varia da 1 a 5 in maniera sequenziale (il successivo di 5 è 1). I comandi che abbiamo a disposizione possono influenzare più di un contatore alla volta; è quindi necessario scegliere la sequenza corretta per arrivare alla configurazione desiderata.

### Soluzione

E' immediato dedurre dalle istruzioni del gioco che fare una mossa 5 volte equivale a non farla affatto. Interagendo con il servizio diverse volte si può notare che:

- Mosse diverse non contengono mai lo stesso contatore.
- Ogni mossa contiene solo contatori che partono dallo stesso valore.

Di conseguenza, ci basterà eseguire ciascuna delle mosse un numero di volte tale da fare arrivare i corrispondenti contatori a 5.

### Exploit

Di seguito uno script python che interagisce con il serivizo implementando questa soluzione:

```python
#!/bin/env python3
from pwn import remote

def solve(stato, mosse):
    resp = ""
    for i in range(len(mosse)):
        for n in mosse[i]:
            while(stato[n] < 5):
                resp += f"{i+1} "
                for j in mosse[i]:
                    stato[j] += 1

    return resp[:-1]

r = remote("test1.challs.olicyber.it", 15004)
r.recvlines(20)

livello = r.recvline()
while livello.startswith(b"Livello"):
    stato = [int(_) for _ in r.recvline(False).decode().split()]
    mosse = []
    while True:
        s = r.recvline(False).decode()
        if s == "":
            break
        mosse.append(["ABCDEFGHIJKLMNOPQRSTUVWXYZ".index(_) for _ in s.split()])
    res = solve(stato, mosse)
    r.sendline(res.encode())
    r.recvlines(2)
    livello = r.recvline(False)

print(livello.decode())
```

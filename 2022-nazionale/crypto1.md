# OliCyber.IT 2022 - Competizione nazionale

## [crypto-1] OTP-as-a-Service 1 (45 risoluzioni)

`Ho letto che "one time pad" è il cifrario più sicuro che esiste, e ho creato un prodotto che lo utilizza!`

La challenge cifra la flag utilizzando una specie di one time pad con numeri casuali.

### Soluzione

Osserviamo che, detto `pt[i]` l'i-esimo carattere del plaintext (ovvero della flag) e `ct[i]` l'i-esimo carattere del ciphertext, la funzione di encryption è `ct[i] = (pt[i]+r[i])%256`, dove `r[i]` è un numero casuale tra 0 e 255.

Questo è una variante di one time pad, ma ugualmente sicura se i valori `r[i]` fossero davvero random e indipendenti. Il punto centrale della challenge è che i valori `r[i]` vengono generati da un PRNG (il modulo `random` di Python) di cui conosciamo il seed grazie alle righe

```python
t = int(time.time())
random.seed(t)
```

Dato che sappiamo il momento in cui iniziamo la connessione al server, possiamo ricavare anche localmente il tempo `t` e seedare `random` con quel valore per genrare gli stessi valori `r[i]` del server e decifrare la flag con la formula `pt[i] = (ct[i]-r[i])%256`.

### Exploit

```python
from pwn import remote
import time
import random

r = remote("otp1.challs.olicyber.it", 12304)
t = int(time.time())
random.seed(t)
r.recvlines(4)
r.sendline("e")
data = r.recvline(False).decode()
data = list(map(int, data.split("-")))

flag = []
for c in data:
    rnd = random.randint(0, 255)
    m = (c-rnd) % 256
    flag.append(m)
print(bytes(flag))
```

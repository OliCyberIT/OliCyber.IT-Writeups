# OliCyber.IT 2022 - Competizione territoriale

## [crypto-3] babyoracle (7 risoluzioni)

`Have you ever thought about what can be done with a simple oracle?`

La challenge crea un modulo RSA $n=p\cdot q$ e cifra la flag con $e=65537$; il server è poi una sorta di "half decryption oracle", ovvero risponde con $c^{d_p}\pmod p$.

### Soluzione

Il server calcola $d_p\equiv e^{-1}\pmod {p-1}$ come in [RSA-CRT](https://en.wikipedia.org/wiki/RSA_%28cryptosystem%29#Using_the_Chinese_remainder_algorithm); l'equazione fondamentale che fa funzionare questa variante di RSA è $$m^{e\cdot d_p}\equiv m\pmod p.$$

Nell'equazione precedente abbiamo due valori sconosciuti: $p$ e $d_p$; tuttavia l'oracle ci permette proprio di fare operazioni modulo $p$ usando $d_p$ come esponente: inviando $2$ al server, questo risponde $c\equiv 2^{d_p}\pmod p$.

Sostituendo nell'equazione fondamentale vediamo che allora $c^e\equiv 2^{d_p\cdot e}\equiv 2\pmod p$, perciò $p\mid c^e-2$; in quest'ultima espressione conosciamo sia $c$ che $e=65537$, quindi calcolandolo esplicitamente (come intero!) sappiamo che $p$ è un divisore di questo numero.

Infine, poiché $p\mid n$ per costruzione, sappiamo anche che $p\mid \gcd(c^e-2, n)$, e anzi molto probabilmente varrà proprio l'uguaglianza. Quindi facendo questo calcolo (un po' lungo) ricaviamo $p$ e possiamo poi decifrare RSA.

### Exploit

```python
from pwn import *
from Crypto.Util.number import long_to_bytes, GCD, inverse

r = remote("babyoracle.challs.olicyber.it", 12007)

n = int(r.recvline(False).split()[-1])
flag = int(r.recvline(False).split()[-1])
r.recvuntil(": ")
r.sendline("2")
c = int(r.recvline(False))
p = GCD(c**65537 - 2, n)

assert n % p == 0

q = n//p
phi = (p-1)*(q-1)
d = inverse(65537, phi)
print(long_to_bytes(pow(flag, d, n)))
```

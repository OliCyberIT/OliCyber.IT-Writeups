# OliCyber.IT 2021 - Competizione nazionale

## [crypto-2] Small bits of RSA (33 risoluzioni)

```
Ho inventato un modo molto più rapido per generare dei primi casuali!

Vuoi provare a controllare se il metodo è sicuro?
```

Lo script fornito genera due primi e cifra la flag con RSA.

La generazione dei primi avviene come segue:

```python
def gen_prime(nbits):
    while True:
        x = random.randint(1, nbits-1)
        y = random.randint(1, nbits-1)
        p = 1 + 2**x + 2**y + 2**nbits
        if isPrime(p):
            return p
```

### Soluzione

Possiamo notare come i primi generati abbiano al massimo 4 bit a 1, quindi è possibile enumerarli tutti e provare quale dei candidati generati divide il modulo `n`.

A questo punto, conoscendo i primi `p` e `q`, è possibile decifrare il messaggio cifrato con RSA.

### Exploit

```python
from Crypto.Util.number import long_to_bytes, inverse

def bruteforce(n):
    for x in range(1, 1024):
        for y in range(1, 1024):
            p = 1 + 2**x + 2**y + 2**1024
            if n % p == 0:
                return p

n = ...
ct = ...

p = bruteforce(n)
q = n//p
phi = (p-1)*(q-1)
d = inverse(65537, phi)
msg = pow(ct, d, n)
print(long_to_bytes(msg))
```

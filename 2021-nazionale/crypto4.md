# OliCyber.IT 2021 - Competizione nazionale

## [crypto-4] Large bits of RSA (0 risoluzioni)

```
Ho migliorato il mio algoritmo di generazione dei primi, ora ha davvero sicurezza massima!
```

La challenge è molto simile a `crypto-2 Small bits of RSA`, con l'unica differenza nella generazione dei primi, che diventa la seguente.

```python
def gen_prime(nbits):
    while True:
        p = 2**(nbits+1) - 1
        for i in range(30):
            x = random.randint(2, nbits-1)
            p -= 2**x
        if isPrime(p):
            return p
```

### Soluzione

Osserviamo che la generazione dei primi avviene in modo che `p` e `q` abbiano molti uni e 30 zeri quando scritti in base 2; cerchiamo allora di sfruttare questa proprietà per fattorizzare `n` con un algoritmo "probabilistico".

In particolare costruiamo p e q bit per bit, tramite una successione tale che $p_k\cdot q_k\equiv n\pmod{2^k}$, con $k$ crescente. Dato che ci sono tante possibili soluzioni, esploriamo per prime quelle in cui il numero di zeri è minore.

Infine si prova quale delle coppie restituite corrisponde effettivamente alla fattorizzazione di n, e si conclude decifrando RSA.

### Exploit

```python
def order(candidates):
    res = []
    for p, q in candidates:
        if bin(p).count('0') <= 31 and bin(q).count('0') <= 31:
            res.append((p, q))
    return sorted(res, key=lambda x: bin(x[0]).count('0') + bin(x[1]).count('0'))[:1000]


def recover(n, bits):
    candidates = {(0, 0)}
    for bit in range(0, bits):
        if bit%50==0:
            print(bit, len(candidates))
        remainder = n % (2 ** (bit + 1))
        new_candidates = set()
        for potential_p, potential_q in candidates:
            # extend by 0 bit and by 1 bit on the left, so candidate 101 -> 0101 and 1101
            extended_p = potential_p, (1 << bit) + potential_p
            extended_q = potential_q, (1 << bit) + potential_q
            for p in extended_p:
                for q in extended_q:
                    if (q, p) not in new_candidates: # p,q and q,p is the same for us
                        if (p * q) % (2 ** (bit + 1)) == remainder:
                            new_candidates.add((p, q))
        candidates = order(list(new_candidates))
    return candidates


candidates = recover(n, 1025)
```

# OliCyber.IT 2022 - Competizione territoriale

## [crypto-2] The encrypted line (16 risoluzioni)

`Another day has passed and I still haven't used y=mx+q`

La challenge cifra la flag utilizzando un cifrario affine in cui la chiave è il moltiplicatore.

### Soluzione

Se $m_i$ è il valore del'i-esimo byte del messaggio, $c_i$ è l'i-esimo byte del cifrato e $k$ è il valore della chiave, l'equazione di cifratura è $$c_i\equiv k\cdot m_i+1\pmod{256}$$

Possiamo recuperare la flag in due modi: il primo è facendo un bruteforce sulla chiave. Nel sorgente è generata come un intero a 128 bit, ma possiamo osservare che quello che interessa davvero è il valore di $k\mod 256$, e possiamo provarli tutti decifrando con la formula inversa $$m_i\equiv k^{-1}(c_i-1)\pmod{256},$$ dove $k^{-1}$ è l'inverso moltiplicativo di $k$ modulo $256$.

Tuttavia, poiché conosciamo una parte della flag, in particolare l'inizio `flag{`, possiamo recuperare il valore preciso di $k$ con l'equazione $$k\equiv m_i^{-1}(c_i-1)\pmod{256},$$ facendo solo attenzione ad utilizzare un $m_i$ dispari, come per esempio `a`. Una volta ottenuto il valore di $k$, decifriamo il resto della flag con la formula precedente.

### Exploit

```python
from Crypto.Util.number import inverse

data = bytes.fromhex("7f1d26c428628a1dd436311d36b0054536f1a79c62f1f59c7b4a8ffc9ca7f19c31bbf16b1dc062e6b2")

k = ((data[2]-1) * inverse(ord("a"), 256)) % 256
dec = lambda x: ((x-1) * inverse(k, 256)) % 256
flag = bytes(map(dec, data))
print(flag)
```

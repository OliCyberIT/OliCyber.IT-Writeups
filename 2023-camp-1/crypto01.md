# OliCyber.IT 2023 - Simulazione training camp 1

## [crypto] Cifrario a caso (42 risoluzioni)

La challenge implementa un cifrario che, per ogni byte della flag, ne fa lo xor con un byte generato casualmente.

### Soluzione

La vulnerabilità del cifrario consiste nel fatto che il seed del generatore casuale viene inizializzato ogni volta con il precedente carattere della flag. Dato che sappiamo che il primo carattere è una `f`, possiamo procedere a decifrare un carattere alla volta:

1. Inizializiamo il seed a `ord(f)`
2. Generiamo un valore casuale con `random.randint(0, 255)`
3. Ne facciamo lo xor con il carattere successivo del ciphertext così da ottenere il corrispondente carattere della flag
4. Procediamo allo stesso modo fino ad aver ottenuto tutta la flag.

### Exploit

```python
import random

encrypted_flag = bytes.fromhex("ce272304927c2776eeb675d622fafaa1fe31c50e2434149922ff44394ffb4a12fe75d622fafaa1dc")

flag = b"f"

for c in encrypted_flag[1:]:
    random.seed(flag[-1])
    key = random.randint(0, 255)
    flag += bytes([key ^ c])

print(flag.decode())
```

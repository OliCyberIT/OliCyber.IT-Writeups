# OliCyber.IT 2024 - Selezione territoriale

## [crypto] ARX designer (59 risoluzioni)

Ho appena creato il mio nuovo cifrario ARX, ma credo sia solo una questione di tempo prima che venga rotto.

Autore: Matteo Protopapa <@matpro>

## Overview

La challenge ci presenta due file: uno script in Python contenente il codice sorgente e il suo output, contenente un intero `n` e una stringa esadecimale `enc`. Lo script in Python genera una chiave e implementa un cifrario ARX utilizzato per cifrare la flag. L'obiettivo è quindi decifrare `enc`.

## Soluzione

Possiamo notare che l'output dipende da due valori casuali: `n` e `seed`, generati dal codice seguente:

```python
    n = getrandbits(2048)
    print(f'{n = }')

    # integer between 0 and 999999, I hope it is enough
    seed  = datetime.now().microsecond
```

La chiave per il cifrario viene quindi ottenuta come:

```python
    key = int.to_bytes(prng(n, seed, 10)[-1], 2048//8, 'big')[16:16+32]
```

Infine, la flag è cifrata con il cifrario ARX implementato.

Osserviamo che, siccome conosciamo `n` e `seed` è compreso tra 0 e 999999 (come suggerito dal commento), possiamo facilmente trovare la chiave tramite forza bruta, provando a decifrare il testo cifrato e controllando se il testo ottenuto ha il formato corretto. L'unica cosa che ci manca da invertire è la `encrypt_block`.

Il codice della funzione è il seguente:

```python
def encrypt_block(key, block):
    assert len(key) == 32
    assert len(block) == 32
    k2, k1 = key[:16], key[16:32]
    b1, b2 = block[:16], block[16:]
    for r in range(ROUNDS):
        b2 = add(b1, b2)
        b2 = xor(b2, k2)
        b1 = ror(b1, 31)
        b1 = xor(b1, k1)
    return b1 + b2
```

Possiamo quindi scriverla come $b_2' = (b_1 \boxplus b_2) \oplus k_2$ e $b_1' = (b_1 \ggg 31) \oplus k_1$. E la sua inversa sarà $b_1 = (b_1' \oplus k_1) \lll 31$ e $b_2 = (b_2' \oplus k_2) \boxminus b_1$, ottenendo quindi:

```python
def decrypt_block(key, block):
    b1, b2 = block[:16], block[16:]
    k2, k1 = key[:16], key[16:+32]
    for r in range(ROUNDS):
        b1 = rol(xor(b1, k1), 31)
        b2 = sub(xor(b2, k2), b1)
    return b1 + b2 
```

Tutto il resto rimane invariato.

## Exploit

```python
from Crypto.Cipher import ChaCha20
from Crypto.Util.Padding import unpad
from tqdm import trange


ROUNDS = 10
n = 8262967280909911491328654566023625758886458617653507183987700617770641147830488629236634563223071245462280478892453079554565508411327556741361890830991319503597213460010729370349970007579961675488844488868966072564991017420527832883580322108519760131927604286295836482454989408386328685365113345609652418571675432249303845483261697702539654643632145811439060181860197047202705373076290373160502611875747185544118905115309216237145433696560653302247663804590123782307652944890194094392612783025638614067821276200119864947028720062303007979009932430186077093744948009754854461881400774037689979300695784993451011030039
enc = bytes.fromhex("aedcc584cf9bc81e41d8a87b32144a81ed650ef3716acbfd9953d92e10dd4430")


def xor(a, b):
    return bytes([x^y for x,y in zip(a,b)])


def add(a, b):
    return int.to_bytes((int.from_bytes(a, 'big') + int.from_bytes(b, 'big')) & ((1 << 128) - 1), 16, 'big')


def sub(a, b):
    return int.to_bytes((int.from_bytes(a, 'big') - int.from_bytes(b, 'big')) % (1 << 128), 16, 'big')


def rol(x, n):
   return int.to_bytes(((int.from_bytes(x, 'big') << n) | (int.from_bytes(x, 'big') >> (128 - n))) & ((1 << 128) -1), 16, 'big')


def ror(x, n):
   return rol(x, 128-n)


def prng(n, seed, iterations):
  numbers = []
  for _ in range(iterations):
    seed = (seed ** 2) % n
    numbers.append(seed)
  return numbers


def decrypt_block(key, block):
    b1, b2 = block[:16], block[16:]
    k2, k1 = key[:16], key[16:+32]
    for r in range(ROUNDS):
        b1 = rol(xor(b1, k1), 31)
        b2 = sub(xor(b2, k2), b1)
    return b1 + b2   


def decrypt(key, enc):
    msg = b''
    for i in range(len(enc) // 32):
       msg += decrypt_block(key, enc[32*i:32*i+32])
    return msg




for seed in trange(10**6):
    key = int.to_bytes(prng(n, seed, 10)[-1], 2048//8, 'big')[16:16+32*ROUNDS]
    msg = decrypt(key, enc[:32])
    if msg.startswith(b'flag{'):
        print(seed, unpad(decrypt(key, enc), 32))
        break
```

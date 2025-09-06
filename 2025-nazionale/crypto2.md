# OliCyber.IT 2025 - Competizione nazionale

## [crypto] Party Mode (7 solves)

Sono stanco del solito AES-CBC.

_Il timeout della challenge in remoto è di 300 secondi._

`nc partymode.challs.olicyber.it 38063`

Author: Matteo Rossi <@mr96>

## Panoramica

La challenge implementa un cifrario che concatena 3 modalità di funzionamento diverse: CFB, CBC e OFB. Il cifrario utilizzato è Blowfish, ma ai fini della soluzione non è necessario conoscere il suo funzionamento interno.

Abbiamo a disposizione la possibilità di cifrare e decifrare messaggi a piacere, specificando anche i 3 IV da utilizzare in caso di decifratura.

Per ottenere la flag dobbiamo recuperare la chiave da 60 bit generata in partenza.

## Soluzione

### Schema del cifrario

Il cifrario è in realtà uno stream cipher, dove lo stream è generato cifrando byte nulli con le 3 modalità descritte sopra consecutivamente. Inoltre la chiave originale da 60 bit viene divisa in 3 chiavi `key1, key2, key3`, una per modalità:

```python
c1 = Blowfish.new(int.to_bytes(key1, 4, 'big'), Blowfish.MODE_CFB, ciphertext[:8])
c2 = Blowfish.new(int.to_bytes(key2, 4, 'big'), Blowfish.MODE_CBC, ciphertext[8:16])
c3 = Blowfish.new(int.to_bytes(key3, 4, 'big'), Blowfish.MODE_OFB, ciphertext[16:24])
```

L'idea è quella di recuperare una chiave alla volta, partendo da `key3` e sfruttando il fatto che possiamo selezionare l'IV da utilizzare nella decryption. Chiamiamo `x1, x2, x3` gli output rispettivamente dopo le cifrature con `c1, c2, c3` rispettivamente. In particolare, chiedendo l'encryption di un plaintext `pt`, il ciphertext che otteniamo è `ct = x3 ^ pt`.

### Recuperare key3

Chiediamo come prima cosa l'encryption del plaintext `pt = bytes(8)` (ignoriamo il blocco di padding aggiunto) e otteniamo gli `ivs` e il ciphertext `ct = x3`. Per come funziona la cifratura con OFB, abbiamo che

$$
ct = Enc(\texttt{key3}, \texttt{ivs}[2]) \oplus x2 \oplus pt = Enc(\texttt{key3}, \texttt{ivs}[2]) \oplus x2.
$$

dove $Enc$ indica l'encryption con Blowfish in modalità ECB.

Chiediamo ora la decryption di `ct` con i primi due `iv` uguali ma il terzo modificato: `my_ivs = ivs[:2] + [bytes(8)]`. L'encryption con le prime due modes of operation, CFB e CBC, sarà quindi uguale; l'output `x3'` di OFB sarà:

$`
x3' = Enc(\texttt{key3}, \texttt{my\_ivs}[2]) \oplus x2
`$

L'output dato dalla decryption sarà quindi

$`
pt1 = x3' \oplus ct =  Enc(\texttt{key3}, \texttt{my\_ivs}[2]) \oplus x2 \oplus ct.
`$

Sostituendo a `ct` il risultato sopra, abbiamo

$`
pt1 = Enc(\texttt{key3}, \texttt{my\_ivs}[2]) \oplus x2 \oplus ct = Enc(\texttt{key3}, \texttt{my\_ivs}[2]) \oplus Enc(\texttt{key3}, \texttt{ivs}[2]).
`$

Possiamo quindi utilizzare questa relazione per brutare `key3`:

```python
def brute_ofb(iv1, iv2, pt1, start=0):
    for k in trange(start, 2**20):
        cipher = Blowfish.new(k.to_bytes(4, "big"), Blowfish.MODE_ECB)
        if xor(cipher.encrypt(iv1), cipher.encrypt(iv2)) == pt1:
            print("FOUND:", k)
            return k
```

### Recuperare key2

Il procedimento per recuperare `key2` è analogo a quello per `key3`. In questo caso, conoscendo già `key3`, possiamo decifrare con il cifrario `c3` e ottenere l'output dopo l'encryption con CBC. Chiamiamo `enc1` ed `enc2` i due output dopo l'encryption con CBC e `iv1, iv2` i due IV utilizzati per CBC in encryption e decryption; dato il funzionamento di CBC, la condizione per il bruteforce di `key2` sarà:

$$
Dec(\texttt{key2}, enc1) \oplus iv1 = Dec(\texttt{key2}, enc2) \oplus iv2.
$$

```python
def brute_cbc(iv1, iv2, enc1, enc2, start=0):
    for k in trange(start, 2**20):
        cipher = Blowfish.new(k.to_bytes(4, "big"), Blowfish.MODE_ECB)
        if xor(cipher.decrypt(enc1), iv1) == xor(cipher.decrypt(enc2), iv2):
            print("FOUND:", k)
            return k
```

### Recuperare key1

Una volta ottenute `key3` e `key2`, per ottenere `key1` è sufficiente una coppia plaintext/ciphertext e verificare quale chiave decifra correttamente:

```python
def brute_cfb(iv, enc, start=0):
    for k in trange(start, 2**20):
        cipher = Blowfish.new(k.to_bytes(4, "big"), Blowfish.MODE_CFB, iv)
        if cipher.decrypt(enc) == pt:
            print("FOUND:", k)
            return k
```

### Ricostruire la chiave completa

Avendo le tre chiavi, la chiave completa `key` è semplicemente data da:

```python
key = (key3 << 40) + (key2 << 20) + key1
```

## Exploit

```python
from pwn import process, remote
from Crypto.Cipher import Blowfish
from tqdm import trange

def xor(a, b):
    return bytes([x^y for x,y in zip(a,b)])

def encrypt(pt):
    chall.sendlineafter(b"> ", b"E" + pt.hex().encode())
    ct = bytes.fromhex(chall.recvline().decode())
    ivs = [ct[i:i+8] for i in range(0, 24, 8)]
    return ivs, ct[24:]

def decrypt(ct):
    chall.sendlineafter(b"> ", b"D" + ct.hex().encode())
    pt = bytes.fromhex(chall.recvline().decode())
    return pt

def brute_ofb(iv1, iv2, target, start=0):
    for k in trange(start, 2**20):
        cipher = Blowfish.new(k.to_bytes(4, "big"), Blowfish.MODE_ECB)
        if xor(cipher.encrypt(iv1), cipher.encrypt(iv2)) == target:
            print("FOUND:", k)
            return k

def brute_cbc(iv1, iv2, enc1, enc2, start=0):
    for k in trange(start, 2**20):
        cipher = Blowfish.new(k.to_bytes(4, "big"), Blowfish.MODE_ECB)
        if xor(cipher.decrypt(enc1), iv1) == xor(cipher.decrypt(enc2), iv2):
            print("FOUND:", k)
            return k

def brute_cfb(iv, enc, start=0):
    for k in trange(start, 2**20):
        cipher = Blowfish.new(k.to_bytes(4, "big"), Blowfish.MODE_CFB, iv)
        if cipher.decrypt(enc) == bytes(8):
            print("FOUND:", k)
            return k

with remote("partymode.challs.olicyber.it", "38063") as chall:

    # find k3
    ivs, ct = encrypt(bytes(8))
    my_ivs = ivs[:2] + [bytes(8)]
    new_pt = decrypt(b"".join(my_ivs) + ct)
    k3 = brute_ofb(ivs[-1], bytes(8), new_pt, start=0)

    # find k2
    ivs, ct = encrypt(bytes(8))
    my_ivs = ivs[:1] + [bytes(8)] + ivs[-1:]
    new_pt = decrypt(b"".join(my_ivs) + bytes(8))
    c3 = Blowfish.new(int.to_bytes(k3, 4, 'big'), Blowfish.MODE_OFB, ivs[2])
    enc1 = c3.decrypt(ct)
    c3 = Blowfish.new(int.to_bytes(k3, 4, 'big'), Blowfish.MODE_OFB, ivs[2])
    enc2 = c3.decrypt(new_pt)
    k2 = brute_cbc(ivs[-2], bytes(8), enc1, enc2, start=0)

    # find k1
    c3 = Blowfish.new(int.to_bytes(k3, 4, 'big'), Blowfish.MODE_OFB, ivs[2])
    c2 = Blowfish.new(int.to_bytes(k2, 4, 'big'), Blowfish.MODE_CBC, ivs[1])
    enc = c2.decrypt(c3.decrypt(ct))
    k1 = brute_cfb(ivs[0], enc, start=0)

    k = (k3 << 40) + (k2 << 20) + k1

    chall.sendlineafter(b"> ", b"G" + hex(k)[2:].encode())
    print(chall.recvline().decode())
```

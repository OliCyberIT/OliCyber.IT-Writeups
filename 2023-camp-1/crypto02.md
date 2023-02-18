# OliCyber.IT 2023 - Simulazione training camp 1

## [crypto] DH revisited (7 risoluzioni)

La challenge propone una variante del protocollo di Diffie-Hellman in cui l'operazione di esponenziazione modulare è sostituita dalla funzione $o(x, y) = x + y -2 \cdot (x\\&y)$.

### Soluzione

Quella funzione è equivalente allo _xor_ tra $x$ e $y$: si può dunque recuperare la chiave privata di Alice con $o(g, A) = g \oplus A = g \oplus g \oplus a = a$, da cui è possibile ricavare il segreto condiviso con un ulteriore _xor_ con la chiave pubblica di Bob $B$.

### Exploit

```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad

#da output.txt
g = 126410689802884623988756200712033318972
A = 52231676617151166420364050960016472553
B = 9306272951250678880959738917375588033
iv = bytes.fromhex('39e9c1023681591d13604a1964cdf62f')
enc_flag = bytes.fromhex('66fc9e95577c38f8a2153e1e7e96be950c1eabd46d8b70a6304fd1b81b70be5f')

a = g^A
shared_secret = a^B

key = shared_secret.to_bytes(16, 'big')

FLAG = unpad(AES.new(key, AES.MODE_CBC, iv).decrypt(enc_flag), 16)
print(f'{FLAG = }')
```

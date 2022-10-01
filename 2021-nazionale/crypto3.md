# OliCyber.IT 2021 - Competizione nazionale

## [crypto-3] SoundOfSystem (7 risoluzioni)

```
Dimmi di stampare la flag senza dirmi di stampare la flag (cit.)
```

### Soluzione

Dalla descrizione dela challenge ci si può collegare a un servizio remoto che emula un terminale con una sintassi personalizzata.

Dal codice sorgente si può notare l'esistenza del comando `show_flag`. Tuttavia, se inviato, il terminale risponde che non si hanno i permessi necessari per effettuare foperazione. I permessi sono gestiti tramite una blacklist nella quale sono riportati i nomi dei comandi non utilizzabili.

I comandi vengono eseguiti invece attraverso un dizionario, che associa un "hash" del comando alla funzione da eseguire quando viene chiamato. Gli hash sono calcolati attraverso una decifratura in [AES-128-PCBC](<https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Propagating_cipher_block_chaining_(PCBC)>) del comando, utilizzando un padding a 48 byte e tenendo solo l'ultimo blocco. La lunghezza dei comandi che si possono inserire è limitata, impedendo attacchi del tipo "length extension".

Ci sono due vulnerabilità:

- il sistema di padding non aggiunge un blocco nuovo se l'input ha già lunghezza multipla di 48
- in generale, nella modalità di funzionamento PCBC è possibile scambiare 2 blocchi consecutivi dell'input di una decifratura senza cambiare la decifratura dei blocchi successivi.

Infatti, l'$i$-esimo blocco del plaintext si calcola come $$P_i = D_K(C_i) \oplus P_{i-1} \oplus C_{i-1},\ P_0 \oplus C_0 = IV$$ con $D_K$ la funzione di decifratura.

Sia quindi $X = P_{i-1} \oplus C_{i-1}$ e notiamo che possiamo scambiare i blocchi $i$ e $i+1$ lasciando inalterata la decifratura dal blocco $i+2$ in poi:

$$
\begin{align*}
P_{i+2} &= D_K(C_{i+2}) \oplus P_{i+1} \oplus C_{i+1}\\
&= D_K(C_{i+2}) \oplus (D_K(C_{i+1}) \oplus P_{i} \oplus C_{i}) \oplus C_{i+1}\\
&= D_K(C_{i+2}) \oplus (D_K(C_{i+1}) \oplus (D_K(C_{i}) \oplus X) \oplus C_{i}) \oplus C_{i+1}\\
&= D_K(C_{i+2}) \oplus D_K(C_{i+1}) \oplus C_{i+1} \oplus D_K(C_{i}) \oplus C_{i} \oplus X
\end{align*}
$$

Poiché lo XOR gode della proprietà commutativa, possiamo scambiare $C_i$ e $C_{i+1}$ modificando solo due blocchi del plaintext.

Basta quindi fare il padding in locale di `show_flag` e scambiare i primi due blocchi, ottenendo un comando con lo stesso hash del comando originale ma un nome diverso.

### Exploit

```python
from pwn import *

r = remote("65.21.179.132", 15000)

def pad(m):
    if len(m)%48 == 0:
        return m
    return m + bytes([48-len(m)%48])*(48-len(m)%48)

padded = pad(b"show_flag")
payload = padded[16:32]+padded[:16]+padded[32:]

r.recvuntil("> ")
r.sendline(payload)
r.interactive()

```

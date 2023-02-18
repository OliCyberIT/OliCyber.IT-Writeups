# OliCyber.IT 2023 - Simulazione training camp 1

## [crypto] A weird trip to Delphi (4 risoluzioni)

La challenge cifra la flag con AES in modalità CBC. Il padding utilizzato è custom: al messaggio viene concatenato un byte nullo, seguito dalla parola "pad" fino a giungere alla lunghezza del blocco (con eventuale troncamento).

Successivamente è possibile invocare una sola volta un oracolo il quale, dato un iv e un ciphertext, restituisce il padding del corrispondente testo in chiaro.

### Soluzione

La vulnerabilità consiste nel fatto che l'oracolo cerca il primo indice di occorrenza di un byte nullo a partire da _sinistra_ (con il metodo `.index(b'\x00')`) e non da _destra_.

Quindi, se si riuscisse a modificare il risultato della decifratura di `enc*flag` in modo da rendere nullo ad esempio il primo byte, l'oracolo ci restituirebbe tutta la flag (paddata) ad eccezione del primo byte (la lettera \_f\*).

Dando un'occhiata allo schemino di decifratura di AES in modalità CBC

<img src="./attachments/CBC_decryption.svg">

si può notare che l'iv viene xorato con il primo blocco senza ripercussione alcuna sui blocchi successivi. Sappiamo quindi che:

$$Dec(\texttt{ctx} \\_ \texttt{0}) \oplus \texttt{iv} = \texttt{ptx} \\_ \texttt{0} = \texttt{flag\\{...}$$

Inviando quindi un _new_iv_ che abbia come primo byte $iv[0] \oplus ord(\texttt{'f'})$, si otterrà un _new_ptx_ che avrà come primo byte un byte nullo.

### Exploit

```python
import os
from pwn import remote

HOST = os.environ.get("HOST", "trip-to-delphi.challs.olicyber.it")
PORT = int(os.environ.get("PORT", 34007))

with remote(HOST, PORT) as chall:
    chall.recvlines(6)
    iv = bytes.fromhex(chall.recvline().decode().strip().split(' ')[-1][1:-1])
    enc_flag = chall.recvline().decode().strip().split(' ')[-1][1:-1]
    chall.recvlines(2)

    iv = bytes([iv[0]^ord('f')] + list(iv[1:]))
    chall.sendline(iv.hex().encode())
    chall.sendline(enc_flag.encode())

    chall.recvlines(4)
    oracle_output = bytes.fromhex(chall.recvline().decode().strip().split(' ')[-1][1:-1])

    flag = 'f' + oracle_output[:oracle_output.index(b'\x00')].decode()
    print(flag)
```

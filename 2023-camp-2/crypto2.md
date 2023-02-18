# OliCyber.IT 2023 - Simulazione training camp 2

## [crypto] Il solito servizio (40 risoluzioni)

La challenge implementa un servizio con la sola opzione di comando "Ottieni la flag". Per riuscirci è necessario falsificare la firma dell'admin per il comando "get_flag".

### Soluzione

Dal codice del server possiamo ottenere i parametri pubblici che sono un primo $p$ e la chiave pubblica dell'admin. Inoltre abbiamo anche il codice che verifica la correttezza della firma:

```python
def check_signature(signature, command, pub_key):
    int_command = bytes_to_long(command.encode())

    return (pub_key * signature) % p == int_command
```

Dal codice è possibile notare che, per ottenere una firma valida, è sufficiente mandare come firma

$$
\texttt{signature = } \texttt{pub}\\_\texttt{key}^{-1} \cdot \texttt{int}\\_\texttt{command} \pmod{p},
$$

calcolabile in Python come

```python
signature = (pow(admin_public_key, -1, p) * bytes_to_long(b"get_flag")) % p
```

### Exploit

```python
import os
from pwn import *
from Crypto.Util.number import bytes_to_long
import logging
logging.disable()

p = 290413720651760886054651502832804977189
admin_public_key = 285134739578759981423872071328979454683

# Se challenge tcp
HOST = os.environ.get("HOST", "il-solito-servizio.challs.olicyber.it")
PORT = int(os.environ.get("PORT", 34001))

chall = remote(HOST, PORT)
chall.sendlineafter(b"> ", b"1")

admin_private_key = pow(admin_public_key, -1, p)
signature = (admin_private_key * bytes_to_long(b"get_flag")) % p

chall.sendlineafter(b": ", str(signature).encode())
chall.recvline()
flag = chall.recvline().decode()

print(flag)
```

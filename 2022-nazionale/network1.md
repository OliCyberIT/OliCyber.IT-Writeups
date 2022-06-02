# OliCyber.IT 2022 - Competizione nazionale

## [network-1] SNECC 1 (87 risoluzioni)

`Proprio non si riesce ad accedere, è un servizio che non si fa sfuggire informazioni!`

Viene fornito `traffic.pcap`, che contiene 502 flussi TCP relativi a comunicazioni avvenute con un server con cui si pu; interagire.

### Soluzione

La challenge consiste in un replay attack. Infatti il server fornisce un numero `x` e si aspetta un secondo numero `y` secondo un mapping sconosciuto, ma tali risposte si trovano, anche ripetute, nel file allegato. Recuperando il valore giusto da mandare, si può interagire col server per ottenere la prima flag e le informazioni necessarie per risolvere la seconda challenge.

### Exploit

```python
import pyshark
from collections import defaultdict
from pwn import remote
import OpenSSL

challenge_response = {}
streams = defaultdict(lambda: b"")
cap = pyshark.FileCapture("attachments/traffic.pcap")

for p in cap:
    try:
        i = int(p.tcp.stream)
        streams[i] += bytes.fromhex(p.data.data)
    except:
        pass
cap.close()

for i in streams:
    stream = streams[i]
    if b"1. Login con password" in stream:
        challenge = int(stream.split(b"\n")[0].decode())
        response = int(stream.split(b"\n")[1].decode())
        challenge_response[challenge] = response

r = remote('snecc.challs.olicyber.it', 12310)
c = int(r.recvline().decode())
r.sendline(str(challenge_response[c]).encode())
r.recvuntil(b'> ')
r.sendline(b'3')
print(r.recvline())
```

